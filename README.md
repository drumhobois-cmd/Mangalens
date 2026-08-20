# MangaLens

**Local-first AI manga translation for the browser.**

MangaLens is an experimental browser extension and local translation backend designed to translate Japanese manga directly while you read.

The goal is to combine the convenience and speed of automatic manga translators with higher-quality, context-aware AI translation, better typesetting, local processing, and the option to run the translation model entirely on your own computer.

> **Current status:** Early development / Alpha  
> **Current version:** v0.4

---

## Why MangaLens?

Most automatic manga translators make a trade-off between:

- translation quality;
- speed;
- clean presentation;
- privacy;
- cost.

MangaLens is being built to reduce those compromises.

Instead of treating every speech bubble as an isolated sentence, MangaLens is designed around a translation pipeline that can eventually understand:

- surrounding dialogue;
- character relationships;
- recurring terminology;
- visual context;
- speech style;
- narration;
- sound effects;
- cultural terminology;
- continuity across an entire chapter.

The long-term goal is to produce translations that approach good human scanlation quality while remaining fast enough to use as a live manga reader.

---

# Current Features

## Browser Translation

MangaLens currently supports Chromium-based browsers such as:

- Opera
- Google Chrome
- Microsoft Edge
- other Chromium-compatible browsers

The extension detects manga images on a webpage and attaches translation layers directly to the manga rather than treating the browser viewport as one disposable screenshot.

Translations therefore remain associated with the page while scrolling.

---

## Japanese OCR

Japanese text detection is performed locally using **PaddleOCR**.

The OCR system provides:

- detected Japanese text;
- text coordinates;
- orientation;
- region information;
- data used for translation placement.

Future versions will also expand the use of manga-specialised OCR models.

---

## Context-Aware AI Translation

MangaLens supports AI-based translation rather than relying on traditional machine translation alone.

The current translation pipeline can use:

```text
Japanese OCR
      │
      ▼
Translation model
      │
      ▼
JNC translation
      │
      ▼
Optional quality-control/editor pass
      │
      ▼
Final English dialogue
```

MangaLens can therefore distinguish between simply translating words and producing dialogue that reads naturally in English.

---

# JNC Translation

The default MangaLens translation philosophy is currently called:

## Japanese Natural Cultural — JNC

JNC aims to:

- preserve the meaning of the Japanese;
- produce natural English dialogue;
- retain culturally significant Japanese terminology where useful;
- preserve character titles, ranks and terminology;
- avoid unnecessary localisation;
- avoid making dialogue more or less explicit than the source;
- preserve character personality and intent.

The aim is to sit between overly literal machine translation and aggressive localisation.

---

# Translation Providers

MangaLens v0.4 supports multiple translation backends.

## OpenAI

OpenAI can currently be used as the high-quality cloud translation provider.

This is useful for:

- benchmarking;
- difficult dialogue;
- visual context;
- translation quality comparisons.

An OpenAI API key is required when using this provider.

API usage may incur charges.

---

## Ollama — Local AI

MangaLens can also use **Ollama** to host the translator entirely on your own computer.

Example:

```text
MangaLens
    │
    ▼
Local Python backend
    │
    ├── PaddleOCR
    │
    ▼
Ollama
    │
    ▼
Qwen
    │
    ▼
Your GPU
```

No translation API fee is required when running locally.

The current recommended local test model is:

```text
qwen3:30b
```

A smaller model can also be used on systems with less available VRAM.

---

# Local / Private Translation

One of the main design goals of MangaLens is allowing the entire translation pipeline to eventually run locally.

A fully local configuration can keep:

- manga images;
- OCR;
- Japanese text;
- translations;
- cleanup;
- typesetting;

on the user's own computer.

This is useful for privacy and removes recurring translation API costs.

---

# Image-Anchored Translation

Earlier MangaLens prototypes translated only the currently visible browser viewport.

v0.4 moves toward an image-oriented reader architecture.

```text
Manga Image 1
├── OCR
├── translation
├── cached result
└── translation layer

Manga Image 2
├── queued
└── pre-translated

Manga Image 3
└── waiting
```

Translation overlays are attached to the manga element and move naturally with the page while scrolling.

---

# Translation Queue & Prefetching

MangaLens can prioritise visible manga pages while preparing nearby pages in advance.

The intended experience is:

```text
Current page
     ↓
Translate immediately

Next page
     ↓
Prefetch

Following page
     ↓
Queue
```

The goal is for translated pages to already be available by the time the reader reaches them.

---

# Translation Progress

Translation should never look like the application has frozen.

MangaLens includes in-page translation status feedback such as:

```text
Reading Japanese…
        ↓
Translating…
        ↓
Typesetting…
        ↓
Complete
```

Long-running translation work can therefore provide immediate visible feedback.

---

# Safe Clean Rendering

MangaLens originally experimented with general-purpose OpenCV inpainting.

This worked poorly on manga artwork because traditional inpainting can damage:

- screentones;
- hair;
- clothing;
- speed lines;
- panel borders;
- hatching;
- detailed backgrounds.

v0.4 instead moves toward **Safe Clean** rendering.

For areas such as white speech bubbles, MangaLens attempts to remove only the original Japanese text while protecting the surrounding artwork.

```text
Japanese speech bubble
        ↓
Detect safe interior
        ↓
Remove Japanese glyph area
        ↓
Preserve bubble/art
        ↓
Typeset English
```

Detailed artwork is deliberately treated more conservatively.

---

# Typesetting

MangaLens automatically creates English text overlays using the detected manga region.

The renderer is being developed to account for:

- available bubble area;
- horizontal English text;
- font size;
- line wrapping;
- text alignment;
- outlined lettering over artwork;
- narration;
- dialogue;
- sound effects.

An important design principle is that the original Japanese OCR box and the final English typesetting area do **not** have to be the same size.

Japanese is frequently written vertically while English requires significantly more horizontal space.

---

# Optional Translation QC

MangaLens includes an optional second translation stage.

Rather than keeping the user waiting for every quality-control operation, the intended system is:

```text
Initial translation
        ↓
DISPLAY IMMEDIATELY
        ↓
Background QC
        ↓
Update improved lines
```

The quality-control stage can examine:

- semantic accuracy;
- lost nuance;
- unnatural English;
- terminology;
- character voice;
- over-translation;
- under-translation;
- cultural meaning.

---

# Planned Chapter Memory

A major planned feature is persistent translation memory.

For example:

```text
魔導騎士 → Arcane Knight
王立魔導院 → Royal Arcane Academy
瘴気 → Miasma
```

MangaLens will eventually maintain information about:

### Characters

```text
Reina
- female
- informal speech
- sarcastic
- calls Takumi "Taku"
```

### Terminology

```text
黒騎士団 → Black Knights
```

### Places

```text
灰の都 → Ashen City
```

This should prevent terminology and character voice from changing randomly between pages.

---

# Planned Translation Pipeline

The long-term translation architecture is:

```text
                   Manga Page
                       │
                       ▼
                Text Detection
                       │
                       ▼
                 Japanese OCR
                       │
              ┌────────┴────────┐
              │                 │
        Chapter Context     Visual Context
              │                 │
              └────────┬────────┘
                       ▼
               JNC Translator
                       │
                       ▼
              Accuracy Critic
                       │
                       ▼
             Dialogue Editor
                       │
                       ▼
            Continuity Checker
                       │
                       ▼
               Final English
```

Simple dialogue should bypass unnecessary stages so that translation remains fast.

---

# Project Structure

A typical MangaLens installation contains:

```text
MangaLens/
│
├── extension/
│   ├── manifest.json
│   ├── background.js
│   ├── content.js
│   ├── popup.html
│   └── ...
│
├── backend/
│   ├── app/
│   ├── requirements.txt
│   ├── setup_windows.bat
│   ├── run_backend.bat
│   └── .env
│
├── test_page.html
├── README.md
└── CHANGELOG.md
```

---

# Installation

> Installation instructions may change significantly while MangaLens remains in alpha.

## 1. Clone the repository

```bash
git clone <YOUR-REPOSITORY-URL>
cd MangaLens
```

---

## 2. Configure the backend

On Windows:

```text
backend\setup_windows.bat
```

This creates the Python environment and installs the required packages.

---

## 3. Configure `.env`

Create or edit:

```text
backend\.env
```

### OpenAI example

```ini
OPENAI_API_KEY=your_api_key_here
```

### Ollama example

```ini
OLLAMA_BASE_URL=http://127.0.0.1:11434
OLLAMA_MODEL=qwen3:30b
OLLAMA_EDITOR_MODEL=qwen3:30b
OLLAMA_KEEP_ALIVE=30m
OLLAMA_TIMEOUT_SECONDS=180
OLLAMA_VISION=false
```

---

# Ollama Setup

Install Ollama and then download a compatible model.

Example:

```bash
ollama pull qwen3:30b
```

Test it with:

```bash
ollama run qwen3:30b
```

Check GPU utilisation with:

```bash
ollama ps
```

For local MangaLens use, the model should ideally remain predominantly or entirely in GPU memory.

---

# Start MangaLens

Run:

```text
backend\run_backend.bat
```

Keep the backend running while MangaLens is active.

---

# Install the Browser Extension

In Opera:

```text
opera://extensions
```

In Chrome:

```text
chrome://extensions
```

Then:

1. Enable **Developer Mode**.
2. Select **Load unpacked**.
3. Choose the `extension` directory.
4. Open a manga reader.
5. Open MangaLens.
6. Enable translation.

---

# Development Goals

MangaLens is currently focused on four main areas:

### Translation Quality

Approach strong human fan translations while maintaining accurate Japanese meaning.

### Speed

Make translation sufficiently fast that pages can be prepared before the reader reaches them.

### Presentation

Produce clean typesetting without damaging the underlying artwork.

### Local Processing

Allow OCR, translation and rendering to run without requiring paid cloud APIs.

---

# Roadmap

## v0.4

- [x] Image-anchored translation
- [x] Translation queue
- [x] Nearby page prefetching
- [x] In-page loading feedback
- [x] Safe Clean renderer
- [x] Local Ollama provider
- [x] OpenAI provider
- [x] Optional background QC
- [x] Local translation model support

## Next

- [ ] Improve speech-bubble segmentation
- [ ] Better Japanese glyph removal
- [ ] Manga-specific comic fonts
- [ ] Improved automatic English layout
- [ ] Chapter translation memory
- [ ] Character profiles
- [ ] Persistent terminology glossary
- [ ] Automatic reading-order detection
- [ ] Improved SFX detection
- [ ] Better handwritten-text OCR
- [ ] Local vision-language model support
- [ ] Hybrid local/cloud translation
- [ ] Translation confidence inspection
- [ ] Translation caching across sessions
- [ ] Automatic chapter processing
- [ ] Fully offline Private Mode

---

# Translation Benchmarking

MangaLens is being tested against particularly difficult Japanese manga rather than only simple dialogue.

Testing currently compares:

```text
Original Japanese
       │
       ├── MangaLens JNC
       ├── MangaLens JNC + QC
       ├── Local AI
       └── Published / professional translation
```

The project evaluates:

- semantic accuracy;
- natural English;
- terminology;
- character voice;
- cultural nuance;
- consistency;
- translation speed;
- typesetting quality.

---

# Privacy

MangaLens is intended to become a **local-first application**.

When using a local Ollama model:

```text
Manga
  ↓
Local OCR
  ↓
Local AI
  ↓
Local rendering
```

Translation data does not need to be sent to a commercial translation API.

When using a cloud provider, content required for that translation may be transmitted to that provider according to its own API terms and privacy policies.

---

# Disclaimer

MangaLens is an independent experimental project.

It is **not affiliated with or endorsed by IsManga, Kodansha, manga publishers, scanlation groups, browser vendors, OpenAI, Ollama, Qwen or other referenced organisations**.

MangaLens does not provide, distribute or host manga.

Users are responsible for ensuring they have the appropriate rights or permission to access, process and translate any content used with MangaLens.

Do not redistribute copyrighted manga or translated works unless you have the legal right to do so.

---

# Contributing

MangaLens is currently in active early development.

Bug reports, translation comparisons, OCR test cases and development contributions are welcome.

Useful reports should ideally include:

- source language;
- original manga screenshot;
- MangaLens translation;
- expected or reference translation;
- OCR result;
- selected translation provider;
- selected MangaLens settings;
- translation timing;
- browser;
- GPU/CPU configuration.

Please avoid uploading copyrighted chapters or other large copyrighted works to the repository when reporting issues. Small excerpts should only be included where legally appropriate.

---

# License

A project license has not yet been finalised.

Until a licence file is added, **all rights are reserved by default**.

Do not assume the source code may be redistributed, incorporated into other projects or commercially reused until the repository includes an explicit licence.

---

# Project Status

MangaLens is currently an experimental personal-development project and APIs, configuration files, translation formats and extension behaviour may change between releases.

Expect bugs.

Expect terrible OCR occasionally.

Expect strange typesetting.

And hopefully expect each version to get considerably better.

**MangaLens — read the manga, not the machine translation.**
