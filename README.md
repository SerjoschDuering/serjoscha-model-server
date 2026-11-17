# Team 0 · model server (class demo)

A model server for the AI Methods Hackathon. It runs the class example model (an autoencoder that flags unusual clothing images) and offers a web page and an API.

## Run it on your computer
```bash
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
python app.py          # open http://127.0.0.1:7860
```

## Put it online
1. Push this folder to GitHub.
2. render.com → New → Blueprint → choose this repository → Apply.
3. Wait for "Live", then open `https://iaac-team0-model-server.onrender.com`.

## Call it
Open `call_api.ipynb` and set `URL`. The first call after 15 minutes of silence takes about a minute: the free server wakes up.
