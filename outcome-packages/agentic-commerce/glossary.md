# Agentic Commerce — Business Glossary

> Status: 🔵 Nearing complete (0.1-beta) · Last reviewed: 2026-08-02 by @shubhamp051991

Business terms for the ORDM outcome package **Agentic Commerce** (Unity Catalog schema). Definitions are vendor-neutral and follow the ORDM [data model standards](../../docs/data-model-standards.md).

## Scope

This package models **autonomous agent-to-agent (A2A) commerce** — a buyer's AI agent negotiating and transacting with a seller's AI agent. Tables carry the `a2a_` prefix.

Human-to-assistant conversational shopping (**Intent Product Discovery**, **Conversational Shopping**) is a **separate scope** within the same package, modeled by the `conversation_session` / `discovery_*` tables. The two scopes share the `agentic_commerce` schema but never share a spine table: `a2a_session` is agent↔agent, `conversation_session` is human↔assistant.

## Tables

| Table | Grain | SCD | Description |
|---|---|---|---|
| `a2a_session` | One agent-to-agent session | None (mutable lifecycle) | Tracks the full negotiation lifecycle between a buying agent (customer) and a selling agent (retailer). |
| `a2a_agent` | One agent version | SCD2 | The autonomous agents themselves as first-class actors — buyer-side or seller-side, with operator, trust status, and protocol version. |
| `a2a_negotiation_event` | One negotiation step | None (append-only) | Immutable log of every offer / counter-offer / acceptance / escalation within a session. |
| `a2a_mandate` | One mandate version | SCD2 | The delegated authority a customer grants a buying agent — spend ceiling, per-transaction limit, approval threshold, allowed categories. |

## Gold views

| View | Grain | Description |
|---|---|---|
| `gold_a2a_session_summary` | One session | Session + rolled-up negotiation outcome (event count, first offer, agreed price/quantity, concession, escalation flag). |
| `gold_a2a_negotiation_funnel` | Day × channel × product | Funnel counts (initiated → negotiating → agreed → completed) and loss, with completion rate. |
| `gold_a2a_agent_scorecard` | One agent | Per-agent participation, completion, success rate, and average time to agreement. |

## Key concepts

- **A2A session** — A single negotiation between two AI agents: one acting on behalf of a buyer (customer) and one acting on behalf of a seller (retailer/store). The session has a lifecycle from initiation through negotiation to completion, abandonment, or failure.
- **Buying agent** — An AI agent that acts on behalf of a customer (linked via `buyer_profile_sk`). It discovers products, requests quotes, negotiates terms, and places orders within delegated authority.
- **Selling agent** — An AI agent that acts on behalf of a retailer or store (linked via `store_sk`). It responds to inquiries, offers pricing, checks inventory, and accepts or counters proposals.
- **Session lifecycle** — The progression of a session through states: `initiated` (first contact), `negotiating` (active exchange), `agreed` (terms accepted), `completed` (order placed), `abandoned` (party withdrew), `failed` (technical/policy failure), `expired` (timed out).
- **Initiator** — The agent that started the session, either `buyer_agent` or `seller_agent`.
- **Agent** (`a2a_agent`) — An autonomous software actor that participates in A2A sessions. A **buyer agent** represents a customer profile; a **seller agent** represents a store. Agents are SCD2-versioned because their trust status, protocol version, and capabilities change over time.
- **Trust status** — The verification state of an agent: `unverified`, `verified`, `suspended`, `revoked`. A counterparty may refuse to transact with an unverified or suspended agent.
- **Negotiation event** (`a2a_negotiation_event`) — One immutable step in a negotiation: `price_request`, `offer`, `counter_offer`, `acceptance`, `rejection`, `availability_check`, or `escalation_to_human`. The append-only history behind a session's current state.
- **Escalation to human** — A negotiation step where the agent hands control to a person (e.g. a proposal exceeds its mandate). Tracked as an `event_type` so the human-escalation rate is measurable.
- **Mandate** (`a2a_mandate`) — The delegated authority a customer grants a buying agent: the spend ceiling, per-transaction limit, the value above which a human must approve (`approval_threshold_amount`), and the product categories the agent may buy. SCD2 — a customer revises the mandate over time. Authorized by a `consent` decision in the customer core (referenced by durable `consent_id`).
- **Price concession** — The reduction from a session's first offer to its agreed price, expressed as a fraction in `gold_a2a_session_summary` — `(first_offer_price − agreed_price) / first_offer_price`. The first offer is the seller agent's opening quote (its ask), so a positive value measures how much ground the seller gave from that quote; a negative value means the settled price rose above the opening.

## Selected terms

| Term | Definition |
|---|---|
| **Agent-to-Agent Transaction** | A purchase completed entirely through automated negotiation between AI agents, without direct human interaction during the transaction itself. |
| **Communication channel** | The communication medium between agents: `api` (programmatic), `marketplace` (platform-mediated), `direct` (peer-to-peer), `voice` (spoken-language interface). Named `communication_channel` to avoid collision with the canonical retail channel concept. |
| **Delegation / mandate** | The authority a human grants to their buying agent to act on their behalf within defined constraints (scope, spend limits, product categories). Modeled in the `a2a_mandate` SCD2 table (see the entities table above). |
| **Session summary** | Free-text description of the session outcome. Treated as PII because agents may relay customer details. |

## Standards used

| Concept | Standard |
|---|---|
| Timestamps | ISO 8601, UTC |
| Session identifiers | UUID v4 (recommended, not enforced) |

## PII / sensitivity classification

Tagged via `dbx_pii` column tags (standard §11):

| Column(s) | Tag |
|---|---|
| `a2a_session.session_summary` | `dbx_pii` |
