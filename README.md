# Online Baghchal Exchange (OBX) Notation and API Documentation

Version: **OBX 2.0**   
Date: Jan 3, 2025

## Introduction
OBX (Online Baghchal Exchange) is a notation system designed to simplify recording Baghchal games. This system provides a concise way to represent the board state, moves, and outcomes for offline and online gameplay. The OBX format also facilitates software development for Baghchal by offering a standardized way to interact with the game state.

This documentation outlines the OBX notation and provides details about a reusable API for developers and researchers. It explains how to represent and track the state of a Baghchal game using a standard notation system. It includes details on how to denote the board state, track turns, capture counts, and represent moves for both goats and tigers, along with their respective move numbers.

Baghchal game start position:

```
  A B C D E
1 T X X X T
2 X X X X X
3 X X X X X
4 X X X X X
5 T X X X T
```

```
OBX: TXXXT/XXXXX/XXXXX/XXXXX/TXXXT g @20 c0 - #
```

Baghchal board with a goat at the center position (C3):

```
  A B C D E
1 T X X X T
2 X X X X X
3 X X G X X
4 X X X X X
5 T X X X T
```

```
OBX: TXXXT/XXXXX/XXGXX/XXXXX/TXXXT t @19 c0 mC3 #g1
```

---

## OBX Notation

The notation format used to represent the state of the game is as follows:
```
[Board State] [Turn t,g] @[Remaining Goat Count] c[Captured Goat Count] m[Last Move] #[t,g][Move Number]
```


### **1. Symbols**
- **T**: Tiger.
- **G**: Goat.
- **X**: Empty space (room).
- **A1** to **E5**: Grid positions (Columns A-E, Rows 1-5).

### **2. Components**
#### **a. Board State**
- Each row is represented using `T`, `G`, or `X`.
- Rows are separated by `/`.
- Example:
  - `GGGGG/GTTTG/GXTXG/GXXXG/GGGGG`

#### **b. Turn**
- `g`: Goat's turn.
- `t`: Tiger's turn.

#### **c. Remaining Goats**
- Format: `@N`, where `N` is the number of remaining goats (maximum: 20).

#### **d. Captured Goats**
- Format: `cY`, where `Y` is the number of captured goats (maximum: 5).

#### **e. Last Move**
- **Goat Placement**: `mN` (e.g., `mC1` for placing a goat on C1).
- **Movement**: `mPN` (e.g., `mA1B1` for a piece moving from A1 to B1).
- **Capture** `mA1C1(B1)`: Moving a tiger from A1 to C1 and capturing a goat at B1.
- Use `-` if no last move is recorded.

#### **f. Move Number (Optional)**
- A string starting with `#` followed by the player (`g` for goats, `t` for tigers) and the move number:
   - `#g1`: Goats' 1st move.
   - `#t3`: Tigers' 3rd move.
- Use `-` if not recorded.


### **3. Example Notation**

```
TXXXT/XXXXX/XXXXX/XXXXX/TXXXT g @20 c0 - #
```

This represents the initial state of the game:
- Board State: `TXXXT/XXXXX/XXXXX/XXXXX/TXXXT`
- Turn: `g` (goats' turn)
- Remaining Goats Count: `@20` (all goats are outside the board)  
- Captured Count: `c0` (no goats captured)
- Move: `-` (no move yet)
- Move Number: `#`

## Example Sequence of Moves

1. **Initial State**:
   ```
   TXXXT/XXXXX/XXXXX/XXXXX/TXXXT g @20 c0 - #
   ```

2. **Place a goat at B2**:
   ```
   TXXXT/XXXXX/XXXXX/XXXXX/TXXXT g @20 c0 - #
   TXXXT/XGXXX/XXXXX/XXXXX/TXXXT t @19 c0 mB2 #g1
   ```

3. **Move a tiger from A1 to B2**:
   ```
   TXXXT/XGXXX/XXXXX/XXXXX/TXXXT t @19 c0 mB2 #g1
   TXTXT/TXXXX/XXXXX/XXXXX/TXXXT g @19 c0 mA1B2 #t1
   ```

4. **Place another goat at C2**:
   ```
   TXTXT/TXXXX/XXXXX/XXXXX/TXXXT g @19 c0 mA1B2 #t1
   TXTXT/TGXXX/XXXXX/XXXXX/TXXXT t @18 c0 mC2 #g2
   ```

---

## Programming a Baghchal bot

When developing a Baghchal engine or AI bot, representing the board state efficiently and precomputing legal move tables can significantly improve performance during move generation and positional evaluation.

### **1. Game State Representation**

To track the progress of a Baghchal game in memory, a structured data type (such as an associative array or hash map) is recommended. The board is mapped to a 1-dimensional array of 25 elements, where index `0` corresponds to position **A1** (top-left) and index `24` corresponds to position **E5** (bottom-right).

```php
$gameState = array(
    'board'          => array(0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0), // Indices 0 to 24 (A1 to E5)
    'turn'           => 'g',  // Current player turn: 'g' for Goat, 't' for Tiger
    'remainingGoats' => 20,   // Goats waiting to be placed on the board (max 20)
    'capturedGoats'  => 0,    // Goats captured by tigers (max 5; tigers win at 5)
    'lastMove'       => "-",  // Recorded in OBX format or "fromCoordinate|toCoordinate"
    'moveNumber'     => 0     // Total half-moves played (initial position is move 0)
);
```

**Explanation:**
- **`board`**: A 25-element flat array representing the 5×5 grid. Each cell can store a value representing an empty space, a goat, or a tiger (e.g., `0` for empty `X`, `1` for goat `G`, and `2` for tiger `T`).
- **`turn`**: Indicates whose turn it is to move (`g` for goats, `t` for tigers).
- **`remainingGoats`**: Tracks how many of the 20 goats have not yet been placed on the board during the placement phase.
- **`capturedGoats`**: The number of goats captured by tigers. The game ends in a tiger victory if this reaches 5.
- **`lastMove`**: Stores the previous move executed, useful for move history, UI highlighting, or OBX generation.
- **`moveNumber`**: The ply count (turn number) since the start of the game.

---

### **2. ASCII Board Print from Game State**

When debugging or logging bots, it is very helpful to visualize the 5×5 board along with game statistics in the terminal or console. Below is a PHP function that takes the `$gameState` array and formats an ASCII representation of the board and game status:

```php
/**
 * Prints an ASCII representation of the Baghchal board and game status from the gameState array.
 * @param array $gameState - The current game state array.
 */
function showBoard($gameState) {
    // Map board values (0, 1, 2 or 'X', 'G', 'T') to display characters
    $symbols = array_map(function($cell) {
        if ($cell === 1 || $cell === 'G' || $cell === 'g') return 'G';
        if ($cell === 2 || $cell === 'T' || $cell === 't') return 'T';
        return 'X';
    }, $gameState['board']);

    // Format board string with grid lines and symbols
    $boardAscii = "  A   B   C   D   E
1 {$symbols[0]}   {$symbols[1]}   {$symbols[2]}   {$symbols[3]}   {$symbols[4]}
  | \\ | / | \\ | / |
2 {$symbols[5]}   {$symbols[6]}   {$symbols[7]}   {$symbols[8]}   {$symbols[9]}
  | / | \\ | / | \\ |
3 {$symbols[10]}   {$symbols[11]}   {$symbols[12]}   {$symbols[13]}   {$symbols[14]}
  | \\ | / | \\ | / |
4 {$symbols[15]}   {$symbols[16]}   {$symbols[17]}   {$symbols[18]}   {$symbols[19]}
  | / | \\ | / | \\ |
5 {$symbols[20]}   {$symbols[21]}   {$symbols[22]}   {$symbols[23]}   {$symbols[24]}\n";

    echo $boardAscii;
    echo "Turn: " . (strtolower($gameState['turn']) === 'g' ? 'Goat' : 'Tiger') . "\n";
    echo "Remaining Goats: " . $gameState['remainingGoats'] . "\n";
    echo "Captured Goats: " . $gameState['capturedGoats'] . "\n\n";
}
```

### **3. Legal Moves Validation**

To generate valid moves rapidly without recalculating board geometry on every turn, bots rely on precomputed adjacency lookup tables. Using the 0–24 indexing scheme, these tables map every board index to its reachable neighbors.

#### **a. Normal Adjacency Lookup (`$legalMovesConnectionMatrix`)**
Maps each starting position (0 to 24) to an array of adjacent square indices reachable by a standard 1-step move along board lines.

```php
$legalMovesConnectionMatrix = [
    0 => [1, 5, 6], 1 => [2, 0, 6], 2 => [3, 1, 7, 6, 8], 3 => [4, 2, 8], 4 => [3, 9, 8],
    5 => [6, 10, 0], 6 => [7, 5, 11, 1, 10, 2, 12, 0], 7 => [8, 6, 12, 2], 8 => [9, 7, 13, 3, 12, 4, 14, 2], 9 => [8, 14, 4],
    10 => [11, 15, 5, 6, 16], 11 => [12, 10, 16, 6], 12 => [13, 11, 17, 7, 16, 8, 18, 6], 13 => [14, 12, 18, 8],
    14 => [13, 19, 9, 18, 8],
    15 => [16, 20, 10], 16 => [17, 15, 21, 11, 20, 12, 22, 10], 17 => [18, 16, 22, 12],
    18 => [19, 17, 23, 13, 22, 14, 24, 12], 19 => [18, 24, 14],
    20 => [21, 15, 16], 21 => [22, 20, 16], 22 => [23, 21, 17, 18, 16], 23 => [24, 22, 18], 24 => [23, 19, 18]
];
```
- **Explanation**: During move generation for goats or tigers, the engine looks up `$legalMovesConnectionMatrix[$currentIndex]`. If the target index on the board contains an empty space (`X`), it is a valid movement destination.

#### **b. Capture Jump Adjacency Lookup (`$captureMovesConnectionMatrix`)**
Maps each starting position (0 to 24) to an array of destination indices reachable by a 2-step tiger capture jump along valid grid lines.

```php
$captureMovesConnectionMatrix = [
    0 => [2, 10, 12], 1 => [3, 11], 2 => [4, 0, 12, 10, 14], 3 => [1, 13], 4 => [2, 14, 12],
    5 => [7, 15], 6 => [8, 16, 18], 7 => [9, 5, 17], 8 => [6, 18, 16], 9 => [7, 19],
    10 => [12, 20, 0, 2, 22], 11 => [13, 21, 1], 12 => [14, 10, 22, 2, 20, 4, 24, 0],
    13 => [11, 23, 3], 14 => [12, 24, 4, 22, 2],
    15 => [17, 5], 16 => [18, 6, 8], 17 => [19, 15, 7], 18 => [16, 8, 6], 19 => [17, 9],
    20 => [22, 10, 12], 21 => [23, 11], 22 => [24, 20, 12, 14, 10], 23 => [21, 13], 24 => [22, 14, 12]
];
```
- **Explanation**: When generating capture moves for a tiger at index `$fromIndex`, the bot iterates over destination indices in `$captureMovesConnectionMatrix[$fromIndex]`. A capture is valid if and only if:
  1. The destination square `$toIndex` is empty (`X`).
  2. The intermediate square, calculated as `$midpoint = ($fromIndex + $toIndex) / 2`, is currently occupied by a goat (`G`).

---

### **4. Evaluation Matrices**

Positional evaluation tables help AI algorithms (such as Minimax or Alpha-Beta pruning) assess board control and piece mobility. In Baghchal, piece mobility depends on the number of grid lines connected to an intersection.

#### **a. OBX Normal Moves Matrix (`$obxMovesMatrix`)**
This matrix tabulates the number of adjacent connected intersections for every position on the 5×5 board. Squares with diagonal markings (such as the center C3 or inner corners B2, D2, B4, D4) offer significantly higher mobility.

```php
$obxMovesMatrix = [
    [ 3, 3, 5, 3, 3 ],
    [ 3, 8, 4, 8, 3 ],
    [ 5, 4, 8, 4, 5 ],
    [ 3, 8, 4, 8, 3 ],
    [ 3, 3, 5, 3, 3 ],
];
```
- **Explanation**: A piece on **C3** (row 2, col 2) can move in 8 different directions, whereas a piece on a corner like **A1** (row 0, col 0) only has 3 possible moves. Bots use these weights to favor placing or moving pieces toward high-mobility squares.

#### **b. OBX Tiger Capture Moves Matrix (`$obxTigerCaptureMovesMatrix`)**
This matrix tabulates the number of potential 2-step jump landing squares available from each board position along valid grid lines.

```php
$obxTigerCaptureMovesMatrix = [
    [ 3, 2, 5, 2, 3 ],
    [ 2, 3, 3, 3, 2 ],
    [ 5, 3, 8, 3, 5 ],
    [ 2, 3, 3, 3, 2 ],
    [ 3, 2, 5, 2, 3 ],
];
```
- **Explanation**: A tiger placed on a square with a high capture move count (such as center **C3** with 8 potential jump paths) poses a much greater threat to goats. This matrix helps evaluate the attacking potential of tigers.

#### **c. OBX Tiger Total Moves Matrix (`$obxTigerMovesMatrix`)**
This matrix is the element-wise sum of the normal moves matrix and the capture moves matrix. It provides a combined score for evaluating tiger mobility and tactical strength.

```php
$obxTigerMovesMatrix = [
    [  6,  5, 10,  5,  6 ],
    [  5, 11,  7, 11,  5 ],
    [ 10,  7, 16,  7, 10 ],
    [  5, 11,  7, 11,  5 ],
    [  6,  5, 10,  5,  6 ],
];
```
- **Explanation**: By combining both standard movement paths and capture paths, this matrix allows the evaluation function to quickly quantify how dominant a tiger's position is on any given square. Notice how the center position (**C3**, score `16.0`) is the most powerful location for a tiger.

---

## API Documentation

### **1. Overview**
The API implementing OBX provides a mechanism to generate the next move based on the current board state represented in OBX format. It supports move validation and board state manipulation.

### **2. Endpoint**
- **URL**: `/obx`
- **Method**: GET
- **Input Format**: JSON
- **Output Format**: JSON

### **3. Request**
#### **Input Parameters**
| Field  | Type   | Description                             |
|--------|--------|-----------------------------------------|
| `obx`  | string | The current board state in OBX format.  |

#### **Example Request**
```json
{
  "obx": "TGXXT/XXXXX/XXXXX/XXXXX/TXXXT t @19 c0 mB1 #g1"
}
```

### **4. Response**
#### **Output Fields**
| Field        | Type   | Description                                 |
|--------------|--------|---------------------------------------------|
| `input`  | string | Current board state OBX.    |
| `obx`| string | Updated board state OBX after the suggested move. |
| `move`| string | Next move. |

#### **Example Response**
```json
{
  "input": "TGXXT/XXXXX/XXXXX/XXXXX/TXXXT t @19 c0 mB1 #g1"
  "obx": "XXTXT/XXXXX/XXGXX/XXXXX/TXXXX g @19 c1 mA1C3(B1) #t1",
  "move" : "mA1C3(B1)"
}
```

### **5. Error Handling**
| Error Code | Message                     | Description                           |
|------------|-----------------------------|---------------------------------------|
| 400        | "Invalid OBX format."      | The input OBX string is malformed.    |
| 422        | "Illegal move detected."   | The suggested move violates game rules. |

---

## Integration Guide

### **1. Step-by-Step Usage**
1. Submit the current board state in OBX format to the `/obx` endpoint.
2. Parse the response to retrieve the suggested move and updated OBX string.
3. Apply the move in your game logic or UI.

### **2. Sample Workflow**
1. **Current Board State**:
   ```json
   {
     "obx": "TGXXT/XXXXX/XXXXX/XXXXX/TXXXT t @19 c0 mB1 #g1"
   }
   ```
2. **API Response with OBX of the next move**:
   ```json
   {
     "input": "TGXXT/XXXXX/XXXXX/XXXXX/TXXXT t @19 c0 mB1 #g1"
	 "obx": "XXTXT/XXXXX/XXGXX/XXXXX/TXXXX g @19 c1 mA1C3(B1) #t1",
	 "move" : "mA1C3(B1)"
   }
   ```
3. **Update Game**:
   - Move tiger from A1 to C3, remove goat at B1 which is captured.
   - Update the game state and turn.

---

### **Projects Using OBX**

- Online Baghchal [www.baghchal.net](https://baghchal.net)
- OBX Draft by Bhupal Sapkota [v1.0](https://bhupalsapkota.com/baghchal/baghchal-obx-notation.pdf)

---


## License
The project is open-sourced under the MIT License.

---

