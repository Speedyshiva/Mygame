# Mines Game API Contracts

## Overview
Point-based mines game with admin panel for user management and customer panel for gameplay across 3 difficulty levels.

## Point System Logic
- Player bets X points before game starts
- Player earns X points for each safe cell revealed
- Player loses bet amount if they hit a mine
- Only admin can manually add/remove points from user accounts

## API Endpoints Required

### Authentication Endpoints

**POST /api/auth/login**
```json
Request: {
  "username": "string",
  "password": "string"
}
Response: {
  "success": true,
  "user": {
    "id": "number",
    "username": "string", 
    "role": "admin|customer",
    "points": "number"
  },
  "token": "string"
}
```

### User Management (Admin Only)

**POST /api/admin/users**
```json
Request: {
  "username": "string",
  "password": "string", 
  "initialPoints": "number"
}
Response: {
  "success": true,
  "user": { "id": "number", "username": "string", "points": "number" }
}
```

**GET /api/admin/users**
```json
Response: {
  "success": true,
  "users": [{ "id": "number", "username": "string", "points": "number", "createdAt": "date" }]
}
```

**POST /api/admin/users/:id/points**
```json
Request: {
  "amount": "number" // positive to add, negative to subtract
}
Response: {
  "success": true,
  "newBalance": "number"
}
```

**GET /api/admin/game-sessions**
```json
Response: {
  "success": true,
  "sessions": [{ 
    "id": "number", 
    "playerId": "number", 
    "playerName": "string",
    "difficulty": "EASY|MEDIUM|HARD",
    "betAmount": "number",
    "result": "won|lost",
    "pointsEarned": "number",
    "timestamp": "datetime"
  }]
}
```

### Game Endpoints

**POST /api/game/start**
```json
Request: {
  "difficulty": "EASY|MEDIUM|HARD",
  "betAmount": "number"
}
Response: {
  "success": true,
  "gameId": "string",
  "board": "array", // board structure without mine positions
  "config": { "gridSize": "number", "mineCount": "number" },
  "playerBalance": "number" // updated balance after bet deduction
}
```

**POST /api/game/:gameId/reveal**
```json
Request: {
  "row": "number",
  "col": "number"
}
Response: {
  "success": true,
  "result": "continue|won|lost",
  "revealedCells": "array", // updated cells to reveal
  "pointsEarned": "number", // points earned this reveal
  "totalPointsEarned": "number",
  "playerBalance": "number" // updated player balance
}
```

**POST /api/game/:gameId/flag**
```json
Request: {
  "row": "number", 
  "col": "number"
}
Response: {
  "success": true,
  "flagged": "boolean"
}
```

**GET /api/game/history**
```json
Response: {
  "success": true,
  "sessions": [{ 
    "id": "number",
    "difficulty": "string", 
    "betAmount": "number",
    "result": "won|lost",
    "pointsEarned": "number",
    "timestamp": "datetime"
  }]
}
```

## Database Models

### User Model
```python
class User:
    id: int
    username: str
    password: str (hashed)
    role: str # "admin" | "customer"  
    points: int
    created_at: datetime
```

### GameSession Model
```python
class GameSession:
    id: str # UUID
    player_id: int
    difficulty: str # "EASY" | "MEDIUM" | "HARD"
    bet_amount: int
    board_state: json # current board state
    mine_positions: json # mine positions array
    game_status: str # "playing" | "won" | "lost"
    points_earned: int
    created_at: datetime
    updated_at: datetime
```

## Mock Data to Replace

### Frontend Mock Data (mock.js)
- `MOCK_USERS` → Replace with API calls to `/api/admin/users`
- `MOCK_GAME_SESSIONS` → Replace with API calls to `/api/admin/game-sessions` and `/api/game/history`
- `generateMinePositions()` → Replace with server-generated positions
- `generateGameBoard()` → Replace with server-generated board

### Frontend Integration Points

1. **Login.jsx**: Replace localStorage auth with API authentication
2. **AdminPanel.jsx**: Replace localStorage operations with admin API calls
3. **CustomerPanel.jsx**: Replace local game logic with game API calls
4. **GameBoard.jsx**: Update to use real-time game state from API

## Game Flow
1. Player selects difficulty and bet amount
2. Frontend calls `/api/game/start` - server deducts bet, generates board
3. Player clicks cells - Frontend calls `/api/game/:gameId/reveal`
4. Server validates move, updates game state, awards points for safe reveals
5. Game ends when player hits mine (lose) or reveals all safe cells (win)
6. Server saves final game session with results

## Security Considerations
- JWT token authentication for all API calls
- Admin role verification for admin endpoints
- Game session validation to prevent cheating
- Rate limiting on game actions
- Input validation on all endpoints