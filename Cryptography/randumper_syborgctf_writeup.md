# Randumper — SyborgCTF Writeup

**Challenge:** Randumper  
**Category:** Cryptography  
**Difficulty:** Easy  
**Points:** 100 pts  
**Flag:** `syborg_ctf{a_doubly_smooth_number_in_this_economy?!?}`

---

## Description

> Making a prng is easy, the hard part is hiding the backdoor...

---

## Background Knowledge (Read This First!)

### What is a PRNG?

A PRNG (Pseudo-Random Number Generator) generates keystream values that are XOR-ed with data to encrypt it. If you can predict the keystream, you can decrypt everything.

### What is a Smooth Number?

A number whose prime factors are all small:

- `120 = 2³ × 3 × 5` → smooth (all factors ≤ 5) ✓
- `127` → not smooth (prime itself) ✗

**Why it matters for crypto:** When a group's order is smooth, the Pohlig-Hellman algorithm makes discrete logarithm easy. Normally discrete log is computationally impossible — with a smooth order it takes seconds.

### The Backdoor

`Totally_Safe_Modulus` is composite with four 64-bit prime factors. SageMath's `factor()` finds them instantly. Then `discrete_log()` recovers the PRNG state using Pohlig-Hellman. This is the "doubly smooth" backdoor.

### ⚠️ Critical: `^` vs `^^` in SageMath

In `.sage` files:

- `^` means **POWER** (same as `**`) — causes `OverflowError` if used for XOR!
- `^^` means **XOR** ← use this!

### ⚠️ nano Must Be Run from `$` Terminal, NOT Inside `sage:`

If you see `sage:` at your prompt you're inside the SageMath interactive console. `nano` there gives `SyntaxError`. Exit first:

```
sage: quit()
```

Then use `nano` from the regular `$` prompt.

---

## Solution — SageMath

### Step 1 — Install SageMath (if not already installed)

First check if sage is available:

```
┌──(zham㉿kali)-[~]
└─$ sage --version
```

If you get "command not found", install it:

```
┌──(zham㉿kali)-[~]
└─$ sudo apt update

┌──(zham㉿kali)-[~]
└─$ sudo apt install sagemath -y
```

This downloads about 1GB and takes several minutes. Let it finish.

If you have conda with a sage environment, you can also use:

```
┌──(zham㉿kali)-[~]
└─$ conda activate sage
```

Either way, confirm sage works before continuing:

```
┌──(zham㉿kali)-[~]
└─$ sage --version
SageMath version 10.x.x
```

### Step 2 — Go to challenge files and extract

Open a regular terminal (make sure you see `$` not `sage:`):

```
┌──(zham㉿kali)-[~]
└─$ cd /media/sf_downloads

┌──(zham㉿kali)-[/media/sf_downloads]
└─$ ls | grep -i rand

┌──(zham㉿kali)-[/media/sf_downloads]
└─$ unzip Randumper.zip

┌──(zham㉿kali)-[/media/sf_downloads]
└─$ ls
```

Confirm you see the challenge files after extracting.

### Step 3 — Create the script

Make sure you are still at the `$` prompt (NOT inside `sage`). Then:

```
┌──(zham㉿kali)-[/media/sf_downloads]
└─$ nano solve.sage
```

Paste this:

```python
# ---- Helper: integer to bytes (no Crypto import needed) ----
def long_to_bytes(n):
    n = int(n)
    if n == 0: return b'\x00'
    length = (n.bit_length() + 7) // 8
    return n.to_bytes(length, 'big')

# ---- Challenge values ----
m = 9792986822963146992114161946549377254001156012300083076259452975070903436656675903033418798369811004678022807932064872669773556983318074157606463933686225452143852071767552582367819932516654167400507310054793760473098421162463267
A = 246920531455433048826966670968977027196902794022573095007307132382393013927558904839982249967656424317118051446771969623490992917094197975459553923550187
B = 3120090466259654766880909097573553950487818021670727992039931232537173262599219376624291488340607228938989670215080876932583555412966984946786628621917436142053528648425399877486481618672307477414787846984446437445450282569184963
e = 4180488827

encrypted = [
    263928838896714332657762314348602240455087661491500131260179929152321790693260683004004767933801216877412896467528419672112455669506824987777448696213888416333755744665868795853306879814179767910033263080874331851322593343273834,
    7500716958005582985508201577146205078774368134456220390802616323312171951781595852608949532619393759348663684985174625019004577610546554994801687569942118260000292201829011312249563036393035204990811392856155820408283216493092108,
    2577794401090299805723177598637436117371525412262771825236813936115237585602114357696686099541164980249580094089526387369627399445447896464668107355247993148990034255262021717192029828447047736912032562575292618672346226630177737,
    9097092753543027532933159793556144814276134227405286080166556624316657274280165256861623248902541076900039285592392074751617153907616670295765377162951858063712964557527212039482031296350052367350738141716598584719951567052177519,
    8174235910244322855637254130843352735528509191186418212056519911128301630950848710182451421471531481098828227735010211554839205099309290314993113489826162548089590990963249899714956197889959999884447438378454452096405595048394416,
    2823506900640363315693941780283568096117331722098156706833610340919194199647098542170689133810904702849664199290565773405646243449194786656181872671323095023235608799444864036190997987871160063941897196360905342203429250068859314,
    4625135637721297092934166648990860031642126233341589454312760229225192256970211899532356300747405146834048765116081788846632527090729839758895834854075755594692611781829177188955301784020596520269704429107749955107365380494152652
]

# ---- Step 1: Factor m ----
print("Factoring m...")
factors = factor(m)
# SageMath's factor() uses ECM to find the prime factors of m
print("Factors:", factors)

# ---- Step 2: Recover first keystream from known plaintext ----
# The flag starts with b"good luck lmao " (given in source!)
# First chunk = first 10 bytes = b"good luck "
known_pt = int.from_bytes(b"good luck ", "big")

first_ks = int(encrypted[0]) ^^ int(known_pt)
# ^^ is XOR in SageMath .sage files — NOT ^ (which means POWER!)
# int() wraps both sides as Python integers for safe bitwise ops
print("First keystream recovered!")

# ---- Step 3: Pohlig-Hellman discrete log ----
print("Computing discrete log (this takes a few minutes)...")
Zm = Integers(m)
# Integers(m) = ring of integers mod m in SageMath

state_partial = int(discrete_log(Zm(first_ks), Zm(B)))
# Finds state such that B^state ≡ first_ks (mod m)
# int() converts SageMath Integer to Python int for safe bitwise ops
print("Done! Brute-forcing lower 16 bits...")

# ---- Step 4: Brute force lower 16 bits (only 65536 tries) ----
initial_state = None
for lower_bits in range(65536):
    candidate = int(state_partial) | int(lower_bits)
    # | is Python bitwise OR — both sides are int() so this is safe
    test_ks = int(pow(B, candidate - (candidate & 0xffff), m))
    if test_ks == first_ks:
        initial_state = candidate
        print("Found initial state!")
        break

# ---- Step 5: Re-run PRNG and decrypt all chunks ----
state = initial_state
plaintext_chunks = []

for ct_chunk in encrypted:
    keystream = int(pow(B, state - (state & 0xffff), m))
    plaintext_chunks.append(keystream ^^ int(ct_chunk))
    # ^^ = XOR in SageMath .sage files

    xor_val = int(pow(A, state, m)) ^^ int(B)
    # ^^ again for XOR
    state = int(pow(xor_val, e, m))

# ---- Step 6: Convert to readable text ----
flag = b''.join(long_to_bytes(p) for p in plaintext_chunks)
print("Flag:", flag)
```

### Step 4 — Save, exit, run

- **Ctrl + O** → **Enter** (save)
- **Ctrl + X** (exit nano)

```
┌──(zham㉿kali)-[/media/sf_downloads]
└─$ sage solve.sage
Factoring m...
Factors: 17297082179958074003 * 10124460123717732577 * ...
First keystream recovered!
Computing discrete log (this takes a few minutes)...
Done! Brute-forcing lower 16 bits...
Found initial state!
Flag: b'good luck lmao syborg_ctf{a_doubly_smooth_number_in_this_economy?!?}'
```

✅ Got the flag! 🎯

> ⚠️ The `discrete_log` step takes several minutes. The terminal will look frozen — this is normal. Do NOT close it. Wait for it to finish.

---

## Common Mistakes

| Mistake | Fix |
|--------|-----|
| `command not found: sage` | Run `sudo apt install sagemath -y` first |
| `SyntaxError` after typing `nano` | You're inside `sage:` console — type `quit()` first |
| Terminal looks frozen after "Computing discrete log..." | Normal — just wait, it takes several minutes |
| `OverflowError: exponent must be at most...` | Used `^` for XOR — change to `^^` in `.sage` files |
| `ModuleNotFoundError: No module named 'Crypto'` | Remove any `from Crypto import` — the script above doesn't need it |

---

## Tools Used

| Tool | Purpose | Level |
|------|---------|-------|
| SageMath | `factor(m)`, `discrete_log()`, Pohlig-Hellman | ⭐⭐⭐ Advanced |

---

## Key Takeaways

- Pohlig-Hellman breaks discrete log when group order has only small prime factors (smooth)
- In SageMath `.sage` files: `^` = POWER, `^^` = XOR — very easy to mix up!
- The known plaintext `"good luck lmao "` in the source code was the entry point for the attack
- `nano` must be run from the `$` terminal prompt, never from inside `sage:` console
- The `discrete_log` step looks frozen but is working — just wait
