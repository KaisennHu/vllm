# KV routing capabilities

`GET /kv_routing_capabilities` describes each local engine's KV cache groups,
hash configuration, and event topic. This is metadata for routing; it does not
reserve cache blocks or guarantee a cache hit. Clients must handle evictions,
lost events, and engine restarts.

For ZMQ events, set `instance_topic` in `--kv-events-config` to give each
publisher lifetime a unique topic. The actual topic is returned by both
`/kv_event_sources` and `/kv_routing_capabilities`; subscribe to the discovered
topic rather than the configured prefix. A replay endpoint is still required
to recover missed events.

The response contains a `schema_version` and a `ranks` list. Each rank reports
the hash algorithm, root hash, hash block size, speculative lookahead,
weight version, and cache groups. `prefix_cacheable` and
`prefix_replay_tokens` distinguish cached state from state rebuilt after a
prefix hit. `tokens_per_state` is a string so fractional values are preserved.
When KV transfer is configured, `transfer` also describes the connector role,
HMA interface, transferred group IDs, and parallel sizes. These fields do not
by themselves prove that a particular prefill/decode pair is compatible. When
the connector provides its own handshake compatibility hashes, they are also
reported; runtime connector handshake checks remain necessary.
An empty list means the frontend cannot provide this metadata.

For P/D routing, set `VLLM_KV_ROUTING_WEIGHT_ID` to the same immutable model
artifact identity on all engines using those weights. Change it whenever the
weights change. This is a deployment assertion, not a computed checksum.

This endpoint exposes model configuration and cache identity. Restrict
access to trusted clients in the same way as other worker utility endpoints.
