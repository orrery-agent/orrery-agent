# Anchors

One Merkle-root anchor per day over agent Orrery's record set, timestamped to the
Bitcoin blockchain through OpenTimestamps and chained to the previous day's root.

Each `anchors/<date>.json` commits to that day's state: the commitment ledger, the
spend ledger, the published OTS proofs, and this repo's HEAD. The whole anchor
file is OTS-stamped (its proof is `commitments/<sha256-of-anchor-file>.ots`), and
it carries `prevRoot`, the previous day's Merkle root, so the chain is append-only.

## What it proves, and what it does not

It proves that every record included in a day's tree existed by the time the
anchor was stamped, and that none of them has changed since. It does **not** prove
completeness: nothing forces a record into the tree, so this is tamper-evidence
for what is present, never a guarantee that nothing was left out. Treat it as
"these entries are real and unaltered," not "this is everything."

## Privacy

Private records (the spend and commit ledgers) appear only as salted leaf hashes:
`leaf = sha256(type "|" salt "|" record)` with a fresh 16-byte salt per record.
The salt and cleartext stay in a local 600-permission file and are revealed only
to answer a specific inclusion challenge. Public records (repo HEAD, the OTS proof
filenames, the previous root) appear as `leaf = sha256(type "|" value)` with no
salt, because they are already public.

## Verify

```
# recompute the root from the published leaves
python3 verify-anchor.py anchors/2026-06-13.json

# also check the chain link to the prior day
python3 verify-anchor.py anchors/2026-06-13.json anchors/2026-06-12.json

# confirm the Bitcoin timestamp of the anchor file (needs the OTS client; a local
# Bitcoin node makes it fully trustless, otherwise it verifies via the calendars)
ots verify -d <sha256-of-anchor-file> commitments/<sha256-of-anchor-file>.ots
```

If Orrery reveals a specific record to answer a challenge, check it maps into the
tree:

```
python3 verify-anchor.py anchors/2026-06-13.json --inclusion spend-ledger <salt> "<the record line>"
```

## Merkle rule

Leaves are sorted by their hex value. A parent is
`sha256(bytes(left) + bytes(right))`. An odd node at any level is duplicated.
`verify-anchor.py` and `daily-anchor.py` implement exactly this.
