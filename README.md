# Online Baghchal Exchange (OBX) Notation and API Documentation

Version: OBX 2.0 

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
OBX: TXXXT/XXXXX/XXXXX/XXXXX/TXXXT g c0 - #
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
OBX: TXXXT/XXXXX/XXGXX/XXXXX/TXXXT t c0 mC3 #g1
```

---

## OBX Notation

The notation format used to represent the state of the game is as follows:
```
[Board State] [Turn t,g] c[Captured Goat Count] n[Last Move] #[t,g][Move Number]
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

#### **c. Captured Goats**
- Format: `cY`, where `Y` is the number of captured goats (maximum: 5).

#### **d. Last Move**
- **Goat Placement**: `mN` (e.g., `mC1` for placing a goat on C1).
- **Movement**: `mPN` (e.g., `mA1B1` for a piece moving from A1 to B1).
- **Capture** `mA1C1(B1)`: Moving a tiger from A1 to C1 and capturing a goat at B1.
- Use `-` if no last move is recorded.

#### **e. Move Number (Optional)**
- A string starting with `#` followed by the player (`g` for goats, `t` for tigers) and the move number:
   - `#g1`: Goats' 1st move.
   - `#t3`: Tigers' 3rd move.
- Use `-` if not recorded.


### **3. Example Notation**

```
TXXXT/XXXXX/XXXXX/XXXXX/TXXXT g c0 - #
```

This represents the initial state of the game:
- Board State: `TXXXT/XXXXX/XXXXX/XXXXX/TXXXT`
- Turn: `g` (goats' turn)
- Captured Count: `c0` (no goats captured)
- Move: `-` (no move yet)
- Move Number: `#`

## Example Sequence of Moves

1. **Initial State**:
   ```
   TXXXT/XXXXX/XXXXX/XXXXX/TXXXT g c0 - #
   ```

2. **Place a goat at B2**:
   ```
   TXXXT/XXXXX/XXXXX/XXXXX/TXXXT g c0 - #
   TXXXT/XGXXX/XXXXX/XXXXX/TXXXT t c0 mB2 #g1
   ```

3. **Move a tiger from A1 to B2**:
   ```
   TXXXT/XGXXX/XXXXX/XXXXX/TXXXT t c0 mB2 #g1
   TXTXT/TXXXX/XXXXX/XXXXX/TXXXT g c0 mA1B2 #t1
   ```

4. **Place another goat at C2**:
   ```
   TXTXT/TXXXX/XXXXX/XXXXX/TXXXT g c0 mA1B2 #t1
   TXTXT/TGXXX/XXXXX/XXXXX/TXXXT t c0 mC2 #g2
   ```

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
  "obx": "TGXXT/XXXXX/XXXXX/XXXXX/TXXXT t c0 mB1 #g1"
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
  "input": "TGXXT/XXXXX/XXXXX/XXXXX/TXXXT t c0 mB1 #g1"
  "obx": "XXTXT/XXXXX/XXGXX/XXXXX/TXXXX g c1 mA1C3(B1) #t1",
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
     "obx": "TGXXT/XXXXX/XXXXX/XXXXX/TXXXT t c0 mB1 #g1"
   }
   ```
2. **API Response with OBX of the next move**:
   ```json
   {
     "input": "TGXXT/XXXXX/XXXXX/XXXXX/TXXXT t c0 mB1 #g1"
	 "obx": "XXTXT/XXXXX/XXGXX/XXXXX/TXXXX g c1 mA1C3(B1) #t1",
	 "move" : "mA1C3(B1)"
   }
   ```
3. **Update Game**:
   - Move tiger from A1 to C3, remove goat at B1 which is captured.
   - Update the game state and turn.

---

### **Projects Using OBX**

- Online Baghchal [baghchal.net](https://baghchal.net) 

---


## License
The project is open-sourced under the MIT License.

---

