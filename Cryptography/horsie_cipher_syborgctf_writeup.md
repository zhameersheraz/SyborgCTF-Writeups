# Horsie Cipher — SyborgCTF Writeup

**Challenge:** Horsie Cipher  
**Category:** Cryptography  
**Difficulty:** Easy  
**Points:** 100 pts  
**Flag:** `syborg_ctf{heavy_duty_multiplication_for_heavy_duty_cryptography}`

---

## Description

> It might not be the fastest cipher but it tries its best...

---

## Background Knowledge (Read This First!)

### What does haru_cipher actually compute?

The function looks complex and recursive but testing with small numbers reveals the pattern:

| Input | Output | What It Really Is |
|-------|--------|------------------|
| `haru_cipher(2, 1, 5)` | 2 | 2 × 1 = 2 ✓ |
| `haru_cipher(3, 2, 0)` | 6 | 3 × 2 = 6 ✓ |
| `haru_cipher(4, 3, 7)` | 12 | 4 × 3 = 12 ✓ |
| `haru_cipher(5, 5, 99)` | 25 | 5 × 5 = 25 ✓ |

**Conclusion:** `haru_cipher(a, b, c) = a × b` — it's just multiplication! The `c` (iv) parameter does absolutely nothing to the result.

So: `encrypted = flag_int × key` → to decrypt: `flag_int = encrypted ÷ key`

### Why is Multiplication Not Encryption?

Multiplication is a linear operation — trivially reversed by division. `c = m × k` → `m = c ÷ k`. Even if the key weren't given, multiplication is easy to attack with integer factoring.

---

## Solution — Python Script 🐍 (No Library Install Needed!)

This script uses only built-in Python — no `pip install` required.

### Step 1 — Go to challenge files

```
┌──(zham㉿kali)-[~]
└─$ cd /media/sf_downloads
```

### Step 2 — Create the script

```
┌──(zham㉿kali)-[/media/sf_downloads]
└─$ nano solve.py
```

Paste this:

```python
# No imports needed — pure built-in Python!

# The encrypted value is 218 digits long so it is split across three lines.
# Python joins adjacent string literals automatically — the result is identical
# to writing the full number on one line.
encrypted = int(
    "804553796158206582658566627808759315915751777653938258176954996551828222067"
    "457359037393471173966329796663980437327284314103830256868388588740885532947"
    "09405831048354726712268796640387889959994148628234871079000536793408"
)

key = 16026692271780980735644115912092238527974502076791135895337087123122081172032
# iv is ignored — haru_cipher doesn't use it at all!

# ---- Recover the flag integer ----
flag_int = encrypted // key
# Since encrypted = flag_int × key, we reverse by dividing
# // means integer division (no decimal point — gives a whole number)

# ---- Convert integer back to readable text ----
# Using Python's built-in .to_bytes() — no pycryptodome needed!
length = (flag_int.bit_length() + 7) // 8
# bit_length() = how many bits the number uses
# + 7 then // 8 rounds up to the nearest whole byte

flag_bytes = flag_int.to_bytes(length, 'big')
# .to_bytes() converts the integer to raw bytes, 'big' = big-endian

print(flag_bytes.decode())
# .decode() converts bytes to readable text
```

### Step 3 — Save, exit, run

- **Ctrl + O** → **Enter** (save)
- **Ctrl + X** (exit nano)

```
┌──(zham㉿kali)-[/media/sf_downloads]
└─$ python3 solve.py
syborg_ctf{heavy_duty_multiplication_for_heavy_duty_cryptography}
```

✅ Got the flag! 🎯

---

## Tools Used

| Tool | Purpose | Level |
|------|---------|-------|
| Python 3 (built-in only) | Integer division to recover flag, `.to_bytes()` to convert to text | ⭐ Easiest |

---

## Key Takeaways

- Always test a recursive function with small numbers — complex code often hides a simple operation
- Multiplication is not encryption — reversed instantly by division
- The `iv` parameter was a complete red herring — it contributed nothing
- "Heavy duty multiplication" is sarcastic — multiplication is the weakest possible cryptography
