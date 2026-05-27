# Fly-in

## Description

**Fly-in** is a strategic simulation that solves a complex logistics challenge: routing a fleet of autonomous drones through a network of interconnected zones, from a starting hub to a final destination, **in the fewest turns possible**.

The problem goes beyond shortest-path routing. Drones compete for limited zone capacity and link throughput, so the system must act as a smart traffic controller — deciding when a drone should fly full speed, take a longer priority route to avoid congestion, or wait in place to let others pass.

The project is built around **Dijkstra's algorithm** extended with dynamic cost scaling, turn-by-turn reservation, and real-time recalculation to handle collisions.

---

## ✨ Features

- **Dynamic Dijkstra pathfinding** with initial path caching and real-time recalculation on collision
- **Predictive zone reservation** — tracks occupancy turn-by-turn to respect capacity constraints before they become collisions
- **Traffic cost scaling** — route cost increases with congestion density, naturally distributing drones across alternative paths
- **Zone type hierarchy** — `NormalZone`, `RestrictedZone`, `PriorityZone`, each with unique movement costs and capacity rules
- **Connection capacity enforcement** — `max_link_capacity` limits simultaneous drone traversal per link
- **Path optimization guard** — a drone only switches routes if the new path is strictly shorter than its remaining current route
- **Real-time Pygame visualization** — see zones at capacity, drones in transit, and waiting states live
- **Custom map support** — load any map file via CLI argument

---

## 🚀 Quick Start

### Install dependencies

```bash
make install
```

### Run the default map

```bash
make run
```

### Run with a custom map

```bash
make run MAP=<file>
# or
python3 -m fly_in <file>
```

---

## 🧠 How the Routing Engine Works

```
Simulation Start
      │
      ▼
[1] Initial Dijkstra Pass
    └─ Compute optimal path for each drone from start_hub → end_hub
    └─ Cache paths to reduce per-turn overhead

      │
      ▼
[2] Turn Loop
    ├─ [2a] Predictive Reservation
    │       └─ Reserve zones & links for planned moves (respects max_drones, max_link_capacity)
    │
    ├─ [2b] Traffic Cost Scaling
    │       └─ Increase edge cost dynamically based on current density
    │       └─ Forces subsequent drones onto less congested alternative routes
    │
    ├─ [2c] Move Drones
    │       └─ Each drone advances along its cached path
    │       └─ Restricted zones require 2 turns of connection transit
    │
    └─ [2d] Collision Check
            └─ If a zone is unexpectedly occupied → trigger real-time Dijkstra recalc
            └─ Switch only if new path length < remaining current path length

      │
      ▼
Simulation ends when all drones reach end_hub
```

---

## 🗺️ Zone & Connection Types

| Entity | Property | Description |
|--------|----------|-------------|
| `NormalZone` | Standard cost | Default movement, standard capacity |
| `RestrictedZone` | Higher cost | Requires 2 turns to traverse (drone occupies connecting link) |
| `PriorityZone` | Lower cost | Preferred routing, used as bypass under congestion |
| Connection | `max_link_capacity` | Max drones allowed to traverse the link simultaneously |
| Zone | `max_drones` | Max drones that can occupy the zone at once |

---

## 🏗️ Object-Oriented Architecture

The simulation is built on three core entity classes:

**Zones** encapsulate their coordinates, movement cost, and real-time occupancy tracking. Subclasses override cost and transit duration logic.

**Connections** are first-class objects linking two zones. They store `max_link_capacity` and track per-turn drone transit to enforce throughput limits.

**Drones** are independent objects identified by ID (e.g. `D1`, `D2`). Each maintains its current position, active path cache, and transit progress when crossing restricted zones.

---

## 📁 Project Structure

```
Test_Dijkstra/
├── src/                # Core simulation: zones, connections, drones, scheduler
├── fly_in.py           # Entry point and CLI
├── __init__.py
├── requirements.txt    # pydantic, pygame, flake8, mypy
├── Makefile
└── README.md
```

---

## 🛠️ Tech Stack

| Component | Library |
|-----------|---------|
| Pathfinding | Custom Dijkstra implementation |
| Visualization | `pygame` |
| Data validation | `pydantic` |
| Type checking | `mypy` |
| Linting | `flake8` |
| Package manager | `uv` |

---

## 📋 Requirements

- Python ≥ 3.10
- [`uv`](https://github.com/astral-sh/uv) installed
- A display environment for Pygame (or terminal color mode)

---

## 📄 License

This project is open source. See [LICENSE](LICENSE) for details.
