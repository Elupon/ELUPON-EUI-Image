# ELUPON Universal Image (EUI Specification)

Welcome to the official technical repository for the **ELUPON Universal Image (EUI)** open-source graphics format. Designed from scratch for high-performance mobile architectures, zero processor overhead, and direct native hardware buffer execution.

---

## 🖼️ ELUPON Universal Image (EUI Format - ELUPON UBRG v3)

The **EUI** standard (Version 3) incorporates the proprietary **ELUPON UBRG v3** (Universal Binary RGB) graphics architecture. It completely discards container inflation, color profiles, EXIF metadata, and compression artifacts. By storing pixel data in a raw 24-bit per pixel sequential `RGB888` block (8-bit Red, 8-bit Green, 8-bit Blue), it delivers mathematically perfect image fidelity, instantly inflating into mobile graphics memory (`android.graphics.Bitmap`) without CPU decoding strain.

### Technical Metadata
- **MIME Media Type:** `image/prs.elupon-eui`
- **File Extension:** `.eui`
- **Magic Number (Signature):** `EUIV3` (`45 55 49 56 33` in Hex)

### Binary Structure Specification
Every `.eui` file contains an 11-byte unpadded structural header followed by a raw 24-bit sequential pixel payload. Total payload size always equals `Width * Height * 3` bytes.

#### File Header Layout

| Offset (Bytes) | Size | Data Type | Field Description |
|----------------|------|-----------|-------------------|
| 0 - 4          | 5    | `char[]`  | Magic Signature: Must be string `"EUIV3"` |
| 5              | 1    | `uint8`   | Format Version: Set to `3` |
| 6 - 7          | 2    | `uint16`  | Image Width in Pixels (Little-Endian) |
| 8 - 9          | 2    | `uint16`  | Image Height in Pixels (Little-Endian) |
| 10             | 1    | `uint8`   | Compression Type: Set to `7` (ELUPON UBRG TrueColor RAW) |

### 🗜️ Reference Python Graphics Encoder
Use this script to programmatically package any native high-definition image into a production-ready `.eui` container running the **ELUPON UBRG v3** engine:

```python
import struct
import os
from PIL import Image

def elupon_encode_image_ubrg3(img_path, eui_path):
    if not os.path.exists(img_path):
        print(f"❌ File {img_path} not found!")
        return

    # Load image and force pure RGB conversion
    img = Image.open(img_path).convert('RGB')
    width, height = img.size
    pixels = list(img.getdata())

    print(f"🛸 Кодек ELUPON UBRG v3... Размер: {width}x{height} px")
    
    compressed_bytes = bytearray()
    
    # Pack perfect 24-bit per pixel (3 bytes) dynamic array without lossless reduction
    for p in pixels:
        compressed_bytes.append(p[0]) # Red
        compressed_bytes.append(p[1]) # Green
        compressed_bytes.append(p[2]) # Blue

    # Write binary stream with strict EUIV3 signature layout
    with open(eui_path, 'wb') as eui:
        eui.write(b'EUIV3')
        eui.write(struct.pack('<B', 3))  # Version 3
        eui.write(struct.pack('<H', width))
        eui.write(struct.pack('<H', height))
        eui.write(struct.pack('<B', 7))  # Type: ELUPON UBRG TrueColor RAW
        eui.write(compressed_bytes)

    print(f"🔥 Success! Ultra-fidelity file created: {eui_path} ({len(compressed_bytes) + 11} bytes)")

elupon_encode_image_ubrg3("input.jpg", "image.eui")
```
