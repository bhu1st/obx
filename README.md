# Online Baghchal Exchange (OBX) Notation and API Documentation

## Introduction
OBX (Online Baghchal Exchange) is a notation system designed to simplify recording Baghchal games. This system provides a concise way to represent the board state, moves, and outcomes for offline and online gameplay. The OBX format also facilitates software development for Baghchal by offering a standardized way to interact with the game state.

This documentation outlines the OBX notation and provides details about a reusable API for developers and researchers.

Baghchal game start position:

```
  A B C D E
1 T X X X T
2 X X X X X
3 X X X X X
4 X X X X X
5 T X X X T
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

---

## OBX Notation

### **1. Symbols**
- **T**: Tiger.
- **G**: Goat.
- **X**: Empty space.
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
- Use `-` if no last move is recorded.

#### **e. Move Number (Optional)**
- Starts at 0 and increments with each turn.
- Use `-` if not recorded.

### **3. Example OBX Strings**
#### **Example 1**:
- OBX: `GGGGG/GTTTG/GXTXG/GXXXG/GGGGG t c1 mC5 -`
- Meaning:
  - **Board State**: Tigers are in the center, surrounded by goats.
  - **Turn**: Tiger's turn.
  - **Captured Goats**: 1 goat captured.
  - **Last Move**: Goat placed on C5.

#### **Example 2**:
- OBX: `GGGGG/GTTTG/GXTXG/GXXXG/GGGGG g c2 mA3B3 12`
- Meaning:
  - **Board State**: Similar to Example 1.
  - **Turn**: Goat's turn.
  - **Captured Goats**: 2 goats captured.
  - **Last Move**: Tiger moved from A3 to B3.
  - **Move Number**: 12.

---

## API Documentation

### **1. Overview**
The API implementing OBX provides a mechanism to generate the next move based on the current board state represented in OBX format. It supports move validation and board state manipulation.

### **2. Endpoint**
- **URL**: `/nextmove`
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
  "obx": "GGGGG/GTTTG/GXTXG/GXXXG/GGGGG t c1 mC5 -"
}
```

### **4. Response**
#### **Output Fields**
| Field        | Type   | Description                                 |
|--------------|--------|---------------------------------------------|
| `next_move`  | string | Suggested move in `mPN` or `mN` format.    |
| `updated_obx`| string | Updated board state after the suggested move. |

#### **Example Response**
```json
{
  "next_move": "mC5C4",
  "updated_obx": "GGGGG/GTTTG/GXTXG/GXTXG/GGGGG g c1 mC5C4 -"
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
1. Submit the current board state in OBX format to the `/nextmove` endpoint.
2. Parse the response to retrieve the suggested move and updated OBX string.
3. Apply the move in your game logic or UI.

### **2. Sample Workflow**
1. **Current Board State**:
   ```json
   {
     "obx": "GGGGG/GTTTG/GXTXG/GXXXG/GGGGG t c1 mC5 -"
   }
   ```
2. **API Response**:
   ```json
   {
     "next_move": "mC5C4",
     "updated_obx": "GGGGG/GTTTG/GXTXG/GXTXG/GGGGG g c1 mC5C4 -"
   }
   ```
3. **Update Game**:
   - Move tiger from C5 to C4.
   - Update the game state and turn.

---

### **Projects Supporting OBX**

- Online Baghchal [baghchal.net](https://baghchal.net) 

---


## License
The project is open-sourced under the MIT License.

---

