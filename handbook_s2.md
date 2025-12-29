# **Handbook: Rapid Prototyping Challenge Session 2**

**Project:** PCB Designing of The Intelligent Toll Gate System 

### **1. The Tool: EasyEDA**

For the scope of this **Rapid Prototyping Challenge**, we will strictly use **EasyEDA**. It is browser-based, fast, and perfect for getting designs done quickly.

### **2. Component Selection (The Shortcut)**

Normally, you would search for components in the EasyEDA/LCSC library (which has 100k+ parts).

* **However, to save time:** We will provide you with a specific **Component Directory/File**.
* This file contains all the correct Symbols, Footprints, and 3D views with labels you need.
* **Action:** Just load these pre-verified components into your project. Don't waste time searching!



### **3. The Workflow (3 Simple Steps)**

**Step 1: Schematic (The Logic)**

* Drag and drop components from the provided file.
* Connect pins using wires.
* **Tip:** Use "Net Labels" (e.g., `5V`, `GND`) to keep connections clean without drawing messy wires everywhere.

**Step 2: Convert to PCB**

* Click "Convert to PCB".
* Arranging components is key: Keep related parts close together (e.g., crystal near the chip).

**Step 3: Routing (The Golden Rules)**
Follow these rules to ensure your board works and wins the challenge:

* **No 90° Angles:** Always use **45°** turns for your tracks.
* **Proper Gaps:** Ensure there is enough empty space (clearance) between two tracks so they don’t accidentally touch during manufacturing.
* **Leverage Both Layers:**
* **Top Layer:** Start routing here.
* **Bottom Layer:** If a route is blocked by other tracks on the Top layer, don't panic.
* **Use Vias:** Drop a **Via** (a small hole) to jump your track from the Top layer to the Bottom layer, continue the route under the blockage, and jump back up if needed.





### **4. Understanding Layers (EasyEDA Colors)**

| Layer | EasyEDA Color | Function |
| --- | --- | --- |
| **Top Layer** | **Red** | Top wiring (copper tracks). |
| **Bottom Layer** | **Blue** | Bottom wiring (copper tracks). |
| **Top SilkLayer** | **Yellow** | Text and component outlines (the printing). |
| **Multi-Layer** | **Silver/Gray** | Pads and holes that go through the board. |



### **5. The Challenge Format (On-Campus)**

* **The Task:** You have to design the PCB from scratch using the concepts learned.
* **Our Support:** While you design, we will be designing the same PCB live on the big screen at a **slow pace** to guide you if you get stuck.
* **Winning Criteria:** The winner will be decided based on:
  1. **Best Design Quality** (Clean routing, compact placement... ).
  2. **Time Taken** (Speed matters!).





#### **6. Resources**

* 🔗 **Session Recording:** [Link to be added]
* 📂 **EasyEDA Project File:** [Link to be added]
* 📘 **PCB Design Guidebook (PDF):** [Link to be added]

---

### **Credits:** Rishab Dhama, Krishn Singh
