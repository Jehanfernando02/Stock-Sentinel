# 🛡️ Stock Sentinel

A real-time stock reservation system built to prevent overselling — the same problem Amazon and Daraz face when multiple users try to buy the last item at the same time.

Instead of letting two people buy the same item and dealing with the mess after, Stock Sentinel uses a **reserve → hold → confirm** flow. You click buy, the item is locked for 60 seconds while you checkout. If you bail, the stock comes back automatically.

## The Problem It Solves

Standard e-commerce CRUD doesn't handle concurrency. Two users hit "buy" at the same millisecond, both see stock = 1, both decrement it, now stock = -1 and two people think they own the same item. This project fixes that using Redis atomic operations and TTL-based reservations.

## Tech Stack

- **FastAPI** — async Python backend
- **Redis** — in-memory stock management, atomic ops, TTL reservations
- **PostgreSQL** — permanent storage for products and orders
- **React + Vite** — frontend with live reservation countdown
- **Docker Compose** — runs everything together

## How It Works

```
User clicks "Reserve"
  → Redis DECR stock atomically (thread-safe)
  → Reservation key set with 60s TTL
  → Timer starts in the UI

User completes checkout
  → Reservation key deleted
  → Order saved to Postgres

User does nothing / cancels
  → TTL expires or manual cancel
  → Redis INCR restores the stock
```

## Getting Started

**Prerequisites:** Docker, Node.js 18+, Python 3.11+

```bash
# Clone and enter the project
git clone https://github.com/your-username/stock-sentinel.git
cd stock-sentinel

# Start Redis and Postgres
docker-compose up -d postgres redis

# Run the backend
cd backend
pip install -r requirements.txt
uvicorn app.main:app --reload --port 8000

# Run the frontend
cd ../frontend
npm install
npm run dev
```

Open `http://localhost:5173` for the UI and `http://localhost:8000/docs` for the auto-generated API docs.

## API Overview

| Method | Endpoint | What it does |
|--------|----------|--------------|
| POST | `/products/` | Create a product |
| GET | `/products/` | List all products with live stock |
| POST | `/reserve/{id}` | Reserve an item (60s hold) |
| POST | `/purchase/{id}` | Confirm purchase after reserving |
| POST | `/cancel/{id}` | Cancel and restore stock |

## Redis Key Design

```
stock:product:{id}          → current available stock (DECR/INCR)
reservation:{user}:{id}     → active reservation (auto-expires in 60s)
lock:product:{id}           → distributed lock to prevent race conditions
```

## Project Structure

```
stock-sentinel/
├── backend/
│   ├── app/
│   │   ├── main.py
│   │   ├── redis_client.py
│   │   ├── models/
│   │   ├── routers/
│   │   └── services/
│   └── requirements.txt
├── frontend/
│   └── src/
│       ├── components/
│       └── pages/
└── docker-compose.yml
```

## Why This Is Interesting

Most portfolio projects are just CRUD apps with a database. This one deals with **concurrency** — a problem every real platform has to solve. The reservation pattern used here is the same approach used in ticket booking systems, flash sales, and limited-drop e-commerce. Redis handles it well because its operations are atomic by design and TTL means no manual cleanup jobs.

---

Built with FastAPI, Redis, and React.
