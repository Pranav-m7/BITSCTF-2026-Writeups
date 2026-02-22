# Queen Roselia's Diamond - Steganography Challenge

## Challenge Overview
A sophisticated steganography challenge involving a 64-bit floating-point TIFF image with multiple hidden layers, including a decoy QR code and a fragmented flag hidden in visual noise.

**Category:** Steganography / Forensics  
**Difficulty:** Hard

---

## Solution Walkthrough

### Step 1: Initial Reconnaissance
First, I checked the metadata and ran strings on the `.tif` file. `exiftool` showed it was a 64-bit floating-point TIFF image.

```bash
exiftool challenge_stego.tif
```

```bash
strings challenge_stego.tif
```

![Strings Output](images/queen-roselia-diamond/strings-output.png)
*Image 2: strings command output showing garbage data*

---

### Step 2: Extracting Layers (Python)
Since standard image viewers can't handle 64-bit floats properly, I used a Python script to extract the 8 individual byte layers from the raw data.

**sl8.py:**
```python
import numpy as np
from PIL import Image

width, height = 635, 635
offset = 8

with open('challenge_stego.tif', 'rb') as f:
    f.seek(offset)
    raw_data = np.fromfile(f, dtype='d', count=width * height)

data_matrix = raw_data.reshape((height, width))
byte_view = data_matrix.view(np.uint8).reshape(height, width, 8)

for i in range(8):
    layer = byte_view[:, :, i]
    if np.max(layer) > np.min(layer):
        enhanced = ((layer - np.min(layer)) / (np.max(layer) - np.min(layer)) * 255).astype(np.uint8)
    else:
        enhanced = layer
    Image.fromarray(enhanced).save(f'layer_{i}_enhanced.png')
```

```bash
python3 sl8.py
```

![Python Script Output](images/queen-roselia-diamond/python-output.png)
*Image 3: Python script terminal output*

![Extracted Layers](images/queen-roselia-diamond/extracted-layers.png)
*Image 4: Extracted layer folder screenshot*

---

### Step 3: The Decoy
Looking through the extracted layers, one of them contained a QR code. I enhanced it to make it scannable.

![Enhanced QR Code](images/queen-roselia-diamond/qr-code.png)
*Image 5: Enhanced QR code image*

Scanning this QR code resulted in a **fake flag** - a clever decoy!

![Fake Flag](images/queen-roselia-diamond/fake-flag.png)
*Image 1: Fake flag from QR scan*

---

### Step 4: Finding the Real Flag
The real flag was hidden in the visual noise of **layer 7**. I opened `layer_7_enhanced.png` in GIMP and adjusted the **Color Threshold** to isolate the bright white text fragments from the gray QR code background.

![GIMP Threshold 1](images/queen-roselia-diamond/gimp-threshold-1.png)
*Image 6.1: GIMP screenshot - Color Threshold adjustment*

![GIMP Threshold 2](images/queen-roselia-diamond/gimp-threshold-2.png)
*Image 6.2: GIMP screenshot - Revealing flag part*

![GIMP Threshold 3](images/queen-roselia-diamond/gimp-threshold-3.png)
*Image 6.3: GIMP screenshot - Revealing flag part*

![GIMP Threshold 4](images/queen-roselia-diamond/gimp-threshold-4.png)
*Image 6.4: GIMP screenshot - Revealing flag part*

---

### Step 5: Reconstructing the Flag
The text was fragmented and sheared. I pieced the visible parts together and had to guess the missing `bu7` part based on the leetspeak context (`qr_bu7_n07_7h47_qr`).

![Flag Reconstruction](images/queen-roselia-diamond/flag-reconstruction.png)
*Image 7: Final flag constructed in text editor*

---

## The Flag
```
bitsctf{qr_bu7_n07_7h47_qr}
```

---

## Tools Used
- `exiftool` - Metadata analysis
- `strings` - String extraction
- Python (NumPy, PIL) - Custom layer extraction
- GIMP - Image manipulation and color threshold analysis

## Key Techniques
- 64-bit floating-point TIFF analysis
- Byte-layer extraction
- Decoy detection
- Color threshold manipulation
- Visual steganography

---

*Writeup by [Your Name]*
