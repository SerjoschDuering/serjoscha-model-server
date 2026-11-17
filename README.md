# serjoscha-model-server (class demo)

A **model server** for the AI Methods Hackathon (IAAC MaAI, 17 Nov 2026). It runs a trained autoencoder (ONNX) that flags unusual clothing images, and offers three doors to the same `predict` function:

| Door | For | Address |
|---|---|---|
| Web page | people | `https://serjoscha-model-server.onrender.com` |
| API | programs (notebook, web app) | `…/gradio_api/call/predict` or `gradio_client` |
| MCP (optional) | AI agents (Claude, Cursor) | `…/gradio_api/mcp/` |

**The approach in one sentence:** a **Gradio** app (`app.py`) turns one Python function into a web page and an API; **Render** runs it online for free; a **Blueprint** (`render.yaml`) tells Render how.

---

## Files

| File | Job |
|---|---|
| `app.py` | the server: loads the model once, preprocesses the image, `predict` = the API endpoint |
| `anomaly_ae.onnx` | the trained model (architecture + weights; no PyTorch needed) |
| `anomaly_ae_config.json` | settings: image size, threshold |
| `requirements.txt` | packages to install (pinned versions) |
| `.python-version` | Python version for Render (3.12) |
| `render.yaml` | the **Blueprint**: what Render builds and how it starts |
| `examples/` | test images |
| `call_api.ipynb` | calls the server from a notebook |

---

## 1 · Run it on your computer

```bash
python -m venv .venv && source .venv/bin/activate     # Windows: .venv\Scripts\activate
pip install -r requirements.txt
python app.py                                          # open http://127.0.0.1:7860
```
Then open `call_api.ipynb`, keep `URL = "http://127.0.0.1:7860"` and run the cells.

---

## 2 · Accounts (once)

1. **GitHub:** you need an account and this folder as a repository on GitHub.
   - Easiest: **GitHub Desktop** → *Add existing repository* → *Publish repository*.
   - Terminal: `gh repo create <your-name>-model-server --public --source=. --push`
2. **Render:** go to **render.com** → *Get started* → **Sign in with GitHub**. No separate password.
   - The first time, Render asks which repositories it may read: choose *Only select repositories* and pick this one (or *All*).
   - Repos inside the **iaac-maai** organisation only appear after an org owner has installed the Render GitHub app on the organisation. If yours is missing, use a repo in your personal account.

---

## 3 · Deploy with the Blueprint (recommended)

The Blueprint is `render.yaml`. Render reads it and creates the service; you click once.

1. In `render.yaml`, change `name:` to your own name (e.g. `team4-model-server`). It becomes your URL.
2. Push to GitHub.
3. Render dashboard → **New** → **Blueprint** → choose your repository → **Apply**.
4. Open the service → **Logs**. Wait until it says **Live** (first build: a few minutes).
5. Copy the URL from the top of the service page: `https://<name>.onrender.com`.

One-click link (replace the repo):
`https://render.com/deploy?repo=https://github.com/SerjoschDuering/serjoscha-model-server`

### What the Blueprint says
```yaml
services:
  - type: web                    # a web service that answers HTTP requests
    name: serjoscha-model-server # → https://serjoscha-model-server.onrender.com
    runtime: python
    plan: free
    region: frankfurt            # servers in Europe; cannot be changed later
    buildCommand: pip install -r requirements.txt   # install, once per deploy
    startCommand: python app.py                     # start, then keep running
    healthCheckPath: /                              # Render checks that it answers
```

---

## 4 · Deploy by hand (alternative, no Blueprint)

Render dashboard → **New** → **Web Service** → choose your repository, then fill in:

| Field | Value |
|---|---|
| Language | Python 3 |
| Region | Frankfurt |
| Build command | `pip install -r requirements.txt` |
| Start command | `python app.py` |
| Instance type | Free |

→ **Deploy web service**. Same result; the Blueprint just saves the clicks and keeps the settings in Git.

---

## 5 · Call it

```python
from gradio_client import Client                       # pip install gradio_client
client = Client("https://serjoscha-model-server.onrender.com")
client.predict(image_b64, api_name="/predict")         # image_b64 = the photo as base64 text
```
Plain HTTP (what a web app does), two steps:
```bash
curl -X POST https://serjoscha-model-server.onrender.com/gradio_api/call/predict \
     -H "Content-Type: application/json" -d '{"data": ["<base64 image>"]}'      # → {"event_id": "…"}
curl https://serjoscha-model-server.onrender.com/gradio_api/call/predict/<event_id>
```
The page footer → **Use via API** lists every endpoint with examples.

---

## 6 · Notes on this approach

- **Free plan:** the server **sleeps after 15 minutes** without requests. The next request wakes it in **about a minute**: slow, not broken. Open it before a demo.
- **No disk:** files the app writes are lost on restart. Results that must survive go to a database (Wednesday: Cloudflare D1).
- **Port:** Render tells the app its port in `$PORT`; `app.py` listens on `0.0.0.0` and that port. Keep that line.
- **Python version:** new Render services default to 3.14; `.python-version` pins 3.12.
- **Model size:** keep the model file under 100 MB (GitHub limit without LFS).
- **Every push to `main` redeploys** automatically.
- **Why Gradio:** one function → a page for people + an API for programs, in one file. For an API only, FastAPI is the usual alternative.

## 7 · Optional: MCP for AI agents

1. In `requirements.txt` write `gradio[mcp]==6.32.0` instead of `gradio==6.32.0`.
2. In `app.py` add `mcp_server=True` to `demo.launch(...)`.
3. Connect an agent, e.g. Claude Code:
   `claude mcp add --transport http model-server https://serjoscha-model-server.onrender.com/gradio_api/mcp/`

The agent sees a tool `predict`; its description is the function's docstring. Tested locally on 17 Nov 2026.

## If something fails

| Log says | Fix |
|---|---|
| `No module named …` | add the package to `requirements.txt` |
| `No open ports detected` | keep `server_name="0.0.0.0", server_port=int(os.environ.get("PORT", 7860))` |
| `FileNotFoundError` | file names are case-sensitive on the server |
| shape / size error | the input must match what the model was trained on |
| page loads forever | the free server is waking up; wait one minute |
