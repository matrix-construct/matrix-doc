# Mass redactions
Matrix, like any platform with public chat rooms, has spammers. Currently,
redacting spam essentially requires spamming redaction events in a 1:1 ratio,
which is not optimal<sup>[1]</sup>. Most clients do not even have any mass
redaction tools, likely in part due to the lack of a mass redaction API. A mass
redaction API on the other hand has not been implemented as it would require
sending lots of events at once. However, this problem could be solved by
allowing a single redaction event to redact many events instead of sending many
redaction events.

## Proposal
This proposal builds upon [MSC2174] and suggests making the `redacts` field
in the content of `m.room.redaction` events an array of event ID strings
instead of a single event ID string.

It would be easiest to do this before MSC2174 is written into the spec, as then
only one migration would be needed: from an event-level redacts string to a
content-level redacts array.

### Number of redactions
Room v4+ event IDs are 44 bytes long, which means the federation event size
limit would cap a single redaction event at a bit less than 1500 targets.
Redactions are not intrinsically heavy, so a separate limit should not be
necessary.

### Client behavior

Clients shall apply existing `m.room.redaction` target behavior over an array
of `event_id` strings.

### Server behavior

#### Auth rules modification

The target events of an `m.room.redaction` shall no longer be considered when
deciding the authenticity of an `m.room.redaction` event. Any other existing
rules remain unchanged.

When a server accepts an `m.room.redaction` using the modified auth rules it
evaluates targets individually for authenticity under the existing auth rules.
Servers MUST NOT include failing and unknown entries to clients.

> Servers do not know whether redaction targets are authorized at the time
> they receive the `m.room.redaction` unless they are in possession of the
> target event. Implementations retain entries in the original list which were
> not shared with clients to later evaluate the target's redaction status.

When the implementation receives a belated target from an earlier
`m.room.redaction` it evaluates at that point whether redaction is authorized.

> Servers should not send belated target events to clients if their redaction
> was found to be authentic, as clients were not made aware of the redaction.
> That fact is also used to simply ignore unauthorized targets and send the
> events to clients normally.

[1]: https://img.mau.lu/hEqqt.png
[MSC2174]: https://github.com/matrix-org/matrix-doc/pull/2174
