# 3d-game-project
3d-game-project
[README.md](https://github.com/user-attachments/files/26575078/README.md)
# Adaptive Maze AI

Adaptive Maze AI is a publishable hackathon-ready RL game prototype where the environment teaches the agent instead of staying static.

It includes:

- Adaptive difficulty that reacts to recent agent performance.
- Hierarchical rewards for immediate, short-term, and long-term planning.
- Replay logging plus a human correction workflow.
- A browser-native Gradio demo with a live difficulty slider and replay review.
- A playable Pygame front end.
- Built-in PyTorch DQN training plus heuristic behavior-clone rescue for a stronger shipped model.
- A custom FastAPI API plus an OpenEnv-compliant server wrapper.

## Game Loop

The agent plays a procedurally generated maze that contains:

- Coins for immediate rewards.
- Trap tiles and moving enemies that punish sloppy planning.
- An exit tile that ends the level.

Difficulty increases when the agent is consistently succeeding and decreases when it struggles. Reward shaping is hierarchical:

- Coin pickup: `+1`
- Three-coin chain: `+10`
- Level complete: `+25`
- Clean finish without touching a trap or enemy: `+100`
- Step cost and collision penalties keep the task meaningful.

Human feedback is stored as offline corrections. When a replay reviewer says "I would have moved here instead", the environment persists that preference and future training runs receive a reward signal for following it.

## Project Structure

```text
server/                 OpenEnv server entrypoint and Docker runtime
scripts/                Validation, demo, deployment, and packaging automation
src/adaptive_maze_ai/
  api.py            FastAPI environment service
  cli.py            Play, review, train, and serve commands
  config.py         Tunable game and reward settings
  difficulty.py     Adaptive curriculum controller
  env.py            Gymnasium environment implementation
  episode_store.py  Episode persistence
  feedback.py       Human correction storage and lookup
  models.py         Shared dataclasses and constants
  renderer.py       Pygame rendering helpers
  rewards.py        Reward graph engine
tests/
  test_env.py
  test_feedback.py
  test_openenv.py
```

## Quick Start

```bash
python -m venv .venv
.venv\Scripts\activate
pip install -e .[dev]
pytest
```

## One-Command Winner Build

```bash
python scripts/build_winner.py
```

Windows wrapper:

```bash
build_winner.bat
```

That pipeline runs training, comparison, demo generation, validation, deployment checks, and submission packaging.

## One-Command Proof

```bash
.venv\Scripts\python scripts/auto_validate.py
```

That command:

- runs 5 random episodes
- writes `validation_report.json`
- verifies episode logging and feedback persistence
- validates the local OpenEnv environment structure
- starts the OpenEnv server and runs runtime validation against it

If everything passes, it prints:

```text
[OK] ALL TESTS PASSED
All systems nominal.
```

## Play The Game

```bash
adaptive-maze-play
```

Controls:

- Arrow keys or `WASD`: move
- `Space`: stay in place
- `R`: start a new episode after finishing
- `Q`: quit

## 3D Demo (Optional)

Install the optional 3D renderer dependencies:

```bash
pip install -e .[3d]
```

Then launch the new 3D mode:

```bash
adaptive-maze-play-3d
```

The renderer will use built-in Ursina primitives by default. To use custom open-source 3D models, place them under:

```text
assets/models/
```

Supported model names:

- `knight.obj`, `knight.glb`, `agent.obj`, `agent.glb`
- `coin.obj`, `coin.glb`, `gold_coin.obj`, `gold_coin.glb`
- `enemy.obj`, `enemy.glb`, `ghost.obj`, `ghost.glb`
- `exit.obj`, `exit.glb`, `castle_gate.obj`, `portal.obj`

## Train An Agent

```bash
adaptive-maze-train --timesteps 20000
```

Artifacts are written to:

- `artifacts/models/`
- `artifacts/metrics/`
- `data/episodes/`
- `data/feedback/`

The trainer is self-contained and uses a PyTorch DQN loop, so it does not depend on an external RL framework to run.

For the full judge-facing pipeline with stronger defaults:

```bash
.venv\Scripts\python scripts/auto_train.py --timesteps 20000 --seed 42
```

That script runs:

- fixed-difficulty warmup
- adaptive fine-tuning
- automatic behavior-clone rescue if the RL checkpoint is still weak

The final model is written to `artifacts/models/dqn_adaptive_maze.pt`.

## Browser Demo

### Gradio Web Interface (Full Features)

```bash
.venv\Scripts\python app.py
```

Or:

```bash
adaptive-maze-web
```

The Gradio browser demo includes:

- manual control buttons
- trained, heuristic, and random autoplay modes
- a live difficulty slider that regenerates the maze
- a real-time difficulty-vs-reward chart
- a web replay-review tab for human corrections
- a baseline comparison chart

### Three.js 3D Web Demo (Interactive 3D)

For the most visually impressive demo with real-time 3D rendering, use the Three.js frontend:

```bash
pip install -e .[web]
adaptive-maze-web-3d
```

Then open `frontend/index.html` in your web browser. This provides:

- **Real-time 3D rendering** with Three.js and WebGL
- **Interactive controls** via keyboard (arrow keys, WASD) or on-screen buttons
- **Live statistics** showing difficulty, coins, rewards, and chains
- **Responsive design** - works on desktop and tablets
- **No installation needed for viewers** - just open the HTML file once the API server is running

**Features:**
- Smooth agent movement with animated coins
- Dynamic lighting and shadows
- Support for custom 3D models (place .obj/.glb files in `assets/models/`)
- Real-time environment state synchronization via REST API

**Controls:**
- Arrow keys or WASD: Move
- Space: Stay in place
- R: Reset the maze
- Buttons: Same actions via mouse click

**For Judges:** The Three.js demo showcases:
1. Beautiful, modern 3D visuals replacing the retro grid
2. Real-time adaptive difficulty (visible in UI stats)
3. Human-in-the-loop feedback (collect coins, avoid traps)
4. Trained AI model integration (AI Step button prepares for future AI replay)

## Review A Replay And Add Human Corrections

```bash
adaptive-maze-review
```

Controls:

- `[` / `]`: move between steps
- Click an action button or press arrow keys / `WASD` / `Space` to record a correction
- `Enter`: save and quit
- `Q`: quit

Corrections are appended to `data/feedback/corrections.jsonl`.

## Run The API

```bash
adaptive-maze-api --host 127.0.0.1 --port 8000
```

Endpoints:

- `GET /health`
- `POST /reset`
- `POST /step`
- `GET /metrics`
- `GET /episodes`

## Run The OpenEnv Server

```bash
.venv\Scripts\python run.py
```

Or run the server directly:

```bash
.venv\Scripts\python -m server.app
```

Validate the OpenEnv package layout:

```bash
.venv\Scripts\openenv validate . --json
```

## Generate Demo Assets

```bash
.venv\Scripts\python scripts/generate_demo_frames.py
```

Outputs:

- `demo_frames/`
- `demo.mp4` if `ffmpeg` is available
- `demo_script.txt`

## Baseline Comparison

```bash
.venv\Scripts\python scripts/compare.py --episodes 10
```

Outputs:

- `comparison.txt`
- `data/metrics/comparison.json`
- `data/metrics/comparison.png`

## Deployment

The repo now includes:

- `openenv.yaml`
- `app.py`
- `server/app.py`
- `server/Dockerfile`
- `Dockerfile`
- `space.py`

Attempt deployment with:

```bash
.venv\Scripts\python scripts/deploy_hf_space.py
```

The script automatically writes `deployment_report.json`. If Hugging Face auth is missing, it fails with a precise message instead of failing silently.
If `openenv push` is blocked, the repo is still ready for a manual Hugging Face Space upload using the root `Dockerfile` or `requirements.txt`.

## Winner Package

Build the final submission bundle with:

```bash
.venv\Scripts\python scripts/build_winner_package.py
```

Outputs:

- `WINNER_PACKAGE.md`
- `submission.zip`

## Publishing Notes

This repo is ready for GitHub publishing and local demoing. The only external blocker to a live Hugging Face Space is authentication: set `HF_TOKEN` or run `huggingface-cli login`, then run the deploy script.
