# SYBORG ACLE — SyborgCTF Writeup

**Challenge:** SYBORG ACLE  
**Category:** Cryptography  
**Difficulty:** Easy  
**Points:** 50 pts  
**Flag:** `syborg_ctf{syborg_acle_2026}`

---

## Description

> Decode this:
> `Vm1wR2IyUXhVWGxVV0d4VFlrZG9WVll3YUVOV1ZteHpWV3hrV0dKR1NsbFpNM0JEWVRBeFdGVnFRbFZpUmtwRVdXdGtTMVl4WkhOaVJscE9ZV3hhYjFkV1kzaFRNV1JIVm01U1RsWnRhRmhaYkZwTFZWWmtXV05GWkd0TlZtd3pWREZXYzJGV1NsVldiRkpXWWtkUmQxUnNXbXRrUjFaR1YyeENWMkV3Y0ZSV1ZWcFNaREZDVWxCVU1EMD0=`

---

## Background Knowledge (Read This First!)

### What is Base64?

Base64 is an encoding scheme that converts data into printable text using 64 characters: `A-Z`, `a-z`, `0-9`, `+`, `/`. The `=` sign at the end is padding.

**Key giveaway:** A string ending with `=` or `==` is almost certainly Base64.

### What is Layered Encoding?

Layered encoding means wrapping one encoding inside another — like boxes inside boxes. This challenge has **7 layers of Base64**. Each time you decode, you get another encoded string. Decode all 7 and you get the flag.

**Why?** Base64 is just encoding, not encryption — it has no key. Stacking layers adds zero security. It just looks scary.

| Layer | Starts With |
|-------|------------|
| 0 (input) | `Vm1wR2IyUXhVW...` |
| After decode 1 | `VmpGb2QxUXlUW...` |
| After decode 2 | `VjFod1QyTXlSb...` |
| After decode 3 | `V1hwT2MyRlhTW...` |
| After decode 4 | `WXpOc2FXSXpTb...` |
| After decode 5 | `YzNsaWIzSm5YM...` |
| After decode 6 | `c3lib3JnX2N0Zn...` |
| **After decode 7** ✅ | **`syborg_ctf{syborg_acle_2026}`** |

---

## Solution — Method 1: CyberChef ✅ Easiest (No Coding)

1. Go to 👉 **https://gchq.github.io/CyberChef/**
2. Paste the full encoded string into the **Input** box
3. Search **From Base64** → drag it into the Recipe
4. Repeat — dragging **From Base64** into the Recipe **7 times total** (stack them on top of each other)
5. Click **BAKE!** 🍞

Output:

```
syborg_ctf{syborg_acle_2026}
```

✅ Got the flag! 🎯

> **Tip:** Not sure how many layers? Keep adding From Base64 blocks one at a time until the output looks like readable text.

---

## Solution — Method 2: Python Script 🐍

### Step 1 — Create the script

```bash
nano solve.py
```

### Step 2 — Paste this

```python
import base64
# import base64 so Python can decode Base64

s = "Vm1wR2IyUXhVWGxVV0d4VFlrZG9WVll3YUVOV1ZteHpWV3hrV0dKR1NsbFpNM0JEWVRBeFdGVnFRbFZpUmtwRVdXdGtTMVl4WkhOaVJscE9ZV3hhYjFkV1kzaFRNV1JIVm01U1RsWnRhRmhaYkZwTFZWWmtXV05GWkd0TlZtd3pWREZXYzJGV1NsVldiRkpXWWtkUmQxUnNXbXRrUjFaR1YyeENWMkV3Y0ZSV1ZWcFNaREZDVWxCVU1EMD0="
# The encoded string from the challenge

while True:
    # Loop forever until we find the flag
    decoded = base64.b64decode(s).decode('ascii')
    # base64.b64decode(s) strips one layer of Base64
    # .decode('ascii') converts the result to a readable string

    if decoded.startswith('syborg_ctf'):
        # Found the flag!
        print("FLAG:", decoded)
        break

    s = decoded
    # Otherwise use the decoded result as input for the next loop (peel next layer)
```

### Step 3 — Save, exit, run

- **Ctrl + O** → **Enter** (save)
- **Ctrl + X** (exit nano)

```bash
python3 solve.py
```

Output:

```
FLAG: syborg_ctf{syborg_acle_2026}
```

✅ Got the flag! 🎯

---

## Tools Used

| Tool | Purpose | Level |
|------|---------|-------|
| CyberChef | Stack 7× "From Base64" blocks | ⭐ Easiest |
| Python 3 | Automated loop to peel all 7 layers | ⭐⭐ Medium |

---

## Key Takeaways

- A string ending with `=` or `==` is almost always **Base64**
- **Encoding ≠ Encryption** — Base64 has no key, anyone can decode it instantly
- Stacking 7 layers of Base64 adds **zero security** — still just peeled one by one
- CyberChef lets you stack multiple operations in a Recipe — perfect for layered encoding
