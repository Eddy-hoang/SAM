# 11 - Event Processing Pipeline

> **Document Status:** `[DECISION]` Event Queue & Normalization Specification  

---

## 1. Event Ingest Queue Architecture

To prevent event drops during traffic bursts (e.g., multiple sensors triggering simultaneously), incoming messages pass through an in-memory Asynchronous Queue:

```text
Incoming Payload ──> Schema Validator ──> LRU Deduplicator ──> In-Memory Queue (FIFO) ──> Event Processor
```

---

## 2. Ingest Queue Parameters

| Configuration Key | Parameter Value | Architectural Rationale |
| :--- | :--- | :--- |
| `MAX_QUEUE_DEPTH` | `1000 Events` | Prevents RAM exhaustion during processing delays |
| `DEDUP_WINDOW_TTL_MS`| `60000 ms` | Deduplication window lifetime for V4 UUIDs |
| `MAX_CLOCK_DRIFT_MS`| `30000 ms` | Maximum allowed timestamp divergence from Gateway time |
| `CONCURRENCY_WORKERS`| `4 Threads` | Dedicated worker pool processing events |

---

## 3. Idempotency & Sequence Order Rules

1. **UUID Uniqueness:** Every event contains an `event_id` (V4 UUID). Gateway drops any incoming event matching an active `event_id` in the deduplication cache.
2. **Sequence Gap Detection:**
   * Gateway tracks `last_seen_seq` per `device_id`.
   * If `incoming_seq == last_seen_seq + 1`: Process normally.
   * If `incoming_seq > last_seen_seq + 1`: Log `WARNING_PACKET_LOSS` (packets dropped over air).
   * If `incoming_seq <= last_seen_seq`: Log `SECURITY_ALERT_REPLAY_ATTEMPT` and immediately drop payload.
