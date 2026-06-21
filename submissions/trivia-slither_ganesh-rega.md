# TriviaSlither

---

## Attendee/Team Details

**Name:** Ganesh Rega
**GitHub Username:** GaneshRega
**LinkedIn Profile:** —
**GitHub Project Repository:** https://github.com/GaneshRega/trivia-slither

---

## Problem Statement Selected

```
Problem statement 61 + 63 (combined)
```

Kahoot-like Quiz Game (#61) fused with Infinite Canvas Multiplayer / Slither.io (#63).

---

## Project Description

TriviaSlither is a real-time multiplayer snake arena that periodically interrupts gameplay with live Kahoot-style trivia rounds.

- Players join a named room and control a snake on a 3000×3000 infinite-canvas world.
- Every 25 seconds the game pauses for a 10-second trivia question broadcast to all players simultaneously.
- Correct answers grow your snake and boost speed; wrong answers shrink it and slow it down.
- A live leaderboard tracks scores across the session.

It is aimed at groups who want a fast, social game that mixes skill (snake movement, collision avoidance) with knowledge (trivia). Valkey is the backbone that makes the whole experience work across multiple server instances without any shared in-process state.

---

## Approach

1. **Understood the dual problem** — slither.io demands ultra-low-latency position sync; Kahoot demands synchronized broadcast of questions to all clients at the same instant. Both map naturally onto Valkey pub/sub.

2. **Designed a three-connection Valkey architecture** — one client for commands (`HSET`, `ZADD`, `GET`), one dedicated publisher, one dedicated subscriber. This separation is the pattern that lets any number of Node.js processes share a single live game.

3. **In-memory physics, Valkey as mirror** — snake positions are computed in memory at 20 Hz for minimum latency. Every 4 ticks (~5 Hz) a compact snapshot is published on `ch:room:{id}:state` and key summaries are persisted to Valkey hashes and sorted sets, so a late-joining client or a second server instance can always reconstruct current state.

4. **Quiz lifecycle fully in Valkey** — the active question (including the correct answer) is stored with a TTL so any server process can validate an answer. The correct answer is never sent to clients on `quiz:start`; it is only revealed on `quiz:reveal` after the window closes.

5. **Leaderboard via Sorted Set** — `ZADD room:{id}:leaderboard score playerId` after every quiz round and food pickup gives O(log N) inserts and O(log N) top-10 reads.

---

## Tech Stack and Tools Used

**Frontend:** Vanilla HTML/CSS/Canvas, Socket.io client
**Backend:** Node.js, Express, Socket.io 4
**Database / State:** Valkey (via ioredis) — pub/sub, sorted sets, hashes, TTL keys
**Other Tools:** Docker (Valkey container), nanoid, dotenv

---

## Key Features

1. Real-time snake movement at 20 Hz with smooth angle interpolation and camera-follow canvas
2. Synchronized trivia interrupts — question broadcast and answer reveal via Valkey pub/sub across all connected clients
3. Correct/wrong answer physics effects — speed boost or slow penalty applied instantly to in-memory snake state
4. Persistent leaderboard in a Valkey sorted set, returned on every new player join
5. Horizontal-scaling ready — three-connection Valkey pattern means multiple Node processes can share one live room
6. Snake-vs-snake collision, wall death, respawn with score penalty, and food drops from dead snakes

---

## What is Working?

- Full join → play → trivia → leaderboard loop verified end-to-end (40/40 automated assertions)
- Two simultaneous socket clients receive state broadcasts, submit answers, get quiz reveal, and see leaderboard updates
- Server boots cleanly against a Docker Valkey container; graceful shutdown closes all three connections

---

## What is Still in Progress?

- React frontend (current frontend is vanilla Canvas/JS — works but does not use React as recommended)
- Breeth AI integration not yet added
- Mobile touch controls
- Spectator mode and room listing UI

---

## Screenshots or Demo

**Deployed Link:** —
**Demo Video Link:** —
**Screenshots:** Run `npm start` locally (requires Docker + Valkey on port 6379) and open http://localhost:3000

---

## Challenges Faced

- Valkey's pub/sub connections cannot issue other commands, requiring three separate ioredis clients — getting the subscriber's `pmessage` routing to correctly fan out to Socket.io rooms took careful channel naming.
- Keeping the `quiz:start` payload free of the correct answer while still storing it in Valkey for server-side validation required splitting the payload before publishing.
- Smooth snake turning on a Canvas without input lag needed angle interpolation (clamped delta per tick) rather than direct angle assignment.

---

## Learnings

- Valkey sorted sets are a natural fit for live leaderboards — a single `ZADD` and `ZREVRANGE` replaces what would otherwise be a full table scan.
- Separating publisher and subscriber connections is not optional in Redis/Valkey — a subscribed connection enters a special mode and cannot issue normal commands.
- Pub/sub as the sole bridge between the game loop and Socket.io emits makes horizontal scaling a first-class property of the design, not an afterthought.

---

## Future Improvements

- Migrate frontend to React + Vite for component-based UI and easier state management
- Add Breeth AI to generate dynamic trivia questions per-room based on a topic the players choose
- Valkey cluster mode demo with multiple Node processes behind a load balancer
- Persistent room history and player profiles stored in Valkey hashes

---

## Final Note

TriviaSlither intentionally combines two problem statements (#61 and #63) because the interesting engineering challenge is making them coexist: the game loop must not stall during a trivia pause, and the trivia system must not desync across clients who may be connected to different server processes. Valkey pub/sub solves both problems with the same mechanism.
