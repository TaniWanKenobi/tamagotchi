## Creating the Schematic

> Note: Your schematic should NOT look like this at the end of the tutorial! You are required to add your own components, and switch it up to personalize it :D 

The XIAO-ESP32-C6 will be used as the MCU! This is because it’s tiny and  includes WiFi, Bluetooth, and built‑in battery charging, which makes it perfect for this.

First, I imported the XIAO-ESP32-C6 from [OPL Library](https://github.com/Seeed-Studio/OPL_Kicad_Library/tree/master). Heres an [excellent tutorial](https://www.youtube.com/watch?v=lnCvyGIQong) on using it! 

<!-- I will create my own video later, which may be easier to use, if trouble is reported undestanding it. I think beginners may struggle. -->

I then click A, search for the part, and import it. 

I ended up with this!

<img width="790" height="500" alt="image" src="https://github.com/user-attachments/assets/41a97672-ab63-43bd-876a-20fb04e2273d" />

Now, for this tutorial I will create the base tamagotchi. Spice your own tamagotchi up more then this one! Add some flavor!

I then added the [0.96 OLED](https://www.lcsc.com/product-detail/C5248080.html) via EasyEDA to KiCAD. Follow this [guide](https://hwdocs.hackclub.dev/user-contrib-guides/easyeda2kicad/)! 


I connected GND to GND, SDA to SDA, and SCL to SCL. 

> Normally you need pull‑up resistors on SDA and SCL, but the XIAO boards already include internal pull‑ups, so you  don’t need external ones here.

I checked the [datasheet](https://www.lcsc.com/datasheet/C5248080.pdf) attached in the LCSC link, and found that VCC should be connected to 3v3. 

<img width="1008" height="54" alt="image" src="https://github.com/user-attachments/assets/15b279c4-acc2-40db-b04b-2e664e778855" />

Since a tamagatchi has 3 buttons, I decided to add those as well!

For this, I decided to use an **active‑low** button layout (which is the most common approach).

Each button connects one side to **GND** and the other to a **GPIO pin** with the XIAO’s **internal pull‑up** enabled.  
This makes the pin read **HIGH** when the button is idle, and **LOW** when the button is pressed.

I also added a buzzer from [here](https://www.lcsc.com/product-detail/C49246964.html)! 

The buzzer is wired with one pin to a GPIO output and the other to GND, letting the GPIO drive it with a square‑wave tone.

<img width="1263" height="584" alt="image" src="https://github.com/user-attachments/assets/68f89dc9-19ea-4992-93f8-83674963e40d" />

This is my final schematic! Lets make the PCB now. First, lets assign parts. Click this:

<img width="236" height="146" alt="image" src="https://github.com/user-attachments/assets/7fdab501-5c92-4df0-b032-ab40fdc7018a" />

and then assign the corresponding EasyEDA / OPL parts! For the buttons, import from [here](https://www.lcsc.com/product-detail/C2888493.html)!

<img width="963" height="178" alt="image" src="https://github.com/user-attachments/assets/cbc73d30-a61e-4a5a-912f-915604c71418" />

Now, press this and open the PCB viewer!

<img width="55" height="54" alt="image" src="https://github.com/user-attachments/assets/5bbc50fa-dbe2-4ab1-b301-5c360a974529" />


You should see something like this:

<img width="808" height="674" alt="image" src="https://github.com/user-attachments/assets/1cf53978-42d2-49b6-ab12-837e863fc6ed" />



## Create a Printed Circuit Board (PCB)

To synchronize changes between your schematic and PCB layout in KiCad:

- Press **F8**
  *or*
- Click the **Update PCB from Schematic** button

You can do this anytime you want to refresh the PCB with the latest schematic updates.

First, create a board outline. For this tutorial, **YOU NEED TO CUSTOMIZE YOUR BOARD WITH A CUSTOM OUTLINE AND ART**

This is done by modifying the `Edge.Cuts` layer on the right side.

There are many ways to do this. You can either manually draw it with the given menue.

I found the default outline options limiting and wanted to create something more complex.
To do this, I:

1. Took a Batman image.
2. Converted the image into a DXF file using an image-to-DXF converter.
3. Imported the DXF file into KiCad.
4. Created a 100x100 mm box as a reference.
5. Used the measuring tool to determine the correct scale.
6. Scaled the outline down so it fit within 100 mm.

At the end, it may look something like this:

![](/old-cdn/06ae2032f870f668a088dafecf0d36b64cacef1e_image.webp)

**Make sure to always keep your board below 100x100mm!**

## PCB Routing

A PCB is made out of multiple layers. Our boards are “two layer” meaning that they have two layers of copper wire.

The layers include:

Top and bottom solder mask: the white ink where you can do art

Top and bottom copper layer: the layers where you make your copper wires

Substrate: The actual plastic (usually green) which makes up your board

Via: the tunnels which connect the top and bottom copper layers

![](/old-cdn/c2ec73f247fdb1f466903fc86d345fe0f4b47b6f_image.webp)

Place all of your components inside the Board Outline. Move components to shorten ratlines, which are are the straight blue lines.

- You can use ‘r’ to rotate them
- Remember to save (Control-S or  ⌘-S) often !!!!

Now it's time to route the PCB! Hit X on your keyboard and hit anything with a thin blue line poking out of it. It should dim the entire screen, show you which direction you need to go with a thin blue line and highlight the destination:

![example here](/old-cdn/e56026795ef53593ec7e75e329f0bc7c9c9d71b6_image.webp)

Join the highlighted points together. If there isn't enough space on the front side, or there is a trace already present that is blocking you, you can route on the back side by clicking B.Cu on the right toolbar. At the same time, if you want to change sides during routing, press V and a via shall be added, which will transfer your trace to the other side of the board. **Wires and pads of different colors (except golden) can't be connected together directly! You must via to the other side.**

![](/old-cdn/94a3f8fd9531b47e0f5322ea7bf1c76724ad1c82_image.webp)

Your routing is complete!

> Tip: place everything based on what shortens the blue lines, and what makes them not cross!

> Tip 2: Use a [ground plane](https://www.kicadtips.com/how-to/make-a-ground-plane) to help with routing and to reduce noise. Thats what the red and blue layers are for! Its not necessary, but looks nice and is easy to set up!

![](https://files.catbox.moe/cvtxud.png)

## Customization

You may have already added some text and art to customize your board. if not, you can click “F.Silkscreen” and use the text tool.

![sample text](/old-cdn/40af4f6bb85a2aadc2e3791e28f2e6e84ed8eb56_image.webp)

To add art, just select the “Top Silkscreen Layer” or “Bottom Silkscreen Layer” in the sidebars. Then you use the kicad image converter to add custom art.

![](https://kicad-info.s3.dualstack.us-west-2.amazonaws.com/original/3X/a/3/a3b5fba9b9455697b0d861d48a028c571ec44403.png)

Your board is now beautiful

![](https://files.catbox.moe/hycsid.png)

## Run Design Rules Check

**DRC** stands for Design Rules Check. This runs a script which makes sure that your board has no interference errors, no components are off the board, and no wires are intersecting. It does not however confirm that your board works.

![](/old-cdn/27ae0765082623ea2988bbe01ce8cb8a4012b0b8_image.webp)

Using the output, correct any errors. This can be confusing, so remember: you can always ask for help!

Here are some common errors:

- Track and copper errors (clearance violations, track width, annular rings)
- Via errors (diameter, micro vias, blind/buried vias)
- Pad and footprint errors (pad-to-pad, hole clearances)
- Edge and board outline errors (copper edge clearance, silkscreen issues)
- Zone errors (copper slivers, starved thermals, unconnected items)
- Net and connection errors (missing connections, net conflicts)
- Courtyard errors (overlaps, missing courtyards)

Once your PCB passes the DRC, it is finished!

In PCB editor click View > 3D Viewer to see your finished work!

![](https://files.catbox.moe/8a6auv.png)

## Add Your Files to Your GitHub Repo

Now it is time to order your board.

Get the following files of your project:

- A screenshot of your 3d view 
  - In PCB Editor: View → 3D Viewer → Edit → Copy 3D Image.
- .kicad_pro (KiCad project file)
- .kicad_sch (schematic, if you used KiCad)
- .kicad_pcb (PCB, if you used KiCad)
- Your Gerber files:
  1. In your PCB editor do File → Fabrication Outputs → Gerbers (.gbr)
  2. Set an output folder (e.g., a new "Gerbers" folder)
  3. Select necessary layers (Generally already selected)
  4. Click Plot
  5. Click Generate Drill Files
  6. Zip the resulting files for your manufacturer

## Upload Your Files to GitHub

Go back to the GitHub repo you created at the start.

Click Add File → Upload files

![](/old-cdn/470145098fd2379b9385c1f49ddf795fbadfa1a3_image.webp)

Drag in your:


- Screenshot of your 3d view 
- .kicad_pro (KiCad project file)
- .kicad_sch (schematic, if you used KiCad)
- .kicad_pcb (PCB, if you used KiCad)
- Gerbers 

(you should have downloaded all of these before)

![](/old-cdn/8116375daf649aa342be5d409e010a5769f5ed81_image.webp)

You can then click to commit your changes.

Finally, edit your ReadME
![](/old-cdn/383215c7b9b871788025c40fd95b2cab387938d2_image.webp)

NOTE: ALL projects you make for Blueprint must have a project photo in your ReadME.

## Getting a JLCPCB Price

Go to [https://jlcpcb.com/](https://jlcpcb.com/) and make an account. Then, add your Gerber file for the instant quote.

![](/old-cdn/baaa0ca887d51110c30cba9d862968acbef618f8_image.webp)

Settings:

You should keep the default settings for everything. The only thing you should/can change is the PCB Color. I did black as seen below:

![](/old-cdn/2b2ea8e606d05ddff382fdcc7934da4bff70615c_image.webp)

For high-spec options, also keep the default. Do not click PCB assembly as we will give you a kit to hand-solder your board.

![](/old-cdn/053061912ca84c66c46323ccef5b12cb71c7d721_image.webp)

Once you have successfully *Not* changed any of the settings (except the board color), on the right, change the shipping method to Global Standard Direct (or Air Registered Mail if it is cheaper), and take a screenshot (this is very important).

![](/old-cdn/6f538d5f301eae997544c43c1d6ce6daca01331d_image.webp)

## Submitting Your Blueprint Project to Get Funding

You are almost done! At this stage you should have:

- A completed board
- A GitHub repo for your board
- A Blueprint project for your board
- A price estimate and screenshot for your board

Go back to your Blueprint project screen from before. You should edit the project so the banner is your 3D render.

![](/old-cdn/a73a0deeb8dbf8e604f0ebfda1f95f6f6ccd6daa_image.webp)

Now, click “Ship It”

![](/old-cdn/6d1688e2383ca416db27b57942698e7f24a88da9_image.webp)

Blueprint will run some checks. If any are red, you need to fix them. (you may need to upload your project banner)

![](/old-cdn/0e646bc3c854746bfd51172fa68cdba8a9df1625_image.webp)

Enter the dollar amount which you previously screenshotted on JPCLCB (don’t worry, we will give you extra for any fluctuations).

![](/old-cdn/22dfe9ac0998e14b4f9418b118f37eeee29a5561_image.webp)

Click “No” for 3d print

![](/old-cdn/cd0755158d34ae939d044a3d350e0ca6c4cb6ea7_image.webp)

Upload your JLCPCB screenshot from earlier

![](/old-cdn/0aaa8f8f94dab1567dfdaff06ca5ac755b07f131_image.webp)

Check your project…. and ship!

![](/old-cdn/9306e1f8bb9e33f20e757051d90f47fe1f8035e0_image.webp)

You may need to verify your Hack Club identity if you have not already.

You are done!

You should wait for  a reviewer to approve your project! Once it is approved, you can complete the checkout on JLCPCB (making sure to use Global Standard Direct (or Air Registered Mail if it is cheaper), and your kit/soldering iron will be sent to you!

While you wait….

Check out more Blueprint projects on [https://blueprint.hackclub.com/explore](https://blueprint.hackclub.com/explore)! You can also make any hardware project you want on Blueprint, and get up to $400 to make it. For future Blueprint projects, you will need to make a journal throughout your development process. Again, if you have any questions, ask in #blueprint-suppport on Slack.


