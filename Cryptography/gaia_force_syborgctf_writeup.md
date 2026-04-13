# gaia force — SyborgCTF Writeup

**Challenge:** gaia force  
**Category:** Cryptography  
**Difficulty:** Easy  
**Points:** 100 pts  
**Flag:** `syborg_ctf{weirdo_horse_that_has_silvers_in_both_turf_and_dirt_g1s}`

---

## Description

> Yesterday was the mile championship, sadly Gaia Force placed 2nd, meaning he likely will not achieve a single g1 win before his retirement 😦

---

## Background Knowledge (Read This First!)

### How Are Flag Bits Encoded?

Each flag bit gets 64 samples `[a1, a2, b1, b2]`. The rule:

| Flag Bit | How b1, b2 are chosen |
|----------|----------------------|
| 0 | Random values → polynomial f(x) probably has NO root in GF(p) |
| 1 | Specially chosen → polynomial f(x) is guaranteed to have a root in GF(p) |

The polynomial is: `f(x) = a2·x^101 + a1·x^100 - b2·x - b1`

**How to tell bit 0 from bit 1:**

- If ANY of the 64 samples has a polynomial with no root → bit is **0**
- If ALL 64 samples have polynomials with roots → bit is **1**

### How to Check if a Polynomial Has a Root in GF(p)?

Use Fermat's little theorem: compute `gcd(f(x), x^p - x)`. If the gcd has degree > 0, f has at least one root.

### ⚠️ The Zip File Has a SPACE in Its Name!

```
┌──(zham㉿kali)-[/media/sf_downloads]
└─$ ls | grep -i gaia
# Shows: gaia force.zip   ← space, not underscore!
```

- `unzip gaia_force.zip` → fails (no underscore in the filename)
- `unzip "gaia force.zip"` → works ✅ (quotes handle the space)

---

## Solution — SageMath Online

### Step 1 — Extract the files

```
┌──(zham㉿kali)-[~]
└─$ cd /media/sf_downloads

┌──(zham㉿kali)-[/media/sf_downloads]
└─$ unzip "gaia force.zip"
```

Check both files exist:

```
┌──(zham㉿kali)-[/media/sf_downloads]
└─$ ls gaia_force_output.py gaia_force.sage
```

### Step 2 — Install xclip if not already installed

```
┌──(zham㉿kali)-[/media/sf_downloads]
└─$ sudo apt install xclip -y
```

Confirm it installed:

```
┌──(zham㉿kali)-[/media/sf_downloads]
└─$ xclip -version
```

### Step 3 — Copy the output file to clipboard

```
┌──(zham㉿kali)-[/media/sf_downloads]
└─$ cat gaia_force_output.py | xclip -selection clipboard
```

The file is ~4MB, one long line starting with `output=[[[...`

If `xclip` doesn't work, try this alternative — it copies to the PRIMARY clipboard instead:

```
┌──(zham㉿kali)-[/media/sf_downloads]
└─$ cat gaia_force_output.py | xclip
```

Then paste with middle mouse button instead of Ctrl+V.

If both fail, open the file in a text editor:

```
┌──(zham㉿kali)-[/media/sf_downloads]
└─$ mousepad gaia_force_output.py
```

Then select all with **Ctrl + A** and copy with **Ctrl + C**.

### Step 4 — Open SageMath Online

Go to 👉 **https://sagecell.sagemath.org/**

### Step 5 — Paste in the correct order

1. Click inside the text box → **Ctrl + A** → **Delete** (clear everything)
2. **Ctrl + V** → paste the `gaia_force_output.py` content (the `output=[[[...` line)
3. Click at the very end of the pasted content → press **Enter** twice
4. Now paste the solver code below

### Step 6 — Paste the solver BELOW the output content

```python
# ---- Fixed values (hardcoded from gaia_force.sage) ----
p = 867782418176367566201081547629
# p = the prime for GF(p), hardcoded in the challenge source
n = 100
# n = the matrix power, also hardcoded in the challenge source

# ---- Helper: integer to bytes ----
def long_to_bytes(num):
    num = int(num)
    if num == 0: return b'\x00'
    length = (num.bit_length() + 7) // 8
    return num.to_bytes(length, 'big')

# ---- Polynomial operations over GF(p) ----
def polymul(a, b, p):
    res = [0]*(len(a)+len(b)-1)
    for i,ai in enumerate(a):
        for j,bj in enumerate(b): res[i+j]=(res[i+j]+ai*bj)%p
    return res

def polymod(f, g, p):
    f=list(f)
    while len(f)>=len(g) and (len(f)>1 or f[0]!=0):
        if f[-1]==0: f.pop(); continue
        r=f[-1]*pow(int(g[-1]),-1,p)%p; d=len(f)-len(g)
        for i,gi in enumerate(g): f[i+d]=(f[i+d]-r*gi)%p
        while f and f[-1]==0: f.pop()
    return f or [0]

def polypowmod(base, exp, mod, p):
    result=[1]; base=polymod(base,mod,p)
    while exp>0:
        if exp&1: result=polymod(polymul(result,base,p),mod,p)
        base=polymod(polymul(base,base,p),mod,p); exp>>=1
    return result

def polygcd(a, b, p):
    a,b=list(a),list(b)
    while b and not(len(b)==1 and b[0]==0): a,b=b,polymod(a,b,p)
    inv=pow(int(a[-1]),-1,p)
    return [c*inv%p for c in a]

def sample_has_no_root(a1, a2, b1, b2, p, n):
    # Build f(x) = a2*x^(n+1) + a1*x^n - b2*x - b1
    f=[0]*(n+2); f[0]=(-b1)%p; f[1]=(-b2)%p; f[n]=int(a1)%p; f[n+1]=int(a2)%p
    while f and f[-1]==0: f.pop()
    if not f or f==[0]: return False
    # Compute x^p mod f — if gcd(x^p - x, f) has degree > 0, f has a root
    xp=polypowmod([0,1],p,f,p)
    if len(xp)<2: xp+=[0]*(2-len(xp))
    xp[1]=(xp[1]-1)%p
    while xp and xp[-1]==0: xp.pop()
    if not xp or xp==[0]: return False
    g=polygcd(f,xp,p)
    return len(g)<=1  # gcd degree 0 = no root in GF(p)

# ---- Decode each flag bit ----
# 'output' comes from gaia_force_output.py pasted above
print("Decoding", len(output), "bits...")
flag_bits = []

for bit_index, samples in enumerate(output):
    bit_is_zero = False
    for a1, a2, b1, b2 in samples:
        if sample_has_no_root(a1, a2, b1, b2, p, n):
            bit_is_zero = True
            break   # Found one sample with no root → bit is 0
    flag_bits.append(0 if bit_is_zero else 1)
    if (bit_index+1) % 50 == 0:
        print("  Progress:", bit_index+1, "/", len(output))

# ---- Convert bits to the flag ----
flag_binary = ''.join(map(str, flag_bits))
flag_int = int(flag_binary, 2)
flag = long_to_bytes(flag_int)
print("\nFlag:", flag.decode())
```

### Step 7 — What your text box should look like

```
output=[[[519087805226444534...   ← gaia_force_output.py content (pasted FIRST)

p = 867782418176367566201081547629   ← solver code (pasted BELOW)
n = 100
def long_to_bytes(num):
    ...
print("\nFlag:", flag.decode())
```

### Step 8 — Click Evaluate and wait

> ⚠️ Takes ~5 minutes (487 bits × ~0.6s each). The page shows a spinner. Do NOT close or reload the tab!

Output:

```
Decoding 487 bits...
  Progress: 50 / 487
  Progress: 100 / 487
  ...
Flag: syborg_ctf{weirdo_horse_that_has_silvers_in_both_turf_and_dirt_g1s}
```

✅ Got the flag! 🎯

---

## Common Mistakes

| Mistake | Fix |
|--------|-----|
| `unzip gaia_force.zip` → "cannot find or open" | Filename has a SPACE: use `unzip "gaia force.zip"` |
| `xclip: command not found` | Run `sudo apt install xclip -y` first |
| Nothing pastes into sagecell | Try `cat gaia_force_output.py | xclip` then paste with middle mouse button; or open the file in mousepad and Ctrl+A, Ctrl+C |
| `NameError: name 'p' is not defined` | `p` is now in the solver itself — make sure you pasted the solver |
| `NameError: name 'output' is not defined` | Paste `gaia_force_output.py` content FIRST, solver BELOW |
| Page spins forever past 10 minutes | sagecell timed out — reload and try again; the site has a compute limit |

---

## Tools Used

| Tool | Purpose | Level |
|------|---------|-------|
| xclip | Copy 4MB output file to clipboard | ⭐ Easy |
| SageMath Online | Run the polynomial GCD solver (~5 min) | ⭐⭐⭐ Advanced |

---

## Key Takeaways

- Filename with a space requires quotes: `unzip "gaia force.zip"` not `unzip gaia_force.zip`
- Bit=1 → all 64 samples have polynomial roots; Bit=0 → at least one sample doesn't
- `gcd(f(x), x^p - x)` tells us if a polynomial has a root in GF(p) — a standard number theory tool
- `p` and `n` are hardcoded in `gaia_force.sage` — I read the source to find them
- Install xclip with `sudo apt install xclip -y` before trying to copy large files
