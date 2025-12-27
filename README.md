# Relay

Relay receives, normalizes, and reliably delivers webhook events.

## Canonical Data Model (CDM)

Relay transforms all incoming webhook events into a **canonical event**.
This canonical event is the single source of truth for all events within Relay. 
This event representation is used for storage, processing, retriers, replays, and delivery.

Providers may change their webhook payload formats or event names, but the canonical event remains consistent.

### Why a Canonical Data Model?

Third-party webhooks are notoriously inconsistent across providers examples:

- Different naming conventions (e.g., `user.created` vs `UserCreated`).
- Varying payload structures for similar events.
- Inconsistent data types for similar fields.

These inconsistencies are prone to breaking changes that can disrupt your integrations/applications.
Relay normalizes these webhooks into a stable, provider agnostic format/contract so downstream systems
can handle handle business logic without worrying about provider-specific changes independently.

### Canonical Event Schema (v1)

This is the public contract Relay guarantess:

```json
{
    "event_id": "string",               // Unique identifier for the event
    "schema_version": "string",         // Version of the canonical schema
    "source": "string",                 // Source/provider of the event (e.g., "stripe", "github")
    "type": "string",                   // Canonical event type (e.g., "user.created")
    "occurred_at": "string",            // ISO 8601 timestamp of when the event occurred
    "data": {},                         // Event-specific data payload
    "provider_metadata": {}             // Original provider-specific metadata
    "raw_event_id": "string"            // Reference to the raw event stored in Relay
}
```

> [!NOTE]
> The `data` field will stay consistent but only within the a canonical event type.
> Meaning for a given `type`, the structure of `data` object must remain consistent across all providers.
> Across different `type` values, the `data` structure is allowed to differ.
> Example: `payment.succeeded` events from Stripe and PayPal will have the same `data` structure,
> but `user.created` events from GitHub and Auth0 will have a different `data` structure.
