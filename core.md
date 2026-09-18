# AJAY Cipher — Core Reference

Clean reference doc: structure, verifications, and arguments. Debate history and chronology live separately; this is the distilled current state.

---

## STRUCTURE (mechanism spec)

- **v1 core**: pick N random byte positions in a file (CSPRNG), record each position's original value, then per-position either:
  - (a) **replace** with a new value — file length unchanged, no shift, or
  - (b) **delete** the byte — file shrinks; deleted positions must be reconstructed in ascending-position order on decrypt
- **Minimal complete key** = three fields per position: `(position, original hex value, type r/d)`. This is the only thing strictly required by the algorithm. Any extra personal shorthand or reconstruction notation beyond this is individual bookkeeping, not part of the cipher itself.
- **Optional 4-byte header marker** `41 4A 41 59` ("AJAY") can be prepended — breaks file-type signature detection immediately if present. Omitting it leaves the file forensically indistinguishable from random/corrupted data.
- **Key storage**: meant to be written to a physical diary, digital key file deleted after — key never persists digitally by design.
- **v2** (built by Ajay, Python first then `ajay-v2.html`) adds a separate **key-protection wrapper**: PBKDF2 (100k iterations) + HMAC-SHA256 authenticated, password-derived armored key container. Replace-vs-delete is decided per-position (50/50 random, mixed per file). This wrapper protects the key from leaking; it is a distinct concern from the core file-scrambling algorithm and doesn't change what the core algorithm's own security claim covers.
- Algorithm is public/documented (Kerckhoffs's principle) — secrecy lives only in the key, not in hiding the mechanism.

---

## VERIFICATIONS (actually run, with results)

| Test | Result |
|---|---|
| N=4, N=2048, N=10000 round-trip on real JPEGs (by Ajay) | Encrypt→decrypt matched original exactly |
| v2 round-trip, verified by Claude in Node.js | Byte-for-byte match; wrong-password rejection confirmed; tampered-key detection confirmed; found a bug in v2's own displayed security-bit-estimate formula (overshooting) — that fix was itself later superseded by the corrected `L`-based merged-mode formula (see ARGUMENTS below) |
| N=4 blind test (encrypted PNG, no key given) | Brute-force confirmed infeasible; the search-space estimate used at the time has since been superseded by the corrected `L`-based merged-mode formula |
| Diary-photo key test (Ajay's personal shift-notation) | Claude could not decode without Ajay's own reconstruction rule; Ajay confirmed even he can't decrypt that exact file outside his original working environment — concluded to be a format-completeness/usability gap, not a cryptographic weakness |
| Clean-format test (N=6 JPEG + explicit 3-field key, via `ajay-clean-format.html`) | Claude wrote a standard decrypt script (sort deletes by position, one linear reconstruction pass, apply replaces) and successfully reconstructed the original file byte-for-byte — confirms deterministic, correct decryption when given a complete key |
| Zero-info blind test (no N, no key) | Proposed, then declined by Ajay — both agreed a Claude failure there would be inconclusive (proves lack of Claude's brute-force tooling, not cipher strength) |

---

## ARGUMENTS (resolved positions)

- **Core combinatorial claim**: a single isolated file without its key is practically unbreakable regardless of hardware/time/quantum computing, for any reasonable N. The original `C(B,N) × 256^N` formula only held for single-global-mode (all-replace or all-delete); in merged mode `B` itself becomes unknown to the attacker (since deletes shrink the observed length `L` by an unknown amount), so the corrected, verified formula is stated in terms of the observable `L`, not `B`:
  ```
  Total Search Space = 256^N × Σ (D=0 to N) C(L+D, N) × C(N, D)
  ```
  Verified twice by brute-force enumeration (toy cases), exact match both times. Security scales with `N`, not file size — a small N (e.g. N=4) is classically strong but quantum-weak (~10 min via Grover), while large N (e.g. N=4000) is unbreakable either way. Not formally peer-reviewed or adversarially battle-tested like AES/RSA — Ajay acknowledges this.
- **Resolved**: file-scrambling mechanism (Ajay's) and key-protection wrapper (v2's PBKDF2/HMAC) are separate concerns — a wrapper failure (e.g. weak password) is not a refutation of the core algorithm's claim.
- **Resolved**: manual/diary key notation can legitimately be personal/ad-hoc without weakening the algorithm, since the algorithm only strictly requires the 3-field `(position, value, type)` data. Consistent with Kerckhoffs's principle once required fields are separated from optional personal shorthand.
- **Resolved**: what can fail is always implementation/operational — weak RNG, predictable human-chosen positions, key/diary leakage, position-pattern reuse across files (the Soviet VENONA risk) — never the combinatorial math itself. A compromised file via implementation weakness doesn't cascade to other files, since there's no shared key-pool to exhaust.
- **Open / unresolved**: non-brute-force attack-vector breakdown —
  - **Known-plaintext**: original file or predictable format header (e.g. JPEG's `FF D8 FF`) known to attacker lets positions be diffed out instantly
  - **File-structural attacks**: checksum/length fields narrow per-position value search
  - **Multi-file correlation**: reused N/position patterns across files
  - **Side-channel/operational risk**: weak RNG misuse, device compromise, coercion ("$5 wrench")
  
  Ajay has not yet responded in depth to this list.
- Ajay deliberately keeps the tool private/unpublished until he finds a loophole himself — a fully untraceable tool would mean he couldn't detect or report misuse if someone else used it for harm.
