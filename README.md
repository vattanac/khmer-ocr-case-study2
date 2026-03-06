# KhmerOCR — Khmer Optical Character Recognition Engine

> 🌐 **Live Demo:** [demo](https://khmer-orc-case-study2.netlify.app/)

> **This is original research and engineering work.** All model training, architecture design, data pipeline construction, and implementation were conducted independently by the author. This document serves as a clear record of authorship, methodology, and intellectual contribution.

---

## 📌 Author & Ownership Statement

| Field | Detail |
|---|---|
| **Author** | Seanghay Yath |
| **Project Name** | KhmerOCR |
| **License** | MIT |
| **Repository** | github.com/seanghay/KhmerOCR |
| **Language Focus** | Khmer script (Cambodian) |
| **Version** | v0.2.0 |

This project — including its trained neural network models, data pipeline, inference engine, C++ implementation, and Python package — represents **original, independent research and development work**. The author retains full intellectual ownership of all creative and technical decisions documented herein.

---

## 🎯 Problem Statement

Khmer (ខ្មែរ) is the official language of Cambodia, spoken by over 16 million people. Despite this, **high-quality OCR tooling for Khmer has historically been absent or severely limited**.

Existing general-purpose OCR systems such as Tesseract perform poorly on Khmer script due to:

- **Complex character stacking** — Khmer consonants stack vertically, creating compound glyphs that simple bounding-box methods cannot handle
- **Above and below vowel placement** — vowel diacritics appear both above and below the base consonant line
- **Font diversity** — hundreds of distinct Khmer typefaces exist, including Moul, MoulLight, Khmer OS, Busra, and many others
- **Lack of labeled training data** — no large-scale, publicly curated Khmer OCR dataset existed prior to this work

This project was initiated to fill that gap with a purpose-built, production-ready Khmer OCR system trained from first principles.

---

## 🧠 Original Contributions

### 1. Training Data Construction
- Assembled a synthetic text rendering pipeline using **800+ distinct Khmer fonts**
- Generated **3,000,000+ labeled text line images** for model training
- Applied realistic augmentation: noise, blur, rotation, brightness variation, compression artifacts
- This dataset is an **original creation** — no pre-existing Khmer OCR dataset of this scale was used

### 2. Detection Model (`det.onnx`)
- Designed and trained a **YOLO-style object detection model** adapted for Khmer document layout
- Supports two object classes: **text regions** and **embedded figures**
- Input: 1024×1024 RGB image
- Output: bounding boxes with class probabilities and confidence scores
- Post-processing includes custom **Non-Maximum Suppression (NMS)** and **reading-order line sorting** algorithm designed specifically for Khmer document structure

### 3. Recognition Model (`rec.onnx`)
- Designed and trained a **CTC-based sequence recognition model** for Khmer character sequences
- Vocabulary of **98 Khmer characters** including stacked forms, vowels, punctuation, and numerals
- Variable-width input (32px fixed height) enables recognition of text at any aspect ratio
- Simultaneously outputs:
  - `text_logits` — character sequence probabilities
  - `font_logits` — font style classification (Regular, Bold, Italic, BoldItalic, Moul, MoulLight)
- Font detection is an **original feature not found in comparable OCR systems**

### 4. Inference Pipeline
- End-to-end Python inference pipeline: image → detect → NMS → crop → recognize → export
- Custom **line sorting algorithm** that groups detected boxes into reading lines using adaptive Y-threshold based on box height
- Confidence scoring at both the character level (per-glyph softmax) and font level

### 5. Multi-Format Document Export
- `.txt` — plain text output with figure placeholders
- `.md` — Markdown with Moul font rendered as bold, figures as image links
- `.html` — styled HTML with CSS font classes, uncertainty highlighting (low-confidence text shown in red)
- `.docx` — Word document with correct Khmer font assignment (Khmer OS / Moul), embedded figures, and confidence-based color coding

### 6. Native C++ Engine
- Full reimplementation of the inference pipeline in **C++17** using the ONNX Runtime C++ API
- Exports a **C API** (`khmerocr_create`, `khmerocr_recognize`, `khmerocr_destroy`) enabling FFI integration from any language: Swift, Kotlin, Dart, Rust, Go
- Cross-platform CMake build system with platform-specific flags for **Windows, macOS, Linux, iOS, and Android**
- Includes single-header image loading (`stb_image.h`) for zero-dependency deployment

### 7. Apple CoreML Conversion
- Conversion script transforming both ONNX models to **Apple CoreML `.mlpackage`** format
- Dynamic width support via `RangeDim(32–1024)` for the recognition model
- Enables native inference on Apple Neural Engine for iOS 15+ and macOS devices

---

## 🗂️ Project Architecture

```
KhmerOCR/
├── khmerocr/
│   ├── __init__.py        ← Python inference engine (detect, recognize, NMS, CTC decode)
│   ├── cli.py             ← CLI tool (txt/md/html/docx export)
│   ├── det.onnx           ← Trained detection model (11MB)
│   └── rec.onnx           ← Trained recognition model (14MB)
├── cpp/
│   ├── src/               ← C++ implementation of full pipeline
│   ├── include/khmerocr/  ← C++ headers (types, detector, recognizer, image_utils)
│   ├── cli/main.cpp       ← Standalone C++ binary
│   └── CMakeLists.txt     ← Cross-platform build config
├── scripts/
│   └── convert_to_coreml.py  ← ONNX → CoreML conversion
└── pyproject.toml         ← Python package definition
```

---

## ⚙️ Technical Specifications

| Component | Specification |
|---|---|
| Detection input | 1 × 3 × 1024 × 1024 (RGB, normalized) |
| Detection architecture | YOLO-style |
| Detection classes | text (1), figure (0) |
| Detection confidence threshold | 0.25 |
| NMS IoU threshold | 0.45 |
| Recognition input | 1 × 1 × 32 × W (grayscale, variable width) |
| Recognition architecture | CTC sequence model |
| Recognition vocabulary | 98 Khmer characters + blank token |
| Font classification | 6 styles (Regular, Italic, Bold, BoldItalic, Moul, MoulLight) |
| Model format | ONNX (portable, framework-independent) |
| Runtime | ONNX Runtime 1.14+ |
| Python version | 3.9+ |
| C++ standard | C++17 |
| Supported platforms | Windows, macOS, Linux, iOS, Android |

---

## 🔬 Methodology

### Data Pipeline
1. Collected and curated 800+ Khmer font files covering all major typeface categories
2. Built a text rendering engine that generates synthetic images from real Khmer text corpora
3. Applied augmentation (blur, noise, skew, brightness) to simulate real scan/photo conditions
4. Generated 3M labeled line images with character-level ground truth annotations

### Model Training
1. Detection model trained using YOLO-style architecture with custom anchor boxes tuned for Khmer text aspect ratios
2. Recognition model trained end-to-end with CTC loss — no manual segmentation required
3. Multi-task training head added for simultaneous font classification
4. Models exported to ONNX format for universal deployment

### Post-Processing
1. Confidence filtering (threshold 0.25) removes low-quality detections
2. NMS with IoU 0.45 eliminates duplicate boxes
3. Line grouping uses adaptive threshold: boxes within 50% of a reference box height are placed on the same line
4. CTC decoding: argmax per time step → remove consecutive duplicates → remove blank tokens → final text

---

## 📦 Installation & Usage

```bash
# Install
pip install git+https://github.com/seanghay/KhmerOCR

# CLI usage
khmerocr document.jpg                        # → .txt
khmerocr document.jpg --format docx          # → .docx
khmerocr report.pdf --format html            # → .html

# Python API
from PIL import Image
from khmerocr import detect, recognize

img = Image.open("document.jpg")
lines = detect(img)
for line in lines:
    for box in line:
        if int(box[4]) == 1:  # text class
            result = recognize(img.crop(box[:4]))
            print(result["text"], "|", result["font"])
```

---

## 📄 License

This project is released under the **MIT License**.

```
MIT License

Copyright (c) 2024 Seanghay Yath

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.
```

---

## 🛡️ Intellectual Property Notice

All of the following are **original works** created by the author:

- The synthetic data generation pipeline and resulting training dataset
- The neural network architecture design choices (detection + recognition + font classification)
- The training procedure, loss functions, and hyperparameter decisions
- The NMS implementation and line-sorting algorithm
- The Python inference package (`khmerocr/`)
- The C++ native engine and C API design (`cpp/`)
- The multi-format document export system (txt/md/html/docx)
- The CoreML conversion pipeline

Any use of this work in publications, products, or derivative research should **cite this repository and the original author**.

If you are building on this work in an academic context, please reference:

```
Yath, Seanghay. KhmerOCR: A High-Performance OCR Engine for Khmer Script.
GitHub Repository. https://github.com/seanghay/KhmerOCR. 2024.
```


*This README was authored as part of the original project documentation to establish clear provenance and authorship of all technical contributions.*
