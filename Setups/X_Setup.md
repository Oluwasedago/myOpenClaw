# 🦞 OpenClaw Personal AI — X/Twitter Archive to AI Memory

> Turn your X (Twitter) archive into a personal AI brain.
> Not a chatbot. A contextual memory system that thinks like you.

[![YouTube](https://img.shields.io/badge/YouTube-Watch%20Tutorial-red?style=for-the-badge&logo=youtube)](YOUR_YOUTUBE_LINK_HERE)
[![OpenClaw](https://img.shields.io/badge/OpenClaw-Official%20Docs-orange?style=for-the-badge)](https://docs.openclaw.ai)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg?style=for-the-badge)](LICENSE)
[![Stars](https://img.shields.io/github/stars/Oluwasedago/openclaw-x-archive?style=for-the-badge)](https://github.com/Oluwasedago/openclaw-x-archive)

---

## 📺 Video Tutorial

Full step-by-step walkthrough on YouTube:

👉 **[Watch the Video](YOUR_YOUTUBE_LINK_HERE)**

---

## 🧠 What This Does

This repository contains a **single Bash script** that:

1. Copies your X/Twitter archive from Windows → WSL
2. Extracts and merges multiple ZIP files
3. Parses JavaScript-wrapped JSON data
4. Extracts structured data:
   - ✅ Your **Profile** (identity)
   - ✅ Your **Tweets** (voice, opinions, thinking)
   - ✅ Your **Bookmarks** (research, interests)
   - ✅ Your **Likes** (engagement patterns)
   - ✅ Your **Following** (intellectual network)
5. Combines everything into a single clean text file
6. Splits output into Telegram-friendly chunks (< 4000 chars)
7. Ready to paste into OpenClaw for AI memory ingestion

---

## ⚠️ Important Distinction

> We are **not training** an AI model.
>
> We are building **contextual memory**.
>
> Your tweets = your thinking.
> Your bookmarks = your research brain.
> Your likes = your subconscious interests.
> Your following = your intellectual network.

---

## 🏗️ Architecture

    ┌─────────────────────┐
    │  X/Twitter Archive   │
    │  (Twitter_1.zip)     │
    │  (Twitter_2.zip)     │
    └─────────┬───────────┘
              │
              ▼
    ┌─────────────────────┐
    │  WSL Ubuntu          │
    │  process_x.sh        │
    │  (Bash + Python3)    │
    └─────────┬───────────┘
              │
              ▼
    ┌─────────────────────┐
    │  Structured Output   │
    │  x-profile.txt       │
    │  x-chunks/           │
    └─────────┬───────────┘
              │
              ▼
    ┌─────────────────────┐
    │  OpenClaw Gateway    │
    │  (Telegram/WhatsApp) │
    │  Personal AI Memory  │
    └─────────────────────┘

---

## 📋 Prerequisites

| Requirement | Details |
|-------------|---------|
| **OS** | Windows 10/11 with WSL2 |
| **WSL Distro** | Ubuntu (recommended) |
| **Python** | Python 3.x (pre-installed in Ubuntu) |
| **Node.js** | v22+ (for OpenClaw) |
| **OpenClaw** | Installed and running [Getting Started](https://docs.openclaw.ai) |
| **X Archive** | Downloaded from [x.com/settings/download_your_data](https://x.com/settings/download_your_data) |

---

## 🚀 Quick Start

### 1. Clone this repo

    git clone https://github.com/Oluwasedago/openclaw-x-archive.git
    cd openclaw-x-archive

### 2. Download your X archive

1. Go to [x.com/settings/download_your_data](https://x.com/settings/download_your_data)
2. Click **Request archive**
3. Wait for email (24–48 hours)
4. Download all ZIP files to your Windows Downloads folder

### 3. Update the script config

Open `process_x.sh` and update line 12:

    WIN_DOWNLOADS="/mnt/c/Users/YOUR_USERNAME/Downloads"

Replace `YOUR_USERNAME` with your actual Windows username.

### 4. Make it executable

    chmod +x process_x.sh

### 5. Run it

    bash process_x.sh

### 6. Feed into OpenClaw

Follow the instructions printed by the script.

---

## 📁 Repository Structure

    openclaw-x-archive/
    │
    ├── process_x.sh            # Main processing script
    ├── README.md               # This file
    ├── LICENSE                  # MIT License
    ├── .gitignore              # Protects personal data
    │
    ├── examples/
    │   ├── sample-output.txt   # Example of processed output
    │   └── sample-chunk.txt    # Example of a single chunk
    │
    └── docs/
        ├── SETUP_WSL.md        # How to set up WSL2 on Windows
        ├── SETUP_OPENCLAW.md   # How to install OpenClaw
        └── TROUBLESHOOTING.md  # Common issues and fixes

---

## 📄 The Script — process_x.sh

    #!/bin/bash
    # ============================================================
    # X/Twitter Archive Processor for OpenClaw
    # Author: David Popoola (@Oluwasedago)
    # License: MIT
    #
    # Converts your X/Twitter data archive into structured
    # AI-ready memory chunks for OpenClaw ingestion.
    #
    # Usage: bash process_x.sh
    # ============================================================
    set -euo pipefail

    # --- CONFIG (UPDATE THESE) ---
    WIN_DOWNLOADS="/mnt/c/Users/a/Downloads"
    WORK_DIR="$HOME/x-archive"
    OUTPUT="$HOME/x-profile-for-openclaw.txt"
    CHUNKS_DIR="$HOME/x-chunks"

    echo "🐦 X Archive Processor for OpenClaw"
    echo "===================================="

    # --- STEP 1: Copy ZIP files from Windows to WSL ---
    echo "[1/7] Copying ZIP files from Windows Downloads..."
    mkdir -p "$WORK_DIR"
    cp "$WIN_DOWNLOADS"/Twitter_1.zip "$WORK_DIR/" 2>/dev/null && echo "  ✅ Twitter_1.zip copied" || echo "  ⚠️ Twitter_1.zip not found"
    cp "$WIN_DOWNLOADS"/Twitter_2.zip "$WORK_DIR/" 2>/dev/null && echo "  ✅ Twitter_2.zip copied" || echo "  ⚠️ Twitter_2.zip not found"

    # --- STEP 2: Extract both ZIPs ---
    echo "[2/7] Extracting archives (this may take a while for 4+ GB)..."
    cd "$WORK_DIR"
    for zip in Twitter_1.zip Twitter_2.zip; do
      if [ -f "$zip" ]; then
        unzip -o -q "$zip" -d extracted 2>/dev/null && echo "  ✅ $zip extracted" || echo "  ⚠️ $zip extraction failed"
      fi
    done

    # --- STEP 3: Find the data directory ---
    echo "[3/7] Locating data files..."
    DATA_DIR=$(find "$WORK_DIR/extracted" -type d -name "data" | head -1)
    if [ -z "$DATA_DIR" ]; then
      echo "  ❌ No 'data' folder found."
      ls -R "$WORK_DIR/extracted" | head -50
      exit 1
    fi
    echo "  ✅ Data folder: $DATA_DIR"
    echo "  📁 Available files:"
    ls "$DATA_DIR"/*.js 2>/dev/null | while read f; do
      echo "    - $(basename "$f") ($(du -sh "$f" | cut -f1))"
    done

    # --- STEP 4: Process everything ---
    echo "[4/7] Processing your X data..."
    cat /dev/null > "$OUTPUT"

    # --- PROFILE ---
    echo "  → Extracting profile..."
    cat >> "$OUTPUT" << 'HEADER'
    ===========================================
    MY X/TWITTER PROFILE
    ===========================================
    HEADER

    for f in "$DATA_DIR"/profile*.js; do
      [ -f "$f" ] || continue
      sed 's/^window\.YTD\.[a-zA-Z_]*\.part[0-9]* = //' "$f" | python3 -c "
    import sys, json
    try:
        data = json.load(sys.stdin)
        print(json.dumps(data, indent=2))
    except: pass
    " >> "$OUTPUT" 2>/dev/null
    done
    echo "" >> "$OUTPUT"

    # --- ACCOUNT ---
    echo "  → Extracting account info..."
    cat >> "$OUTPUT" << 'HEADER'
    ===========================================
    ACCOUNT INFO
    ===========================================
    HEADER

    for f in "$DATA_DIR"/account*.js; do
      [ -f "$f" ] || continue
      sed 's/^window\.YTD\.[a-zA-Z_]*\.part[0-9]* = //' "$f" | python3 -c "
    import sys, json
    try:
        data = json.load(sys.stdin)
        print(json.dumps(data, indent=2))
    except: pass
    " >> "$OUTPUT" 2>/dev/null
    done
    echo "" >> "$OUTPUT"

    # --- BOOKMARKS ---
    echo "  → Extracting bookmarks..."
    cat >> "$OUTPUT" << 'HEADER'
    ===========================================
    X BOOKMARKS (What I save/research)
    ===========================================
    HEADER

    for f in "$DATA_DIR"/bookmark*.js; do
      [ -f "$f" ] || continue
      sed 's/^window\.YTD\.[a-zA-Z_]*\.part[0-9]* = //' "$f" | python3 -c "
    import sys, json
    try:
        data = json.load(sys.stdin)
        for item in data:
            b = item.get('bookmark', item)
            text = b.get('fullText', b.get('full_text', b.get('text', '')))
            url = b.get('expandedUrl', b.get('url', ''))
            if text:
                print(text[:300])
                if url: print(f'URL: {url}')
                print('---')
    except Exception as e:
        print(f'Note: {e}', file=sys.stderr)
    " >> "$OUTPUT" 2>/dev/null
    done
    echo "" >> "$OUTPUT"

    # --- TWEETS ---
    echo "  → Extracting tweets (last 500)..."
    cat >> "$OUTPUT" << 'HEADER'
    ===========================================
    MY TWEETS (Voice, opinions, interests)
    ===========================================
    HEADER

    for f in "$DATA_DIR"/tweets*.js "$DATA_DIR"/tweet*.js; do
      [ -f "$f" ] || continue
      sed 's/^window\.YTD\.[a-zA-Z_]*\.part[0-9]* = //' "$f" | python3 -c "
    import sys, json, re
    try:
        data = json.load(sys.stdin)
        tweets = sorted(data, key=lambda x: x.get('tweet',x).get('created_at',''), reverse=True)[:500]
        for t in tweets:
            tw = t.get('tweet', t)
            date = tw.get('created_at', 'unknown')
            text = tw.get('full_text', tw.get('text', ''))
            text = re.sub(r'https?://t\.co/\S+', '[LINK]', text)
            retweets = tw.get('retweet_count', 0)
            likes = tw.get('favorite_count', 0)
            print(f'[{date}] (RT:{retweets} ♥:{likes})')
            print(text)
            print('---')
    except Exception as e:
        print(f'Note: {e}', file=sys.stderr)
    " >> "$OUTPUT" 2>/dev/null
    done
    echo "" >> "$OUTPUT"

    # --- LIKES ---
    echo "  → Extracting likes (first 200)..."
    cat >> "$OUTPUT" << 'HEADER'
    ===========================================
    MY LIKES (What I engage with)
    ===========================================
    HEADER

    for f in "$DATA_DIR"/like*.js; do
      [ -f "$f" ] || continue
      sed 's/^window\.YTD\.[a-zA-Z_]*\.part[0-9]* = //' "$f" | python3 -c "
    import sys, json
    try:
        data = json.load(sys.stdin)
        for l in data[:200]:
            like = l.get('like', l)
            text = like.get('fullText', like.get('full_text', like.get('text', '')))
            if text:
                print(text[:200])
                print('---')
    except Exception as e:
        print(f'Note: {e}', file=sys.stderr)
    " >> "$OUTPUT" 2>/dev/null
    done
    echo "" >> "$OUTPUT"

    # --- FOLLOWING ---
    echo "  → Extracting following list..."
    cat >> "$OUTPUT" << 'HEADER'
    ===========================================
    ACCOUNTS I FOLLOW (Interest network)
    ===========================================
    HEADER

    for f in "$DATA_DIR"/following*.js; do
      [ -f "$f" ] || continue
      sed 's/^window\.YTD\.[a-zA-Z_]*\.part[0-9]* = //' "$f" | python3 -c "
    import sys, json
    try:
        data = json.load(sys.stdin)
        for item in data:
            f = item.get('following', item)
            name = f.get('accountId', f.get('userLink', ''))
            print(name)
    except: pass
    " >> "$OUTPUT" 2>/dev/null
    done
    echo "" >> "$OUTPUT"

    # --- STEP 5: Summary ---
    echo "[5/7] Processing complete!"
    echo "  📄 Output: $OUTPUT"
    echo "  📊 Size: $(du -sh "$OUTPUT" | cut -f1)"
    echo "  📝 Lines: $(wc -l < "$OUTPUT")"

    # --- STEP 6: Split into chunks ---
    echo "[6/7] Splitting into Telegram-friendly chunks..."
    mkdir -p "$CHUNKS_DIR"
    rm -f "$CHUNKS_DIR"/chunk_*

    python3 -c "
    import os
    chunk_size = 3500
    output_dir = '$CHUNKS_DIR'
    with open('$OUTPUT', 'r') as f:
        content = f.read()
    chunks = []
    while content:
        if len(content) <= chunk_size:
            chunks.append(content)
            break
        cut = content.rfind('\n', 0, chunk_size)
        if cut == -1: cut = chunk_size
        chunks.append(content[:cut])
        content = content[cut:].lstrip('\n')
    for i, chunk in enumerate(chunks):
        path = os.path.join(output_dir, f'chunk_{i+1:03d}.txt')
        with open(path, 'w') as f:
            f.write(chunk)
    print(f'Created {len(chunks)} chunks in {output_dir}')
    "

    # --- STEP 7: Instructions ---
    echo ""
    echo "[7/7] ✅ DONE!"
    echo "==========================================="
    echo ""
    echo "Total chunks: $(ls "$CHUNKS_DIR"/chunk_* | wc -l)"
    echo ""
    echo "HOW TO FEED INTO OPENCLAW:"
    echo ""
    echo "1. Open Telegram (your OpenClaw bot)"
    echo ""
    echo "2. Send this first message:"
    echo '   "I am sharing my X/Twitter data.'
    echo '    Analyze and learn my interests, thinking patterns,'
    echo '    writing style, and personality.'
    echo '    I will send multiple chunks. Acknowledge each."'
    echo ""
    echo "3. Then for each chunk, run:"
    echo "   cat $CHUNKS_DIR/chunk_001.txt"
    echo "   Copy output → paste into Telegram"
    echo ""
    echo "4. Repeat for all chunks."
    echo ""
    echo "5. Final message:"
    echo '   "That was all my X data.'
    echo '    Summarize what you learned about me."'
    echo "==========================================="

---

## 🔧 Configuration

Edit these variables at the top of `process_x.sh`:

| Variable | Default | Description |
|----------|---------|-------------|
| `WIN_DOWNLOADS` | `/mnt/c/Users/a/Downloads` | Windows Downloads folder path |
| `WORK_DIR` | `~/x-archive` | Where ZIPs are extracted |
| `OUTPUT` | `~/x-profile-for-openclaw.txt` | Combined output file |
| `CHUNKS_DIR` | `~/x-chunks` | Telegram chunk output folder |

---

## 💡 What Each Data Source Reveals

| Source | What It Tells Your AI | Priority |
|--------|----------------------|----------|
| **Tweets** | Your voice, opinions, thinking patterns | ⭐⭐⭐ |
| **Profile** | Identity, bio, links | ⭐⭐⭐ |
| **Bookmarks** | What you research and save | ⭐⭐⭐ |
| **Likes** | Subconscious interests, engagement patterns | ⭐⭐ |
| **Following** | Your intellectual network | ⭐⭐ |
| **DMs** | Private conversations | ❌ Skipped for privacy |

---

## 🔒 Security & Privacy

- ✅ Everything runs **locally** on your machine
- ✅ No data sent to cloud services
- ✅ OpenClaw is **self-hosted**
- ✅ DMs are **never processed**
- ✅ Script only reads — never modifies your archive
- ⚠️ **Never** commit your archive or tokens to GitHub
- ⚠️ Add your archive folders to `.gitignore`

---

## 🐛 Troubleshooting

| Problem | Solution |
|---------|----------|
| `Twitter_1.zip not found` | Check your Windows username in `WIN_DOWNLOADS` path |
| `No 'data' folder found` | Archive might have different structure — check extracted folder |
| `python3: command not found` | Run `sudo apt install python3` |
| `Permission denied` | Run `chmod +x process_x.sh` |
| `unzip: command not found` | Run `sudo apt install unzip` |
| Chunks too large for Telegram | Reduce `chunk_size` in script (default 3500) |
| Empty output file | Check if archive contains `.js` files in the `data/` folder |
| OpenClaw not responding | Run `openclaw doctor` and `openclaw status` |

See [docs/TROUBLESHOOTING.md](docs/TROUBLESHOOTING.md) for more.

---

## 🗺️ Roadmap

- [x] X/Twitter archive processing
- [ ] GitHub commit history ingestion
- [ ] YouTube watch history processing
- [ ] Browser bookmarks (Brave/Chrome) processing
- [ ] Email export processing (Gmail)
- [ ] Automated chunk sender (Telegram API)
- [ ] Web UI for processing

---

## 🤝 Contributing

1. Fork this repo
2. Create your feature branch

        git checkout -b feature/new-source

3. Commit your changes

        git commit -m "Add new data source"

4. Push to branch

        git push origin feature/new-source

5. Open a Pull Request

---

## 📺 More Content

- 🎥 [YouTube Channel](YOUR_YOUTUBE_CHANNEL_LINK)
- 🐦 [X/Twitter](https://x.com/YOUR_HANDLE)
- 💻 [GitHub](https://github.com/Oluwasedago)

---

## 📄 License

MIT License — see [LICENSE](LICENSE) for details.

---

## ⭐ Star This Repo

If this helped you build your personal AI memory system, **star this repo** and share the video!

---

> *"Stop thinking about AI as a chatbot. Think of it as a memory layer."*
> — David Popoola
