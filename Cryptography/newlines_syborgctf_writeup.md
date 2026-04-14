# Newlines — SyborgCTF Writeup

**Challenge:** Newlines  
**Category:** Forensics  
**Difficulty:** Medium  
**Points:** 100 pts  
**Flag:** `syborg_ctf{s0_m4ny_n3wl1n35_b7w_7h15_15_4_v3ry_l0ng_fl4g_s0_h0p3fully_y0u_wr073_4_5cr1p7_f0r_7h1s!!!}`

---

## Description

> Why are there so many newlines?

**Attachment:** `Newlines.zip`

---

## Background Knowledge (Read This First!)

### What is inside the zip?

When you unzip the attachment you get **95 text files** named `0.txt`, `1.txt`, `2.txt` ... `94.txt`.

Each file contains **nothing but newlines** — no letters, no words, just blank lines.

```
┌──(zham㉿kali)-[/media/sf_downloads]
└─$ unzip Newlines.zip

┌──(zham㉿kali)-[/media/sf_downloads]
└─$ wc -l 0.txt
105 0.txt

┌──(zham㉿kali)-[/media/sf_downloads]
└─$ wc -l 1.txt
99 1.txt

┌──(zham㉿kali)-[/media/sf_downloads]
└─$ wc -l 2.txt
116 2.txt
```

### The Trick — Newline Count = ASCII Value

Each file is named with a position number, and its **number of newlines equals the ASCII value of the character at that position** in the flag.

- `0.txt` has 105 newlines → ASCII 105 = `i`
- `1.txt` has 99 newlines → ASCII 99 = `c`
- `2.txt` has 116 newlines → ASCII 116 = `t`
- `3.txt` has 102 newlines → ASCII 102 = `f`
- `4.txt` has 123 newlines → ASCII 123 = `{`
- ...and so on for all 95 files

Read them in order → the full flag!

### Why Is Each File Only Newlines?

A newline character (`\n`) is 1 byte. So a file with 105 newlines is 105 bytes — all `\n`. There is no other content. The **count itself** is the data.

---

## Solution — Python Script 🐍 (Recommended)

The flag is 95 characters long. Counting lines manually across 95 files would take forever. A script does it in seconds.

### Step 1 — Extract the zip

```
┌──(zham㉿kali)-[~]
└─$ cd /media/sf_downloads

┌──(zham㉿kali)-[/media/sf_downloads]
└─$ unzip Newlines.zip

┌──(zham㉿kali)-[/media/sf_downloads]
└─$ ls
0.txt  1.txt  2.txt  ...  94.txt
```

### Step 2 — Create the script

```
┌──(zham㉿kali)-[/media/sf_downloads]
└─$ nano solve.py
```

Paste this:

```python
# No imports needed — pure built-in Python!

chars = []
# We will build the flag one character at a time

for i in range(95):
    # Open each file from 0.txt to 94.txt
    with open(f'{i}.txt', 'rb') as f:
        data = f.read()
        # 'rb' = read as raw bytes

    count = data.count(b'\n')
    # Count how many newline bytes (\n) are in the file
    # This count IS the ASCII value of the character

    chars.append(chr(count))
    # chr() converts a number to its ASCII character
    # e.g. chr(105) = 'i', chr(99) = 'c', chr(116) = 't'

print(''.join(chars))
# Join all characters into one string and print the flag
```

### Step 3 — Save, exit, run

- **Ctrl + O** → **Enter** (save)
- **Ctrl + X** (exit nano)

```
┌──(zham㉿kali)-[/media/sf_downloads]
└─$ python3 solve.py
ictf{s0_m4ny_n3wl1n35_b7w_7h15_15_4_v3ry_l0ng_fl4g_s0_h0p3fully_y0u_wr073_4_5cr1p7_f0r_7h1s!!!}
```

School prefix → `syborg_ctf{s0_m4ny_n3wl1n35_b7w_7h15_15_4_v3ry_l0ng_fl4g_s0_h0p3fully_y0u_wr073_4_5cr1p7_f0r_7h1s!!!}` ✅ Got the flag! 🎯

---

## Solution — One-Liner (Fastest)

If you just want the flag fast without creating a file:

```
┌──(zham㉿kali)-[/media/sf_downloads]
└─$ python3 -c "
chars = []
for i in range(95):
    with open(f'{i}.txt', 'rb') as f:
        count = f.read().count(b'\n')
    chars.append(chr(count))
print(''.join(chars))
"
```

---

## Decoding Table (First 10 characters)

| File | Newline Count | ASCII Character |
|------|--------------|----------------|
| `0.txt` | 105 | `i` |
| `1.txt` | 99 | `c` |
| `2.txt` | 116 | `t` |
| `3.txt` | 102 | `f` |
| `4.txt` | 123 | `{` |
| `5.txt` | 115 | `s` |
| `6.txt` | 48 | `0` |
| `7.txt` | 95 | `_` |
| `8.txt` | 109 | `m` |
| `9.txt` | 52 | `4` |
| ... | ... | ... |
| `94.txt` | 125 | `}` |

---

## Tools Used

| Tool | Purpose | Level |
|------|---------|-------|
| Python 3 (built-in only) | Count newlines in each file, convert to ASCII, build flag | ⭐ Easiest |

---

## Key Takeaways

- Data can be hidden in **file metadata** — here, the number of newlines IS the data
- The file name (0, 1, 2...) gives the **position** in the flag
- The line count gives the **ASCII value** of the character at that position
- Always look at file sizes and line counts when a challenge says "why so many newlines?"
- The flag itself hints at the solution: "hopefully you wrote a script for this"
