## Button Footprint Mismatch

> **Always verify your footprint before finalizing your PCB layout.** Even if a component looks correct in the schematic, the footprint may have incorrect pad numbering that causes your circuit to fail silently. A common red flag: any pin without a net almost always means something went wrong during import.

---

### Why This Happens

After importing a button from EasyEDA into KiCad, the footprint may look fine at first glance:

![KiCad imported switch footprint](https://cdn.hackclub.com/019d64e0-1e84-77a2-b640-9ae44ec2f111/image.png)

EasyEDA uses a **redundant 4-pin symbol**, showing all four pins separately even though two pairs are internally shorted:

![EasyEDA button symbol](https://cdn.hackclub.com/019d64e1-af44-78ba-9264-cd050bf0913b/image.png)

KiCad uses a **cleaner 2-pin representation**:

![KiCad button symbol](https://cdn.hackclub.com/019d64e4-065a-7b78-947d-76c2c4166b31/image.png)

When imported, EasyEDA's pins 1 and 2 map to opposite footprint pads, but on a tactile switch they're on the **same physical side** (internally shorted). This means pads that should be electrically separated end up sharing a net, and the button does nothing because one side is permanently shorted to itself.

| EasyEDA pins | Physical side | KiCad net |
|:---:|:---:|:---:|
| 1, 2 | Side A (shorted) | Net 1 |
| 3, 4 | Side B (shorted) | Net 2 |

The switch connects **Side A to Side B** when pressed. The footprint must reflect this.

---

### Confirming with the Datasheet

Verify the pinout using the component's datasheet. If sourcing from LCSC, the datasheet is available directly on the product page. For example:

- [Product page](https://www.lcsc.com/product-detail/C2888493.html)
- [Datasheet](https://www.lcsc.com/datasheet/C2888493.pdf)

![Physical pin diagram](https://cdn.hackclub.com/019d64f2-0ab2-7a97-a6a2-c0e4637bd711/image.png)
![Internal schematic](https://cdn.hackclub.com/019d64f2-5c5b-7c82-8d01-eba0fbff8d62/image.png)

These confirm that **pads 1 and 2 are shorted** (Side A) and **pads 3 and 4 are shorted** (Side B).

---

### Fixing the Pad Numbering

Renumber the footprint pads to match the correct mapping:

- Pads **1 and 2** (same physical side) → renumber both to **`1`**
- Pads **3 and 4** (other physical side) → renumber both to **`2`**

1. Double-click the footprint in the PCB editor and click **Edit Library Footprint**.

![Edit Library Footprint dialog](https://cdn.hackclub.com/019d64f5-e570-70fe-a8fb-22ab0854085b/image.png)

2. Double-click each pad hole and update the pad number to match the corrected mapping:

![Corrected pad numbering](https://cdn.hackclub.com/019d64f7-c4ab-78f2-adc6-450d63ee774e/image.png)

3. Press `Ctrl+S` to save.

---

### Applying the Fix

1. Go to **Tools > Update Footprint from Library**.

![Update Footprint from Library menu](https://cdn.hackclub.com/019d64f9-6a84-7663-a0b4-69b215dbb021/image.png)

2. Click **Update**.

The footprint will refresh with the corrected pad numbering: one net on Side A, a separate net on Side B, and the switch bridging them when pressed.

> Does this not work? Try deleting the buttons, and updating from schematic again :)