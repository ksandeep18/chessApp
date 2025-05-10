
# ⚙️ Backend Setup & Real-Time Game Server (WebSocket + TypeScript)

## 📁 Folder Structure

```
chess/
│
├── backend1/
│   ├── src/
│   │   ├── index.ts
│   │   ├── gameManager.ts
│   │   ├── game.ts
│   │   └── messages.ts
│   ├── dist/ (compiled JS)
│   ├── tsconfig.json
│   └── package.json
└── frontend/ (will be done later)
```

---

## 🔧 TypeScript Setup

In `backend1/`:

```bash
npm init -y
npm install typescript ws
npm install --save-dev @types/ws
npx tsc --init
```

Update `tsconfig.json`:

```json
{
  "compilerOptions": {
    "outDir": "./dist",
    "rootDir": "./src",
    "module": "CommonJS",
    "target": "ES6",
    "esModuleInterop": true,
    "strict": true
  }
}
```

---

## 🚀 Start WebSocket Server (src/index.ts)

```ts
import { WebSocketServer } from "ws";
import { GameManager } from "./gameManager";

const wss = new WebSocketServer({ port: 8000 });
const gameManager = new GameManager();

wss.on("connection", (ws) => {
  console.log("Client connected");
  gameManager.addUser(ws);

  ws.on("message", (message) => {
    gameManager.handleMessage(ws, message.toString());
  });

  ws.on("close", () => {
    gameManager.removeUser(ws);
  });
});
```

**Commands to run on Windows terminal**:

```bash
npx tsc -b
node dist/index.js
```

Use [Hoppscotch WebSocket client](https://hoppscotch.io) → `ws://localhost:8000` to test. If messages echo/respond, it's working.

---

## 🧠 GameManager Class (src/gameManager.ts)

```ts
import { WebSocket } from "ws";
import { Game } from "./game";
import { INIT_GAME, MOVE } from "./messages";

export class GameManager {
  private games: Game[] = [];
  private pendingUser: WebSocket | null = null;
  private users: WebSocket[] = [];

  addUser(ws: WebSocket) {
    this.users.push(ws);
  }

  removeUser(ws: WebSocket) {
    this.users = this.users.filter((user) => user !== ws);
  }

  handleMessage(ws: WebSocket, rawMessage: string) {
    const msg = JSON.parse(rawMessage);

    if (msg.type === INIT_GAME) {
      if (this.pendingUser) {
        const game = new Game(this.pendingUser, ws);
        this.games.push(game);
        game.start();
        this.pendingUser = null;
      } else {
        this.pendingUser = ws;
      }
    }

    if (msg.type === MOVE) {
      const game = this.games.find(g => g.hasPlayer(ws));
      if (game) game.makeMove(ws, msg.data);
    }
  }
}
```

---

## 💬 Message Types (src/messages.ts)

```ts
export const INIT_GAME = "init_game";
export const MOVE = "move";
```

---

## 🧩 Game Logic Class (src/game.ts)

```ts
import { WebSocket } from "ws";
import { Chess } from "chess.js";

export class Game {
  player1: WebSocket;
  player2: WebSocket;
  board: string;
  moves: string[];
  startTime: Date;
  chess: any;

  constructor(p1: WebSocket, p2: WebSocket) {
    this.player1 = p1;
    this.player2 = p2;
    this.chess = new Chess();
    this.board = this.chess.fen();
    this.moves = [];
    this.startTime = new Date();
  }

  start() {
    const payload = JSON.stringify({ type: "start_game", board: this.board });
    this.player1.send(payload);
    this.player2.send(payload);
  }

  hasPlayer(ws: WebSocket) {
    return this.player1 === ws || this.player2 === ws;
  }

  makeMove(ws: WebSocket, move: string) {
    const turn = this.chess.turn() === "w" ? this.player1 : this.player2;
    if (ws !== turn || !this.chess.move(move)) return;

    this.moves.push(move);
    this.board = this.chess.fen();

    const payload = JSON.stringify({ type: "update", board: this.board, moves: this.moves });
    this.player1.send(payload);
    this.player2.send(payload);
  }
}
```

### ✅ Install chess.js

```bash
npm install chess.js
npm install --save-dev @types/chess.js
```

---

## 🧠 Why JSON.stringify / parse?

- WebSocket only sends **string data**.
- We convert JS objects → string using `JSON.stringify(...)`.
- On receiving, parse back to object via `JSON.parse(...)`.

```ts
// Example
ws.send(JSON.stringify({ type: "start_game", board: "..." }));
```

---

## ✅ Validations to Consider in `makeMove()`

1. **Is it player’s turn?**
2. **Is move legal?** (use `chess.js.move(move)`)
3. **Push move to moves[]**
4. **Update board string**
5. **Check for game over** (using `chess.js`)
6. **Send updates to both players**

---

## 🚀 Future Improvements

- Persist game state in Redis or DB
- Use UUIDs for games and players
- Auto matchmaking queue instead of one pending user
- Chess timer and ELO calculation
- Game history storage

---
