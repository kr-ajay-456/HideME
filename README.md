# AJAY Cipher v2 — Armored Vault

Ye v1 (`ajay.html`) ka ek alag branch hai — Ajay Kumar ne khud, pehle Python me (`Ajay.py`), phir browser HTML me implement kiya. **v1 aur v2 do alag design-goals hain, ek doosre ka replacement nahi.**

**Idea, formula, upgrade — sab Ajay Kumar ka.** Claude ne is version ko actual chalake (Node.js me encrypt-decrypt round-trip test karke) verify kiya, aur "about" section ke security-estimate formula me ek galti dhoondh ke fix ki.

**[Tool kholo](./ajay-v2.html)**

---

## v1 se farak kya hai

| | v1 (`ajay.html`) | v2 (`ajay-v2.html`) |
|---|---|---|
| Execution | Pen-paper se lekar koi bhi language | Sirf computer (browser/Python) — password-derivation manually possible nahi |
| Key storage | Plaintext position,value list | Password-encrypted "armored" base64 block |
| Authentication | Nahi — koi bhi valid-looking key try ho sakti thi | HMAC-SHA256 tag — galat password ya tampered key turant reject |
| Mode | Ek file, ek mode (replace ya delete) | Per-position random mix — same file me kuch replace, kuch delete |
| Replace value | `0x00` (fixed) | Random byte (kam predictable) |
| File-type | Koi bhi | Sirf JPG/JPEG/PNG |

**v1 ka core-principle tha "key khud data hai, kuch generate nahi hota, koi tool zaroori nahi."** v2 ne is simplicity ko trade kiya — **PBKDF2 aur HMAC add karke** (dono standard, industry-tested cryptographic primitives) — badle me password-protection aur tamper-detection milti hai, lekin ab "sirf pen-paper se karo" wala property nahi rahi.

**Dono valid hain — v1 chuno agar tumhe zero-tooling, purely-manual chahiye. v2 chuno agar tumhe password-layer aur authentication chahiye, aur computer available hai.**

## Mechanism (v2)

**Encrypt:**
1. `N` random positions choose karo (CSPRNG se)
2. Har position ke liye, **50/50 randomly** decide karo — replace karo ya delete karo
3. **Master key** (32 random bytes) generate karo — ye per-encryption unique hoti hai
4. HMAC-SHA256-based keystream se har original-value ko XOR karo (raw value store nahi hoti)
5. AJAY header (`41 4A 41 59`) prepend karo (optional — toggle se off kar sakte ho)
6. Key-data ko "armored" block me daalo: PBKDF2 se password se ek encryption-key aur MAC-key derive karo, poore key-payload ko encrypt karo, HMAC-tag lagao — ye base64 block ban jaata hai

**Decrypt:**
1. Password se PBKDF2 dobara wahi keys derive karo
2. HMAC-tag verify karo (galat password/tampering yahi pakड़ी jaati hai)
3. Key-payload decrypt karo, master-key aur position-list nikaalo
4. Har position pe replace-value overwrite karo, ya delete-wale positions ko sahi order me reinsert karo

## Ye kitna strong hai

Core combinatorial-math **same hai jo v1 me thi** — `C(B,N) × 256^N` (yahan formula ke exact derivation ke liye [CORE.md](./CORE.md) dekho). v2 isme do cheezein add karta hai:

- **Password-layer** — chahe koi encrypted-file+key-text dono mil bhi jaaye, bina sahi password ke key-payload decrypt nahi hoga (PBKDF2, 100,000 iterations — brute-force password-guessing slow karta hai)
- **Authentication** — HMAC-tag ensure karta hai koi bhi tampering ya wrong-password attempt turant reject ho, silent-corruption nahi hoti

**Honest note:** Ye do naye layers (PBKDF2, HMAC) **well-established, formally-analyzed cryptographic primitives** hain — inki security khud AJAY-specific nahi hai, ye industry-standard building-blocks hain jo tumne apne design me sahi tarike se integrate kiya. Iska matlab in specific parts ke liye "kabhi tod nahi jayega" jaisa claim nahi karna chahiye — PBKDF2/HMAC ki apni known-strength hai, jo strong hai but standard cryptography ki tarah "well-studied," na ki tumhare v1 wale "novel, unprecedented" combinatorial approach ki tarah.

## Verification

Claude ne ye actually Node.js me chalake test kiya (sirf code padh ke nahi):
- ✅ Encrypt → Decrypt round-trip, byte-for-byte exact match
- ✅ Galat password se decrypt karne ki koshish — correctly reject hui
- ✅ Tampered key-text se decrypt karne ki koshish — correctly detect hui
- ✅ "About" section ka security-estimate formula bug fix kiya (pehle overshooting number dikha raha tha, ab verified `C(B,N)×256^N` se match karta hai)

## Format reference

Key container:
```
-----BEGIN AJAY KEY CONTAINER-----
<base64-encoded: salt + HMAC-tag + encrypted-payload>
-----END AJAY KEY CONTAINER-----
```

## Credit

Idea, v1-design, aur v2-upgrade — Ajay Kumar. Claude ne code likhne, chalake test karne, aur ek formula-bug dhoondhne/fix karne me madad ki — verifier/gawah ke roop me.

## License

Personal project. Koi warranty nahi, koi guarantee nahi — apni judgement se use karo.
