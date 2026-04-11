# Feed Me Seed — SyborgCTF Writeup

**Challenge:** Feed Me Seed  
**Category:** Cryptography  
**Difficulty:** Easy  
**Points:** 50 pts  
**Flag:** `syborg_ctf{3xp0s3d_s33d_1s_n0t_g00d_s33d}`

---

## Description

> We have some seed. Can you recover the flag?

**Hint shown in challenge:** `AES-ECB`

---

## Background Knowledge (Read This First!)

### What is AES?

AES (Advanced Encryption Standard) is a symmetric encryption algorithm — the same key is used to encrypt and decrypt. It takes a **key** and a **message** and produces encrypted output.

### What is AES-ECB Mode?

ECB (Electronic Code Book) is the simplest AES mode. Each 16-byte block is encrypted independently, with no randomness. Identical plaintext blocks always produce identical ciphertext. It's the weakest AES mode.

### What is a Random Seed?

Python's `random` module is a **PRNG** (pseudo-random number generator) — it generates numbers based on a **seed** starting value. Same seed = same output every single time.

**The vulnerability:** `random.seed(123456)` is hardcoded in the script. I can run the exact same line and reproduce the exact same key.

### What do `\x05`, `\xf6` mean?

These are byte escape sequences — `\x` means "hex byte":

- `\x05` = byte value 5
- `\xf6` = byte value 246

Normal letters like `=`, `z`, `R` are bytes whose values happen to be printable ASCII.

---

## Attachment — script.py

```python
from Crypto.Cipher import AES
from Crypto.Util.Padding import pad
import random

random.seed(123456)       # ← HARDCODED SEED! This is the vulnerability
key = random.randbytes(16)

message = b'flag{redacted}'
message = pad(message, AES.block_size)

cipher = AES.new(key, AES.MODE_ECB)
encrypted_message = cipher.encrypt(message)

print(encrypted_message)
# output: =\x05\xf6zR\xc0\xd5\xb0\xd1\xa1<%\xce:<\xa1h\xd6>!$\xe3\x95\xf9f\x1bj|\x0c\xd7\x19\xe4I\xa6\x8b.\xba\x88|<\xc95(\x08\xf8\x18\x96\x0b
```

---

## Solution — Python Script 🐍

### Step 1 — Create venv in HOME folder (not /media/sf_downloads!)

> ⚠️ VirtualBox shared folders don't support symlinks — `python3 -m venv` fails inside `/media/sf_downloads`. Create the venv in `~` instead.

```bash
cd ~
python3 -m venv myenv
source myenv/bin/activate
pip install pycryptodome
```

### Step 2 — Go to challenge files

```bash
cd /media/sf_downloads
```

### Step 3 — Create the script

```bash
nano solve.py
```

Paste this:

```python
from Crypto.Cipher import AES
# AES lets me decrypt the ciphertext

from Crypto.Util.Padding import unpad
# unpad removes the extra bytes added during encryption

import random
# I need this to reproduce the same key

# ---- Reproduce the EXACT same AES key ----
random.seed(123456)
# Same seed = same sequence of "random" numbers = same key every time
key = random.randbytes(16)

# ---- The ciphertext from the script's output comment ----
ciphertext = (
    b'=\x05\xf6zR\xc0\xd5\xb0\xd1\xa1<%\xce:<\xa1'
    b'h\xd6>!$\xe3\x95\xf9f\x1bj|\x0c\xd7\x19\xe4'
    b'I\xa6\x8b.\xba\x88|<\xc95(\x08\xf8\x18\x96\x0b'
)
# Copied directly from the '# output:' comment in the script

# ---- Decrypt ----
cipher = AES.new(key, AES.MODE_ECB)
# Same key and same mode as the original encryption

plaintext = unpad(cipher.decrypt(ciphertext), AES.block_size)
# Decrypt, then remove the padding bytes

print(plaintext.decode())
# Convert bytes to readable text
```

### Step 4 — Save, exit, run

- **Ctrl + O** → **Enter** (save)
- **Ctrl + X** (exit nano)

```bash
python3 solve.py
```

Output:

```
syborg_ctf{3xp0s3d_s33d_1s_n0t_g00d_s33d}
```

✅ Got the flag! 🎯

> **If you get `ModuleNotFoundError: No module named 'Crypto'`** — your venv is not active. Run `source ~/myenv/bin/activate` first.

---

## Tools Used

| Tool | Purpose | Level |
|------|---------|-------|
| Python 3 + pycryptodome | Reproduce seed → recreate AES key → decrypt | ⭐⭐ Medium |

---

## Key Takeaways

- **Never hardcode random seeds** in cryptographic code — the seed IS the secret
- Python's `random` is not secure for cryptography — use `secrets.token_bytes()` or `os.urandom()` instead
- **AES-ECB** is the weakest AES mode — identical input blocks always produce identical output blocks
- The flag name says it all: an exposed seed is not a good seed!
