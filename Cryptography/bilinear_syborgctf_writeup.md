# Bilinear — SyborgCTF Writeup

**Challenge:** Bilinear  
**Category:** Cryptography  
**Difficulty:** Easy  
**Points:** 100 pts  
**Flag:** `syborg_ctf{4dD1ng+m0re_op3raTIon5~1sNT_alwAyS_stR0ngEr}`

---

## Description

> Xor is linear, addition is too... does combining them make it bilinear???

---

## Background Knowledge (Read This First!)

### What is an Affine Transformation?

An affine transformation applies: `output = A × input + B (mod 256)`. Both XOR and the formula `(key + val ^ key) mod 256` are affine transformations.

**Key fact:** Applying one affine function after another always gives another affine function. So 256 keys × 32 rounds each = still just ONE formula: `output = A × input + B (mod 256)`. That's a monoalphabetic substitution cipher — broken with 2 known bytes.

### What is a Known-Plaintext Attack?

I know the flag starts with `ictf{`. This gives me known plaintext-ciphertext pairs:

- Plaintext byte 0 = `i` (value 105) → Ciphertext byte 0 = `0xe9` (233)
- Plaintext byte 2 = `t` (value 116) → Ciphertext byte 2 = `0xb4` (180)

With two pairs I solve for `A` and `B`, then decrypt everything.

### ⚠️ Why I Use `i` and `t` (Not `s` and `y`)

The difference between chosen bytes must be **odd** to have a modular inverse mod 256:

- `i` - `t` = 105 - 116 = -11 → mod 256 = 245 (odd ✅)
- `s` - `y` = 115 - 121 = -6 → mod 256 = 250 (even ❌ → causes `ValueError`!)

---

## Solution — Python Script 🐍 (No Library Install Needed!)

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

# The ciphertext from the challenge output
ciphertext = bytes([
    0xe9, 0x63, 0xb4, 0x26, 0x7b, 0x74, 0xa4, 0x84,
    0xb1, 0x2e, 0x67, 0x2b, 0xed, 0x70, 0x32, 0xe5,
    0x5f, 0x6f, 0xb0, 0x33, 0x32, 0xe1, 0x94, 0xc9,
    0x6f, 0x2e, 0xb5, 0x3e, 0xb1, 0x73, 0x0e, 0x94,
    0x5f, 0xe1, 0xac, 0x77, 0xc1, 0xf9, 0x53, 0x5f,
    0x73, 0xb4, 0x12, 0x70, 0x2e, 0x67, 0xc5, 0x32, 0xfd
])

# ---- Known plaintext: flag starts with 'ictf{' ----
x1 = ord('i')   # 105
y1 = 0xe9       # 233
x2 = ord('t')   # 116
y2 = 0xb4       # 180
# Their difference: 105 - 116 = -11 → mod 256 = 245 (ODD = invertible ✓)

# ---- Solve for A in: A*(x1-x2) ≡ (y1-y2) mod 256 ----
diff_x = (x1 - x2) % 256   # = 245
diff_y = (y1 - y2) % 256   # = 53

A = (diff_y * pow(diff_x, -1, 256)) % 256
# pow(diff_x, -1, 256) = modular inverse of diff_x mod 256
# Multiply by diff_y to get A

B = (y1 - A * x1) % 256
# Rearranging A*x1 + B = y1 gives us B

# ---- Decrypt all bytes ----
A_inv = pow(A, -1, 256)
# Modular inverse of A — needed to reverse the multiplication

flag = bytes([(A_inv * (c - B)) % 256 for c in ciphertext])
# For each ciphertext byte c:
#   Subtract B → undo the addition
#   Multiply by A_inv → undo the multiplication
#   Take mod 256

print(flag.decode())
```

### Step 3 — Save, exit, run

- **Ctrl + O** → **Enter** (save)
- **Ctrl + X** (exit nano)

```
┌──(zham㉿kali)-[/media/sf_downloads]
└─$ python3 solve.py
ictf{4dD1ng+m0re_op3raTIon5~1sNT_alwAyS_stR0ngEr}
```

School prefix → `syborg_ctf{4dD1ng+m0re_op3raTIon5~1sNT_alwAyS_stR0ngEr}` ✅ Got the flag! 🎯

---

## Tools Used

| Tool | Purpose | Level |
|------|---------|-------|
| Python 3 (built-in only) | Known-plaintext attack to find A and B, then decrypt all bytes | ⭐⭐ Medium |

---

## Key Takeaways

- Combining linear operations (XOR + addition) still stays linear — stacking more doesn't help
- Known-plaintext attacks are powerful — just knowing `ictf{` broke the entire cipher
- The two chosen bytes must have an **odd** difference mod 256 or the math fails
- Always assume the attacker knows your flag format prefix!
