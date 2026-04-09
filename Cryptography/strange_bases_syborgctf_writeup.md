# Strange Bases — SyborgCTF Writeup

**Challenge:** Strange Bases  
**Category:** Cryptography  
**Difficulty:** Easy  
**Points:** 50 pts  
**Flag:** `syborg_ctf{less_common_90s_base}`

---

## Description

> base64 isn't the only text encoding scheme...

**Given attachment:**
```
0x47,0x27,0x48,0x72,0x58,0x2d,0x63,0x28,0x40,0x5e,0x74,0x2e,0x6f,0x39,
0x4d,0x46,0x48,0x4d,0x76,0x5a,0x4d,0x76,0x57,0x52,0x34,0x7b,0x68,0x33,
0x42,0x5d,0x31,0x4a,0x25,0x51
```

---

## Background Knowledge (Read This First!)

### What is Hex?
Hex (hexadecimal) is base-16 — numbers written using digits 0–9 and letters A–F. The `0x` prefix just means "this number is in hex." Each hex value here maps to one ASCII character.

### What is ASCII?
ASCII is a table that assigns numbers to characters:
- `0x47` = decimal `71` = letter **G**
- `0x7b` = decimal `123` = symbol **{**
- `0x25` = decimal `37` = symbol **%**

### What is Base92?
Base92 is a text encoding from the early 1990s. Like Base64, it converts binary data into printable text — but uses 92 different characters instead of 64. The description says "less common 90s base" — that points directly at Base92.

| Encoding | Characters Used | Efficiency |
|----------|----------------|------------|
| Base64 | A-Z, a-z, 0-9, +, /, = | ~75% |
| **Base92** | **92 printable ASCII chars** | **~82%** |

---

## Step 1 — Full Hex-to-ASCII Table

| Hex | Decimal | Character |
|-----|---------|-----------|
| 0x47 | 71 | **G** |
| 0x27 | 39 | **'** |
| 0x48 | 72 | **H** |
| 0x72 | 114 | **r** |
| 0x58 | 88 | **X** |
| 0x2d | 45 | **-** |
| 0x63 | 99 | **c** |
| 0x28 | 40 | **(** |
| 0x40 | 64 | **@** |
| 0x5e | 94 | **^** |
| 0x74 | 116 | **t** |
| 0x2e | 46 | **.** |
| 0x6f | 111 | **o** |
| 0x39 | 57 | **9** |
| 0x4d | 77 | **M** |
| 0x46 | 70 | **F** |
| 0x48 | 72 | **H** |
| 0x4d | 77 | **M** |
| 0x76 | 118 | **v** |
| 0x5a | 90 | **Z** |
| 0x4d | 77 | **M** |
| 0x76 | 118 | **v** |
| 0x57 | 87 | **W** |
| 0x52 | 82 | **R** |
| 0x34 | 52 | **4** |
| 0x7b | 123 | **{** |
| 0x68 | 104 | **h** |
| 0x33 | 51 | **3** |
| 0x42 | 66 | **B** |
| 0x5d | 93 | **]** |
| 0x31 | 49 | **1** |
| 0x4a | 74 | **J** |
| 0x25 | 37 | **%** |
| 0x51 | 81 | **Q** |

Reading all together → **`G'HrX-c(@^t.o9MFHMvZMvWR4{h3B]1J%Q`**

---

## Solution — CyberChef ✅ Easiest

### Step 1 — Convert Hex to ASCII

1. I go to 👉 **https://gchq.github.io/CyberChef/**
2. In the **Input** box, I paste (spaces only, no `0x`, no commas):
   ```
   47 27 48 72 58 2d 63 28 40 5e 74 2e 6f 39 4d 46 48 4d 76 5a 4d 76 57 52 34 7b 68 33 42 5d 31 4a 25 51
   ```
3. I search **From Hex** → drag into Recipe → set Delimiter to `Space`
4. Click **BAKE!** 🍞

Output: `G'HrX-c(@^t.o9MFHMvZMvWR4{h3B]1J%Q`

### Step 2 — Identify the Encoding

I look at the characters: `'`, `(`, `@`, `^`, `{`, `]`, `%`. These are NOT valid Base64 characters. Base64 only uses `A-Z`, `a-z`, `0-9`, `+`, `/`, `=`. The description says "less common 90s base" → this is **Base92**.

### Step 3 — Decode Base92

1. I clear the Recipe in CyberChef
2. In the Input box, I paste: `G'HrX-c(@^t.o9MFHMvZMvWR4{h3B]1J%Q`
3. I search **From Base92** → drag into Recipe
4. Click **BAKE!** 🍞

Output:
```
syborg_ctf{less_common_90s_base}
```
✅ Got the flag! 🎯

---

## Tools Used

| Tool | Purpose | Level |
|------|---------|-------|
| CyberChef | From Hex → From Base92 | ⭐ Easiest |

---

## Key Takeaways

- Hex values (`0x47`) are just numbers — look them up in an ASCII table to get the character
- **Base64 is not the only encoding** — symbols like `'`, `@`, `^`, `]`, `%` hint at Base92
- The description is always a clue — "less common 90s base" pointed directly at Base92
- CyberChef has operations for dozens of encodings — always browse the operations list when stuck
