

## 🔄 Real-Time Communication: Why WebSockets?

### ❌ Why not HTTP?
- HTTP is **request-response** only.
- Server **cannot push** events to clients.
- Not ideal for **constant state exchange** in real-time games.

### ✅ Why WebSockets?
- Full-duplex, persistent connection.
- **Server can push** game events anytime.
- Required for low-latency moves, live board updates.

---

## 🎮 Game Session: In-Memory Game Class Design

```js
class Game {
  constructor(player1) {
    this.player1 = player1;
    this.player2 = null;
    this.state = "waiting"; // "active", "ended"
    this.moves = [];
    this.startTime = null;
  }

  join(player2) {
    this.player2 = player2;
    this.state = "active";
    this.startTime = new Date();
  }

  makeMove(move) {
    this.moves.push(move);
    // validate move legality, checkmate status etc.
  }
}
```

### 🌐 Where to store it?
- Store in **in-memory variable** (like a Map of games).
- Lightweight and fast.
- Downside: **data loss** on server crash.

```js
const games = new Map(); // gameId => Game instance
```

---

## 🔁 Events Between Client & Server

| Event Name     | Trigger                                       |
|----------------|-----------------------------------------------|
| `init_game`    | Player 1 joins, server waits for Player 2     |
| `start_game`   | Player 2 joins, game begins                   |
| `make_move`    | Either player makes a valid move              |
| `game_over`    | Checkmate/resign/draw                        |
| `invalid_move` | Server rejects illegal move                  |
| `update_state` | Server pushes new board state after each move |

### 🔐 Server-side Validations
- Validate user identity
- Check it’s the user’s turn
- Validate move legality (e.g., no moving into check)
- Update state only after passing all checks

---

## ⚠️ Limitations of Current Design

- Game state in-memory: lost on crash
- No persistence = no game resume after failure
- Not horizontally scalable without sticky sessions or shared state

---

## 📈 Future Improvements

| Improvement          | Reason                                   |
|----------------------|-------------------------------------------|
| Use Redis/DB for game state | To persist and restore on crash         |
| WebSocket clustering | To support multi-server deployments       |
| Sticky sessions or session sharing | Keep user on same server node     |
| Auto-scaling with load balancer | Handle user surge automatically      |
| Game state snapshots | Restore point-in-time game state          |
| Validation engine module | Decouple chess rules from server logic   |
