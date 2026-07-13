# 🎙️ Voice Note Transcriber

A Streamlit app that turns voice notes into downloadable text transcripts.
Built for **English, Urdu, and Pashto** — including **code-switched** recordings
where the languages are mixed inside a single voice note.

- **Upload** one or many voice notes (including WhatsApp **.opus** / **.m4a**),
  **or record straight from your microphone** in the browser.
- Choose an engine: **OpenAI**, **ElevenLabs Scribe**, or **Groq Whisper**.
- Get an editable transcript per file — download each as **.txt**, or all as a **.zip**.
- **Romanize** toggle: view Urdu/Pashto in Roman (Latin) script or their native
  script, switchable instantly per transcript.
- **Multi-key failover:** give each engine several API keys and the app rotates
  to the next automatically if one hits an auth / quota / rate-limit error.

## Supported formats

`mp3, wav, m4a, ogg, opus, flac, webm, mp4, aac, amr`

`.opus` (WhatsApp) and `.amr` are auto-transcoded to mp3 with `pydub` + `ffmpeg`.
If `ffmpeg` is missing, the app falls back to sending the original file.

## Engines

| Engine | Models | Secret key name | Notes |
|---|---|---|---|
| **OpenAI** | gpt-4o-transcribe, gpt-4o-mini-transcribe, whisper-1 | `OPENAI_API_KEYS` | needs OpenAI credits |
| **ElevenLabs Scribe** | scribe_v2 | `ELEVENLABS_API_KEYS` | often native-script output |
| **Groq Whisper** | whisper-large-v3-turbo, whisper-large-v3 | `GROQ_API_KEYS` | free tier; keys start `gsk_` |

> **Groq**, not "Grok": the Whisper host is **Groq** (console.groq.com, keys
> `gsk_`). xAI's *Grok* is a chat model with no transcription API.

## Language & romanization

Keep **Auto-detect** (default) for mixed-language notes — forcing a single
language can push the whole transcript into the wrong script.

The **Romanize output** toggle transliterates Urdu/Pashto into Latin script
(English stays English). Two methods:

- **Offline (free)** — instant, no API. Great on Devanagari; weaker on Arabic
  script (which omits short vowels).
- **OpenAI (higher quality)** — more natural spelling, needs OpenAI credits;
  auto-falls back to offline.

Toggle it any time — each transcript switches between Roman and native **instantly,
without re-transcribing**.

---

## Run locally

Requires **Python 3.9+** and **ffmpeg**.

```bash
python3 -m venv venv
source venv/bin/activate          # Windows: venv\Scripts\activate
pip install -r requirements.txt

# ffmpeg (needed for .opus / .amr):
#   macOS:          brew install ffmpeg
#   Debian/Ubuntu:  sudo apt install ffmpeg
#   conda:          conda install -c conda-forge ffmpeg

# add your keys to .streamlit/secrets.toml (see below), then:
streamlit run app.py
```

Open the local URL Streamlit prints (usually http://localhost:8501).

> 🎤 Mic recording needs a secure context — it works on `localhost` and on HTTPS
> (Streamlit Cloud), but **not** over a plain-http LAN address. File upload works
> everywhere.

### API keys & failover

Each engine accepts **multiple keys**, gathered in this order and tried one by
one (skipping keys that fail with 401/403/429/5xx until one works):

1. `OPENAI_API_KEYS` / `ELEVENLABS_API_KEYS` / `GROQ_API_KEYS` — TOML **lists**.
2. `OPENAI_API_KEY` / `ELEVENLABS_API_KEY` / `GROQ_API_KEY` — single keys (still supported).
3. Extra keys typed into the **sidebar** box (one per line), used as a fallback.

A non-key error (e.g. 400 for bad audio) fails fast without burning the other
keys, and each result shows which **key + model** produced it.

Example `.streamlit/secrets.toml`:

```toml
OPENAI_API_KEYS     = ["sk-proj-...", "sk-proj-..."]
ELEVENLABS_API_KEYS = ["sk_...", "sk_..."]
GROQ_API_KEYS       = ["gsk_...", "gsk_..."]
```

`.streamlit/secrets.toml` is **gitignored** so your keys are never committed.

---

## Deploy to Streamlit Community Cloud

1. Push to a **GitHub repo** (`secrets.toml` stays out via `.gitignore`).
2. **https://share.streamlit.io** → sign in with GitHub → **Create app** → pick the
   repo, branch `main`, main file `app.py`.
3. **Advanced settings → Secrets:** paste your key arrays (same TOML as above).
4. `packages.txt` (ffmpeg) + `requirements.txt` build automatically. **Deploy**.

The theme in `.streamlit/config.toml` is applied automatically (viewers can still
switch light/dark from the app's ☰ menu → Settings → Theme).

---

## Project layout

```
voice-transcriber/
├── app.py                  # the Streamlit app
├── requirements.txt        # streamlit, requests, pydub, unidecode
├── packages.txt            # ffmpeg (apt package for Streamlit Cloud)
├── .streamlit/
│   ├── config.toml          # theme (committed — no secrets)
│   └── secrets.toml         # your API keys — gitignored, never committed
├── .gitignore
├── README.md
└── context.md              # architecture / decisions orientation doc
```
