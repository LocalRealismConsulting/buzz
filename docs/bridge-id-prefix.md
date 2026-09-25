# Event ID prefix lookup

The authenticated HTTP bridge accepts a bounded event-ID prefix filter on
`POST /query`:

```json
[{"ids_prefix":"a1b2c3d4"}]
```

The prefix is 8–64 hexadecimal characters. An optional `#h` array restricts
the lookup to channel UUIDs. Without `#h`, Buzz searches all channels the
authenticated identity can currently read. Requested channels are intersected
with that access set; an inaccessible-only scope returns the same empty result
as an absent match and includes no channel metadata.

Prefix lookups include only message kinds `9`, `40002`, `40008`, `45001`, and
`45003` from the last 30 days. They exclude soft-deleted events and all
channel-less global events. The request accepts only `ids_prefix` and optional
`#h`; it rejects mixed filters and other filter fields rather than silently
ignoring them.

The response contains up to 500 matching events:

```json
{"events":[],"complete":true,"ambiguous":false}
```

`complete` means the readable channel scope was exhausted for this prefix and
30-day window; it is false if a 501st candidate proves the result exceeded the
response bound. `ambiguous` is true
whenever more than one match exists or the result is incomplete. A caller may
treat a single match as resolved only when
`complete` is true and `ambiguous` is false.

The CLI uses the authenticated bridge path:

```sh
buzz messages resolve --prefix a1b2c3d4
buzz messages resolve --prefix a1b2c3d4 --channel <UUID>
```
