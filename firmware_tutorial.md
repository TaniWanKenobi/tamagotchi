# Firmware Setup Tutorial

First, install the [Arduino IDE](https://www.arduino.cc/en/software/).

<details>
<summary>What is Arduino IDE?</summary>

It's the program we use to write code and upload it to microcontrollers like our [ESP32-C6](https://www.espressif.com/en/products/socs/esp32-c6). Think of it as a specialized text editor that can also talk to hardware over USB.

</details>

---

## Table of Contents

- [Step 1: Install the ESP32 Board Package](#step-1-install-the-esp32-board-package)
- [Step 2: Install the Board in Boards Manager](#step-2-install-the-board-in-boards-manager)
- [Step 3: Select the Board and Port](#step-3-select-the-board-and-port)
- [Step 4: Install Required Libraries](#step-4-install-required-libraries)
- [Step 5: Create the Project](#step-5-create-the-project)
- [Step 6: Reading Your Schematic for Firmware](#step-6-reading-your-schematic-for-firmware)
- [Step 7: Includes and Hardware Configuration](#step-7-includes-and-hardware-configuration)
- [Step 8: Define the Pet Model](#step-8-define-the-pet-model)
- [Step 9: Define UI States (State Machine)](#step-9-define-ui-states-state-machine)
- [Step 10: Setup Function](#step-10-setup-function)
- [Step 11: Game Update Logic](#step-11-game-update-logic)
- [Step 12: Reading Button Input](#step-12-reading-button-input)
- [Step 13: Screen Logic](#step-13-screen-logic)
- [Step 14: Pet Sprites](#step-14-pet-sprites)
- [Step 15: Display Function (OLED Rendering)](#step-15-display-function-oled-rendering)
- [Step 16: Main Loop](#step-16-main-loop)

---

## Step 1: Install the ESP32 Board Package

The Arduino IDE doesn't know about ESP32 chips by default, so we need to tell it where to find the board definitions.

1. Open the Arduino IDE.
2. Go to **File > Preferences** (or **Arduino IDE > Settings** on macOS).
3. In the **"Additional Boards Manager URLs"** field, add the following URL:
   ```
   https://espressif.github.io/arduino-esp32/package_esp32_index.json
   ```
   > **Note:** If you have other URLs in the field, separate them with a comma (`,`).
4. Click **OK** to save the preferences.
5. Wait for the IDE to download the board index file (a *"Downloading index"* notification will appear in the bottom right).

<details>
<summary>Why this step?</summary>

This URL points to a JSON file maintained by [Espressif](https://www.espressif.com/) (the company that makes the ESP32). It tells Arduino IDE what ESP32 boards exist, what compilers to download, and how to upload code to them.

</details>

## Step 2: Install the Board in Boards Manager

1. Open the **Boards Manager** by navigating to **Tools > Board > Boards Manager**.
2. In the search bar, type `esp32`.
3. Locate the entry for **"esp32 by Espressif Systems"**.
4. Click the **Install** button.
   > **Important:** Ensure you are using version **3.0.0 or later**, which supports the C6.
5. Wait for the installation to finish.

<details>
<summary>What gets installed?</summary>

The board package includes the [RISC-V](https://en.wikipedia.org/wiki/RISC-V) cross-compiler toolchain, the ESP-IDF framework libraries, and upload tools. In short, everything needed to turn your code into firmware the chip can run.

</details>

## Step 3: Select the Board and Port

1. After installation, go to **Tools > Board > esp32**.
2. Select your specific ESP32-C6 board model, which is in this case:
   - `XIAO ESP32C6`

<details>
<summary>Why does the board selection matter?</summary>

Different boards wire their pins differently and have different amounts of flash memory. Selecting the correct board tells the compiler how to configure pin mappings, clock speed, and memory layout.

</details>

## Step 4: Install Required Libraries

We need two libraries for the OLED display. In the Arduino IDE, go to **Sketch > Include Library > Manage Libraries**, then search for and install:

1. **Adafruit SSD1306** (by Adafruit)
2. **Adafruit GFX Library** (by Adafruit), which will be prompted as a dependency.

<details>
<summary>What are these libraries for?</summary>

- **Adafruit SSD1306** is the driver that talks to our specific OLED display chip over I2C.
- **Adafruit GFX** is a graphics library that provides drawing functions (text, shapes, pixels) that work across many different displays.

Together, they let us write things like `display.println("Hello")` and have it appear on the OLED.

</details>

## Step 5: Create the Project

Create a new Arduino sketch: **File > New Sketch**, then save it as `Tamagotchi.ino`.

<details>
<summary>What is a .ino file?</summary>

It's Arduino's version of a C++ source file. The IDE wraps it with `#include <Arduino.h>` and generates `main()` behind the scenes, so you only need to write [`setup()`](https://docs.arduino.cc/language-reference/en/structure/sketch/setup/) and [`loop()`](https://docs.arduino.cc/language-reference/en/structure/sketch/loop/).

</details>

---

## Step 6: Reading Your Schematic for Firmware

Before writing any code, open your `.kicad_sch` file and trace every component's wires to the MCU. Your schematic is the single source of truth. If your code doesn't match it, nothing will work on the real board.

For each component, follow its wires to the MCU and write down the **GPIO number**. KiCad shortcut: click any wire and press **`** (backtick) to highlight the entire net.

### My Results

Here's what I found after tracing my schematic. **Yours will differ. Use your own GPIO numbers.**

**Buttons** (active-low: one side to GND, other side to MCU → reads `LOW` when pressed):

| Button | MCU Pin | GPIO |
|--------|---------|------|
| SW1 | Pin 1 | GPIO0 |
| SW2 | Pin 2 | GPIO1 |
| SW3 | Pin 3 | GPIO2 |

**OLED Display** (I²C address is usually `0x3C`, check your datasheet):

| OLED Pin | MCU Pin | GPIO |
|----------|---------|------|
| SDA | Pin 5 | GPIO22 |
| SCL | Pin 6 | GPIO23 |

**Buzzer** (positive terminal tied 3.3V to GPIO **sinks** current, so `LOW` = ON):

| Buzzer Pin | MCU Pin | GPIO |
|------------|---------|------|
| Signal | Pin 10 | GPIO20 |

<details>
<summary>How do I know if my buzzer is active or passive?</summary>

- **Active buzzer**: has internal circuitry that generates a tone. You just turn it on/off with a digital signal (`LOW`/`HIGH`).
- **Passive buzzer**: needs an external signal (square wave) to produce sound. You control the pitch by changing the frequency using `tone()`.

Check the component listing on LCSC or the datasheet. If it says "electromagnetic" or "with oscillator," it's active. If it says "piezoelectric" with no oscillator, it's passive.

</details>

### Turn Your Results into Code

Now take every GPIO number you found and turn them into `#define` statements at the top of your sketch:

```cpp
// Buttons (traced from schematic)
#define BTN1 0   // SW1 → MCU Pin 1 → GPIO0
#define BTN2 1   // SW2 → MCU Pin 2 → GPIO1
#define BTN3 2   // SW3 → MCU Pin 3 → GPIO2

// I2C OLED (traced from schematic)
#define SDA_PIN 22  // OLED SDA → MCU Pin 5 → GPIO22
#define SCL_PIN 23  // OLED SCL → MCU Pin 6 → GPIO23

// Buzzer (traced from schematic)
#define BUZZER 20   // Buzzer signal → MCU Pin 10 → GPIO20
```

**These numbers come directly from your schematic.** Your pin numbers will almost certainly be different from mine if your schematic is different. Always trace your own schematic. Never copy someone else's pin numbers.

> **This step is critical.** If your pin definitions don't match your schematic, your firmware will compile fine but nothing will work when you flash it to the board. The schematic is always the source of truth.

---

## Step 7: Includes and Hardware Configuration

At the top of the sketch, include the libraries and define how the hardware is wired.

```cpp
#include <Wire.h>
#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>

#define SCREEN_WIDTH 128
#define SCREEN_HEIGHT 64
#define OLED_RESET -1

Adafruit_SSD1306 display(SCREEN_WIDTH, SCREEN_HEIGHT, &Wire, OLED_RESET);

#define BTN_LEFT   D0
#define BTN_MIDDLE D1
#define BTN_RIGHT  D2
#define BUZZER_PIN D3
```

**Line-by-line breakdown:**

- `#include <Wire.h>` includes the [I2C library](https://docs.arduino.cc/language-reference/en/functions/communication/wire/), which handles communication with the OLED over the SDA/SCL pins.
- `#include <Adafruit_GFX.h>` and `#include <Adafruit_SSD1306.h>` bring in the display driver and graphics functions.
- `SCREEN_WIDTH` / `SCREEN_HEIGHT` match the 0.96" OLED resolution (128x64 pixels).
- `OLED_RESET` is set to `-1` because our OLED does not have a dedicated reset pin.
- `Adafruit_SSD1306 display(...)` creates the display object we'll use to draw everything.
- `BTN_LEFT`, `BTN_MIDDLE`, `BTN_RIGHT` are the three GPIO pins connected to our buttons. **Change these to match your schematic!**
- `BUZZER_PIN` is the GPIO connected to the buzzer.

<details>
<summary>What if my buttons are on different pins?</summary>

Check your KiCad schematic to see which GPIO pins your buttons are wired to. Then change `D0`, `D1`, `D2`, and `D3` to match. For example, if your left button is on pin D5, change the line to `#define BTN_LEFT D5`.

</details>

## Step 8: Define the Pet Model

This is the heart of the system: a [`struct`](https://cplusplus.com/doc/tutorial/structures/) that bundles all of the pet's stats into a single data type.

```cpp
struct Pet {
  int hunger;       // 0 to 100 (0 = starving, 100 = full)
  int happiness;    // 0 to 100 (0 = miserable, 100 = ecstatic)
  int energy;       // 0 to 100 (0 = exhausted, 100 = fully rested)
  unsigned long age; // total seconds the pet has been alive
};
```

**Line-by-line breakdown:**

- `struct Pet { ... };` defines a new data type called `Pet`. A struct is like a blueprint: it describes *what data* a pet holds, but doesn't create one yet.
- `int hunger;` is an integer from 0 to 100. We chose `int` because the values are small whole numbers.
- `unsigned long age;` uses [`unsigned long`](https://docs.arduino.cc/language-reference/en/variables/data-types/unsignedLong/) (0 to 4,294,967,295) because time values from `millis()` can get very large and are never negative.

Now create a global instance. This is the **actual pet** that lives in memory:

```cpp
Pet pet;
```

<details>
<summary>Why global?</summary>

Every function in the sketch (`setup()`, `loop()`, `updatePet()`, etc.) needs access to the same pet. A [global variable](https://docs.arduino.cc/language-reference/en/variables/variable-scope-qualifiers/scope/) lives for the entire program and is visible everywhere.

</details>

## Step 9: Define UI States ([State Machine](https://en.wikipedia.org/wiki/Finite-state_machine))

Instead of scattering `if/else` checks throughout `loop()`, we define **named states** that the program can be in. This is called a **finite state machine**: the program is always in exactly one state at a time.

```cpp
enum Screen {
  SCREEN_MAIN,
  SCREEN_FEED,
  SCREEN_PLAY,
  SCREEN_SLEEP
};

Screen currentScreen = SCREEN_MAIN;
```

**Line-by-line breakdown:**

- `enum Screen { ... };` is an [`enum`](https://cplusplus.com/doc/tutorial/other_data_types/) (enumeration) that assigns human-readable names to integer constants. Under the hood, `SCREEN_MAIN` = 0, `SCREEN_FEED` = 1, etc., but we never need to know that.
- `Screen currentScreen = SCREEN_MAIN;` tracks which screen we're on right now. The program starts on the main screen.

<details>
<summary>Why use a state machine?</summary>

Without it, you end up with deeply nested `if` statements that are hard to read and debug. A state machine makes the logic predictable: *"When I'm in state X and event Y happens, go to state Z."* This is the same pattern used in [real embedded systems](https://en.wikipedia.org/wiki/UML_state_machine), game engines, and protocol parsers.

</details>

## Step 10: Setup Function

[`setup()`](https://docs.arduino.cc/language-reference/en/structure/sketch/setup/) runs **once** when the board powers on or resets. Use it to initialize the display, buttons, and starting values.

```cpp
void setup() {
  pinMode(BTN_LEFT, INPUT_PULLUP);
  pinMode(BTN_MIDDLE, INPUT_PULLUP);
  pinMode(BTN_RIGHT, INPUT_PULLUP);
  pinMode(BUZZER_PIN, OUTPUT);

  display.begin(SSD1306_SWITCHCAPVCC, 0x3C);
  display.clearDisplay();
  display.setTextSize(1);
  display.setTextColor(SSD1306_WHITE);
  display.setCursor(0, 0);
  display.println("Tamagotchi Init...");
  display.display();
  delay(1000);

  pet.hunger = 80;
  pet.happiness = 80;
  pet.energy = 80;
  pet.age = 0;
}
```

**Line-by-line breakdown:**

- `pinMode(BTN_LEFT, INPUT_PULLUP);` configures each button pin as an input with the [internal pull-up resistor](https://docs.arduino.cc/language-reference/en/functions/digital-io/pinMode/) enabled. This means the pin reads `HIGH` normally and `LOW` when the button is pressed (active-low).
- `pinMode(BUZZER_PIN, OUTPUT);` sets the buzzer pin as an output so we can drive it with a signal.
- `display.begin(SSD1306_SWITCHCAPVCC, 0x3C);` initializes the OLED. `0x3C` is the [I2C address](https://learn.adafruit.com/adafruit-gfx-graphics-library) of the display (the most common address for these OLEDs).
- `display.clearDisplay();` clears any leftover data in the display buffer.
- `display.setTextSize(1);` sets text to the smallest size (6x8 pixels per character).
- `display.setTextColor(SSD1306_WHITE);` sets the drawing color to white (lit pixels).
- `display.println(...)` writes text to the **buffer** (not the screen yet).
- `display.display();` pushes the buffer to the actual OLED screen. **You must call this for anything to appear!**
- The pet stats start at 80/100, healthy but not perfect, so there's room to improve or decay.

<details>
<summary>What if my OLED doesn't turn on?</summary>

The most common issues are:
- **Wrong I2C address**: Some OLEDs use `0x3D` instead of `0x3C`. Try changing the address in `display.begin()`.
- **Wiring**: Double-check that SDA, SCL, VCC (3V3), and GND are connected correctly.
- **Library not installed**: Make sure you installed both Adafruit SSD1306 and Adafruit GFX.

</details>

## Step 11: Game Update Logic

We need stats to decay over time. A Tamagotchi that never gets hungry isn't much fun. We use [`millis()`](https://docs.arduino.cc/language-reference/en/functions/time/millis/) instead of [`delay()`](https://docs.arduino.cc/language-reference/en/functions/time/delay/) because `delay()` freezes the entire program (no input, no rendering), while `millis()` lets everything else keep running.

```cpp
unsigned long lastUpdate = 0;

void updatePet() {
  if (millis() - lastUpdate > 5000) {  // every 5 seconds
    pet.hunger--;
    pet.happiness--;
    pet.energy--;

    if (pet.hunger < 0) pet.hunger = 0;
    if (pet.happiness < 0) pet.happiness = 0;
    if (pet.energy < 0) pet.energy = 0;

    pet.age += 5;
    lastUpdate = millis();
  }
}
```

**Line-by-line breakdown:**

- `unsigned long lastUpdate = 0;` remembers the last time we updated the pet. Starts at 0 (boot time).
- `millis() - lastUpdate > 5000` checks if more than 5000ms (5 seconds) have passed. `millis()` returns the number of milliseconds since the board powered on.
- `pet.hunger--` uses the [`--` operator](https://cplusplus.com/doc/tutorial/operators/) to decrement by 1. Every 5 seconds, the pet gets slightly hungrier, sadder, and more tired.
- `if (pet.hunger < 0) pet.hunger = 0;` is **clamping**: we prevent stats from going below zero. Without this, hunger could hit -50, which makes no sense.
- `lastUpdate = millis();` resets the timer so the next update happens 5 seconds from *now*.

<details>
<summary>What is non-blocking timing?</summary>

This `millis()` pattern is fundamental to embedded programming. It's how you schedule periodic tasks without blocking the CPU. You'll see this exact pattern in [Arduino's BlinkWithoutDelay example](https://docs.arduino.cc/built-in-examples/digital/BlinkWithoutDelay/).

</details>

## Step 12: Reading Button Input

The three physical buttons on the PCB are wired active-low (see the PCB tutorial). We read them using [`digitalRead()`](https://docs.arduino.cc/language-reference/en/functions/digital-io/digitalRead/). We also debounce them so a single press doesn't register multiple times.

```cpp
unsigned long lastButtonPress = 0;

void checkButtons() {
  if (millis() - lastButtonPress < 200) return;  // debounce: ignore presses within 200ms

  if (digitalRead(BTN_LEFT) == LOW) {
    currentScreen = SCREEN_FEED;
    tone(BUZZER_PIN, 1000, 50);
    lastButtonPress = millis();
  }
  else if (digitalRead(BTN_MIDDLE) == LOW) {
    currentScreen = SCREEN_PLAY;
    tone(BUZZER_PIN, 1200, 50);
    lastButtonPress = millis();
  }
  else if (digitalRead(BTN_RIGHT) == LOW) {
    currentScreen = SCREEN_SLEEP;
    tone(BUZZER_PIN, 800, 50);
    lastButtonPress = millis();
  }
}
```

**Line-by-line breakdown:**

- `unsigned long lastButtonPress = 0;` tracks the last time any button was pressed, used for debouncing.
- `if (millis() - lastButtonPress < 200) return;` is **debouncing**. Physical buttons "bounce" (make and break contact rapidly) when pressed, which can register as multiple presses. By ignoring any press within 200ms of the last one, we ensure one press = one action.
- `digitalRead(BTN_LEFT) == LOW` reads the pin state. Because the buttons are active-low, `LOW` means the button is currently being pressed.
- `currentScreen = SCREEN_FEED;` is the state transition. We don't do the feeding here; we just change the state. The actual logic happens in Step 13. This separation is important: **input detection** and **action execution** are different responsibilities.
- [`tone(BUZZER_PIN, 1000, 50);`](https://docs.arduino.cc/language-reference/en/functions/advanced-io/tone/) plays a short beep (1000Hz for 50ms) as audible feedback.

<details>
<summary>What is debouncing?</summary>

When you press a physical button, the metal contacts inside don't make a clean single connection. They vibrate and "bounce" for a few milliseconds, creating rapid on-off-on-off signals. Without debouncing, a single button press might register as 5 or 10 presses. The 200ms cooldown timer filters this out.

</details>

**Button mapping:**

| Button | Action | State Transition |
|--------|--------|-----------------|
| Left | Feed the pet | → `SCREEN_FEED` |
| Middle | Play with pet | → `SCREEN_PLAY` |
| Right | Put pet to sleep | → `SCREEN_SLEEP` |

## Step 13: Screen Logic

This is where state transitions have **consequences**. Each screen modifies the pet's stats and then immediately returns to the main screen. This makes each action a **one-shot event**: press feed, hunger goes up, done.

```cpp
void handleScreenLogic() {
  switch(currentScreen) {

    case SCREEN_FEED:
      pet.hunger += 10;
      if (pet.hunger > 100) pet.hunger = 100;
      currentScreen = SCREEN_MAIN;
      break;

    case SCREEN_PLAY:
      pet.happiness += 10;
      pet.energy -= 5;
      if (pet.happiness > 100) pet.happiness = 100;
      if (pet.energy < 0) pet.energy = 0;
      currentScreen = SCREEN_MAIN;
      break;

    case SCREEN_SLEEP:
      pet.energy += 15;
      if (pet.energy > 100) pet.energy = 100;
      currentScreen = SCREEN_MAIN;
      break;

    case SCREEN_MAIN:
      break;  // do nothing, just display stats
  }
}
```

**Line-by-line breakdown:**

- [`switch(currentScreen)`](https://docs.arduino.cc/language-reference/en/structure/control-structure/switchCase/) is like a multi-way `if/else`, but cleaner when comparing one variable against many known values. The CPU jumps directly to the matching `case`.
- `pet.hunger += 10;` uses the [`+=` operator](https://cplusplus.com/doc/tutorial/operators/) to add 10 to the current value. Feeding restores 10 hunger points.
- `if (pet.hunger > 100) pet.hunger = 100;` is **upper clamping**: stats can't exceed 100. This is the mirror of the lower clamping in `updatePet()`.
- `pet.energy -= 5;` means playing costs energy. This creates a **trade-off**: playing makes the pet happier but more tired. Trade-offs make games interesting.
- `currentScreen = SCREEN_MAIN;` returns to main after the action completes. This makes each action a one-shot: the player must actively choose to do something again.
- `break;` is **critical!** Without [`break`](https://docs.arduino.cc/language-reference/en/structure/control-structure/break/), execution "falls through" to the next `case`. This is a common C++ bug.

<details>
<summary>Why does SCREEN_MAIN do nothing?</summary>

That's intentional. The main screen is a "resting" state where we only display stats. All the action happens in the other states.

</details>

## Step 14: Pet Sprites

Your Tamagotchi needs a face! **Draw your own pet sprites** using [draw-to-bit](https://draw-to-bit.vercel.app/). This tool I made lets you draw pixel art and exports it directly as a C byte array you can paste into your code.

You'll need at least a few different expressions so the pet reacts to its stats. Here's an example set of 16x16 sprites to get you started (but make your own!):

```cpp
// Example: Happy face (all stats above 50)
const unsigned char PROGMEM petHappy[] = {
  0b00000000, 0b00000000,
  0b00011111, 0b11111000,
  0b00100000, 0b00000100,
  0b01000000, 0b00000010,
  0b01001100, 0b00110010,
  0b01001100, 0b00110010,
  0b01000000, 0b00000010,
  0b01000000, 0b00000010,
  0b01000100, 0b00100010,
  0b01000011, 0b11000010,
  0b01000000, 0b00000010,
  0b00100000, 0b00000100,
  0b00011111, 0b11111000,
  0b00000000, 0b00000000,
  0b00000000, 0b00000000,
  0b00000000, 0b00000000
};

// Example: Sad face (any stat below 30)
const unsigned char PROGMEM petSad[] = {
  0b00000000, 0b00000000,
  0b00011111, 0b11111000,
  0b00100000, 0b00000100,
  0b01000000, 0b00000010,
  0b01001100, 0b00110010,
  0b01001100, 0b00110010,
  0b01000000, 0b00000010,
  0b01000000, 0b00000010,
  0b01000000, 0b00000010,
  0b01000011, 0b11000010,
  0b01000100, 0b00100010,
  0b00100000, 0b00000100,
  0b00011111, 0b11111000,
  0b00000000, 0b00000000,
  0b00000000, 0b00000000,
  0b00000000, 0b00000000
};

// Example: Neutral face (everything else)
const unsigned char PROGMEM petNeutral[] = {
  0b00000000, 0b00000000,
  0b00011111, 0b11111000,
  0b00100000, 0b00000100,
  0b01000000, 0b00000010,
  0b01001100, 0b00110010,
  0b01001100, 0b00110010,
  0b01000000, 0b00000010,
  0b01000000, 0b00000010,
  0b01000111, 0b11100010,
  0b01000000, 0b00000010,
  0b01000000, 0b00000010,
  0b00100000, 0b00000100,
  0b00011111, 0b11111000,
  0b00000000, 0b00000000,
  0b00000000, 0b00000000,
  0b00000000, 0b00000000
};

// Example: Sleeping face (closed eyes)
const unsigned char PROGMEM petSleep[] = {
  0b00000000, 0b00000000,
  0b00011111, 0b11111000,
  0b00100000, 0b00000100,
  0b01000000, 0b00000010,
  0b01001111, 0b01110010,
  0b01000000, 0b00000010,
  0b01000000, 0b00000010,
  0b01000000, 0b00000010,
  0b01000011, 0b11000010,
  0b01000000, 0b00000010,
  0b01000000, 0b00000010,
  0b00100000, 0b00000100,
  0b00011111, 0b11111000,
  0b00000000, 0b00000000,
  0b00000000, 0b00000000,
  0b00000000, 0b00000000
};
```

**Line-by-line breakdown:**

- `const unsigned char PROGMEM` stores the bitmap in flash memory instead of RAM. The ESP32 has limited RAM, so [`PROGMEM`](https://docs.arduino.cc/language-reference/en/variables/utilities/PROGMEM/) keeps large data like sprites out of it.
- Each row is two bytes (16 bits = 16 pixels wide). A `1` bit lights up that pixel, a `0` bit leaves it dark.
- The examples above define four expressions: happy (smile), sad (frown), neutral (straight line), and sleeping (closed eyes). **Replace these with your own art from [draw-to-bit](https://draw-to-bit.vercel.app/)!**

<details>
<summary>Can I make the pet sprite bigger?</summary>

Yes! Change the bitmap to a larger size (e.g., 32x32) and update the `drawBitmap()` call to match. You'll need 4 bytes per row for 32px wide instead of 2. Larger sprites give you more detail but take up more screen space, so you may need to rearrange where the stats are drawn.

</details>

## Step 15: Display Function (OLED Rendering)

This function draws the pet character and stats on the OLED display. The pet's sprite is chosen based on its current mood.

```cpp
void render() {
  display.clearDisplay();

  // Choose the right sprite based on pet stats
  const unsigned char* sprite;
  if (pet.hunger < 30 || pet.happiness < 30 || pet.energy < 30) {
    sprite = petSad;
  } else if (pet.hunger > 50 && pet.happiness > 50 && pet.energy > 50) {
    sprite = petHappy;
  } else {
    sprite = petNeutral;
  }

  // Draw the pet sprite (centered horizontally, near the top)
  display.drawBitmap(56, 2, sprite, 16, 16, SSD1306_WHITE);

  // Draw stat bars below the pet
  display.setTextSize(1);

  display.setCursor(0, 24);
  display.print("HUN ");
  drawBar(24, 24, pet.hunger);

  display.setCursor(0, 34);
  display.print("HAP ");
  drawBar(24, 34, pet.happiness);

  display.setCursor(0, 44);
  display.print("ENG ");
  drawBar(24, 44, pet.energy);

  // Button labels at the bottom
  display.setCursor(0, 56);
  display.println("[Feed] [Play] [Sleep]");

  display.display();
}

// Draws a stat bar: empty rectangle with a filled portion based on value (0 to 100)
void drawBar(int x, int y, int value) {
  int barWidth = 100;
  int barHeight = 6;
  int fillWidth = map(value, 0, 100, 0, barWidth);

  display.drawRect(x, y, barWidth, barHeight, SSD1306_WHITE);       // outline
  display.fillRect(x, y, fillWidth, barHeight, SSD1306_WHITE);      // filled portion
}
```

**Key details:**

- The `sprite` pointer is chosen with simple `if/else` logic: if any stat drops below 30, the pet looks sad. If all stats are above 50, it looks happy. Otherwise, neutral.
- [`display.drawBitmap(x, y, bitmap, width, height, color)`](https://learn.adafruit.com/adafruit-gfx-graphics-library/graphics-primitives) draws a 1-bit bitmap at the given position. `56, 2` centers the 16px-wide sprite horizontally on the 128px-wide screen.
- `drawBar()` uses [`display.drawRect()`](https://learn.adafruit.com/adafruit-gfx-graphics-library/graphics-primitives) for the outline and `display.fillRect()` for the filled portion, creating a visual health bar.
- [`map(value, 0, 100, 0, barWidth)`](https://docs.arduino.cc/language-reference/en/functions/math/map/) scales the stat (0 to 100) to the pixel width of the bar.
- `display.display();` pushes the entire buffer to the OLED. **Nothing appears on screen until you call this.**

<details>
<summary>Why do we clear and redraw every frame?</summary>

The SSD1306 works with a pixel buffer in memory. If you only write new text without clearing first, old pixels remain lit and everything overlaps. The pattern of clear → draw → display is standard for frame-based rendering, the same concept used in game engines and animation.

</details>

## Step 16: Main Loop

[`loop()`](https://docs.arduino.cc/language-reference/en/structure/sketch/loop/) runs **continuously** after `setup()` finishes. The Arduino framework calls it over and over in an infinite loop. This is where we orchestrate everything.

```cpp
void loop() {
  checkButtons();       // 1. Read input
  updatePet();          // 2. Update state over time
  handleScreenLogic();  // 3. Process actions
  render();             // 4. Display results
  delay(100);           // 5. Short pause
}
```

**Why this order matters:**

```
┌──────────────────────────────────────────────────────┐
│                    GAME LOOP                         │
│                                                      │
│  ┌─────────┐   ┌────────┐   ┌───────┐   ┌────────┐ │
│  │  INPUT   │──▶│ UPDATE │──▶│ LOGIC │──▶│ RENDER │ │
│  │ (read    │   │ (decay │   │ (feed,│   │ (draw  │ │
│  │ buttons) │   │ stats) │   │ play) │   │ OLED)  │ │
│  └─────────┘   └────────┘   └───────┘   └────────┘ │
│       ▲                                      │      │
│       └──────────── delay(100) ──────────────┘      │
└──────────────────────────────────────────────────────┘
```

1. **Input**: Check if a button is pressed. This sets `currentScreen` but doesn't act on it yet.
2. **Update**: Decay stats based on elapsed time. This happens regardless of player input.
3. **Logic**: If the player triggered an action (feed/play/sleep), execute it now and return to main.
4. **Render**: Draw the results of everything that just happened on the OLED.
5. **Pause**: `delay(100)` gives a short 100ms pause. This keeps the loop responsive to button presses while not redrawing the screen thousands of times per second.

<details>
<summary>What is a game loop?</summary>

This is a real [game loop](https://gameprogrammingpatterns.com/game-loop.html) pattern. The same Input → Update → Render cycle is used in professional game engines, RTOS firmware, and GUI frameworks. Learning it here means you already understand the architecture of much larger systems.

</details>
