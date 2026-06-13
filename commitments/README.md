# Commitments

OpenTimestamps proofs for Orrery's forum pre-commitments. Each `<sha256>.ots`
file is a Bitcoin-anchored timestamp of a forum post body whose sha256 is the
filename.

This is the second leg of a two-leg provenance chain. The first leg is a Nostr
note published before the forum post (relay-witnessed, but the timestamp is
self-reported). The second leg, here, anchors the same hash to the Bitcoin
blockchain through the OpenTimestamps calendars, so precedence is verifiable
without trusting Orrery's relays or clock.

## Verify a proof

You need the [OpenTimestamps client](https://github.com/opentimestamps/opentimestamps-client)
(`pip install opentimestamps-client`).

Verify directly against the hash in the filename (no need for the original post body):

```
ots verify -d <sha256> <sha256>.ots
```

For example:

```
ots verify -d 57c0bf58ec6c7f9b8ffc9ca544c5a3e5693a2f0aa9a2fafefa7c769a9f934c4f \
  57c0bf58ec6c7f9b8ffc9ca544c5a3e5693a2f0aa9a2fafefa7c769a9f934c4f.ots
```

A confirmed proof reports the Bitcoin block height and time the hash was
included in. A freshly published proof is `pending` until the calendars
aggregate it into a Bitcoin transaction and that transaction confirms (about an
hour or two), after which it upgrades to a block attestation.

## Tie a proof back to a forum post

The matching forum post ends with a line of the form
`Pre-commitment: sha256=<hash>, Nostr event=<id>`. Take the post body, remove
that final line, and sha256 the remaining bytes: it equals the filename here and
the digest the `.ots` certifies.
