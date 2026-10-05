<div align="center">

# 🔐 AJAY Vault · `414A4159`

### Ek personal byte-scrambling algorithm — **key hamesha tumhare paas rahegi**

![Offline](https://img.shields.io/badge/Runs-Offline-9fd46c?style=for-the-badge)
![No Upload](https://img.shields.io/badge/Nothing-Uploaded-ee3b3b?style=for-the-badge)
![Pure JS](https://img.shields.io/badge/Pure-HTML%20%2B%20JS-f6b93b?style=for-the-badge)
![Max Size](https://img.shields.io/badge/Max%20File-5%20MB-bfe4f7?style=for-the-badge)

</div>

---

## ✨ Ye kya hai?

**AJAY Vault** ek single-file tool hai jo kisi bhi file ke bytes ko scramble kar deta hai aur ek **key file** banata hai. Bina key ke original file wapas nahi aa sakti. Sab kuch tumhare browser ke andar hota hai, koi server nahi, koi upload nahi.

---

## 🧠 Algorithm kaise kaam karta hai

Algorithm **byte-level** pe chalta hai. Ye teen simple steps mein samajh lo:

### 1️⃣ Positions chunna
File ke `n` bytes mein se `N` random positions chuni jaati hain. Ye selection **Fisher-Yates shuffle** se hota hai, aur randomness `crypto.getRandomValues` se aati hai (secure random, `Math.random` nahi).

> `N` auto mode mein file size ka **random 10% – 15%** hota hai. Chaho to khud bhi number daal sakte ho.

### 2️⃣ Har position pe Replace ya Delete
Chuni hui har position ke saath ek of do kaam hote hain:

| Action | Kya hota hai | Key mein kya save hota hai |
|:---:|---|---|
| 🔁 **Replace** | Us byte ki jagah ek random byte (0–255) daal di jaati hai | `position, original_hex, r` |
| ❌ **Delete** | Wo byte file se hata di jaati hai (file chhoti ho jaati hai) | `position, original_hex, d` |

Default mein har position ka Replace/Delete **random 50/50** decide hota hai. Chaho to **Custom Replace / Delete** toggle on karke slider se ratio set kar sakte ho.

### 3️⃣ Key banana
Har change ki asli value `position,value,type` format mein key file mein likhi jaati hai. Ye key ke bina kisi ko nahi milti.

```
AJAY
file: photo.jpg
size: 482113
count: 61240
mode: merged
marker: yes
1042,ff,r
5530,2a,d
...
```

### 🔓 Decrypt kaise hota hai
1. Pehle `AJAY` header (agar hai) hata diya jaata hai.
2. **Delete** wale bytes ko unki original positions pe wapas daala jaata hai (position ke hisaab se sorted order mein).
3. **Replace** wale bytes ko unki original value se overwrite kiya jaata hai.
4. Original file wapas mil jaati hai, byte-for-byte. ✅

---

## 🚀 Features

- 🔒 **Byte-level scrambling** with secure random positions
- 🎚️ **Replace / Delete ratio slider** → ek side kheenchoge to uska percentage kam hoga aur dusre ka badh jayega (hamesha total 100%)
- 🔘 **Custom toggle** → off ho to random 50/50, on ho to slider ka ratio
- 🎯 **Auto count** (10–15%) ya **manual count** apni marzi se
- 🏷️ **AJAY header toggle** → encrypted file ke start mein `AJAY` marker (`41 4A 41 59`) add karna ya nahi
- 📅 **Rename output** → `DD-MM-YYYY  #N` format mein file ka naam
- 🗝️ **Key file download** + Decrypt ke time key **paste** ya **upload** dono ka option
- 📴 **100% offline** → file aur key browser se bahar kabhi nahi jaati
- 🧹 **Reset Session** ek click mein sab clear
- 📱 Mobile pe bhi smooth chalta hai

---

## 🛠️ Use kaise karein

**Encrypt**
1. `ajay-vault.html` browser mein kholo
2. File choose karo (max **5 MB**)
3. Count check karo, Replace/Delete ratio chaho to set karo
4. **Encrypt** dabao, phir **encrypted file** aur **key file** dono download karo

**Decrypt**
1. **Decrypt** tab pe jao
2. Encrypted file choose karo
3. Key paste karo ya key file upload karo
4. **Decrypt** dabao, original file download karo

> ⚠️ **Key sambhal ke rakhna.** Key kho gayi to file kabhi restore nahi hogi.

---

## 📬 Contact

<div align="center">

| Platform | Handle |
|:---:|:---:|
| 𝕏 **X** | [@kr_ajay_456](https://x.com/kr_ajay_456) |
| 🐙 **GitHub** | [kr-ajay-456](https://github.com/kr-ajay-456) |
| 📸 **Instagram** | [kr_ajay_456](https://instagram.com/kr_ajay_456) |
| ✈️ **Telegram** | [kr_ajay_456](https://t.me/kr_ajay_456) |
| 📧 **Email** | kr.ajaychauhaan@gmail.com |
| 📧 **Email (alt)** | Dinno@duck.com |

</div>

---

<div align="center">

**Idea & Design:** Ajay Kumar (ARK) · **Code:** Claude (Anthropic)

Made with ❤️ by **Ajay**

</div>
