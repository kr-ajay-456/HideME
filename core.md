<div align="center">

# 🧮 core.md

### AJAY Vault ka core: first principles se

*Ek naya tareeka, apne hi usool pe khada.*

</div>

---

## 🧱 Pehla usool

> **Jo jaankari ciphertext mein hai hi nahi, use koi wapas nahi la sakta.**

AJAY Vault isi ek line pe bana hai. Ye kisi cheez ko "chhupata" nahi, ye jaankari ko file se **nikaal deta hai**, aur wo jaankari sirf key mein rehti hai.

---

## 🏷️ Naam kya samjhein

App ke buttons mein "Encrypt / Decrypt" likha hai, par asal mein ye **key-based byte scrambling** hai:

- key ke bina file kharab dikhti hai, aur
- key ke saath **byte-for-byte** wapas aati hai.

"Encryption" ek bada shabd hai jiske kai matlab hote hain, aur har kisi ke liye iska matlab alag ho sakta hai. Is file mein jo bhi claim hai, wo isi **scrambling ke apne math** se aata hai, shabd ke naam se nahi.

---

## 🔤 Notation

| Symbol | Matlab |
|:---:|---|
| `n` | original file ke bytes |
| `N` | kitni positions chhedi gayi |
| `p = N / n` | density (kitna hissa chheda gaya) |
| `R` | replace hui positions |
| `D` | delete hui positions |
| `N = R + D` | total chhedi hui |
| `m = n − D` | encrypted file ke bytes |

---

## 1️⃣ Kya hota hai (do hi operation)

| Operation | File pe asar | Kya chali jaati hai |
|:---:|---|---|
| 🔁 **Replace** | byte ki jagah ek random byte | original value (8 bit) |
| ❌ **Delete** | byte hat jaata hai, aage ka sab khisakta hai | original value **aur** uski jagah ka pata |

Dono operation **many-to-one** hain: bahut saari alag original files ek hi encrypted file ban sakti hain. Isiliye ek taraf se wapas jaana possible nahi, jab tak key na ho.

Encrypted file mein sirf teen tarah ke bytes hote hain:

- ✅ **Untouched** (`n − N` bytes): bilkul asli
- 🎲 **Replaced** (`R` bytes): random, par asli jaise hi dikhte hain
- 🕳️ **Deleted** (`D` bytes): hain hi nahi, sirf length aur alignment mein nishaan

Koi bhi byte par label nahi lagta ki wo kis tarah ka hai.

---

## 2️⃣ Kitni jaankari nikal gayi

Har chhedi position pe original byte ki value **8 bit** ki hoti hai, aur wo sirf key mein jaati hai:

$$
I_{values} = 8N \text{ bits}
$$

Uske upar, kaun si positions chhedi gayi aur kaun si delete hui, ye bhi chhupa hai:

$$
I_{where} = \log_2\binom{n}{N} + \log_2\binom{N}{D}
$$

Dono milke **key ka asli vazan** hai:

$$
\boxed{\;I_{key} = \log_2 T = \log_2\!\left[\frac{n!}{R!\,D!\,(n-N)!}\cdot 256^{N}\right]\;}
$$

Yahi **combination formula** hai. `T` = wo kul kitni alag keys ho sakti hain jo is encrypted file se judi ho sakti hain.

---

## 3️⃣ Total search space

Attacker ko `N`, `R`, `D` pata nahi hote, to saari possibilities ka sum:

$$
T_{total}=\sum_{N}\sum_{D=0}^{N}\frac{n!}{(N-D)!\,D!\,(n-N)!}\;256^{N}
$$

Is sum mein sabse bada term sabse badi `N` pe aata hai, baaki sirf ek chhota multiplier jodte hain.

### ⚡ Seedha andaaza (bits per byte)

$$
\frac{\log_2 T}{n} \;\approx\; H(p_u,p_r,p_d) \;+\; 8p
\qquad H=-\sum p_i\log_2 p_i
$$

Jahan `p_u = 1−p` (untouched), `p_r = R/n`, `p_d = D/n`.

| Density `p` | Replace : Delete | Bits per byte of file |
|:---:|:---:|:---:|
| 10% | 50 : 50 | **1.37** |
| 12.5% | 50 : 50 | **1.67** |
| 15% | 50 : 50 | **1.96** |
| 30% | 50 : 50 | **3.58** |
| 56.1% | 25 : 75 | **5.93** |

Matlab **`p` badhao to search space seedha upar jaata hai**, kyunki `8p` wala hissa `p` ke saath chalta hai.

---

## 4️⃣ File kab wapas nahi aati (do wajah)

### 🔗 Wajah 1: sequential data tootta hai, tukdon mein nahi

Compressed formats (PNG, JPEG, MP4, ZIP) ek **sequential code** hain: har byte ka matlab pichle bytes pe tika hai. Ek bhi byte badla ya hata to uske **aage ka sab** galat decode hota hai.

Pehla change aane se pehle kitna hissa saaf bachta hai:

$$
E[\text{clean prefix}] = \frac{1-p}{p} \text{ bytes}
$$

| `p` | 10% | 12.5% | 15% | 56.1% |
|:---:|:---:|:---:|:---:|:---:|
| Saaf prefix | ~9 B | ~7 B | ~5.7 B | ~0.8 B |

Yaani decoder shuru mein hi rukta hai.

### 🕳️ Wajah 2: khoyi jaankari ke liye redundancy chahiye

Khoye hue `8N` bit tabhi guess ho sakte hain jab source mein utni **redundancy** ho (jo bits pehle se predict ho sakte hon). Shart:

$$
r_{source} \;\ge\; p
$$

jahan `r_source` = source ke wo bits ka hissa jo pehle se predictable hain.

| Source | Redundancy `r` | Asar |
|---|:---:|---|
| Compressed (PNG, JPEG, MP4, ZIP) | ≈ 0 | `r < p` hamesha, khoye bytes wapas nahi aate |
| Raw / uncompressed (BMP, WAV) | bada | tabhi mushkil, jab `p` bhi bada ho |
| Plain text | bada | tabhi mushkil, jab `p` bhi bada ho |

Isliye **compressed pic aur video pe ye tareeka sabse mazboot** hai, aur raw ya text pe `p` bada rakhna chahiye.

---

## 5️⃣ Asli test ka record

Ek PNG pe poora test chalaya gaya, aur sirf encrypted file di gayi, key aur original nahi.

| Cheez | Value |
|---|---|
| Original size | 2,218,448 bytes |
| Encrypted size | 1,284,594 bytes |
| Chhedi gayi positions | 12,45,139 (**56.1%**) |
| Delete / Replace | 9,33,854 / 3,11,285 (**75 : 25**) |
| Marker | off |
| Search space | `≈ 10^39,63,242` (**5.93 bits/byte**) |

**Bina key ke:**
- Pehli `IDAT` ke zlib stream se **0 bytes** decode hue.
- File mein 300 random jagah se deflate decode karne pe zyada se zyada **16 bytes** mile.
- Sirf header ke tukde aur metadata ke kuch tukde dikhe, jo untouched bytes the, koi byte recover nahi hua.

**Key ke saath:**
- File exactly `2,218,448` bytes ki wapas aayi.
- 841×1870 ki valid PNG mili.
- **37 chunks ke CRC sab sahi**, yaani ek bhi byte galat nahi.

Yaani bina key ke kuch nahi, aur key ke saath **byte-for-byte** wapas.

---

## 6️⃣ Dono settings ka asar (`p` aur ratio)

| Setting | Kya badalta hai |
|---|---|
| **`p` badhao** | Search space badhta hai, par key bhi badi hoti hai (key mein kam se kam `N` bytes) |
| **Delete zyada** | Length aur alignment dono hilte hain, sequential data jaldi toot'ta hai |
| **Replace zyada** | Length wahi rehti hai, par bytes galat hote hain |
| **50 : 50 ratio** | Har chhedi position pe type ki entropy sabse zyada (**1 bit**), 75 : 25 pe `0.81 bit` |
| **Marker off** | File pe koi fingerprint nahi, ye khud pehchaan mein nahi aati |

Praktik salah: pic ya video ke liye `p` zyada aur ratio balanced rakho, aur encrypt karne se pehle metadata hata do, kyunki metadata ke bache hue tukde file ki pehchaan bata dete hain.

---

## 🎭 Camouflage: file kharab download jaisi dikhti hai

Marker off ho to encrypted file mein koi header, koi fixed pattern, koi fingerprint nahi hota. Wo bas ek **kharab download** jaisi dikhti hai.

**Ye kyun hota hai (usool se):**
- Chhedi hui positions ke bytes random hain, aur untouched bytes asli format ke tukde hain.
- Dono ka mix bilkul wahi dikhta hai jo ek adhoori ya bigdi hui download mein dikhta hai.
- Koi cheez ye nahi batati ki ye jaan-boojhke bigaadi gayi hai.

**Density kitna "format" bachne deti hai (asli tests se):**

| File | Density | Kya bacha dikhta hai |
|---|:---:|---|
| PDF | 12.5% | `%PDF` header aur `obj` ke 58 mein se 41, yaani damaged PDF jaisi |
| PDF | 56% | `obj` sirf 6 bache |
| MP3 | 12.5% | ~918 frame headers mein se 597 |
| MP3 | 56% | sirf 82 headers |
| PNG | 56.1% | kuch `IDAT` tags aur metadata ke tukde |

Matlab **density jitna bada, utna kam format dikhta hai**, aur file utni hi random garbage ke kareeb hoti hai.

**Camouflage badhane ke teen tareeke:**
1. **Marker off rakho:** `AJAY` header file ki pehchaan bata deta hai.
2. **Metadata hata do:** C2PA, EXIF jaisa text aadha-adhoora bhi bacha rahe to file ki pehchaan deta hai.
3. **Density bada rakho:** jitna zyada chhedoge, utne kam tukde bachenge. Dhyaan rahe, key bhi utni badi hoti hai.

---

## 🧾 Ek line mein

- **Idea:** jaankari ko file se nikalo, wo sirf key mein rahe.
- **Formula:** `T = n! / (R!·D!·(n−N)!) · 256^N`
- **Andaaza:** `log₂T / n ≈ H + 8p`
- **Compressed data:** pehle change ke baad aage ka sab toot'ta hai, pehle `(1−p)/p` bytes hi saaf rehte hain.
- **Test:** 56.1% density pe bina key **0 bytes**, key ke saath **poori file, sab CRC sahi**.
- **Camouflage:** marker off ho to file ek kharab download jaisi dikhti hai, density jitna bada utna kam format dikhta hai.
