# Sodashi — SyborgCTF Writeup

**Challenge:** Sodashi  
**Category:** Forensics  
**Difficulty:** Easy  
**Points:** 50 pts  
**Flag:** `syborg_ctf{this_horse_is_kinda_majestic_frfr}`

---

## Description

> The owner of this horse is quite protective of Sodashi's likeness, to respect their wishes we've reframed from even showing her.

**Flag Format:** `syborg_ctf{}`  
**NOTE:** spaces are to be replaced with `_`

**Hint:** This challenge requires some OSINT to solve.

**Attachment:** `Sodashi.zip` → contains `Sodashiless.bmp` and `sodashi.py`

---

## Background Knowledge (Read This First!)

### What is Sodashi?

Sodashi is a real racehorse — a famous white horse known for winning major races. The challenge is named after her and the attachment is a blurred image related to her.

### What is OSINT?

OSINT (Open Source Intelligence) means finding information using publicly available sources. In this challenge, it means finding the original image of Sodashi online.

### What does sodashi.py do?

Reading the script tells us exactly how the image was made:

```python
flag = "ictf{REDACTED}"
img = Image.open("Sodashi.bmp")          # original image
draw = ImageDraw.Draw(img)

font = ImageFont.truetype("calibri.ttf", 100)   # Calibri font, size 100
draw.text((200, 500), flag, fill="black", font=font)  # draw flag at (200, 500)

horsie_region = img.crop((100, 250, 1800, 800))  # crop region that includes the flag text
blurred_horsie = horsie_region.filter(ImageFilter.GaussianBlur(50))  # blur with radius 50
img.paste(blurred_horsie, (100, 250, 1800, 800))  # paste blurred region back
img.save("Sodashiless.bmp")
```

**What this means:**
1. The flag was drawn in **black text, Calibri font size 100, at position (200, 500)**
2. A region around the text was **blurred with GaussianBlur radius 50**
3. We are given the blurred result `Sodashiless.bmp`

We know the **original source image** (via OSINT) and we know the **exact blur parameters**. This means we can recreate what the blurred text looks like for any candidate character, and compare it to the actual blurred image to find which character matches best.

---

## Solution — OSINT + Image Comparison

### Step 1 — Find the Original Image

The original Sodashi image is publicly available online:

👉 **https://idolhorse.com/wp-content/uploads/2024/07/SodashiWW.jpg**

You can find this by searching for "Sodashi horse" and looking for the white racehorse image that matches the blurred background in `Sodashiless.bmp`.

### Step 2 — Understand the Method

Since we know:
- The original image (found via OSINT)
- The font: Calibri, size 100
- The text position: (200, 500)
- The blur: GaussianBlur radius 50
- The crop region: (100, 250, 1800, 800)

We can test **every possible character** at each position in the flag. For each candidate character, we:
1. Draw it on the original image at the known position
2. Apply the same blur
3. Compare the result to `Sodashiless.bmp`
4. The character that makes the **smallest difference** is the correct one

This is a **known-plaintext / brute-force-by-comparison** attack on the blur.

### Step 3 — Set up the environment

> ⚠️ Create the venv in `~` (home), not inside `/media/sf_downloads` — VirtualBox shared folders don't support symlinks.

```
┌──(zham㉿kali)-[~]
└─$ python3 -m venv myenv
                                                                                
┌──(zham㉿kali)-[~]
└─$ source myenv/bin/activate
                                                                                
┌──(myenv)─(zham㉿kali)-[~]
└─$ pip install pillow
```

### Step 4 — Go to challenge files and download the original image

```
┌──(myenv)─(zham㉿kali)-[~]
└─$ cd /media/sf_downloads

┌──(myenv)─(zham㉿kali)-[/media/sf_downloads]
└─$ unzip Sodashi.zip

┌──(myenv)─(zham㉿kali)-[/media/sf_downloads]
└─$ wget "https://idolhorse.com/wp-content/uploads/2024/07/SodashiWW.jpg" -O Sodashi.jpg
```

Convert it to BMP:

```
┌──(myenv)─(zham㉿kali)-[/media/sf_downloads]
└─$ python3 -c "
from PIL import Image
img = Image.open('Sodashi.jpg').convert('RGB')
img.save('Sodashi.bmp')
"
```

### Step 5 — Get Calibri font

Calibri is a Windows font. On Kali you need to get it:

```
┌──(myenv)─(zham㉿kali)-[/media/sf_downloads]
└─$ sudo apt install ttf-mscorefonts-installer -y

┌──(myenv)─(zham㉿kali)-[/media/sf_downloads]
└─$ fc-list | grep -i calibri
```

Then copy the calibri.ttf to the current folder:

```
┌──(myenv)─(zham㉿kali)-[/media/sf_downloads]
└─$ cp /path/shown/by/fc-list/calibri.ttf .
```

### Step 6 — Create the solver script

```
┌──(myenv)─(zham㉿kali)-[/media/sf_downloads]
└─$ nano solve.py
```

Paste this:

```python
from PIL import Image, ImageDraw, ImageFont, ImageFilter
import numpy as np

# ---- Load the blurred target image (given in the challenge) ----
target = np.array(Image.open("Sodashiless.bmp").convert("RGB"))
# Convert to numpy array for pixel-by-pixel comparison

# ---- Load the original image (found via OSINT) ----
original = Image.open("Sodashi.bmp").convert("RGB")

# ---- These values come directly from sodashi.py ----
FONT_PATH = "calibri.ttf"    # need calibri.ttf in the same folder
FONT_SIZE = 100
TEXT_X = 200
TEXT_Y = 500
CROP = (100, 250, 1800, 800)
BLUR_RADIUS = 50

# ---- Characters to try at each position ----
CHARSET = "abcdefghijklmnopqrstuvwxyz0123456789_{}!? "

font = ImageFont.truetype(FONT_PATH, FONT_SIZE)

flag = "ictf{"
# We already know the prefix from the flag format

print("Brute-forcing flag characters...")

while not flag.endswith("}"):
    best_char = None
    best_score = float('inf')

    for ch in CHARSET:
        # Try this candidate: current known flag + candidate character
        candidate_flag = flag + ch

        # Recreate what the image would look like with this candidate
        img = original.copy()
        draw = ImageDraw.Draw(img)
        draw.text((TEXT_X, TEXT_Y), candidate_flag, fill="black", font=font)

        # Apply same blur
        region = img.crop(CROP)
        blurred = region.filter(ImageFilter.GaussianBlur(BLUR_RADIUS))
        img.paste(blurred, CROP)

        # Compare to target image — lower score = better match
        candidate_arr = np.array(img.convert("RGB"))
        score = np.sum((candidate_arr.astype(int) - target.astype(int)) ** 2)
        # Sum of squared pixel differences

        if score < best_score:
            best_score = score
            best_char = ch

    flag += best_char
    print(f"Found so far: {flag}")

print("\nFull flag:", flag)
print("Syborg flag: syborg_ctf{" + flag[5:-1] + "}")
```

### Step 7 — Save, exit, run

- **Ctrl + O** → **Enter** (save)
- **Ctrl + X** (exit nano)

```
┌──(myenv)─(zham㉿kali)-[/media/sf_downloads]
└─$ python3 solve.py
Brute-forcing flag characters...
Found so far: ictf{t
Found so far: ictf{th
Found so far: ictf{thi
Found so far: ictf{this
...
Full flag: ictf{this_horse_is_kinda_majestic_frfr}
Syborg flag: syborg_ctf{this_horse_is_kinda_majestic_frfr}
```

✅ Got the flag! 🎯

> ⚠️ This will take several minutes — it tests every character at every position, recreating and blurring the image each time.

---

## Why This Works

GaussianBlur mixes neighboring pixels together. If you draw a `t` at a position and blur it, the resulting pixels are slightly different from drawing an `h` at the same position and blurring it. The difference is small but measurable. By comparing the sum of squared pixel differences to the actual blurred image, we find which character produces the closest match — that's the real character.

---

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| `FileNotFoundError: calibri.ttf` | You need the actual Calibri font file — copy it from a Windows machine or install `ttf-mscorefonts-installer` |
| `FileNotFoundError: Sodashi.bmp` | You need to download the original image first and save it as `Sodashi.bmp` |
| Script gives wrong characters | Make sure you downloaded the correct original image from the exact URL above |

---

## Tools Used

| Tool | Purpose | Level |
|------|---------|-------|
| OSINT (Google search) | Find original Sodashi image URL | ⭐ Easy |
| Python 3 + Pillow | Recreate blur per candidate character, compare pixel differences | ⭐⭐⭐ Advanced |

---

## Key Takeaways

- **OSINT** means using public info — the original horse image was findable online
- Knowing the **exact blur parameters** (type, radius, crop region, font, position) lets you reverse-engineer the blurred text
- **GaussianBlur is not encryption** — if you know the original image, you can compare blurred versions of candidates to find the real text
- The script in the zip (`sodashi.py`) told us everything we needed: font, size, position, blur radius, crop coordinates
- Spaces in the flag become `_` as the challenge states
