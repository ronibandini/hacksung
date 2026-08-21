# 🧺⚡ Hacksung

**An Arduino replacement controller for a failed Samsung washing machine, combining relay-based motor and valve control, physical buttons, Wi-Fi, Telegram notifications, and a few deliberately unnecessary extras.**

Hacksung began after a **Samsung WS1702 washing machine** developed erratic behavior after only a few years of use.

Several repairs were attempted, including work on the motor, carbon brushes, wiring, coils, pumps, and electronic control board. The individual electromechanical components still appeared usable, so instead of discarding the complete appliance, the original controller was replaced with an **Arduino** and external control electronics.

The resulting machine provides:

- manual control of water and motor functions
- experimental automatic wash cycles
- a Nokia 5110 menu interface
- Wi-Fi connectivity
- Telegram notifications
- a dollar-exchange-rate screen
- a fake "Will it run Doom?" sequence
- custom literary washing programs

```text
             ┌────────────────────┐
             │ Arduino MKR WiFi   │
             │       1010         │
             └─────────┬──────────┘
                       │
      ┌────────────────┼───────────────────┐
      │                │                   │
      ▼                ▼                   ▼
 Nokia 5110       3 push buttons      Wi-Fi / Internet
   display                                 │
                                          ├── Telegram
      │                                   │
      │                                   └── Dollar quote
      │
      ▼
  Menu system
      │
      ▼
4 relay outputs
      │
 ┌────┼───────────────┐
 │    │               │
 ▼    ▼               ▼
Motor Water In    Water Out
```

---

## ✨ Features

- 🧺 Replacement washing-machine control unit
- ⚡ Arduino MKR WiFi 1010
- 🔌 Four relay outputs
- 💧 Water-in valve control
- 🚰 Drain / water-out control
- 🌀 Washing-machine motor control
- 📟 Nokia 5110 / PCD8544 display
- 🔘 Three-button menu interface
- 📶 Wi-Fi connectivity through WiFiNINA
- 📲 Telegram completion notification
- 🌐 Direct HTTP/HTTPS requests
- 💵 Dollar exchange-rate submenu
- 🎮 "Will it run Doom?" bitmap animation
- 📚 Washing cycles named after writers
- 🖨️ Custom 3D-printed controller enclosure
- 📜 MIT licensed

---

## 🧠 Concept

Most of a washing machine is not the control board.

A conventional machine contains independently useful components such as:

```text
motor
water inlet valves
drain pump
tachometer
door hardware
drum
mechanical structure
```

Hacksung keeps those components and replaces the failed electronic controller.

```text
Original control PCB
        ✕
        │
        ▼
Arduino controller
        │
        ├── read buttons
        ├── display menus
        ├── execute sequence
        ├── switch relays
        ├── connect to Wi-Fi
        └── send notifications
```

The project was built as an experiment in extending the life of an otherwise mechanically functional appliance.

---

## 🏗️ Architecture

```text
                       USER
                        │
                 ┌──────┴───────┐
                 │              │
                 ▼              ▼
           Nokia 5110       3 buttons
                 │              │
                 └──────┬───────┘
                        ▼
              ┌───────────────────┐
              │ Arduino MKR WiFi  │
              │       1010        │
              │                   │
              │ C++               │
              │ Menu system       │
              │ WiFiNINA          │
              └─────────┬─────────┘
                        │
                 ┌──────┴───────┐
                 │              │
                 ▼              ▼
          Relay outputs       Internet
                 │              │
       ┌─────────┼───────┐      ├── Telegram
       ▼         ▼       ▼      │
     Motor    Water In  Drain    └── Custom PHP
                                         │
                                         ▼
                                  Dollar exchange
                                      quote
```

---

## 🧰 Hardware

The original build uses:

| Qty | Component | Purpose |
|---:|---|---|
| 1 | **Arduino MKR WiFi 1010** | Main controller and Wi-Fi |
| 1 | **4-channel relay board / shield** | Motor and valve switching |
| 1 | **Nokia 5110 LCD** | User interface |
| 3 | **Push buttons** | Menu navigation |
| 1 | **SCR 2000 W AC voltage controller** | Experimental motor-speed control |
| 1 | **ZMPT101B AC voltage sensor** | Experimental motor tachometer interface |
| 1 | Samsung WS1702 washing machine | Original appliance |
| 1 | Custom 3D-printed controller enclosure | Electronics housing |

Project tutorial:

[Dead Washing Machine Returns to Life with Arduino MKR — Hackster.io](https://www.hackster.io/roni-bandini/dead-washing-machine-returns-to-life-with-arduino-mkr-81a970)

---

# ⚡ Arduino MKR WiFi 1010

The project uses an **Arduino MKR WiFi 1010**, based on the SAMD21 microcontroller.

Relevant characteristics include:

- Arm Cortex-M0+ processor
- up to 48 MHz
- 256 KB Flash
- 32 KB SRAM
- Wi-Fi
- Bluetooth
- hardware cryptographic support
- compact MKR form factor

Official documentation:

[Arduino MKR WiFi 1010](https://docs.arduino.cc/hardware/mkr-wifi-1010/)

The MKR WiFi 1010 uses:

```text
3.3 V I/O
```

and its GPIO pins are **not 5 V tolerant**.

---

# 📟 Nokia 5110 display

The current source uses:

```cpp
#include <Adafruit_GFX.h>
#include <Adafruit_PCD8544.h>
```

and initializes the display with:

```cpp
Adafruit_PCD8544 display =
    Adafruit_PCD8544(7, 6, 5, 4, 3);
```

The pin assignment is:

| Function | MKR pin |
|---|---:|
| SCLK | `7` |
| DIN | `6` |
| D/C | `5` |
| CS | `4` |
| RST | `3` |

The screen resolution is:

```text
84 × 48
```

which is also used for the Hacksung logo and Doom animation bitmaps.

---

# 🔘 Buttons

The three menu buttons are configured as:

```cpp
const int BUTTON_PIN  = 0;
const int BUTTON_PIN1 = 1;
const int BUTTON_PIN2 = 2;
```

with:

```cpp
INPUT_PULLUP
```

The firmware implements software debouncing:

```cpp
const int DEBOUNCE_DELAY = 75;
```

The buttons provide:

```text
UP
DOWN
ENTER
```

style navigation through the menu system.

---

# 🔌 Relay outputs

The current source defines four relay outputs:

```cpp
int relayMotor1   = 8;
int relayWaterIn  = 9;
int relayWaterOut = 10;
int relayMotor2   = 11;
```

| Function | MKR pin |
|---|---:|
| Motor relay 1 | `8` |
| Water inlet | `9` |
| Drain / water out | `10` |
| Motor relay 2 | `11` |

The relay logic is active-low in the current firmware.

At startup:

```cpp
digitalWrite(relayMotor1, HIGH);
digitalWrite(relayMotor2, HIGH);
digitalWrite(relayWaterIn, HIGH);
digitalWrite(relayWaterOut, HIGH);
```

keeps all outputs inactive.

An output is activated with:

```cpp
digitalWrite(..., LOW);
```

---

# 🧭 Main menu

The current firmware has four top-level menu options:

```text
> Manual
  Auto
  Extra
  Info
```

or:

```text
  Manual
> Auto
  Extra
  Info
```

The menu state is maintained using:

```cpp
int screenCursor = 1;
int menuLevel = 0;
```

---

# 🕹️ Manual mode

The manual submenu contains:

```text
Water In
Motor on
Water out
Back
```

Each selection toggles the corresponding relay.

Example:

```cpp
if (waterIn == 0) {
    digitalWrite(relayWaterIn, LOW);
    waterIn = 1;
}
else {
    digitalWrite(relayWaterIn, HIGH);
    waterIn = 0;
}
```

The same basic logic is used for:

```text
motor
drain
```

This mode was useful during development for identifying and testing the original washing-machine hardware independently.

---

# 📟 Status icons

The display interface also shows small status indicators when outputs are active.

The current `myMessage()` function adds:

```text
↑   water entering
∞   motor running
↓   water leaving
```

at the right side of the Nokia display.

Conceptually:

```text
Hacksung              ↑
> Water In
  Motor on            ∞
  Water out           ↓
  Back
```

---

# 📚 Literary washing cycles

Instead of conventional appliance names, Hacksung uses argentine writers as washing programs:

```text
Lucio Mansilla
Gombrowicz
Bioy Casares
```

The intended personalities of the cycles are described in the original project as:

### Lucio V. Mansilla

```text
Prewash: 5 min
Wash:     5 min
Spin:   400 rpm
```

A deliberately rough cycle, referencing _Una excursión a los indios ranqueles_.

### Witold Gombrowicz

```text
Prewash: 15 min
Wash:    30 min
Spin:   800 rpm
```

### Adolfo Bioy Casares

```text
Prewash: 30 min
Wash:    60 min
Spin:  1000 rpm
```

These names turn the replacement appliance controller into something more personal than a conventional list such as:

```text
Cotton
Eco
Quick
Delicate
```

---

# ⚙️ Current automatic-cycle implementation

The current source is explicitly marked:

```text
work in progress
```

and the automatic cycles should be understood accordingly.

### Mansilla

The first cycle contains an actual relay-control demonstration.

The current sequence is:

```text
Water inlet ON
      │
      │ 20 seconds
      ▼
Water inlet OFF
      │
      ▼
Motor ON
      │
      │ 20 seconds
      ▼
Motor OFF
      │
      ▼
Drain ON
      │
      │ 20 seconds
      ▼
Drain OFF
      │
      ▼
Motor ON
      │
      │ 20 seconds
      ▼
Motor OFF
      │
      ▼
Telegram notification
```

The values displayed on screen:

```text
Prewash: 5 min
Wash: 5 min
Cent: 400 rpm
```

---

# 🌀 Motor control

The original washing-machine motor has seven connections.

During development, the motor was tested independently from the failed controller to identify the connections required to run it.

The full project also experimented with an:

```text
SCR 2000 W AC voltage regulator
```

to vary motor speed.

This was necessary because simply switching the motor on and off was not sufficient for a complete wash cycle.

Different operations require very different speeds:

```text
agitation
      ↓
low speed

spinning
      ↓
high speed
```

---

# 📈 Tachometer feedback

The original motor includes a tachometer.

The project documentation describes measuring approximately:

```text
40 V AC
```

from that tachometer during development.

A **ZMPT101B AC voltage sensor** was added as an experimental interface so the signal could eventually be measured through the microcontroller.

The current source contains placeholders:

```cpp
//motorRpm = analogRead(A0);
```

and:

```cpp
//analogWrite(A1, 100);
```

for future closed-loop motor-speed control.

---

# 🔄 Motor reversal

The original project also investigated reversing motor direction.

The published build notes that this requires changing the motor connections through additional switching hardware.

The current repository declares:

```cpp
relayMotor2
```

on pin:

```text
11
```

but does not implement a complete automatic reversing sequence.

Motor reversal therefore remains an unfinished part of the prototype.

---

# 📶 Wi-Fi

The project uses:

```cpp
#include <WiFiNINA.h>
```

Credentials are configured in:

```cpp
char ssid[] = "SSID";
char pass[] = "PASSWORD";
```

At startup the firmware waits until:

```cpp
status == WL_CONNECTED
```

before entering the main interface.

The startup screen displays a Wi-Fi bitmap while connecting.

---

# 📲 Telegram notification

When the implemented automatic cycle finishes, Hacksung sends a message through the **Telegram Bot API**.

The current source connects directly to:

```text
api.telegram.org
```

using:

```cpp
client.connectSSL(server, 443)
```

and sends an HTTP request equivalent to:

```text
GET /bot<TOKEN>/sendMessage
    ?chat_id=<CHAT_ID>
    &text=Finished
```

This avoids relying on a dedicated Telegram Arduino library.

---

## 🔑 Telegram configuration

Replace the placeholders in:

```cpp
client.println(
    "GET /XXXXXXXXXXXXXXXXXYYYYYYYYYYYYYYYY/"
    "sendMessage?chat_id=-ZZZZZZZZZZZZZZZZ"
    "&text=Finished HTTP/1.1"
);
```

with the corresponding:

```text
bot token
chat ID
```

Do not commit real credentials to a public repository.

Telegram Bot API:

[Telegram Bot API](https://core.telegram.org/bots/api)

---

# 💵 Dollar exchange-rate menu

Because a washing machine obviously needs an exchange-rate screen, the `Extra` menu contains:

```text
Doom
Dollar
Stats
Back
```

The Dollar option connects to a custom HTTP server:

```cpp
char server2[] = "yourPHPServer.com";
```

and requests:

```text
/server/dolar.php
```

over port:

```text
80
```

The response is parsed using:

```cpp
quote = getValue(
    serverAnswer,
    '~',
    1
);
```

and displayed as:

```text
Dollar quote
Sell $
<value>
```

The example therefore expects the custom PHP endpoint to return a response containing a `~`-delimited value.

---

# 🎮 Will it run Doom?

Not quite.

The Doom submenu displays four monochrome bitmaps sequentially on the Nokia 5110:

```cpp
doom1
doom2
doom3
doom4
```

with delays of:

```text
1.5 s
1.5 s
1.0 s
1.0 s
```

The result is a short Doom-style animation.

```text
Hacksung
   │
   ▼
Extra
   │
   ▼
Doom
   │
   ▼
84 × 48 bitmap animation
```

No Doom game engine is running on the washing machine.

---

# 📊 Stats screen

The Extras menu also includes:

```text
Stats
```

The current firmware displays:

```text
Manual: 12
Auto: 0
Run: 8 hrs
```

These values are currently hardcoded:

```cpp
myMessage(
    "Stats",
    "Manual: 12",
    "Auto: 0",
    "Run: 8 hrs"
);
```

They are placeholders rather than persistent usage counters.

---

# ℹ️ Info screen

The Info menu displays:

```text
Roni Bandini
Dec 2020
@RoniBandini
Argentina
```

and then returns automatically to the main menu.

---

# 🧩 Software dependencies

The sketch includes:

```cpp
#include <Adafruit_GFX.h>
#include <Adafruit_PCD8544.h>
#include <SPI.h>
#include <WiFiNINA.h>
```

Required libraries:

- **Adafruit GFX Library**
- **Adafruit PCD8544 Nokia 5110 LCD Library**
- **WiFiNINA**
- **SPI** — included with Arduino

---

# 🚀 Installation

## 1. Clone the repository

```bash
git clone https://github.com/ronibandini/hacksung.git
cd hacksung
```

---

## 2. Install Arduino IDE

Download:

[Arduino IDE](https://www.arduino.cc/en/software)

---

## 3. Install MKR board support

In Arduino IDE:

```text
Tools
→ Board
→ Boards Manager
```

install:

```text
Arduino SAMD Boards
```

Then select:

```text
Arduino MKR WiFi 1010
```

---

## 4. Install libraries

From:

```text
Sketch
→ Include Library
→ Manage Libraries
```

install:

```text
Adafruit GFX Library
Adafruit PCD8544
WiFiNINA
```

---

## 5. Configure Wi-Fi

Edit:

```cpp
char ssid[] = "SSID";
char pass[] = "PASSWORD";
```

---

## 6. Configure Telegram

Replace the placeholder bot-token and chat-ID values in the Telegram request.

For testing, configure a Telegram bot using:

```text
@BotFather
```

and obtain the target group/chat ID.

---

## 7. Optional dollar service

The Dollar submenu expects an external server.

Configure:

```cpp
char server2[] = "yourPHPServer.com";
```

and provide:

```text
/server/dolar.php
```

with the response format expected by the parser.

The washing-machine control functions do not depend on this feature.

---

## 8. Connect the user interface

Current source mapping:

### Buttons

```text
D0 → button
D1 → button
D2 → button
```

### Nokia 5110

```text
D7 → SCLK
D6 → DIN
D5 → D/C
D4 → CS
D3 → RST
```

### Relays

```text
D8  → Motor 1
D9  → Water In
D10 → Water Out
D11 → Motor 2
```

---

## 9. Upload

Open:

```text
hacksung.ino
```

compile and upload it to the MKR WiFi 1010.

Serial output uses:

```text
9600 baud
```

and begins with:

```text
Hacksung started...
Roni Bandini - Argentina
```

---

# 📁 Repository structure

```text
hacksung/
├── Hacksung.pdf
├── LICENSE
├── README.md
└── hacksung.ino
```

### `hacksung.ino`

Main Arduino firmware.

Contains:

- relay control
- button debouncing
- menu system
- LCD bitmaps
- manual operation
- automatic-cycle prototype
- Telegram request
- dollar exchange-rate request
- Doom animation
- status screen

### `Hacksung.pdf`

Additional project documentation.

### `LICENSE`

MIT License.

---

# 🖨️ 3D-printed enclosure

The control unit enclosure was designed in **Fusion 360** and printed as a custom replacement interface.

The original build documentation describes it as a three-part case.

3D files:

**[Hacksung washing machine control unit — Thingiverse](https://www.thingiverse.com/thing:4697383)**

The enclosure houses:

- Arduino
- relay hardware
- Nokia display
- three physical buttons
- related control electronics

---

# 🔬 Ideas for extending the project

1. **🌀 Closed-loop motor control** — safely condition the tachometer signal and implement actual RPM feedback for controlled wash and spin speeds.

---

# 📰 External references

## 🗞️ Arduino

### Arduino control system puts defunct washing machine back into operation

Arduino published a dedicated feature on **December 28, 2020**.

The article covers:

- the failed Samsung controller
- Arduino MKR WiFi 1010
- four relays
- three buttons
- motor and valve control
- Telegram notifications
- reuse of the original appliance
- future motor-speed and reverse-control ideas

**[Read the Arduino article](https://blog.arduino.cc/2020/12/28/arduino-control-system-puts-defunct-washing-machine-back-into-operation/)**

---

## 📰 Electrogeek

### Arduino vuelve a poner en funcionamiento una lavadora rota

Electrogeek published Spanish-language coverage of Hacksung in September 2025, describing the MKR WiFi replacement controller, relays, Telegram integration, GitHub source, and 3D-printable enclosure.

**[Read on Electrogeek](https://www.electrogeekshop.com/arduino-vuelve-a-poner-en-funcionamiento-una-lavadora-rota/)**

---

## 📰 RoboCraft

### Система управления для стиральной машинки на Arduino

RoboCraft also covered the project, describing the Arduino MKR WiFi 1010 replacement controller, three-button interface, Nokia display, four relays, and Telegram notification.

**[Read on RoboCraft](https://robocraft.ru/projects/4151)**

---

## 📰 Time Out

A later Time Out article about other Maker Counterculture projects also mentions the hacked Samsung washing machine and its unusual features:

- writer-themed washing cycles
- Telegram
- dollar exchange rate
- Doom

**[Read on Time Out](https://www.timeout.es/barcelona/es/noticias/un-hombre-crea-un-aparato-para-detectar-reggaeton-y-apagarlo-y-nos-explica-como-fabricarlo-en-casa-022524)**

---

# 🛠️ Project tutorials

## Hackster.io

### Dead Washing Machine Returns to Life with Arduino MKR

The complete project tutorial was published on **December 25, 2020**.

It documents:

- original washing-machine failure
- motor investigation
- seven-pin motor
- water valves
- Arduino MKR WiFi 1010
- relay controller
- SCR motor-speed experiment
- ZMPT101B tachometer experiment
- Nokia 5110 interface
- literary washing cycles
- Telegram
- dollar exchange rates
- Doom
- enclosure design
- future development

**[Dead Washing Machine Returns to Life with Arduino MKR — Hackster.io](https://www.hackster.io/roni-bandini/dead-washing-machine-returns-to-life-with-arduino-mkr-81a970)**

---

## DFRobot Maker Community

### Dead Washing Machine Returns to Life with Arduino MKR

DFRobot Maker Community republished the build with its complete hardware list and project explanation.

**[Read on DFRobot Maker Community](https://community.dfrobot.com/makelog-308553.html)**

---

# ✍️ Medium

## Hackear el lavarropas

The original Spanish-language article documents the transformation from Samsung washing machine to Hacksung.

It covers:

- the unsuccessful repair history
- motor and valve testing
- MKR WiFi controller
- relay hardware
- Nokia 5110 display
- literary cycles
- Telegram
- exchange-rate menu
- Doom screen

**[Hackear el lavarropas — Medium](https://bandini.medium.com/hackear-el-lavarropas-dd238f7da29f)**

---

# 📕 Contracultura Maker

Hacksung is part of a broader collection of projects based on modifying existing technologies, repurposing discarded hardware, building unusual interfaces, and treating hardware as something that can be understood and transformed rather than simply replaced.

More projects and context are collected in:

**[Contracultura Maker — book](https://bandini.medium.com/libro-de-contracultura-maker-94d1bb0d951c)**

---

# 📚 Useful references

- **[Arduino MKR WiFi 1010](https://docs.arduino.cc/hardware/mkr-wifi-1010/)**
- **[Arduino IDE](https://www.arduino.cc/en/software)**
- **[WiFiNINA](https://docs.arduino.cc/libraries/wifinina/)**
- **[Adafruit PCD8544 Nokia 5110 Library](https://github.com/adafruit/Adafruit-PCD8544-Nokia-5110-LCD-library)**
- **[Telegram Bot API](https://core.telegram.org/bots/api)**
- **[Hacksung enclosure — Thingiverse](https://www.thingiverse.com/thing:4697383)**

---

# 🔗 You may also be interested in...

Other projects by **Roni Bandini** involving hardware reuse, appliances, unusual interfaces, and Maker Counterculture.

## 🧮🕵️ P101

**Electronic art installation built from a discarded Olivetti calculator using an Arduino UNO R4 Minima, six-digit LCD, and the original printer mechanism.**

Another project that keeps useful hardware while replacing the original electronic behavior.

**[github.com/ronibandini/p101](https://github.com/ronibandini/p101)**

---

## 📺🧠 Tachistoscope

**Raspberry Pi-powered media installation built around a CRT monitor, physical controls, video processing, and historical technology.**

Another project combining obsolete hardware with a completely new control system and purpose.

**[github.com/ronibandini/Tachistoscope](https://github.com/ronibandini/Tachistoscope)**

---

## 🤖⚙️ Elektro

**Full-scale ESP8266-controlled recreation of the 1939 Westinghouse Elektro robot using recycled metal, motors, audio, smoke, and Wi-Fi control.**

Another project centered on reconstructing and reprogramming electromechanical hardware.

**[github.com/ronibandini/elektro](https://github.com/ronibandini/elektro)**

---

# ⚠️ Electrical safety

## Mains voltage

A washing machine contains exposed circuits and components operating directly from mains electricity.

Depending on the country, this may be approximately:

```text
110–120 V AC
```

or:

```text
220–240 V AC
```

Both can cause fatal electric shock, burns, fire, or equipment damage.

Do not reproduce the mains-voltage portions of this project unless you are qualified to work safely on mains-powered appliances.

---

## Water and electricity

A washing machine combines:

```text
mains electricity
+
water
+
metal enclosure
+
high-current motor
```

which makes safe grounding, insulation, fusing, strain relief, clearances, connectors, and enclosure design critical.

Never test exposed mains wiring in wet conditions.

---


# 📜 License

Hacksung is released under the **MIT License**.

See [`LICENSE`](LICENSE) for details.

---

# 👤 Author

**Roni Bandini**

Maker, AI developer, electronic artist and writer.

- 🐙 GitHub: [@ronibandini](https://github.com/ronibandini)
- 💼 LinkedIn: [Roni Bandini](https://www.linkedin.com/in/ronibandini/)
- 📸 Instagram: [@ronibandini](https://www.instagram.com/ronibandini/)
- 🐦 X: [@RoniBandini](https://x.com/RoniBandini)
- ✍️ Medium: [bandini.medium.com](https://bandini.medium.com/)
- 🛠️ Hackster: [Roni Bandini](https://www.hackster.io/roni-bandini)
- 🔧 Hackaday.io: [Roni Bandini](https://hackaday.io/ronibandini)

Buenos Aires, Argentina.
