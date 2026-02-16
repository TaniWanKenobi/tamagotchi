# Firmware Setup Tutorial

First, install the [Arduino IDE](https://www.arduino.cc/en/software/).

> **What is Arduino IDE?** It's the program we use to write code and upload it to microcontrollers like our [ESP32-C6](https://www.espressif.com/en/products/socs/esp32-c6). Think of it as a specialized text editor that can also talk to hardware over USB.

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

> **Why this step?** This URL points to a JSON file maintained by [Espressif](https://www.espressif.com/) (the company that makes the ESP32). It tells Arduino IDE what ESP32 boards exist, what compilers to download, and how to upload code to them.

## Step 2: Install the Board in Boards Manager

1. Open the **Boards Manager** by navigating to **Tools > Board > Boards Manager**.
2. In the search bar, type `esp32`.
3. Locate the entry for **"esp32 by Espressif Systems"**.
4. Click the **Install** button.
   > **Important:** Ensure you are using version **3.0.0 or later**, which supports the C6.
5. Wait for the installation to finish.

> **What gets installed?** The board package includes the [RISC-V](https://en.wikipedia.org/wiki/RISC-V) cross-compiler toolchain, the ESP-IDF framework libraries, and upload tools. In short, everything needed to turn your code into firmware the chip can run.

## Step 3: Select the Board and Port

1. After installation, go to **Tools > Board > esp32**.
2. Select your specific ESP32-C6 board model, which is in this case:
   - `XIAO ESP32C6`

> **Why does the board selection matter?** Different boards wire their pins differently and have different amounts of flash memory. Selecting the correct board tells the compiler how to configure pin mappings, clock speed, and memory layout.

## Step 4: Create the Project

Create a new Arduino sketch: **File > New Sketch**, then save it as `Tamagotchi_Preliminary.ino`.

> **What is a `.ino` file?** It's Arduino's version of a C++ source file. The IDE wraps it with `#include <Arduino.h>` and generates `main()` behind the scenes, so you only need to write [`setup()`](https://docs.arduino.cc/language-reference/en/structure/sketch/setup/) and [`loop()`](https://docs.arduino.cc/language-reference/en/structure/sketch/loop/).

---

## Step 5: Define the Pet Model

This is the heart of the system: a [`struct`](https://cplusplus.com/doc/tutorial/structures/) that bundles all of the pet's stats into a single data type.

```cpp
struct Pet {
  int hunger;       // 0–100 (0 = starving, 100 = full)
  int happiness;    // 0–100 (0 = miserable, 100 = ecstatic)
  int energy;       // 0–100 (0 = exhausted, 100 = fully rested)
  unsigned long age; // total seconds the pet has been alive
};
```

**Line-by-line breakdown:**

- `struct Pet { ... };` - Defines a new data type called `Pet`. A struct is like a blueprint: it describes *what data* a pet holds, but doesn't create one yet.
- `int hunger;` - An integer from 0 to 100. We chose `int` because the values are small whole numbers.
- `unsigned long age;` - We use [`unsigned long`](https://docs.arduino.cc/language-reference/en/variables/data-types/unsignedLong/) (0 to 4,294,967,295) because time values from `millis()` can get very large and are never negative.

Now create a global instance. This is the **actual pet** that lives in memory:

```cpp
Pet pet;
```

> **Why global?** Every function in the sketch (`setup()`, `loop()`, `updatePet()`, etc.) needs access to the same pet. A [global variable](https://docs.arduino.cc/language-reference/en/variables/variable-scope-qualifiers/scope/) lives for the entire program and is visible everywhere.

## Step 6: Define UI States ([State Machine](https://en.wikipedia.org/wiki/Finite-state_machine))

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

- `enum Screen { ... };` - An [`enum`](https://cplusplus.com/doc/tutorial/other_data_types/) (enumeration) assigns human-readable names to integer constants. Under the hood, `SCREEN_MAIN` = 0, `SCREEN_FEED` = 1, etc., but we never need to know that.
- `Screen currentScreen = SCREEN_MAIN;` - This variable tracks which screen we're on right now. The program starts on the main screen.

> **Why use a state machine?** Without it, you end up with deeply nested `if` statements that are hard to read and debug. A state machine makes the logic predictable: *"When I'm in state X and event Y happens, go to state Z."* This is the same pattern used in [real embedded systems](https://en.wikipedia.org/wiki/UML_state_machine), game engines, and protocol parsers.

## Step 7: Setup Function

[`setup()`](https://docs.arduino.cc/language-reference/en/structure/sketch/setup/) runs **once** when the board powers on or resets. Use it to initialize hardware and set starting values.

```cpp
void setup() {
  Serial.begin(115200);

  pet.hunger = 80;
  pet.happiness = 80;
  pet.energy = 80;
  pet.age = 0;

  Serial.println("Tamagotchi Initialized");
}
```

**Line-by-line breakdown:**

- `Serial.begin(115200);` - Opens the [serial port](https://docs.arduino.cc/language-reference/en/functions/communication/serial/begin/) at **115200 baud** (bits per second). This is the communication speed between the board and your computer. Both sides must agree on the same baud rate or you'll see garbled text.
- `pet.hunger = 80;` - The dot (`.`) operator accesses a field inside the struct. We start the pet at 80/100 for each stat, healthy but not perfect, so there's room to improve or decay.
- `Serial.println(...)` - Prints a line of text to the **Serial Monitor** (the console window in Arduino IDE). This is our temporary "display" until we connect an OLED screen.

> Since there is no OLED yet, [`Serial`](https://docs.arduino.cc/language-reference/en/functions/communication/serial/) is our display. Open it with **Tools > Serial Monitor** and set the baud rate to **115200**.

## Step 8: Game Update Logic

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

- `unsigned long lastUpdate = 0;` - Remembers the last time we updated the pet. Starts at 0 (boot time).
- `millis() - lastUpdate > 5000` - `millis()` returns the number of milliseconds since the board powered on. By subtracting `lastUpdate`, we get the elapsed time. If more than 5000ms (5 seconds) have passed, it's time to update.
- `pet.hunger--` - The [`--` operator](https://cplusplus.com/doc/tutorial/operators/) decrements by 1. Every 5 seconds, the pet gets slightly hungrier, sadder, and more tired.
- `if (pet.hunger < 0) pet.hunger = 0;` - **Clamping**: we prevent stats from going below zero. Without this, hunger could hit -50, which makes no sense.
- `lastUpdate = millis();` - Reset the timer so the next update happens 5 seconds from *now*.

> **Key concept: non-blocking timing.** This `millis()` pattern is fundamental to embedded programming. It's how you schedule periodic tasks without blocking the CPU. You'll see this exact pattern in [Arduino's BlinkWithoutDelay example](https://docs.arduino.cc/built-in-examples/digital/BlinkWithoutDelay/).

## Step 9: Simulating Button Input (No Hardware)

Since no physical buttons exist yet, we simulate them using the [Serial Monitor](https://docs.arduino.cc/software/ide-v2/tutorials/ide-v2-serial-monitor/). You type a character, the board reads it, and changes state, exactly like a button press would.

```cpp
void checkSerialInput() {
  if (Serial.available()) {
    char input = Serial.read();

    if (input == 'f') {
      currentScreen = SCREEN_FEED;
    }
    else if (input == 'p') {
      currentScreen = SCREEN_PLAY;
    }
    else if (input == 's') {
      currentScreen = SCREEN_SLEEP;
    }
    else if (input == 'm') {
      currentScreen = SCREEN_MAIN;
    }
  }
}
```

**Line-by-line breakdown:**

- [`Serial.available()`](https://docs.arduino.cc/language-reference/en/functions/communication/serial/available/) - Returns the number of bytes waiting in the serial input buffer. If it's 0, no one has typed anything, so we skip the rest.
- [`Serial.read()`](https://docs.arduino.cc/language-reference/en/functions/communication/serial/read/) - Reads **one byte** from the buffer and removes it. We store it as a `char` (a single character).
- `if (input == 'f')` - Single quotes `'f'` denote a [character literal](https://cplusplus.com/doc/tutorial/constants/). We compare the typed character to our command set.
- `currentScreen = SCREEN_FEED;` - This is the state transition. We don't do the feeding here; we just change the state. The actual logic happens in Step 10. This separation is important: **input detection** and **action execution** are different responsibilities.

**Serial Monitor commands:**

| Key | Action | State Transition |
|-----|--------|-----------------|
| `f` | Feed the pet | → `SCREEN_FEED` |
| `p` | Play with pet | → `SCREEN_PLAY` |
| `s` | Put pet to sleep | → `SCREEN_SLEEP` |
| `m` | Return to main | → `SCREEN_MAIN` |

## Step 10: Screen Logic

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

- [`switch(currentScreen)`](https://docs.arduino.cc/language-reference/en/structure/control-structure/switchCase/) - A switch is like a multi-way `if/else`, but cleaner when comparing one variable against many known values. The CPU jumps directly to the matching `case`.
- `pet.hunger += 10;` - The [`+=` operator](https://cplusplus.com/doc/tutorial/operators/) adds 10 to the current value. Feeding restores 10 hunger points.
- `if (pet.hunger > 100) pet.hunger = 100;` - **Upper clamping**: stats can't exceed 100. This is the mirror of the lower clamping in `updatePet()`.
- `pet.energy -= 5;` - Playing costs energy. This creates a **trade-off**: playing makes the pet happier but more tired. Trade-offs make games interesting.
- `currentScreen = SCREEN_MAIN;` - After the action completes, return to main. This makes each action a one-shot: the player must actively choose to do something again.
- `break;` - **Critical!** Without [`break`](https://docs.arduino.cc/language-reference/en/structure/control-structure/break/), execution "falls through" to the next `case`. This is a common C++ bug.

> **Design insight:** Notice that `SCREEN_MAIN` does nothing; it just `break`s. That's intentional. The main screen is a "resting" state where we only display stats. All the action happens in the other states.

## Step 11: Display Function (Serial UI)

This function prints the pet's current stats to the Serial Monitor. It runs every loop cycle, giving us a live dashboard.

```cpp
void render() {
  Serial.println("------");
  Serial.print("Hunger: ");
  Serial.println(pet.hunger);

  Serial.print("Happiness: ");
  Serial.println(pet.happiness);

  Serial.print("Energy: ");
  Serial.println(pet.energy);

  Serial.print("Age: ");
  Serial.println(pet.age);

  Serial.println("------");
}
```

**Key details:**

- [`Serial.print()`](https://docs.arduino.cc/language-reference/en/functions/communication/serial/print/) vs [`Serial.println()`](https://docs.arduino.cc/language-reference/en/functions/communication/serial/println/) - `print()` writes text **without** a newline; `println()` adds one. We use `print()` for the label and `println()` for the value so they appear on the same line: `Hunger: 80`.
- The `"------"` separators create a visual frame so each update is easy to read in the scrolling Serial Monitor output.

> **Phase 2 upgrade:** When we add the OLED display, this function gets rewritten to draw pixels instead of printing text, but the *interface* stays the same: one function called `render()` that shows the current state.

## Step 12: Main Loop

[`loop()`](https://docs.arduino.cc/language-reference/en/structure/sketch/loop/) runs **continuously** after `setup()` finishes. The Arduino framework calls it over and over in an infinite loop. This is where we orchestrate everything.

```cpp
void loop() {
  checkSerialInput();   // 1. Read input
  updatePet();          // 2. Update state over time
  handleScreenLogic();  // 3. Process actions
  render();             // 4. Display results
  delay(1000);          // 5. Wait 1 second
}
```

**Why this order matters:**

```
┌──────────────────────────────────────────────────────┐
│                    GAME LOOP                         │
│                                                      │
│  ┌─────────┐   ┌────────┐   ┌───────┐   ┌────────┐   │
│  │  INPUT   │──▶│ UPDATE │──▶│LOGIC│──▶│ RENDER│   │
│  │ (read    │   │ (decay │   │ (feed,│   │ (print│   │
│  │ serial)  │   │ stats) │   │ play) │   │ stats)│   │
│  └─────────┘   └────────┘   └───────┘   └────────┘   │
│       ▲                                      │       │
│       └──────────── delay(1000) ─────────────┘       │
└──────────────────────────────────────────────────────┘
```

1. **Input** - Check if the player typed a command. This sets `currentScreen` but doesn't act on it yet.
2. **Update** - Decay stats based on elapsed time. This happens regardless of player input.
3. **Logic** - If the player triggered an action (feed/play/sleep), execute it now and return to main.
4. **Render** - Display the results of everything that just happened.
5. **Wait** - `delay(1000)` pauses for 1 second so the Serial Monitor doesn't flood with output.

> **This is a real [game loop](https://gameprogrammingpatterns.com/game-loop.html) pattern.** The same Input → Update → Render cycle is used in professional game engines, RTOS firmware, and GUI frameworks. Learning it here means you already understand the architecture of much larger systems.

---

## Step 13: What Students Just Learned

Without any hardware beyond a USB cable, you now understand:

| Concept | Where You Used It |
|---------|------------------|
| [Structs](https://cplusplus.com/doc/tutorial/structures/) | `Pet` data type (Step 5) |
| [Enums](https://cplusplus.com/doc/tutorial/other_data_types/) | `Screen` states (Step 6) |
| [State machines](https://en.wikipedia.org/wiki/Finite-state_machine) | `currentScreen` transitions (Steps 6, 9, 10) |
| [Non-blocking timing](https://docs.arduino.cc/built-in-examples/digital/BlinkWithoutDelay/) | `millis()` pattern (Step 8) |
| [Serial communication](https://docs.arduino.cc/language-reference/en/functions/communication/serial/) | Input & output (Steps 7, 9, 11) |
| [Switch statements](https://docs.arduino.cc/language-reference/en/structure/control-structure/switchCase/) | Screen logic (Step 10) |
| [Game loop architecture](https://gameprogrammingpatterns.com/game-loop.html) | `loop()` structure (Step 12) |
| Value clamping | Keeping stats in 0 to 100 range (Steps 8, 10) |
| Trade-off design | Playing costs energy (Step 10) |

> That is far more valuable than blinking LEDs.

## Step 14: When Hardware Arrives (Phase 2 Preview)

The brilliant part of this architecture: **swapping in real hardware only changes the edges, not the core.**

| Current (Software Simulation) | Future (Real Hardware) | What Changes |
|-------------------------------|------------------------|--------------|
| `Serial.read()` for input | [GPIO button reads](https://docs.arduino.cc/language-reference/en/functions/digital-io/digitalRead/) | Only `checkSerialInput()` |
| `Serial.print()` for display | [SSD1306 OLED drawing](https://github.com/adafruit/Adafruit_SSD1306) | Only `render()` |
| No feedback | [Piezo buzzer tones](https://docs.arduino.cc/language-reference/en/functions/advanced-io/tone/) | Add to `handleScreenLogic()` |

The `Pet` struct, the state machine, the game loop, the update logic: **none of that changes**. The architecture remains unchanged.

**That is the lesson:** write firmware around *clean abstractions*, and the hardware becomes interchangeable.
