# Plan2Reality
# Plan2Reality AI

**Turn a 2D floor-plan image into an interactive, walkable 3D model, then ask questions about the building.**

Plan2Reality uses classical computer vision (OpenCV) to read walls, rooms, doors and windows from a blueprint, converts them into real-world coordinates, and renders the result as a live 3D scene in the browser with Three.js. A built-in spatial assistant answers questions such as *"How many rooms?"* or *"Show the emergency exit"* directly from the extracted data.

Built for **HackMint** (Problem ID: HACM006).

> `Blueprint image → Image processing → Feature extraction → JSON (metres) → 3D reconstruction → Interactive navigation`

<!-- Add a screenshot or GIF here, e.g. ![Demo](docs/demo.gif) -->

---

## Why this project?

Reading a floor plan still takes imagination. Clients can't easily picture a space from flat lines, and professional CAD tools are heavy and need structured input (dimensions, constraints, clean geometry). Old, scanned or visually complex drawings usually need manual tracing first.

Plan2Reality automates that **visual interpretation stage**. It does not replace CAD. It acts as a computer-vision front end that turns a plain image into structured spatial data and a navigable 3D model with minimal manual input.

## Features

- Upload a floor plan (drag and drop) or pick one of 6 built-in samples
- Detects **walls** (horizontal, vertical and angled), **rooms** (including L-shaped), **doors** and **windows**
- **Scale calibration**: enter the real width in metres to get true dimensions
- Interactive 3D view: orbit and zoom, or **first-person walk mode** with collision
- Click to inspect rooms, rename rooms, exterior exits highlighted in red
- **Spatial assistant** (rule-based, no external API): room counts, largest/smallest room, total area, door and window counts, where a room is, exits
- **Download the model as `.glb`**
- Login, saved projects and a dashboard (SQLite)
- Original blueprint shown side by side with the 3D model

## How it works

1. **Preprocess**: grayscale, lighting normalisation, Otsu thresholding
2. **Walls**: adaptive wall thickness (distance transform) and morphological filtering extract straight and angled walls
3. **Openings**: gaps in walls are classified as doors (empty gap) or windows (thin glazing line)
4. **Rooms**: enclosed free space becomes room polygons via connected components and contours
5. **Labels**: room names read by template matching (optional Tesseract OCR), editable in the UI
6. **Reasoning**: adjacency between rooms and exterior exits are computed
7. **Output**: everything is saved as JSON in metres, then built into a 3D scene with Three.js

The pipeline is explainable and deterministic and needs no training data.

## Tech stack

| Layer | Technology |
|---|---|
| Computer vision | Python, OpenCV, NumPy (optional: Tesseract OCR) |
| Backend | FastAPI, Uvicorn, SQLite |
| Frontend | React 18, Vite 5, Three.js |
| Auth | PBKDF2 password hashing and signed tokens |

## Getting started

**Requirements:** Python 3.9+ and Node.js 18+

**Quick start (one port)**

```bash
# Windows
start.bat

# Mac / Linux
./start.sh
```

Then open **http://localhost:8000**.

**Development mode (two terminals)**

```bash
# Terminal 1 (project root)
python -m venv venv
venv\Scripts\activate          # Mac/Linux: source venv/bin/activate
pip install -r backend/requirements.txt
uvicorn backend.main:app --reload --port 8000

# Terminal 2
cd frontend
npm install
npm run dev                    # http://localhost:5173
```

## Quick demo

1. Register or log in
2. Upload `sample_data/house3.png` and enter a width of `14` m
3. Rotate and zoom the 3D model
4. Ask: *"How many rooms?"*, *"Where is the kitchen?"*, *"Show the emergency exit"*
5. Switch to walk mode and move through the building
6. Download the `.glb`
7. Try `angled.png` and `lshape.png` for non-rectangular layouts

## Project structure

```
PLAN2REALITY/
├── ai/                  # Computer vision + assistant (pure Python)
│   ├── parser.py        # image -> walls, rooms, doors, windows (JSON, metres)
│   ├── assistant.py     # rule-based spatial Q&A
│   └── make_samples.py  # generates the 6 sample blueprints
├── backend/             # FastAPI server
│   ├── main.py          # API routes (also serves frontend/dist)
│   ├── auth.py          # password hashing and tokens
│   ├── db.py            # SQLite setup (users, projects)
│   └── requirements.txt
├── frontend/            # React + Vite + Three.js
│   └── src/
│       ├── pages/       # Login, Dashboard, Workspace
│       ├── components/  # Viewer.jsx (3D scene, walk mode, .glb export)
│       └── api.js       # all backend calls
├── sample_data/         # house1, apartment, house3, office, lshape, angled
├── start.bat / start.sh
└── README.md
```

## API overview

All routes are under `/api`. Everything except `/auth/*` and `/samples` needs `Authorization: Bearer <token>`.

| Method | Route | Purpose |
|---|---|---|
| POST | `/auth/register`, `/auth/login` | Create account / get token |
| GET | `/auth/me` | Current user |
| POST | `/analyze` | Run the CV pipeline on an uploaded image (optional `width_m`) |
| GET / POST | `/samples`, `/sample/{name}` | List / analyze a built-in blueprint |
| GET | `/projects`, `/projects/{id}` | List / open saved projects |
| PUT / DELETE | `/projects/{id}` | Save edits / delete |
| GET | `/projects/{id}/image` | Original blueprint |
| POST | `/ask` | Ask the spatial assistant |

## Limitations

- Works best on clean digital plans (dark walls, light background)
- Hand-drawn sketches and phone photos are handled only approximately; lighting is corrected but perspective skew is not
- Curved walls are not detected
- Room names need a standard font, otherwise a size-based guess is used (names can be edited in the UI)
- One image equals one floor; no multi-storey stacking yet

## Future work

- Trained ML model (e.g. CubiCasa5k / YOLO) for hand-drawn and photographed plans
- Perspective correction for angled photos
- PDF / DXF / CAD import with multi-floor support
- LLM-powered assistant for free-form questions
- Furniture recognition from blueprint icons

## Team

2nd Year, B.Tech Artificial Intelligence and Data Science, KPR Institute of Engineering and Technology

- Arishni Devi M
- Asu M
- Annehilda Mtisi

