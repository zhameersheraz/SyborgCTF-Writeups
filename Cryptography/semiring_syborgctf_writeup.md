# semiring — SyborgCTF Writeup

**Challenge:** semiring  
**Category:** Cryptography  
**Difficulty:** Easy  
**Points:** 100 pts  
**Flag:** `syborg_ctf{l3@Rn_s0m3_Tr0pic4l_s3M1R1n9}`

---

## Description

> Wanna define some operations on a fake ring?

---

## Background Knowledge (Read This First!)

### What is a Tropical Semiring?

The tropical semiring replaces normal math with different rules:

| Operation | Normal Math | Tropical (this challenge) |
|-----------|------------|--------------------------|
| "Addition" | a + b | min(a, b) — take the smaller |
| "Multiplication" | a × b | a + b — add them |
| "Zero" | 0 | -1 (represents +∞) |
| "One" | 1 | 0 |

Matrix multiplication and exponentiation use these operations. It sounds exotic, but the important thing is what it encodes.

### The Key Insight — Flag Bits in the First Row

After raising the matrix to a large power, the first row of the result matrix directly encodes the flag bits:

| Entry Value | Flag Bit |
|------------|----------|
| 1 | 1 |
| 2 | 0 |

That's it. Read the first row, map `1→bit1` and `2→bit0`, convert to text.

### ⚠️ Extract the Zip First!

```
┌──(zham㉿kali)-[~]
└─$ cd /media/sf_downloads

┌──(zham㉿kali)-[/media/sf_downloads]
└─$ unzip semiring.zip

┌──(zham㉿kali)-[/media/sf_downloads]
└─$ ls output.txt
```

`output.txt` must be in your current folder before running the script.

---

## Solution — Python Script 🐍 (No Library Install Needed!)

### Step 1 — Extract and go to files

```
┌──(zham㉿kali)-[~]
└─$ cd /media/sf_downloads

┌──(zham㉿kali)-[/media/sf_downloads]
└─$ unzip semiring.zip
```

### Step 2 — Create the script

```
┌──(zham㉿kali)-[/media/sf_downloads]
└─$ nano solve.py
```

Paste this:

```python
# No pip install needed — ast is a built-in Python module!

from ast import literal_eval
# literal_eval safely reads a Python data structure from a text file
# This is built-in — no pip install needed

# ---- Step 1: Load the result matrix from output.txt ----
M = literal_eval(open("output.txt").read())
# Reads output.txt as a string, converts it into a Python 2D list
# NOTE: output.txt must be in your CURRENT folder!

# ---- Step 2: Read the flag bits from the first row ----
# M[0] = first row of the result matrix
# M[0][0] = diagonal (always 0, skip it — start from index [1:])
# M[0][1], M[0][2], ... = the encoded flag bits

flag_bits = "".join(str((c - 1) ^ 1) for c in M[0][1:])
# For each entry c (starting at index 1):
#   c - 1 → maps: 1 → 0, 2 → 1
#   ^ 1   → flips: 0 → 1, 1 → 0
# Final result: entry 1 → bit "1",  entry 2 → bit "0"
# "".join(...) combines all bits into one string like "01101001..."

# ---- Step 3: Binary string → integer → bytes ----
flag_int = int(flag_bits, 2)
# int(string, 2) reads the binary string as a base-2 number

# Built-in Python — no pycryptodome needed!
length = (flag_int.bit_length() + 7) // 8
flag_bytes = flag_int.to_bytes(length, 'big')

print(flag_bytes.decode())
```

### Step 3 — Save, exit, run

- **Ctrl + O** → **Enter** (save)
- **Ctrl + X** (exit nano)

```
┌──(zham㉿kali)-[/media/sf_downloads]
└─$ python3 solve.py
syborg_ctf{l3@Rn_s0m3_Tr0pic4l_s3M1R1n9}
```

✅ Got the flag! 🎯

---

## Tools Used

| Tool | Purpose | Level |
|------|---------|-------|
| Python 3 (built-in only) | Parse output.txt, decode first row, convert bits to flag | ⭐⭐ Medium |

---

## Key Takeaways

- Tropical semiring sounds scary — the important thing is just: entry 1 = bit 1, entry 2 = bit 0
- `output.txt` must be in your current folder — always `cd /media/sf_downloads` first
- `ast.literal_eval()` is a safe built-in way to load large Python data from text files
- No pycryptodome needed — Python's built-in `.to_bytes()` does the same job
