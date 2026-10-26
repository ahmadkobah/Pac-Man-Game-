# Pac-Man Game (Browser Clone)

A small recreation of the classic Pac-Man arcade game, written as a single HTML page with plain
JavaScript and the HTML5 Canvas API. It was built as a learning project and runs in any modern
web browser without a build step or server.

---

## Table of Contents

1. [Overview](#overview)
2. [Project Status](#project-status)
3. [Gameplay and Controls](#gameplay-and-controls)
4. [Features](#features)
5. [How It Works](#how-it-works)
6. [Technology Stack](#technology-stack)
7. [Project Structure](#project-structure)
8. [Getting Started](#getting-started)
9. [Configuration](#configuration)
10. [Known Issues](#known-issues)
11. [Possible Improvements](#possible-improvements)
12. [License](#license)

---

## Overview

### Purpose

This project is a simplified clone of the arcade game Pac-Man. The player steers Pac-Man through
a maze, eats pellets to collect points and avoids a ghost. The goal of the project is to practise
browser game programming, not to reproduce the original game exactly.

### What it demonstrates

- Drawing a tile-based maze, characters and items on an HTML5 `<canvas>`
- A frame-based game loop with `requestAnimationFrame`
- Keyboard input handling with the `keydown` event
- Axis-aligned collision detection between round characters and square wall tiles
- Distance-based collision between characters and pellets
- Simple random movement for an enemy (ghost)
- Game state handling: start screen, lives, score, game over, win and restart

---

## Project Status

The game is playable from start to finish, but it is an early, simplified version. Several
elements of the original game are not implemented (see [Known Issues](#known-issues) and
[Possible Improvements](#possible-improvements)). The repository contains one commit
("GameUpload").

---

## Gameplay and Controls

1. Open the game in a browser. The board is blurred and a **Start Game** button is shown.
2. Click **Start Game**. The board becomes sharp and the game begins.
3. Move Pac-Man with the arrow keys and eat all pellets in the maze.
4. Avoid the red ghost. Touching it costs one life.
5. When all pellets are eaten, you win. When all lives are lost, the game is over.
6. Click **Play Again** to start a new game.

| Input            | Action                                         |
|------------------|------------------------------------------------|
| Arrow Up         | Move up                                        |
| Arrow Down       | Move down                                      |
| Arrow Left       | Move left                                      |
| Arrow Right      | Move right                                     |
| Start Game (button) | Start the first game                        |
| Play Again (button) | Reset the board and start again (shown after game over or win) |

Pac-Man keeps moving in the last chosen direction until it hits a wall. A new direction is only
accepted if Pac-Man can move in that direction at the moment the key is pressed.

### Scoring

| Item          | Maze symbol | Points |
|---------------|-------------|--------|
| Pellet        | `.`         | 10     |
| Power pellet  | `o`         | 50     |

- The player starts with **3 lives**.
- Lives and score are shown below the game board.
- There is a single level. There is no high-score list.

---

## Features

- 28 x 31 tile maze defined as text in the source code (16 px per tile, 448 x 496 px canvas)
- Pac-Man drawn as a yellow circle with an open mouth
- One red ghost that moves through the maze in random directions
- Normal pellets and larger power pellets, with different point values
- Lives counter and score counter
- Short invincibility after losing a life: Pac-Man blinks and cannot be hit again for a moment
- One-second pause after losing a life, with Pac-Man and the ghost moved back to a reset position
- "GAME OVER" screen with the final score
- "YOU WIN!" message with the final score when all pellets are eaten
- Start screen with a blurred board, and a restart button

---

## How It Works

All game code is inside a `<script>` block in `pacman/draw2.html`.

### Game loop

```
 page load
     |
     v
+------------+     +-------------------------------+
| initGame() | --> | board blurred, "Start Game"   |
+------------+     +-------------------------------+
     |                         | click
     v                         v
+--------------------------------------------------+
| gameLoop()  (runs every frame)                   |
|                                                  |
|  1. clear canvas                                 |
|  2. drawWalls, drawPacman, drawGhosts,           |
|     drawPellets                                  |
|  3. if running:   update()                       |
|     if game over: displayGameOver()              |
|  4. requestAnimationFrame(gameLoop)              |
+--------------------------------------------------+
     |
     v
+--------------------------------------------------+
| update()                                         |
|  - count down invincibility timer                |
|  - move Pac-Man if the next position is free     |
|  - remove eaten pellets, add points              |
|  - no pellets left -> win                        |
|  - ghost touches Pac-Man -> loseLife()           |
|  - moveGhosts()                                  |
+--------------------------------------------------+
```

### Main state variables

| Variable          | Meaning                                                   |
|-------------------|-----------------------------------------------------------|
| `pacman`          | Position, radius, speed and current direction (`dx`, `dy`) |
| `ghosts`          | Array of ghost objects (currently one ghost)              |
| `pellets`         | Remaining pellets, each with position, radius and type    |
| `walls`           | Wall rectangles built from the maze                       |
| `lives`, `score`  | Player lives (starts at 3) and points                     |
| `gameRunning`     | True while the game is being updated                      |
| `gameOver`        | True after the last life is lost                          |
| `invincible`, `invincibleTimer` | Short protection after a hit (60 frames)     |

### Maze

The maze is an array of 31 strings with 28 characters each:

| Character | Meaning                                  |
|-----------|------------------------------------------|
| `#`       | Wall tile (drawn as a blue square)       |
| `.`       | Pellet                                   |
| `o`       | Power pellet                             |
| space     | Empty floor (used for the ghost area)    |

`initGame()` reads this array once and builds the `walls` and `pellets` lists from it.

### Movement

- Pac-Man moves 1 pixel per frame (`tileSize / 16`) in its current direction.
- Before each step, the game checks whether the new position would touch a wall. If it would,
  Pac-Man stops.
- Speed is counted per frame, so the game runs faster on screens with a higher refresh rate.

### Collision detection

- **Walls:** `checkCollision(x, y)` treats the character as a square around its centre (using
  Pac-Man's radius) and tests it against every wall rectangle.
- **Pellets:** a pellet is eaten when the distance between Pac-Man's centre and the pellet is
  smaller than Pac-Man's radius.
- **Ghost:** Pac-Man loses a life when the distance between the two centres is smaller than the
  sum of their radii.

### Ghost behaviour

The ghost does not chase Pac-Man. It uses simple random movement:

1. Every 30 frames, or right after it hits a wall, the ghost picks a new direction.
2. It looks at the four directions (up, down, left, right) and keeps only those that are free.
3. It picks one of the free directions at random.

There are no chase, scatter or frightened modes as in the original game.

### Lives, win and game over

- `loseLife()` removes one life and starts a 60-frame invincibility period.
- If lives remain, Pac-Man and the ghost are moved back and the game pauses for one second.
- If no lives remain, `gameOver` is set, the "GAME OVER" screen is drawn and the
  **Play Again** button appears.
- If no pellets remain, the game stops, shows "YOU WIN!" and the **Play Again** button.

---

## Technology Stack

| Area      | Technology                                    |
|-----------|-----------------------------------------------|
| Language  | JavaScript (ES6, no framework)                |
| Rendering | HTML5 Canvas 2D API                           |
| Layout    | HTML and inline CSS (Flexbox)                 |
| Runtime   | Any modern web browser                        |
| Build     | None (no package manager, no bundler)         |

---

## Project Structure

```
Pac-Man-Game-/
|-- pacman/
|   `-- draw2.html             # The complete game: HTML, CSS and JavaScript
|-- Ghost.png                  # Image of a ghost (not used by the code)
|-- PacMan.png                 # Image of Pac-Man (not used by the code)
|-- PacMan Maze.png            # Image of a maze (not used by the code)
|-- PacMan Documtation.docx    # Project documentation (Word)
`-- PacMan Documtation.pdf     # Project documentation (PDF)
```

The images and the documentation files are separate project material. The game draws all
graphics with canvas shapes and does not load any image file.

---

## Getting Started

### Prerequisites

- A modern web browser (for example Chrome, Firefox, Edge or Safari)
- Git (optional, only to clone the repository)

### Get the code

```
git clone https://github.com/ahmadkobah/Pac-Man-Game-.git
cd Pac-Man-Game-
```

### Run the game

There is nothing to install or build. Open the game file in a browser:

- Double-click `pacman/draw2.html`, or
- Drag the file into an open browser window.

If you prefer to serve it over HTTP, any static file server works, for example:

```
python -m http.server 8000
```

Then open `http://localhost:8000/pacman/draw2.html`.

### Tests

The project has no automated tests.

---

## Configuration

There is no configuration file. Game values are constants and initial values in the script of
`pacman/draw2.html`. You can change them there:

| Value                      | Where                         | Default            |
|----------------------------|-------------------------------|--------------------|
| Tile size                  | `tileSize`                    | 16 px              |
| Maze layout                | `maze` array                  | 28 x 31 tiles      |
| Pac-Man speed              | `pacman.speed`                | `tileSize / 16`    |
| Starting lives             | `lives` in `initGame()`       | 3                  |
| Ghost list and colour      | `ghosts` in `initGame()`      | one red ghost      |
| Ghost direction change     | `directionChangeCooldown`     | every 30 frames    |
| Invincibility after a hit  | `invincibleTimer`             | 60 frames          |
| Pellet points              | `update()`                    | 10 / 50            |

If you change the tile size or the maze, also change the canvas size (`width` and `height` of
`<canvas id="gameCanvas">`) and the width of `#gameUI` in the CSS.

---

## Known Issues

These points were found by reading the source code:

- **Power pellets have no special effect.** They only give 50 points. Ghosts never turn blue
  and cannot be eaten.
- **Only one ghost.** The original game has four ghosts with different behaviour.
- **The ghost moves randomly.** It does not chase Pac-Man.
- **The win message is only visible for one frame.** `displayWinMessage()` is called once from
  `update()`, but the next frame clears the canvas and does not draw the message again. The
  **Play Again** button still appears.
- **Ghost reset position differs from its start position.** The ghost starts at tile row 16
  but is moved to row 13 after Pac-Man loses a life.
- **Ghost wall check uses Pac-Man's size.** `checkCollision()` always uses `pacman.radius`, so
  the ghost (which is drawn larger) can overlap walls slightly.
- **No turn buffering.** A direction key is ignored if the turn is not possible at that exact
  moment, so the player must press the key at the right position.
- **Speed depends on the frame rate.** Movement is per frame, not per second.
- **Pac-Man always faces right** and its mouth does not open and close.
- **No side tunnels, no fruit, no sound, no levels and no high score.**
- **Unused assets.** The PNG images in the repository root are not loaded by the game.

---

## Possible Improvements

These are ideas only. None of them is implemented:

- Frightened mode after a power pellet, with points for eating ghosts
- Four ghosts with individual chase and scatter behaviour
- Keep the win message on screen until restart
- Remember the next direction and turn as soon as it is possible
- Time-based movement instead of frame-based movement
- Mouth animation and rotation in the movement direction
- Side tunnels, levels, sound effects and a saved high score
- Use the sprite images, or remove them from the repository

---

## License

No license file is included. Private. All rights reserved.

Pac-Man is a trademark of Bandai Namco Entertainment Inc. This project is an independent,
non-commercial recreation for learning purposes and is not affiliated with or endorsed by
Bandai Namco.

Author: Ahmad Kobah
