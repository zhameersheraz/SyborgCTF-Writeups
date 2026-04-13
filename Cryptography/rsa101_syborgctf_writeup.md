# RSA101 — SyborgCTF Writeup

**Challenge:** RSA101  
**Category:** Cryptography  
**Difficulty:** Easy  
**Points:** 100 pts  
**Flag:** `syborg_ctf{RSA_nice_try}`

---

## Description

> We received a message from our agent but we don't know how to use our key to read the message.

**Attachments:** `key.pem`, `cipher`

---

## Background Knowledge (Read This First!)

### What is RSA?

RSA is one of the most widely used encryption algorithms (used in HTTPS, SSH, etc.):

- **Encrypt:** `ciphertext = plaintext ^ e mod n` (uses public key `e` and `n`)
- **Decrypt:** `plaintext = ciphertext ^ d mod n` (uses private key `d`)

Security depends entirely on `d` being kept secret.

### What is a .pem File?

`key.pem` is a text file containing the RSA private key. Opening it shows:

```
-----BEGIN RSA PRIVATE KEY-----
MIICWwIBAAKBgQDuwkcYDIcJAuLPCTf0d4Wz...
-----END RSA PRIVATE KEY-----
```

The data between the headers contains `n`, `e`, and `d`. Since we have `d`, decryption is trivial.

### ⚠️ Two Important Kali Notes

**Note 1 — Files are in `/media/sf_downloads`**
Always `cd /media/sf_downloads` first. Running commands from `~` will cause "No such file or directory."

**Note 2 — Cannot create venv inside `/media/sf_downloads`**
VirtualBox shared folders don't support symlinks. Create the venv in `~` (home) instead.

---

## Solution — Method 1: OpenSSL ✅ Easiest (No Coding, No Setup)

### Step 1 — Go to your files and extract

```
┌──(zham㉿kali)-[~]
└─$ cd /media/sf_downloads

┌──(zham㉿kali)-[/media/sf_downloads]
└─$ ls | grep -i rsa

┌──(zham㉿kali)-[/media/sf_downloads]
└─$ unzip RSA101.zip
```

Press `y` if asked to replace files. Confirm both files exist:

```
┌──(zham㉿kali)-[/media/sf_downloads]
└─$ ls key.pem cipher
```

### Step 2 — Decrypt with one command

```
┌──(zham㉿kali)-[/media/sf_downloads]
└─$ openssl pkeyutl -decrypt -inkey key.pem -in cipher
flag{RSA_nice_try}
```

**What each part means:**

| Part | Purpose |
|------|---------|
| `openssl` | RSA tool — already installed on Kali |
| `pkeyutl` | Use this! (NOT `rsautl` — that's deprecated in OpenSSL 3.0) |
| `-decrypt` | I want to decrypt |
| `-inkey key.pem` | The private key file |
| `-in cipher` | The file to decrypt |

✅ Got the flag! 🎯 (school competition prefix → `syborg_ctf{RSA_nice_try}`)

---

## Solution — Method 2: Python Script 🐍

### Step 1 — Create venv in HOME folder

```
┌──(zham㉿kali)-[~]
└─$ python3 -m venv myenv
                                                                                
┌──(zham㉿kali)-[~]
└─$ source myenv/bin/activate
                                                                                
┌──(myenv)─(zham㉿kali)-[~]
└─$ pip install pycryptodome
```

### Step 2 — Go to challenge files

```
┌──(myenv)─(zham㉿kali)-[~]
└─$ cd /media/sf_downloads
```

### Step 3 — Create the script

```
┌──(myenv)─(zham㉿kali)-[/media/sf_downloads]
└─$ nano solve.py
```

Paste this:

```python
from Crypto.PublicKey import RSA
# Lets me read and parse the key.pem file

from Crypto.Util.number import long_to_bytes
# Converts a large number back to readable text

# ---- Load the private key ----
key = RSA.import_key(open("key.pem").read())
# Reads key.pem from /media/sf_downloads (must be current folder!)
# Extracts key.n (modulus), key.e (public exponent), key.d (private key)

# ---- Load the encrypted file ----
ciphertext_bytes = open("cipher", "rb").read()
# "rb" = read as raw binary bytes

c = int.from_bytes(ciphertext_bytes, "big")
# Convert bytes to one large integer for RSA math

# ---- RSA Decrypt: plaintext = ciphertext ^ d mod n ----
m = pow(c, key.d, key.n)
# pow(base, exponent, modulus) computes c^d mod n efficiently
# This is the RSA decryption formula

# ---- Strip PKCS#1 v1.5 padding ----
# Raw RSA output looks like: 0x00 0x02 <random bytes> 0x00 <actual message>
# The random bytes cause UnicodeDecodeError if not stripped
raw = long_to_bytes(m)
idx = raw.index(b'\x00', 2)
# Find the 0x00 separator byte (skip the first 2 bytes)
plaintext = raw[idx + 1:]
# Everything after that 0x00 is the actual message

print(plaintext.decode())
```

### Step 4 — Save, exit, run

- **Ctrl + O** → **Enter** (save)
- **Ctrl + X** (exit nano)

```
┌──(myenv)─(zham㉿kali)-[/media/sf_downloads]
└─$ python3 solve.py
flag{RSA_nice_try}
```

✅ Got the flag! 🎯

---

## Tools Used

| Tool | Purpose | Level |
|------|---------|-------|
| OpenSSL `pkeyutl` | One-command RSA decrypt — already on Kali | ⭐ Easiest |
| Python 3 + pycryptodome | Script-based RSA decrypt with padding strip | ⭐⭐ Medium |

---

## Key Takeaways

- RSA security depends on `d` being secret — giving away `key.pem` completely breaks it
- Use `pkeyutl` NOT `rsautl` — `rsautl` is deprecated in OpenSSL 3.0 (which Kali uses)
- Raw RSA output has PKCS#1 padding — OpenSSL strips it automatically; Python script must do it manually
- Always `cd /media/sf_downloads` and confirm files exist with `ls` before running anything
