# Sea Birdie — SyborgCTF Writeup

**Challenge:** Sea Birdie  
**Category:** Cryptography  
**Difficulty:** Easy  
**Points:** 50 pts  
**Flag:** `syborg_ctf{french_horses_deserve_more_love_too}`

---

## Description

> can combining weak ciphers make a stronger one, well sometimes...

---

## Background Knowledge (Read This First!)

### What is a Monoalphabetic Substitution Cipher?

A substitution cipher replaces each character with a fixed different character throughout the entire message. Every `a` always becomes `x`, every `b` always becomes `q`, etc. The substitution never changes.

**Why is it weak?** English text has predictable patterns (spaces are most common, then `e`, `t`, `a`...). Matching those patterns to the ciphertext reveals the whole substitution table.

### Why Doesn't Stacking 1000 Rounds Help?

Both XOR and the affine formula `(key + val ^ key) mod 256` are **affine transformations**. Applying one affine function after another always gives another affine function. So 1000 rounds of XOR + affine = one single monoalphabetic substitution cipher.

### Why Did quipqiup Fail?

quipqiup is designed for letter-only ciphers (A-Z). Our ciphertext also has symbols like `<`, `;`, `~`, `.` — quipqiup gets confused and outputs garbage. Use a Python script instead.

### The Hidden Flag

The ciphertext decodes to a long horse racing story. The flag `ictf{french_horses_deserve_more_love_too}` is embedded inside it. I found the substitution table by noticing the `~` character separating flag words:

| Ciphertext | Decoded |
|-----------|---------|
| `~` | `_` (separator between words) |
| `j` | `{` (opens the flag) |
| `y5XQ\x02o` | `french` |
| `o.52X2` | `horses` |
| `+X2X5)X` | `deserve` |

---

## Solution — Python Script 🐍

### Step 1 — Go to challenge files

```bash
cd /media/sf_downloads
```

### Step 2 — Create the script

```bash
nano solve.py
```

Paste this:

```python
# The full ciphertext from Sea_Birdie.py's output comment
ct = (
    b'tliHwc6D5\x02w+Xw;5<.pgoX\x11w%XwUXC<DQ\x02X\x03w'
    b'\x1b.gw.yw,.\x085wvDpX\x03w\x08Q+XyXD\x1bX+w&<\x1bow'
    b'[w\x16tw&<Q2\x03w\x05<vvX2\x1bw\x02.pgX\x1b<\x1b<.Qw'
    b'<2w2XDw\x05<5+w\x05\x08\x1bw<\x1bw2XXp2wC<:Xw,.\x08w'
    b'oD)XwDwv..+w\x02oDQ\x02Xw.yw&<QQ<Qv\x11w<\x02\x1byj'
    b'y5XQ\x02o~o.52X2~+X2X5)X~p.5X~C.)X~\x1b.. wUD\x02Xw'
    b'2\x1bD5\x1b2wDQ+w\x1boD\x1bwC\x08QD\x1b<\x02w2XDw\x05'
    b'<5+w)XX52w.yw<Q\x1b.w\x1boXwyD5w\x02.5QX5wDQ+w<2w\x1b'
    b'5DggX+w2.pX&oX5Xw<Qw\x1boXwp<++CXw.yw\x1boXwgD\x02:\x11'
    b'wL.\x08wpDQDvXw\x1b.w\x1bD:Xw\x1boXwCXD+wD\x1bw\x1boXw'
    b'\x1boXwy<QDCw\x02.5QX5\x03wgCXQ\x1b,w.ywXQX5v,w<pg.22'
    b'<\x05CXw\x1b.wC.2XwQ.&\x11\x11\x11w%\x08\x1bw.\x08\x1bw'
    b'.ywQ.&oX5Xw\x02.pX2w2XDw\x05<5+w\x11\x11\x11w&o.w\x1boX'
    b'Qw\x05XD\x1b2w,.\x08w\x05,wiwCXQv\x1bo2\x1e\x04\x1ew'
    b'\x07CD\x02XwuQ+w\x05XD\x1b<Qvw\x1bo<5+w\x05,wDQ.\x1boX'
    b'5wHwCXQv\x1bo2\x11wR.wXp\x05D55D22X+w\x1boD\x1bw,.\x08w'
    b'5X\x1b<5Xw<ppX+<D\x1bXC,'
)

# ---- Substitution table built from the flag word structure ----
sub = {
    ord('w'): ord(' '),   # space — most frequent byte
    ord('~'): ord('_'),   # underscore (separates flag words)
    ord('j'): ord('{'),   # opens the flag
    32:       ord('}'),   # closes the flag
    ord('y'): ord('f'),   # from 'french'
    ord('5'): ord('r'),
    ord('X'): ord('e'),
    ord('Q'): ord('n'),
    0x02:     ord('c'),
    ord('o'): ord('h'),
    ord('.'): ord('o'),   # from 'horses'
    ord('2'): ord('s'),
    ord('+'): ord('d'),   # from 'deserve'
    ord(')'): ord('v'),
    ord('p'): ord('m'),   # from 'more'
    ord('C'): ord('l'),   # from 'love'
    0x1b:     ord('t'),   # from 'too'
    ord('D'): ord('a'),
    ord('<'): ord('i'),
    0x05:     ord('b'),
    ord(':'): ord('k'),
    ord('v'): ord('g'),
    ord(','): ord('y'),
    0x08:     ord('u'),
    ord('&'): ord('w'),
    ord('%'): ord('b'),
    ord('R'): ord('S'),
    ord('L'): ord('Y'),
    ord('i'): ord('i'),
}

# ---- Decode the ciphertext ----
decoded = ''.join(chr(sub.get(b, ord('?'))) for b in ct)
# For each byte, look it up in the substitution table
# Unknown bytes show as '?'

print(decoded)
print()

# ---- Find the flag ----
idx = decoded.find('ictf{')
if idx >= 0:
    print("Flag:", decoded[idx : idx + 50])
```

### Step 3 — Save, exit, run

- **Ctrl + O** → **Enter** (save)
- **Ctrl + X** (exit nano)

```bash
python3 solve.py
```

Output:

```
...ictf{french_horses_deserve_more_love_too}...

Flag: ictf{french_horses_deserve_more_love_too}
```

School prefix → `syborg_ctf{french_horses_deserve_more_love_too}` ✅ Got the flag! 🎯

---

## Tools Used

| Tool | Purpose | Level |
|------|---------|-------|
| Python 3 | Build substitution table from flag word structure, decode story | ⭐⭐ Medium |

---

## Key Takeaways

- **Affine + Affine = still Affine** — 1000 rounds of simple math still gives one simple cipher
- quipqiup fails on non-letter ciphertext — use Python when ciphertext has symbols
- **Flag format is your best friend** — recognizing `french`, `horses`, `deserve` in the ciphertext gave away the substitution table
- The flag was hidden inside a horse racing story — always read the full decoded output
