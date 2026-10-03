# SchGen 训练样本预览（前 2 条）

> 从 `SchGen_dataset.jsonl` 提取，便于人工查看输入与标签结构。

---

## 样本 1

### Meta（不参与训练，仅溯源）

- **module**: `SparkFun_Micro_Pressure_Sensor_BMP581_Qwiic-Schematic`
- **schematic**: `sch_1_0.kicad_sch`
- **schematic_path**: `/home/anonymous/workspace/llm4circuit/dataset/SparkFun_Micro_Pressure_Sensor_BMP581_Qwiic-Schematic/sch_1_0.kicad_sch`
- **code_path**: `/home/anonymous/workspace/llm4circuit/dataset/SparkFun_Micro_Pressure_Sensor_BMP581_Qwiic-Schematic/sch_1_0_int.py`
- **thinking_model**: `gpt-oss-20b`
- **style**: `concise`

### 消息结构概览

| 序号 | role | 字段 | 字符数 |
|---|---|---|---:|
| 1 | `system` | content + - | 4017 |
| 2 | `user` | content + - | 169 |
| 3 | `assistant` | content + thinking(4786) | 1135 |

### Message 1: `system`

#### 输入内容：`content`

```text

You need to complete a user request by outputting executable Python code that generates a KiCad schematic file corresponding to the request. Generate the Python code to edit the schematic file using the KiCad Python API. YOU MUST MAKE SURE THE FINAL CODE ALIGN WITH THE THINKING PROCESS.
###
You have the following functions available to you and can create new functions based on them:
- def add_schematic_symbol(symbol_lib="RF_Module", symbol_name="ESP-WROOM-02", pos_x=150, pos_y=100, reference="U1", value="", rotation=0, mirror:str =None): Add any component symbol from a KiCad library into your schematic. The symbol_lib and symbol_name specify the library and symbol name of the component to add. The pos_x and pos_y specify the position of the center of the symbol in mm. The reference is the unique identifier for the component, e.g., "U1", "R1", "C1". The value is the value of the component, e.g., "10K", "100nF", "ESP32". The rotation is the angle in degrees to rotate the symbol, e.g., 0, 90, 180, 270. The mirror can be "X" or "Y" to flip the symbol according to X or Y axis. If mirror is None, no mirroring is applied.
NOTE:If there is no related information for value, you need to set a value based on your knowledge about what the schematic design. For example, for a pull up resistor, you can set a value of "10K", for a decoupling capacitor, you can set a value of "100nF". value string should NOT include space or `()` or use `TBD`, for example, `12pF (NC)` should be set as "12pF", and `10K (pull up)` should be set as "10K". 

- def get_pin_location(symbol_ref: str, pin_name: str): Get the location of a pin in the schematic. Args: symbol_ref (str): The reference of the symbol or label. pin_name (str): The name or id of the pin, for power symbol or label, this should be "1".

- def add_label(label_pos: list, label_text: str, label_ref: str, label_type: str = "input", text_orient: str ="left"): Add a label to the schematic. The label_pos is a list of two floats [x, y] specifying the position of the label's pin in mm. The label_text is the text of the label, e.g., "IO1", "SDA", "RXD". The label_ref is the unique identifier for the label, e.g., "IO1_0", "SDA_0", "SDA_1". The label_type can be "input", "output", "bidirectional" to specify the type of the label. The text_orient can be "left", "right", "top", or "bottom" to specify the orientation of the text relative to the label pin position.

- def connect_pins(sym_a: str, pin_a: str, sym_b: str, pin_b: str). Create a connection between pin_a of symb_a and pin_b of sym_b. The sym_a and sym_b are the references of the symbols or labels to connect. The pin_a and pin_b are the names or ids of the pins to connect. Note: We treat labels as a kind of symbols, identified by their unique reference. For power symbols and labels, they only have one pin, so pin_a or pin_b should be "1".

- def write_out_all_wires(). Write out all wires of connections in the schematic.

###
# # Example code that uses these functions:
# ```
# {example_code_str}
# ```
###
   
NOTE:
1. You should mind the spatial placement of the components. Make sure they are at reasonable positions and ample spacing so that they do not overlap with each other!
2. The size of the schematic is 210 by 297 mm, size of a A4 paper. It uses a X-Y axes based coordinate system. The origin is [0,0] at bottom left corner of the sheet. X axis is horizontal, and Y axis is vertical. To keep the circuit in the center region.
3. You should check the symbol context to see the spatial information, including the size, orientation, pin locations. The center of the symbol is at (0, 0) and the pin locations are relative to the center of the symbol. X axis is horizontal, and Y axis is vertical. For symbol definition, the Y axis points upward, that means higher Y position means higher position, same direction as the schematic coordinate system.
4. The code should be valid Python code with correct indentation and syntax. For example, comment should start with #. 
        
```

### Message 2: `user`

#### 输入内容：`content`

```text
I want a PCB that provides a test point on the SCK/SCL (clock) line for probing an I²C/SPI interface.
You may need the following symbol information from KiCad library: .
```

### Message 3: `assistant`

#### 训练标签 A：`thinking`（CoT 推理过程）

```text
**1. Overall goal**

The user wants a *probe‑point* that can be used to monitor the clock line of an I²C or SPI bus.  
In a schematic this is usually represented by a **TestPoint** symbol that is wired to the bus net.  
The script therefore only needs to:

1. Place a TestPoint component on the board.  
2. Attach a human‑readable label (“SCK/SCL”) to the point so the designer knows which net it belongs to.  
3. Connect the label to the pin of the TestPoint so the net name is carried into the PCB layout.  

No other components are required because the test point is just a passive probe.

---

**2. Physical placement of the TestPoint**

- **Library & symbol** – `Connector::TestPoint`.  
  The library is chosen because the symbol is a simple probe with a single pin.  
- **Reference** – `"TP1"`.  
  A unique identifier that will be used for all subsequent connections.  
- **Position** – `pos_x=150.0, pos_y=110.0`.  
  The centre of an A4 sheet is roughly (105 mm, 148 mm).  
  Placing the point at (150 mm, 110 mm) keeps it comfortably in the middle of the layout while leaving room for other components.  
- **Rotation** – `270°`.  
  Rotating the symbol so the pin points downwards makes it easier to route a vertical trace from the test point to the rest of the board.  
- **Mirror** – `None`.  
  No mirroring is needed; the symbol already faces the correct direction after rotation.  
- **Value** – omitted (`value=""`).  
  Test points do not carry a value; the label will describe the net.

---

**3. Obtaining the pin location**

`get_pin_location(symbol_ref="TP1", pin_name="TestPoint")`  
- `symbol_ref="TP1"` tells the function which component to query.  
- `pin_name="TestPoint"` is the name of the pin in the symbol’s pin list (often the single pin is called “TestPoint” or “1”).  
The function returns the absolute X,Y coordinates of that pin, which are needed to place the label and to connect it.

---

**4. Adding the label**

- **Label position** – `[x_TP1_1 + (-19), y_TP1_1 + 0]`.  
  The X offset of –19 mm moves the label to the left of the pin.  
  This keeps the label away from the pin so the net name is readable and the wire can be drawn cleanly.  
- **Label text** – `"SCK/SCL"`.  
  The net name that the designer will see on the PCB.  
- **Label reference** – `"SCK/SCL_0"`.  
  A unique ID for the label so it can be referenced in the connection command.  
- **Label type** – `"input"`.  
  Indicates that this is a net coming **into** the test point (the pin is an output from the bus).  
- **Text orientation** – `"left"`.  
  With the label positioned left of the pin, the text will read from right‑to‑left, keeping the label’s orientation consistent with the pin direction.

---

**5. Connecting the label to the pin**

`connect_pins("SCK/SCL_0", "1", "TP1", "TestPoint")`  
- **sym_a** – `"SCK/SCL_0"` (the label).  
- **pin_a** – `"1"` (labels are treated as 1‑pin symbols).  
- **sym_b** – `"TP1"` (the test point).  
- **pin_b** – `"TestPoint"` (the pin on the test point).  

This creates a wire that carries the net name “SCK/SCL” from the bus to the test point pin.  
Because the test point pin is now part of the same net, the PCB layout will automatically place a trace from the test point to the clock line.

---

**6. Final wire output**

`write_out_all_wires()`  
- After all `connect_pins` calls, this function writes the collected wire data into the schematic file.  
- It ensures that the net graph is updated and that the label is correctly linked to the pin in the final KiCad project.

---

**7. Why the chosen values make the schematic readable**

- **Rotation 270°** aligns the pin vertically, so the trace to the clock net can run straight down, avoiding unnecessary bends.  
- **Offset –19 mm** for the label places the text far enough from the pin to avoid overlap, yet close enough to be clearly associated with the test point.  
- **Label orientation “left”** keeps the text aligned with the pin direction, improving legibility when the schematic is printed or viewed at a glance.  
- **Unique references** (`TP1`, `SCK/SCL_0`) prevent naming conflicts and make the script easily extendable if more test points are added later.

---

**8. Summary of the script logic**

1. **Add TestPoint symbol** at a central, well‑spaced location, rotated for optimal routing.  
2. **Query the pin location** to know where to place the label and to connect it.  
3. **Place a label** to the left of the pin, naming the net “SCK/SCL”.  
4. **Connect the label to the pin** so the net name propagates to the PCB.  
5. **Write out all wires** so the schematic reflects the connections.

This minimal, well‑commented script fulfills the user’s request: it creates a clear, routable test point on the clock line of an I²C/SPI interface.
```

#### 训练标签 B：`content`（Python 原理图代码）

```python
# Auto-generated schematic symbols
import sys
import os

# Get project path and import kicad schematic interface
PROJECT_PATH = os.environ['PROJECT_PATH']
sys.path.append(PROJECT_PATH)
from modules.kicad_sch_interface import *

### Placing center symbol 1 : Connector:TestPoint###

center_x_1, center_y_1 = 150.0, 110.0

add_schematic_symbol(symbol_lib="Connector", symbol_name="TestPoint", pos_x=center_x_1, pos_y=center_y_1, reference="TP1", value="TestPoint", rotation=270, mirror="None")

### Placing other symbols in the Schematic with respect to the center symbol 1###


### Placing all global labels in the Schematic and connect them to the neighbor pin ###

# Add label SCK/SCL next to TP1 pin TestPoint 
x_TP1_1, y_TP1_1 = get_pin_location(symbol_ref="TP1", pin_name="TestPoint")
add_label(label_pos=[x_TP1_1+(-19), y_TP1_1+(0)], label_text="SCK/SCL", label_ref="SCK/SCL_0", label_type="input", text_orient="left")
# Connecting Label SCK/SCL label_id:0 to TP1 pin TestPoint (Pin ID 1 -- Name TestPoint)
connect_pins("SCK/SCL_0", "1", "TP1", "TestPoint")


### Connecting all wires in the Schematic ###


write_out_all_wires()

```

---

## 样本 2

### Meta（不参与训练，仅溯源）

- **module**: `22032-SparkFun_Digi_XBee_Regulated_Qwiic-Schematic`
- **schematic**: `sch_4_0.kicad_sch`
- **schematic_path**: `/home/anonymous/workspace/llm4circuit/dataset/22032-SparkFun_Digi_XBee_Regulated_Qwiic-Schematic/sch_4_0.kicad_sch`
- **code_path**: `/home/anonymous/workspace/llm4circuit/dataset/22032-SparkFun_Digi_XBee_Regulated_Qwiic-Schematic/sch_4_0_int.py`
- **thinking_model**: `gpt-oss-120b`
- **style**: `concise`

### 消息结构概览

| 序号 | role | 字段 | 字符数 |
|---|---|---|---:|
| 1 | `system` | content + - | 4017 |
| 2 | `user` | content + - | 157 |
| 3 | `assistant` | content + thinking(6657) | 2175 |

### Message 1: `system`

#### 输入内容：`content`

```text

You need to complete a user request by outputting executable Python code that generates a KiCad schematic file corresponding to the request. Generate the Python code to edit the schematic file using the KiCad Python API. YOU MUST MAKE SURE THE FINAL CODE ALIGN WITH THE THINKING PROCESS.
###
You have the following functions available to you and can create new functions based on them:
- def add_schematic_symbol(symbol_lib="RF_Module", symbol_name="ESP-WROOM-02", pos_x=150, pos_y=100, reference="U1", value="", rotation=0, mirror:str =None): Add any component symbol from a KiCad library into your schematic. The symbol_lib and symbol_name specify the library and symbol name of the component to add. The pos_x and pos_y specify the position of the center of the symbol in mm. The reference is the unique identifier for the component, e.g., "U1", "R1", "C1". The value is the value of the component, e.g., "10K", "100nF", "ESP32". The rotation is the angle in degrees to rotate the symbol, e.g., 0, 90, 180, 270. The mirror can be "X" or "Y" to flip the symbol according to X or Y axis. If mirror is None, no mirroring is applied.
NOTE:If there is no related information for value, you need to set a value based on your knowledge about what the schematic design. For example, for a pull up resistor, you can set a value of "10K", for a decoupling capacitor, you can set a value of "100nF". value string should NOT include space or `()` or use `TBD`, for example, `12pF (NC)` should be set as "12pF", and `10K (pull up)` should be set as "10K". 

- def get_pin_location(symbol_ref: str, pin_name: str): Get the location of a pin in the schematic. Args: symbol_ref (str): The reference of the symbol or label. pin_name (str): The name or id of the pin, for power symbol or label, this should be "1".

- def add_label(label_pos: list, label_text: str, label_ref: str, label_type: str = "input", text_orient: str ="left"): Add a label to the schematic. The label_pos is a list of two floats [x, y] specifying the position of the label's pin in mm. The label_text is the text of the label, e.g., "IO1", "SDA", "RXD". The label_ref is the unique identifier for the label, e.g., "IO1_0", "SDA_0", "SDA_1". The label_type can be "input", "output", "bidirectional" to specify the type of the label. The text_orient can be "left", "right", "top", or "bottom" to specify the orientation of the text relative to the label pin position.

- def connect_pins(sym_a: str, pin_a: str, sym_b: str, pin_b: str). Create a connection between pin_a of symb_a and pin_b of sym_b. The sym_a and sym_b are the references of the symbols or labels to connect. The pin_a and pin_b are the names or ids of the pins to connect. Note: We treat labels as a kind of symbols, identified by their unique reference. For power symbols and labels, they only have one pin, so pin_a or pin_b should be "1".

- def write_out_all_wires(). Write out all wires of connections in the schematic.

###
# # Example code that uses these functions:
# ```
# {example_code_str}
# ```
###
   
NOTE:
1. You should mind the spatial placement of the components. Make sure they are at reasonable positions and ample spacing so that they do not overlap with each other!
2. The size of the schematic is 210 by 297 mm, size of a A4 paper. It uses a X-Y axes based coordinate system. The origin is [0,0] at bottom left corner of the sheet. X axis is horizontal, and Y axis is vertical. To keep the circuit in the center region.
3. You should check the symbol context to see the spatial information, including the size, orientation, pin locations. The center of the symbol is at (0, 0) and the pin locations are relative to the center of the symbol. X axis is horizontal, and Y axis is vertical. For symbol definition, the Y axis points upward, that means higher Y position means higher position, same direction as the schematic coordinate system.
4. The code should be valid Python code with correct indentation and syntax. For example, comment should start with #. 
        
```

### Message 2: `user`

#### 输入内容：`content`

```text
I want a 4‑pin Qwiic I²C connector providing +3.3V and GND with SDA on D11 and SCL on D1.
You may need the following symbol information from KiCad library: .
```

### Message 3: `assistant`

#### 训练标签 A：`thinking`（CoT 推理过程）

```text
<reasoning>
We need to produce thinking process explanation, not the code. Summarize reasoning: request is a 4-pin Qwiic I2C connector with +3.3V, GND, SDA on D11, SCL on D1. Need to place connector, power symbols, labels, connections. Explain placement, offsets etc. Must be under 1000 words.

We'll describe overall request, decide connector orientation, choose positions, add power symbols, add labels, connect pins accordingly. Explain each function call arguments. Also discuss why mirror none, rotation 0, offset values etc.

Proceed.
</reasoning>

**Overall goal**

The user wants a simple I²C “Qwiic” connector that brings four signals onto the schematic:

1. **Pin 1 – SCL** – to be routed to the MCU pin D1.  
2. **Pin 2 – SDA** – to be routed to the MCU pin D11.  
3. **Pin 3 – +3.3 V** – a power rail that supplies the connector.  
4. **Pin 4 – GND** – the ground rail.

To satisfy this we must place a generic 4‑pin header, add the two power symbols (+3.3 V and GND), create human‑readable net‑labels for the two data lines, and finally wire the appropriate pins together.

---

### 1. Choosing a location for the connector

The sheet size (A4) is 210 mm × 297 mm, with the origin at the lower‑left corner.  
To keep the design centred and give enough room for the surrounding symbols, I selected a centre point around **(150 mm, 110 mm)**. This is well inside the printable area and leaves space on the right side for the power symbols and on the left side for the labels.

```python
center_x_1, center_y_1 = 150.0, 110.0
```

The connector is added with:

- **symbol_lib = "Connector_Generic"** – the library that contains generic pin‑header symbols.  
- **symbol_name = "Conn_01x04"** – the 4‑pin vertical header.  
- **reference = "J1"** – a conventional reference for a connector.  
- **value = "QWIIC"** – marks the connector as a Qwiic I²C socket.  
- **rotation = 0** – the default orientation (pins numbered from top to bottom).  
- **mirror = "y"** – I mirrored the part on the Y‑axis so that the pin numbers increase from bottom to top (matching the usual Qwiic pin order) and making the wiring to the labels on the right side more natural.

```python
add_schematic_symbol(..., reference="J1", value="QWIIC", rotation=0, mirror="y")
```

---

### 2. Adding the power symbols

The power pins of the connector (Pin 3 and Pin 4) need to be tied to the appropriate rails.  
I placed the **+3.3 V** symbol a short distance to the right‑upper side of the connector and the **GND** symbol a short distance to the right‑lower side. Offsets of **+10 mm** in X ensure no overlap, while **+7 mm** (for +3.3 V) and **‑8 mm** (for GND) in Y give a clear vertical separation.

```python
add_schematic_symbol(symbol_lib="power", symbol_name="+3.3V",
    pos_x=center_x_1 + 10, pos_y=center_y_1 + 7,
    reference="#PWR_3V1", value="+3.3V", rotation=0, mirror="None")
add_schematic_symbol(symbol_lib="power", symbol_name="GND",
    pos_x=center_x_1 + 10, pos_y=center_y_1 - 8,
    reference="#PWR_GND1", value="GND", rotation=0, mirror="None")
```

The references are prefixed with `#PWR_` to indicate they are global power symbols, which also makes the later net‑connections clearer.

---

### 3. Creating and positioning net‑labels for SDA and SCL

For readability, explicit net‑labels are placed next to the relevant connector pins.  
The workflow:

1. **Query the pin location** of the connector pins using `get_pin_location`.  
   - Pin 2 (`Pin_2`) is the SDA line.  
   - Pin 1 (`Pin_1`) is the SCL line.  

2. **Add a label** a few millimetres to the right of each pin (offset **+8 mm** in X, 0 mm in Y). This keeps the label on the same horizontal line as the pin while leaving enough space for the wire that will later run to the MCU.

3. **Connect the label to the pin** with `connect_pins`. The label is treated as a one‑pin symbol, therefore we always use pin “1” on the label side.

```python
# SDA label
x_J1_2, y_J1_2 = get_pin_location("J1", "Pin_2")
add_label([x_J1_2+8, y_J1_2], "D11/SDA", "D11/SDA_0", "input", "right")
connect_pins("D11/SDA_0", "1", "J1", "Pin_2")

# SCL label
x_J1_1, y_J1_1 = get_pin_location("J1", "Pin_1")
add_label([x_J1_1+8, y_J1_1], "D1/SCL", "D1/SCL_0", "input", "right")
connect_pins("D1/SCL_0", "1", "J1", "Pin_1")
```

The `label_type` is set to `"input"` because, from the MCU’s perspective, these pins are inputs to the external device (the Qwiic bus). The `text_orient="right"` aligns the text to the right of the label’s pin, matching the direction of the offset.

---

### 4. Wiring the power pins

The remaining two pins of the connector are purely power rails:

- **Pin 3 (`Pin_3`) → +3.3 V rail**  
- **Pin 4 (`Pin_4`) → GND rail**

These connections are made directly with `connect_pins`. No intermediate labels are needed because the power symbols already carry the net names.

```python
connect_pins("J1", "Pin_3", "#PWR_3V1", "1")   # +3.3 V
connect_pins("J1", "Pin_4", "#PWR_GND1", "1")  # GND
```

The pin numbers on the power symbols are always “1” (they are single‑pin symbols). Using the connector reference “J1” makes the intent explicit and keeps the netlist tidy.

---

### 5. Finalising the schematic

After all connections are described, `write_out_all_wires()` is called to commit the wire objects to the schematic file. This function aggregates every `connect_pins` call and draws the corresponding nets.

```python
write_out_all_wires()
```

---

### Summary of design decisions

| Item | Reasoning |
|------|------------|
| **Connector placement** (150 mm, 110 mm) | Central, leaves room for power symbols and labels on the right side. |
| **Mirror = "y"** | Flips the header so pins are numbered from bottom to top, matching the usual Qwiic pin order and making the right‑hand side label placement natural. |
| **Power symbol offsets (+10 mm X, ±7/‑8 mm Y)** | Guarantees no overlap with the connector while keeping the power rails visually close to the appropriate pins. |
| **Label offsets (+8 mm X)** | Places the net name a short distance to the right of each pin, keeping the schematic tidy and avoiding line crossings. |
| **Label orientation “right”** | Aligns text outward from the connector, improving readability. |
| **`connect_pins` for power** | Directly ties the connector’s power pins to the global rails; no extra labels are required. |
| **`connect_pins` for data lines** | Connects each label to its respective connector pin, establishing the SDA and SCL nets that will later be linked to the MCU pins D11 and D1. |

With these steps the generated Python script creates a clean, functional Qwiic I²C connector schematic that meets the user’s specification.
```

#### 训练标签 B：`content`（Python 原理图代码）

```python
# Auto-generated schematic symbols
import sys
import os

# Get project path and import kicad schematic interface
PROJECT_PATH = os.environ['PROJECT_PATH']
sys.path.append(PROJECT_PATH)
from modules.kicad_sch_interface import *

### Placing center symbol 1 : Connector_Generic:Conn_01x04###

center_x_1, center_y_1 = 150.0, 110.0

add_schematic_symbol(symbol_lib="Connector_Generic", symbol_name="Conn_01x04", pos_x=center_x_1, pos_y=center_y_1, reference="J1", value="QWIIC", rotation=0, mirror="y")

### Placing other symbols in the Schematic with respect to the center symbol 1###

add_schematic_symbol(symbol_lib="power", symbol_name="+3.3V", pos_x=center_x_1 + (10), pos_y=center_y_1 + (7), reference="#PWR_3V1", value="+3.3V", rotation=0, mirror="None")
add_schematic_symbol(symbol_lib="power", symbol_name="GND", pos_x=center_x_1 + (10), pos_y=center_y_1 + (-8), reference="#PWR_GND1", value="GND", rotation=0, mirror="None")

### Placing all global labels in the Schematic and connect them to the neighbor pin ###

# Add label D11{slash}SDA next to J1 pin Pin_2 
x_J1_2, y_J1_2 = get_pin_location(symbol_ref="J1", pin_name="Pin_2")
add_label(label_pos=[x_J1_2+(8), y_J1_2+(0)], label_text="D11{slash}SDA", label_ref="D11{slash}SDA_0", label_type="input", text_orient="right")
# Connecting Label D11{slash}SDA label_id:0 to J1 pin Pin_2 (Pin ID 2 -- Name Pin_2)
connect_pins("D11{slash}SDA_0", "1", "J1", "Pin_2")

# Add label D1{slash}SCL next to J1 pin Pin_1 
x_J1_1, y_J1_1 = get_pin_location(symbol_ref="J1", pin_name="Pin_1")
add_label(label_pos=[x_J1_1+(8), y_J1_1+(0)], label_text="D1{slash}SCL", label_ref="D1{slash}SCL_0", label_type="input", text_orient="right")
# Connecting Label D1{slash}SCL label_id:0 to J1 pin Pin_1 (Pin ID 1 -- Name Pin_1)
connect_pins("D1{slash}SCL_0", "1", "J1", "Pin_1")


### Connecting all wires in the Schematic ###


# Connecting J1 pin Pin_4 (Pin ID 4 -- Name Pin_4) to #PWR_GND1 pin 1 (Pin ID 1 -- Name None)
connect_pins("J1", "Pin_4", "#PWR_GND1", "1")

# Connecting J1 pin Pin_3 (Pin ID 3 -- Name Pin_3) to #PWR_3V1 pin +3.3V (Pin ID 1 -- Name +3.3V)
connect_pins("J1", "Pin_3", "#PWR_3V1", "+3.3V")

write_out_all_wires()

```
