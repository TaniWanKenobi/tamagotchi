# Tamagotchi PCB Tutorial

> **Note:** Your schematic should NOT look like this at the end of the tutorial! You are required to add your own components and switch it up to personalize it :D

---

## Table of Contents

- [Tamagotchi PCB Tutorial](#tamagotchi-pcb-tutorial)
  - [Table of Contents](#table-of-contents)
  - [Installing KiCad](#installing-kicad)
  - [Creating the Schematic](#creating-the-schematic)
    - [Importing the MCU](#importing-the-mcu)
    - [Adding the OLED Display](#adding-the-oled-display)
    - [Adding Buttons](#adding-buttons)
    - [Adding a Buzzer](#adding-a-buzzer)
    - [Assigning Footprints](#assigning-footprints)
  - [Creating the PCB](#creating-the-pcb)
    - [Laying Out Components](#laying-out-components)
    - [Defining the Board Outline](#defining-the-board-outline)
  - [PCB Routing](#pcb-routing)
    - [Placing Components](#placing-components)
    - [Routing Traces](#routing-traces)
  - [Customization](#customization)
  - [Run Design Rules Check](#run-design-rules-check)
    - [Common DRC Errors](#common-drc-errors)
    - [Late Additions](#late-additions)
  - [Add Your Files to Your GitHub Repo](#add-your-files-to-your-github-repo)
  - [Upload Your Files to GitHub](#upload-your-files-to-github)
    - [Edit Your README](#edit-your-readme)
  - [Getting a JLCPCB Price](#getting-a-jlcpcb-price)
    - [Settings](#settings)
  - [Designing the Case](#designing-the-case)

---

## Installing KiCad

Download KiCad from the official site: **[https://www.kicad.org/download/](https://www.kicad.org/download/)**

1. Go to the download page and select your operating system (Windows, macOS, or Linux).
2. Download the latest **stable release** (KiCad 9.x).
3. Run the installer and follow the on-screen instructions (the defaults are fine).
4. When prompted, make sure to **install the default libraries** (they're selected by default).
5. Launch KiCad and create a new project (**File → New Project** or **Ctrl+N**).

<details>
<summary>What are the default libraries?</summary>

KiCad ships with a large set of schematic symbols, PCB footprints, and 3D models. These cover most common components (resistors, capacitors, connectors, etc.) so you don't have to create them from scratch. You'll still need to import specialty parts (like the XIAO or OLED), but the defaults cover the basics.

</details>

> **New to KiCad?** Read the official getting started guide before continuing:
> [KiCad 9.0: Getting Started](https://docs.kicad.org/9.0/en/getting_started_in_kicad/getting_started_in_kicad.html)

---

## Creating the Schematic

The XIAO-ESP32-C6 will be used as the MCU! This is because it's tiny and includes WiFi, Bluetooth, and built‑in battery charging, which makes it perfect for this.

<details>
<summary>What is an MCU?</summary>

MCU stands for **Microcontroller Unit**, the tiny computer (brain) of your project. It runs your code and controls all the other components (display, buttons, buzzer, etc.). The XIAO-ESP32-C6 is the MCU we're using here.

</details>

<details>
<summary>Useful schematic editor keybinds</summary>

| Key | Action |
|-----|--------|
| **A** | Add a symbol (component) |
| **W** | Draw a wire |
| **P** | Add a power symbol (VCC, GND, 3V3, etc.) |
| **L** | Add a net label |
| **M** | Move a component |
| **R** | Rotate a component |
| **G** | Grab/drag a component (keeps wires attached) |
| **E** | Edit component properties |
| **C** | Copy a component |
| **Delete** | Delete selected item |
| **Ctrl+Z** | Undo |
| **Ctrl+S** | Save |

</details>

### Importing the MCU

First, I imported the XIAO-ESP32-C6 from the [OPL Library](https://github.com/Seeed-Studio/OPL_Kicad_Library/tree/master). Here's an [excellent tutorial](https://www.youtube.com/watch?v=lnCvyGIQong) on using it!

<!-- I will create my own video later, which may be easier to use, if trouble is reported understanding it. I think beginners may struggle. -->

I then clicked **A** (Add Symbol), searched for the part, and imported it. I ended up with this:

![Xiao](https://cdn.hackclub.com/019c6725-d62c-77f5-beae-23fda60a21ef/image.png)

Now, for this tutorial, I will create the base Tamagotchi. Spice your own Tamagotchi up more than this one! Add some flavor!

### Adding the OLED Display

I then added the [0.96" OLED](https://www.lcsc.com/product-detail/C5248080.html) via EasyEDA to KiCad. Follow this [guide](https://hwdocs.hackclub.dev/user-contrib-guides/easyeda2kicad/)!

I connected GND to GND, SDA to SDA, and SCL to SCL. Use **W** to draw wires between pins.

<details>
<summary>What are SDA, SCL, and GND?</summary>

- **GND** = Ground, the common reference point for all electrical signals.
- **SDA** = Serial Data, the line that carries data back and forth over I2C.
- **SCL** = Serial Clock, the line that provides the timing signal for I2C communication.

I2C is a protocol that lets multiple devices talk over just two wires (SDA + SCL).

</details>

<details>
<summary>Do I need pull-up resistors on SDA and SCL?</summary>

Normally yes. I2C requires pull‑up resistors on SDA and SCL; however, the XIAO boards already include internal pull‑ups, so you don't need external ones here.

</details>

I checked the [datasheet](https://www.lcsc.com/datasheet/C5248080.pdf) attached on the LCSC page and found that VCC should be connected to 3V3.

![](https://github.com/user-attachments/assets/15b279c4-acc2-40db-b04b-2e664e778855)

### Adding Buttons

Since a Tamagotchi has three buttons, I decided to add those as well!

For this, I decided to use an **active‑low** button layout (which is the most common approach).

Each button connects one side to **GND** and the other to a **GPIO pin** with the XIAO's **internal pull‑up** enabled. This makes the pin read **HIGH** when the button is idle, and **LOW** when the button is pressed.

<details>
<summary>What does "active-low" mean?</summary>

"Active-low" means the button is considered "pressed" when the signal goes LOW (0V / GND). When the button is not pressed, the internal pull-up resistor keeps the pin HIGH (3.3V). This is the most common button wiring approach because it's simple: you only need the button and a ground connection, no extra resistors.

</details>

<details>
<summary>What is a GPIO pin?</summary>

GPIO stands for **General Purpose Input/Output**. These are the programmable pins on your microcontroller that can be configured as either inputs (to read sensors/buttons) or outputs (to drive LEDs/buzzers). On the XIAO, pins like D0–D10 are GPIOs.

</details>

### Adding a Buzzer

I also added a buzzer from [here](https://www.lcsc.com/product-detail/C49246964.html)!

The buzzer is wired with one pin to a GPIO output and the other to GND, letting the GPIO drive it with a square‑wave tone.

![](https://github.com/user-attachments/assets/68f89dc9-19ea-4992-93f8-83674963e40d)

### Assigning Footprints

This is my final schematic! Let's make the PCB now. First, let's assign parts. Click this:

![](https://github.com/user-attachments/assets/7fdab501-5c92-4df0-b032-ab40fdc7018a)

Then assign the corresponding EasyEDA / OPL parts! For the buttons, import from [here](https://www.lcsc.com/product-detail/C2888493.html)!

![Footprint assignment](https://github.com/user-attachments/assets/cbc73d30-a61e-4a5a-912f-915604c71418)

Now, press this and open the PCB viewer:

![Open PCB viewer](https://github.com/user-attachments/assets/5bbc50fa-dbe2-4ab1-b301-5c360a974529)

---

## Creating the PCB

**The maximum size of your PCB should be 100mm²!**

You should see something like this:

![PCB editor](https://github.com/user-attachments/assets/1cf53978-42d2-49b6-ab12-837e863fc6ed)

To synchronize changes between your schematic and PCB layout in KiCad:

- Press **F8**, *or*
- Click the **Update PCB from Schematic** button

You can do this anytime you want to refresh the PCB with the latest schematic updates.

### Laying Out Components

First, I laid my PCB out in a 100×100 box. I then laid out all my components:

![Component layout](https://cdn.hackclub.com/019c545b-804f-7cba-a7bb-06a9ee3376d9/image.png)

### Defining the Board Outline

You'll notice a white outline surrounding the entire board. This is the **Edge Cuts** layer, which defines the physical boundary where the PCB will be cut.

This is done by modifying the `Edge.Cuts` layer on the right side. There are many ways to do this, such as manually drawing it with the given menu.

I found the default outline options limiting and wanted to create something more complex. To do this, I:

1. Drew out the edge cuts layer.
2. Converted the image into a DXF file using an image-to-DXF converter.
3. Imported the DXF file into KiCad.
4. Created a 100×100 mm box as a reference.
5. Used the measuring tool to determine the correct scale.
6. Scaled the outline down so it fit within 100 mm.

---

## PCB Routing

A PCB is made up of multiple layers. Our boards are "two-layer," meaning that they have two layers of copper wire.

<details>
<summary>What are the PCB layers?</summary>

| Layer | Description |
|-------|-------------|
| **Top & bottom silkscreen** | The white ink layer where you can add art |
| **Top & bottom copper** | The layers where you make your copper wires |
| **Substrate** | The actual plastic (usually green) which makes up your board |
| **Via** | The tunnels which connect the top and bottom copper layers |

![PCB layers](https://cdn.hackclub.com/019c545f-32f8-71b8-9b46-079aee118944/c2ec73f247fdb1f466903fc86d345fe0f4b47b6f_image.webp)

</details>

### Placing Components

Place all of your components inside the board outline. Move components to shorten **ratlines**, which are the straight blue lines.

- **M**: Move a component
- **R**: Rotate a component
- **F**: Flip a component to the other side of the board
- **Ctrl+S** / **⌘+S**: Save (do this often!)

<details>
<summary>What are ratlines?</summary>

Ratlines (also called "ratsnest lines") are the thin straight lines that show unrouted connections between pads. They indicate which pads need to be connected with copper traces. Your goal is to arrange components so these lines are as short as possible and don't cross each other, which makes routing much easier.

</details>

### Routing Traces

Now it's time to route the PCB! Hit **X** on your keyboard and click anything with a thin blue line poking out of it. It should dim the entire screen, show you which direction you need to go with a thin blue line, and highlight the destination:

![Routing example](https://cdn.hackclub.com/019c5460-5bb4-737a-8875-2d15ef8f3715/image.png)

**Key routing shortcuts:**

- **X**: Start routing a trace
- **V**: Add a via (switch trace to the other side of the board mid-route)
- **Backspace**: Delete the last trace segment while routing
- **Esc**: Cancel the current route
- **D**: Drag a trace (reposition it without breaking connections)
- **U**: Select the entire trace from the point you click

Join the highlighted points together. If there isn't enough space on the front side, or there is a trace already present that is blocking you, you can route on the back side by clicking **B.Cu** on the right toolbar. If you want to change sides during routing, press **V** and a via will be added, which will transfer your trace to the other side of the board.

> **Important:** Wires and pads of different colors (except golden) can't be connected together directly! You must use a via to the other side.

<details>
<summary>What is a via?</summary>

A via is a small plated hole that connects a copper trace on one layer of the board to a trace on another layer. Think of it as a tiny tunnel through the PCB. You use them when you can't route a trace on one side because another trace is in the way. Press **V** while routing to drop a via and continue the trace on the other side.

</details>

Your routing is complete!



![Ground plane](https://cdn.hackclub.com/019c5461-63bf-785e-915e-34c92be3fb2b/image.png)

> **Tip:** Place everything based on what shortens the blue lines, and what makes them not cross!

> **Tip:** Use a [ground plane](https://www.kicadtips.com/how-to/make-a-ground-plane) to help with routing and to reduce noise. That's what the red and blue layers are for! It's not necessary, but looks nice and is easy to set up!

> **Note:** You may notice [stitching vias](https://resources.altium.com/p/everything-you-need-know-about-stitching-vias) in my board (the small holes scattered across the ground plane). **You do NOT need to add these.** They are completely optional and unnecessary for a board like this. I only added them for fun.
>
> ![Stitching vias](https://cdn.hackclub.com/019c546b-8acc-7d6c-9c2b-b1a579d315ef/image.png)

---

## Customization

You may have already added some text and art to customize your board like me. If not, you can click **F.Silkscreen** and use the text tool.

![Sample text](https://cdn.hackclub.com/019c546c-c7b1-7df9-b96b-68dae071b550/image.png)

To add art, select the **Top Silkscreen Layer** or **Bottom Silkscreen Layer** in the sidebars, then use the KiCad image converter to add custom art.

![KiCad image converter](https://kicad-info.s3.dualstack.us-west-2.amazonaws.com/original/3X/a/3/a3b5fba9b9455697b0d861d48a028c571ec44403.png)

Your board is now beautiful! 

---

## Run Design Rules Check

**DRC** stands for **Design Rules Check**. This runs a check that makes sure your board has no interference errors, no components are off the board, and no wires are intersecting. It does **not**, however, confirm that your board works.

![DRC](https://cdn.hackclub.com/019c5876-0777-7c3f-9817-931007221588/27ae0765082623ea2988bbe01ce8cb8a4012b0b8_image.webp)

Using the output, correct any errors. This can be confusing, so remember: you can always ask for help!

<details>
<summary>What are common DRC errors?</summary>

- **Track and copper errors** : clearance violations, track width, annular rings
- **Via errors** : diameter, micro vias, blind/buried vias
- **Pad and footprint errors** : pad-to-pad, hole clearances
- **Edge and board outline errors** : copper edge clearance, silkscreen issues
- **Zone errors** : copper slivers, starved thermals, unconnected items
- **Net and connection errors** : missing connections, net conflicts
- **Courtyard errors** : overlaps, missing courtyards

</details>

Once your PCB passes the DRC, it is finished!

In the PCB editor, click **View → 3D Viewer** to see your finished work!

![3D Viewer](https://cdn.hackclub.com/019c5472-6d23-70be-b813-a9eb320bb4e3/image.png)

### Late Additions

There will be moments when you realize you want to add something after you've done most of the work. That's totally fine!

Here, I decided to add battery functionality. Since the XIAO's underside pins are extremely difficult to solder, I designed the board so those pins connect to a pad instead. The battery can then plug into a set of male pins that interface with that pad. Here's what it looks like:

![Battery schematic](https://cdn.hackclub.com/019c5518-d617-7c27-b74c-7047a566dc60/image.png)

Here's my final schematic:

![Final Schematic](https://cdn.hackclub.com/019c551a-0c44-74de-b335-e5125c46b190/image.png)

Here's my final PCB:

![PCB](https://cdn.hackclub.com/019c5520-3b13-777e-a4fe-af7fee4a87db/image.png)

![3D Model Front](https://cdn.hackclub.com/019c5521-3bb5-7571-b746-06b7ccf2c9c5/image.png)

![3D Model Back](https://cdn.hackclub.com/019c5521-e2f7-7c83-8f44-04fc46b14a6a/image.png)

---

## Add Your Files to Your GitHub Repo

Now it is time to order your board. Get the following files from your project:

- **A screenshot of your 3D view**
  - In PCB Editor: **View → 3D Viewer → Edit → Copy 3D Image**
- `.kicad_pro` (KiCad project file)
- `.kicad_sch` (schematic)
- `.kicad_pcb` (PCB layout)
- **Your Gerber files** (see below)

<details>
<summary>How do I export Gerber files?</summary>

1. In your PCB editor: **File → Fabrication Outputs → Gerbers (.gbr)**
2. Set an output folder (e.g., a new "Gerbers" folder)
3. Select necessary layers (generally already selected)
4. Click **Plot**
5. Click **Generate Drill Files**
6. Zip the resulting files for your manufacturer

Gerber files are the industry-standard format that PCB manufacturers use to fabricate your board. They contain the copper layers, silkscreen, solder mask, drill locations, and board outline.

</details>

---

## Upload Your Files to GitHub

Go back to the GitHub repo you created at the start. Click **Add File → Upload files**.

Drag in your:

- Screenshot of your 3D view
- `.kicad_pro` (KiCad project file)
- `.kicad_sch` (schematic)
- `.kicad_pcb` (PCB layout)
- Gerbers (zipped)

You should have downloaded all of these in the previous step.

Click to commit your changes.

### Edit Your README

Finally, edit your README to include the submission requirements:

- A short description of what your project is
- A couple sentences on why you made the project
- A couple sentences on how to use your project

**Pictures of your project:**

- A screenshot of a full 3D model of your project
- A screenshot of your PCB, if you have one
- A wiring diagram, if you're doing any wiring that isn't on a PCB

**A BOM (Bill of Materials) in table format at the end of the README, with links.**

> **NOTE:** ALL projects you make for Blueprint must have a project photo in your README.

---

## Getting a JLCPCB Price

Go to [jlcpcb.com](https://jlcpcb.com/) and make an account. Then, add your Gerber file for the instant quote.

![JLCPCB upload](https://cdn.hackclub.com/019c5535-24e8-75f6-b7c0-adec3e312dc5/baaa0ca887d51110c30cba9d862968acbef618f8_image.webp)

### Settings

You should keep the default settings for everything. The only thing you should/can change is the **PCB Color**. I did black as seen below:

![PCB color](https://cdn.hackclub.com/019c5535-26c4-788a-8508-d053d383fd8d/2b2ea8e606d05ddff382fdcc7934da4bff70615c_image.webp)

For high-spec options, also keep the default. **Do not** click PCB assembly. We will give you a kit to hand-solder your board.

![High-spec options](https://cdn.hackclub.com/019c5535-28a4-787e-8bf4-ba4fcbce1653/053061912ca84c66c46323ccef5b12cb71c7d721_image.webp)

Once you have successfully *not* changed any of the settings (except the board color), on the right, change the shipping method to **Global Standard Direct** (or **Air Registered Mail** if it is cheaper), and take a screenshot (this is very important).

![Shipping](https://cdn.hackclub.com/019c5535-2ab1-7ba1-aa97-7d7a87e4203a/6f538d5f301eae997544c43c1d6ce6daca01331d_image.webp)

---

## Designing the Case

Export the PCB from KiCad into your CAD platform! For me, this is Onshape.

I googled and found headers! I found this from [here](https://www.3dcontentcentral.com/download-model.aspx?catalogid=171&id=1025256)!

I then experimented with the CAD a lot. I used revolve, but it did not look nice and was hard to use. I also tried multiple placements for the battery and found it was optimal if it's placed vertically.

To make these, I created a sketch by clicking the sketch button in the upper-left-hand side and made multiple shapes that include tolerance and my shell size.

![](https://cdn.hackclub.com/019c5c8f-7d48-7dc5-9f80-8dcef93f84e0/image.png)

![](https://cdn.hackclub.com/019c5985-794b-78d8-bbec-b885edabe178/image.png)

![](https://cdn.hackclub.com/019c5983-207f-747f-b33a-d92806aac17e/image.png)

I also decided to change the PCB to add the vibration motor to the back and raise the screen!

From

![](https://cdn.hackclub.com/019c59d1-a434-7b99-a5bf-7d486ab388f6/image.png)

to

![](https://cdn.hackclub.com/019c59cf-3ea1-7c97-8367-5ef8e0558b2a/image.png)

My headers overlapped with the buzzer, so I decided to reposition the buzzer to the middle of the board! The gap between the LCD and the headers should give me enough space!

![example](https://cdn.hackclub.com/019c5a0c-fc35-7127-b439-55fb132f13c3/image.png)

I made the back portion with 3D-printed standoffs and added large fillets to round the edges. I made the top part and added some tolerance between the top of the plate and the bottom of the top half of the case!

![](https://cdn.hackclub.com/019c5c85-9499-7b71-b5ed-3d3325909b62/image.png)

I added a hole for the battery and a hole for the USB-C. Generally, there may be issues printing this, but for this small amount of bridging there shouldn't be any issues!

![](https://cdn.hackclub.com/019c5c85-ee74-7721-ab42-03937224ebe6/image.png)

![](https://cdn.hackclub.com/019c5c87-d8d2-7de5-bc94-b7690c845c4c/image.png)

Finally, I used the section view tool to confirm that there is no overlapping:

![](https://cdn.hackclub.com/019c5c89-1791-78d6-9b35-902744dcb72a/image.png)

Now, my design is finished! I uploaded all necessary files to my GitHub repo.

**Note: Design is an iterative process. I spent about 20 hours manually adjusting everything to look as pretty as possible! But don't worry, #fallout is here to guide you! This is a normal process and you are not alone.**

I recommend **not** following the above 1:1; this was my process, and it will differ greatly from yours!

I also recommend checking my [Onshape](https://cad.onshape.com/documents/fa5791d8e7f345b436054923/w/ad8e09905eb572b0a8a40e9d/e/e81f06aa82a76dd04f8e6832?renderMode=0&uiState=6990af61d8ec4667253f72ea) for inspiration! Check my feature tree and see my iterations yourself.

Enjoy making :D!



