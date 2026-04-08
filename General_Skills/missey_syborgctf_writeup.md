# Missey — SyborgCTF Writeup

**Challenge:** Missey    
**Category:** General Skills    
**Difficulty:** Very Hard    
**Points:** 300 pts    
**Flag:** `syborg_ctf{M15SED_INBY7$}`    

---

## Description

> Seems someone tried to bruteforce on IPs, so can you reveal what he hides?

We're given a `.pcap` file. Someone is "brute-forcing" IP addresses — but something is secretly hidden inside the traffic.

---

## Background Knowledge (Read This First!)

Before solving, you need to understand two things:

### What is a PCAP file?
A `.pcap` file is a **recording of network traffic** — like a security camera but for internet packets. Tools like Wireshark can open and read them.

### What is an IP address?
An IP address looks like this: `192.126.117.112`

It has **4 parts** separated by dots. The last part (`112`) is called the **last octet**.
- Minimum value: `0`
- Maximum value: `255`
- ASCII characters also use values `0–255` — so a last octet **CAN represent a letter!**

### The Trick in This Challenge

The attacker disguised a secret message by sending packets to IPs where the **last octet = ASCII value of a character**. For example:

| Destination IP | Last Octet | ASCII Value | Character |
|----------------|-----------|-------------|-----------|
| `192.126.117.70` | 70 | 70 | **F** |
| `192.126.117.76` | 76 | 76 | **L** |
| `192.126.117.65` | 65 | 65 | **A** |
| `192.126.117.71` | 71 | 71 | **G** |

Read them in order → **FLAG**! 🎯

---

## ⚠️ Important: Why "First Occurrence Only" Matters

Each IP was contacted **many times** to fake a brute-force. If you read ALL packets:
```
pppppPPPPPQQQQQvvvvv3333...
```
Each character repeated hundreds of times — garbage!

But if you only take the **first time** each IP appears, you get each character exactly **once**, in the correct order:
```
pPQv3XRWzu2KZOUjJV8w4CcHorxh6qk09|FLAG{M15SED_INBY7$}
```

**The order of first contact tells the story. Duplicates are just camouflage noise.** ✅

---

## Solution — Method 1: Wireshark + CyberChef ✅ Easiest (No Coding)

### Step 1: Open the file in Wireshark

Download Wireshark free at 👉 https://www.wireshark.org/

Open `Missey.pcap` by dragging it into Wireshark. You'll see thousands of packets — don't panic!

### Step 2: Filter the suspicious traffic

In the **filter bar** at the top, type this and press Enter:

```
ip.dst == 192.126.117.0/24
```

This shows **only packets going to the `192.126.117.x` subnet** — the brute-force traffic.

> 💡 `/24` means "match any IP starting with `192.126.117.`"

### Step 3: Go to Statistics → Endpoints

1. Click the **Statistics** menu at the top
2. Click **Endpoints**
3. Click the **IPv4** tab
4. Sort by **first seen** order

You'll see all unique destination IPs:
```
192.126.117.112
192.126.117.80
192.126.117.81
192.126.117.118
...
```

### Step 4: Extract the last octets in order

Write down the last number of each IP **in the order they first appeared**:
```
112, 80, 81, 118, 51, 88, 82, 87, 122, 117, 50, 75 ... 70, 76, 65, 71, 123, 77, 49, 53, 83, 69, 68, 95, 73, 78, 66, 89, 55, 36, 125
```

### Step 5: Convert to ASCII using CyberChef

1. Go to 👉 https://gchq.github.io/CyberChef/
2. Search **From Charcode** → drag it into the Recipe
3. Set **Delimiter** to `Comma`
4. Paste the numbers into the Input box
5. Click **BAKE!**

Output:
```
pPQv3XRWzu2KZOUjJV8w4CcHorxh6qk09|FLAG{M15SED_INBY7$}
```

The flag content is: **`M15SED_INBY7$`** 🎉

---

## Solution — Method 2: Python Script 🐍 Recommended for Speed

### Step 1: Install Scapy

```bash
pip install scapy
```

### Step 2: Save this script as `solve.py`

```python
from scapy.all import *

# Load the pcap file
print("[*] Loading pcap file...")
pkts = rdpcap('Missey.pcap')
print(f"[*] Total packets loaded: {len(pkts)}")

# Track IPs we've already seen (first occurrence only!)
seen_set = set()
seen_octets = []

# Loop through every packet
for pkt in pkts:
    if IP in pkt:                        # Only look at IP packets
        dst = pkt[IP].dst                # Get destination IP

        if dst.startswith('192.126.117.'):   # Only the brute-force subnet

            if dst not in seen_set:          # Only first occurrence!
                seen_set.add(dst)

                last_octet = int(dst.split('.')[-1])   # Get last number
                seen_octets.append(last_octet)

# Convert all the numbers to characters
result = ''.join(chr(b) for b in seen_octets)

print(f"\n[*] Decoded message:")
print(result)
print(f"\n[*] Flag: syborg_ctf{{{result.split('{')[1].split('}')[0]}}}")
```

### Step 3: Run it

```bash
python3 solve.py
```

### Output:

```
[*] Loading pcap file...
[*] Total packets loaded: 12532

[*] Decoded message:
pPQv3XRWzu2KZOUjJV8w4CcHorxh6qk09|FLAG{M15SED_INBY7$}

[*] Flag: syborg_ctf{M15SED_INBY7$}
```

---

## Solution — Method 3: Tshark One-Liner ⚡ Fastest

**Tshark** is the command-line version of Wireshark. One single command does everything.

### Install Tshark

```bash
# Linux
sudo apt install tshark

# Mac
brew install wireshark

# Windows — install Wireshark, Tshark comes with it
```

### Run this command

```bash
tshark -r Missey.pcap -Y "ip.dst == 192.126.117.0/24" -T fields -e ip.dst | awk -F. '!seen[$0]++ {printf "%c", $4}'
```

**What each part does:**

| Part | Purpose |
|------|---------|
| `-r Missey.pcap` | Read the pcap file |
| `-Y "ip.dst == 192.126.117.0/24"` | Filter to only our subnet |
| `-T fields -e ip.dst` | Print only destination IP |
| `awk -F. '!seen[$0]++'` | Keep only first occurrence of each IP |
| `{printf "%c", $4}` | Print last octet as ASCII character |

### Output:

```
pPQv3XRWzu2KZOUjJV8w4CcHorxh6qk09|FLAG{M15SED_INBY7$}
```

Done in one line! ⚡

---

## Full Decoding Table (Flag Part Only)

| Destination IP | Last Octet | Character |
|----------------|-----------|-----------|
| `192.126.117.70` | 70 | **F** |
| `192.126.117.76` | 76 | **L** |
| `192.126.117.65` | 65 | **A** |
| `192.126.117.71` | 71 | **G** |
| `192.126.117.123` | 123 | **{** |
| `192.126.117.77` | 77 | **M** |
| `192.126.117.49` | 49 | **1** |
| `192.126.117.53` | 53 | **5** |
| `192.126.117.83` | 83 | **S** |
| `192.126.117.69` | 69 | **E** |
| `192.126.117.68` | 68 | **D** |
| `192.126.117.95` | 95 | **_** |
| `192.126.117.73` | 73 | **I** |
| `192.126.117.78` | 78 | **N** |
| `192.126.117.66` | 66 | **B** |
| `192.126.117.89` | 89 | **Y** |
| `192.126.117.55` | 55 | **7** |
| `192.126.117.36` | 36 | **$** |
| `192.126.117.125` | 125 | **}** |

Reading all → **`FLAG{M15SED_INBY7$}`** ✅

---

## Tools Used

| Tool | Purpose | Level |
|------|---------|-------|
| Wireshark + CyberChef | Visual, no coding needed | ⭐ Easiest |
| Python + Scapy | Automated script | ⭐⭐ Medium |
| Tshark (command line) | One-liner, fastest | ⭐⭐⭐ Advanced |

---

## Key Takeaways

- **PCAP files** record real network traffic — Wireshark reads them
- Data can be hidden inside **IP address last octets** — a sneaky steganography trick
- The "brute-force" was fake — it was actually **covert data exfiltration**
- Always check **what's changing** in repetitive traffic — that's where the secret is
- **First occurrence order** matters — duplicates are just camouflage noise
