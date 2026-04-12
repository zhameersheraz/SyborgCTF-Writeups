# Missey — SyborgCTF Writeup

**Challenge:** Missey  
**Category:** General Skills  
**Difficulty:** Very Hard  
**Points:** 300 pts  
**Flag:** `syborg_ctf{M15SED_INBY7$}`

---

## Description

> Seems someone tried to bruteforce on IPs, so can you reveal what he hides?

We are given a `.pcap` file. The traffic looks like a port scanner brute-forcing IPs, but a hidden message is secretly encoded inside the destination IP addresses.

---

## What You Need to Know First

### What is a PCAP file?

A `.pcap` file is a **recording of network traffic** — like a video replay of every packet sent and received. Wireshark can open and read it.

### What is an IP address?

An IP address looks like: `192.126.117.112`

It has **4 parts** separated by dots. The last part is called the **last octet**. In this case, the last octet is `112`.

- IP octets range from `0` to `255`
- ASCII characters also use values `0` to `127` (and beyond)
- So a last octet **can represent a letter or symbol** — for example, octet `65` = the letter `A`

### The Hidden Message Trick

The attacker sent packets to a series of destination IPs where the **last octet equals the ASCII value of a character**. Example:

| Destination IP | Last Octet | ASCII Character |
|---|---|---|
| `192.126.117.70` | 70 | `F` |
| `192.126.117.76` | 76 | `L` |
| `192.126.117.65` | 65 | `A` |
| `192.126.117.71` | 71 | `G` |

Reading those in order → `FLAG`

### Why "First Occurrence Only"?

Each IP was contacted **hundreds of times** to fake a real brute-force scan. If you read all packets, every character repeats hundreds of times — useless noise.

The trick: take only the **first time each unique IP appears**, in the order they appear. That gives you each character exactly **once**, in the correct sequence.

---

## Solution — Method 1: Wireshark + CyberChef (No Coding, Recommended)

### Step 1 — Open the PCAP in Wireshark

Open Wireshark on Kali. Drag `Missey.pcap` into it, or go to **File → Open**.

You will see thousands of packets. Do not panic.

### Step 2 — Filter to Show Only the Suspicious Traffic

At the top of Wireshark, there is a green filter bar. Click it and type this filter exactly, then press **Enter**:

```
ip.dst == 192.126.117.0/24
```

This hides everything except packets going to the `192.126.117.x` range — the subnet the attacker was targeting. The bar turns green when the filter is valid. You should now see only those packets.

> **What does `/24` mean?** It means "match any IP that starts with `192.126.117.`" — it is a subnet mask shorthand.

### Step 3 — Extract the Unique IPs in First-Seen Order

Now you need to find the destination IPs **in the order they first appeared**. Here is the most reliable way:

Go to **Statistics → Endpoints** in the menu bar.

Click the **IPv4** tab. You will see a list of all unique destination IPs like:

```
192.126.117.36
192.126.117.48
192.126.117.49
192.126.117.50
...
192.126.117.125
```

> **Important:** The Endpoints list is sorted alphabetically by default, NOT by first-seen order. This is fine for getting the full list of IPs, but to get them in the right ORDER, use the tshark method in Step 3b, OR use the Python script method below.

#### Step 3b — Getting the Correct Order (the reliable way)

The easiest way to get the correct order is to use **tshark** in the terminal. Open a terminal and run:

```
┌──(zham㉿kali)-[/media/sf_downloads]
└─$ tshark -r Missey.pcap -Y "ip.dst == 192.126.117.0/24" -T fields -e ip.dst | awk -F. '!seen[$0]++ {print $4}'
```

This prints the last octet of each unique destination IP, in the exact order they first appeared in the capture. You will see a list like:

```
112
80
81
118
51
88
82
87
122
117
50
75
...
70
76
65
71
123
77
49
53
83
69
68
95
73
78
66
89
55
36
125
```

**This is where those numbers come from.** Each number is the last octet of a destination IP. Each octet = one ASCII character.

### Step 4 — Convert the Numbers to Text with CyberChef

1. Go to [https://gchq.github.io/CyberChef/](https://gchq.github.io/CyberChef/)
2. In the Operations search box on the left, type `from char` and find **From Charcode**
3. Drag **From Charcode** into the Recipe box in the middle
4. Set **Delimiter** to `Comma`
5. Set **Base** to `10`
6. In the Input box on the right, paste the numbers **separated by commas** like this:

```
112, 80, 81, 118, 51, 88, 82, 87, 122, 117, 50, 75, 90, 79, 85, 106, 74, 86, 56, 119, 52, 67, 99, 72, 111, 114, 120, 104, 54, 113, 107, 48, 57, 124, 70, 76, 65, 71, 123, 77, 49, 53, 83, 69, 68, 95, 73, 78, 66, 89, 55, 36, 125
```

7. Click **BAKE!**

Output:

```
pPQv3XRWzu2KZOUjJV8w4CcHorxh6qk09|FLAG{M15SED_INBY7$}
```

The flag is at the end: `FLAG{M15SED_INBY7$}`, so the full flag is `syborg_ctf{M15SED_INBY7$}`.

---

### Why Base 10, NOT Base 16?

This is a very common mistake. Here is why it must be Base 10:

IP address octets are written in **decimal** (base 10). The number `112` in an IP address literally means one hundred and twelve — it is already a decimal number.

- In base 10: `112` → ASCII character `p` ✅
- In base 16: `112` → the tool reads it as hex `0x112` = `274` → no valid ASCII character ❌

So always use **Base 10** when your input numbers came from IP address octets.

---

## Solution — Method 2: Python Script

If you prefer to automate everything in one step, use this script.

### Step 1 — Go to the folder where Missey.pcap is

First find the file if you are not sure where it is:

```
┌──(zham㉿kali)-[~]
└─$ find / -name "Missey.pcap" 2>/dev/null
```

Then `cd` into that folder:

```
┌──(zham㉿kali)-[~]
└─$ cd /media/sf_downloads
```

Confirm the file is there:

```
┌──(zham㉿kali)-[/media/sf_downloads]
└─$ ls
Missey.pcap
```

### Step 2 — Install Scapy

On Kali Linux, the normal `pip install` command will give you an error. Use this instead:

```
┌──(zham㉿kali)-[/media/sf_downloads]
└─$ pip install scapy --break-system-packages
```

> **Why `--break-system-packages`?** Kali protects the system Python from pip installs by default. This flag bypasses that. It is safe to use on a CTF machine.

### Step 3 — Create the script using nano

Go to your **home directory** first. This is important — there is a file called `code.py` inside `/media/sf_downloads/` that conflicts with Python's built-in module and causes Scapy to crash. Running from home avoids that conflict.

```
┌──(zham㉿kali)-[/media/sf_downloads]
└─$ cd ~
```

Now open a new file:

```
┌──(zham㉿kali)-[~]
└─$ nano solve.py
```

Your terminal turns into a text editor. Type or paste this code. Notice the **full path** to `Missey.pcap` is written directly in the script so it does not matter which folder you run from:

```python
from scapy.all import *

pkts = rdpcap('/media/sf_downloads/Missey.pcap')

seen_set = set()
seen_octets = []

for pkt in pkts:
    if IP in pkt:
        dst = pkt[IP].dst
        if dst.startswith('192.126.117.'):
            if dst not in seen_set:
                seen_set.add(dst)
                last_octet = int(dst.split('.')[-1])
                seen_octets.append(last_octet)

result = ''.join(chr(b) for b in seen_octets)
print(result)
```

Now save and exit nano:

1. Press **Ctrl + O** — saves the file
2. Press **Enter** — confirms the filename
3. Press **Ctrl + X** — exits nano

### Step 4 — Run it

Make sure you are still in the home directory (`~`), then run:

```
┌──(zham㉿kali)-[~]
└─$ python3 solve.py
pPQv3XRWzu2KZOUjJV8w4CcHorxh6qk09|FLAG{M15SED_INBY7$}
```

> **Why did it fail before?** There is a file called `code.py` in `/media/sf_downloads/`. Python mistook it for its own built-in `code` module, which Scapy depends on. Running the script from home (`~`) where there is no `code.py` fixes this completely.

---

## Solution — Method 3: Tshark One-Liner (Fastest)

One single command does everything — extracts IPs, keeps first occurrences only, and converts to ASCII.

### Step 1 — Go to the folder where Missey.pcap is

tshark cannot find the file if you are in the wrong folder. First locate it:

```
┌──(zham㉿kali)-[~]
└─$ find ~ -name "Missey.pcap" 2>/dev/null
```

Then `cd` into that folder:

```
┌──(zham㉿kali)-[~]
└─$ cd /media/sf_downloads
```

Confirm the file is there:

```
┌──(zham㉿kali)-[/media/sf_downloads]
└─$ ls
Missey.pcap
```

### Step 2 — Run the command

```
┌──(zham㉿kali)-[/media/sf_downloads]
└─$ tshark -r Missey.pcap -Y "ip.dst == 192.126.117.0/24" -T fields -e ip.dst | awk -F. '!seen[$0]++ {printf "%c", $4}'
pPQv3XRWzu2KZOUjJV8w4CcHorxh6qk09|FLAG{M15SED_INBY7$}777
```

> **Why does `777` appear at the end?** The pcap contains a few extra packets after the closing `}` that also target the same subnet. Octet `55` = ASCII `7`, and it appears as 3 extra unique IPs near the end of the capture. This is just extra noise the attacker left in — it does not affect the flag. The flag ends at `}` and is: `syborg_ctf{M15SED_INBY7$}` ✅

**Breaking down what each part does:**

| Part | What it does |
|---|---|
| `tshark -r Missey.pcap` | Open and read the pcap file |
| `-Y "ip.dst == 192.126.117.0/24"` | Filter to only show our target subnet |
| `-T fields -e ip.dst` | Print only the destination IP field |
| `awk -F. '!seen[$0]++'` | Only pass through each unique IP the first time it appears |
| `{printf "%c", $4}` | Print the 4th field (last octet) as an ASCII character |

---

## Flag Decoding Table (Final Part)

| Destination IP | Last Octet | ASCII Character |
|---|---|---|
| `192.126.117.70` | 70 | `F` |
| `192.126.117.76` | 76 | `L` |
| `192.126.117.65` | 65 | `A` |
| `192.126.117.71` | 71 | `G` |
| `192.126.117.123` | 123 | `{` |
| `192.126.117.77` | 77 | `M` |
| `192.126.117.49` | 49 | `1` |
| `192.126.117.53` | 53 | `5` |
| `192.126.117.83` | 83 | `S` |
| `192.126.117.69` | 69 | `E` |
| `192.126.117.68` | 68 | `D` |
| `192.126.117.95` | 95 | `_` |
| `192.126.117.73` | 73 | `I` |
| `192.126.117.78` | 78 | `N` |
| `192.126.117.66` | 66 | `B` |
| `192.126.117.89` | 89 | `Y` |
| `192.126.117.55` | 55 | `7` |
| `192.126.117.36` | 36 | `$` |
| `192.126.117.125` | 125 | `}` |

Read together → `FLAG{M15SED_INBY7$}` ✅

---

## Tools Used

| Tool | Purpose | Difficulty |
|---|---|---|
| Wireshark + CyberChef | Visual, no coding needed | ⭐ Easiest |
| Python + Scapy | Automated script | ⭐⭐ Medium |
| Tshark (command line) | One-liner, fastest | ⭐⭐⭐ Advanced |

---

## Key Takeaways

- PCAP files record real network traffic — Wireshark and tshark read them
- Data can be hidden inside the **last octet of IP addresses** — a steganography technique
- The fake brute-force was a cover — the real goal was **covert data exfiltration**
- Only the **first occurrence** of each IP matters — duplicates are just noise
- IP octets are decimal numbers, so always use **Base 10** in CyberChef when converting them to ASCII
