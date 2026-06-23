# HAND-E: Robotic Hand Exoskeleton

## Overview

HAND‑E is a tendon‑driven anthropomorphic robotic hand engineered to replicate natural human finger motion through a real‑time sensing‑to‑actuation software pipeline.  
At its core, HAND‑E is a software‑defined robotic mechanism: every motion, every response, and every behaviour is governed by embedded algorithms designed for stability, noise‑resilience, and biomimetic performance.

The system integrates:
- **Flex‑sensor‑based motion capture**  
- **Real‑time embedded signal processing** (moving average filtering, hysteresis, per‑finger calibration)  
- **Servo‑tendon actuation** with inversion logic and torque‑aware mapping  
- **Iterative CAD modelling** for biomimetic joint geometry  
- **A custom PCB** optimised for clean analogue sensing  

Unlike a typical student project, HAND‑E was built end‑to‑end as a complete robotics system:

- **Concept → sketches → CAD → 3D‑printable mechanism**  
- **Electronics → PCB → embedded firmware → control algorithms**  
- **Mechanical v1 → analysis → redesign → v2 with torsion springs and 95° joint rotation**  

The result is a compact, modular platform for exploring biomimetic manipulation, human‑to‑robot motion mapping, and tendon‑driven actuation — the same principles used in modern humanoid robotics research.

---
## Project Goals

HAND‑E was designed as a full-stack robotics system with a strong emphasis on software-driven control, biomimetic motion, and research‑level engineering principles.  
The primary goals of the project were:

### **1. Develop a real-time sensing → actuation software pipeline**
Create a stable embedded architecture capable of:
- reading noisy analogue flex sensors  
- filtering signals using a circular buffer + moving average  
- applying hysteresis to prevent jitter  
- mapping calibrated sensor ranges to servo angles  
- driving a multi-channel PCA9685 PWM controller  
- maintaining consistent 50 Hz actuation timing  

This pipeline forms the “brain” of the robotic hand.

### **2. Build a tendon‑driven anthropomorphic hand with biomimetic joint behaviour**
Design a mechanical system that:
- uses tendon actuation similar to human flexor tendons  
- incorporates 180° torsion springs for natural restoring force  
- achieves ~95° rotation per joint and ~270° total finger flexion  
- maintains mechanical stability with tight tolerances (0.5–6 mm)  

The goal was to replicate human finger kinematics as closely as possible.

### **3. Integrate hardware, firmware, and mechanics into a unified robotics platform**
HAND‑E is not a single component — it is a system.  
The project aimed to combine:
- CAD modelling  
- embedded firmware  
- signal processing  
- PCB design  
- servo‑tendon mechanics  

into one coherent, modular architecture.

### **4. Explore human‑to‑robot motion mapping**
Use flex sensors to capture human finger motion and translate it into robotic motion in real time.  
This lays the foundation for:
- teleoperation  
- exoskeleton‑style control  
- learning‑based imitation  
- humanoid manipulation research  

### **5. Build a platform suitable for future AI/ML and closed‑loop control**
The long‑term goal is to extend HAND‑E into:
- ML‑based gesture recognition  
- adaptive control  
- force feedback  
- proprioceptive sensing  
- autonomous grasping  

The current system is engineered to be modular and scalable so these capabilities can be added without redesigning the entire architecture.



---

##  System Architecture


HAND‑E is built as a full real‑time robotics pipeline, where sensing, filtering, mapping, and actuation operate in a tightly coupled loop.  
The system is intentionally modular, with each layer isolated but synchronised to achieve stable, biomimetic finger motion.

Below is the high‑level architecture:

<img width="1024" height="1536" alt="image" src="https://github.com/user-attachments/assets/fc151ca1-422d-40c6-bf89-3a712772a9e7" />



### **1. Sensing Layer — Flex Sensor Acquisition**
Each finger uses a resistive flex sensor embedded in a potential divider, with **local decoupling capacitors to ground** to suppress high‑frequency noise before the signal reaches the ADC.  
The ESP32 samples all five sensors using its ADC channels.

Key design choices:
- Independent analogue channels per finger  
- Short, clean analogue traces (in th PCB design)  
- Decoupling capacitors placed close to the flex sensor outputs to reduce high‑frequency noise and stabilise ADC readings  
- 5‑sample circular buffer for additional noise smoothing  
- Per‑finger calibration ranges stored in firmware  

This layer provides a clean, stable “human motion” signal for the rest of the pipeline.

---

### **2. Signal Processing Layer — Noise‑Resilient Filtering**
Raw flex readings are noisy and unsuitable for direct control.  

A Software filtering pipeline was added (buffering, hysteresis, calibration, inversion) — see Signal Processing section for full details.
This ensures stable, predictable behaviour even with low‑cost sensors.

---

### **3. Mapping Layer — Human Motion → Robotic Motion**
After filtering, each finger’s flex value is mapped to a servo angle:

- `flexMin → MAX_SERVO_BEND`  
- `flexMax → MIN_SERVO_BEND`  

This creates a **biomimetic mapping** where human finger flexion directly controls robotic tendon tension.

The mapping is:
- continuous  
- real‑time  
- calibrated per finger  
- constrained to safe mechanical limits  

---

### **4. Actuation Layer — PCA9685 PWM Driver**
The ESP32 offloads all servo timing to the PCA9685:

- 16‑channel hardware PWM  
- 50 Hz stable output  
- No jitter from ESP32 task scheduling  
- Allows future expansion (e.g., wrist, thumb abduction)  

This ensures smooth, consistent tendon actuation.

---

### **5. Mechanical Layer — Tendon‑Driven Anthropomorphic Hand**
Servo rotation pulls tendons routed through the finger segments.  
The mechanical system includes:

- **Curved joint geometry** (10 mm radius)  
- **95° rotation per joint**  
- **~270° total finger flexion**  
- **180° torsion springs** for restoring force  
- **0.5–6 mm tolerances** for stability and clearance  

This layer converts software‑defined motion into physical biomimetic behaviour.

---

### **6. Control Loop — Real‑Time Embedded Execution**
The entire pipeline runs at ~50 Hz:

1. Read flex sensors  
2. Update circular buffer  
3. Compute filtered value  
4. Apply hysteresis  
5. Map to servo angle  
6. Convert to PWM pulse  
7. Send to PCA9685  
8. Update last‑angle state  

This loop is optimised for:
- low latency  
- deterministic behaviour  
- smooth motion  
- minimal back‑driving load on servos  

---

### **7. Modularity & Scalability**
The architecture is intentionally modular:

- Add new fingers → extend the `Finger` struct  
- Add new sensors → extend ADC channels  
- Add ML/AI → replace mapping layer  
- Add force sensing → extend sensing layer  
- Add wrist/arm → extend PCA9685 channels  

HAND‑E is designed as a **platform**, not a one‑off prototype.

---

##  Mechanical Design & CAD Iteration

The mechanical design of HAND‑E went through multiple iterations, starting from hand‑drawn sketches and evolving into a compact, tendon‑driven anthropomorphic hand with biomimetic joint behaviour.

---

### **1. Concept Sketches — Tendon Routing & Joint Concepts**

The design began with side‑profile sketches exploring:
- **Servo–tendon actuation**: a servo pulling a tendon routed through each finger segment, analogous to human flexor tendons.  
- **Rubber‑based restoring force (early idea)**: rubber sections between segments to provide a passive return to neutral.  
- **Joint concepts**: pivot‑based joints with tendon tunnels to guide motion and prevent snagging.

These sketches helped define:
- tendon paths  
- servo placement in the palm  
- finger segment proportions  
- the overall layout of a 5‑finger anthropomorphic hand.

<img width="1504" height="1162" alt="image" src="https://github.com/user-attachments/assets/6faba733-8e8b-4d7a-b513-a43dbe3b2b9e" />


---

### **2. First CAD Prototype — Functional but Geometrically Limited**

The first CAD model translated the sketches into a 3D mechanism:

- **Cube‑like finger segments** with only **3 mm clearance** between joints.  
- This limited each joint to ~**15°** of rotation, resulting in stiff, unnatural motion.  
- The palm housing was ~**120 × 110 mm**, large and boxy, to fit servos and electronics.  
- No built‑in restoring mechanism — once bent, fingers stayed flexed unless actively driven back.

**Key lessons from the first version:**
- Joint geometry directly controls achievable range of motion.  
- Aesthetics and ergonomics matter — the hand looked bulky and non‑anthropomorphic.  
- A restoring mechanism is essential for smooth, repeatable motion cycles.

<img width="1764" height="863" alt="image" src="https://github.com/user-attachments/assets/cbc6463a-7cb9-4197-bf7d-d038dffb3a29" />

<img width="1769" height="963" alt="image" src="https://github.com/user-attachments/assets/3172074d-9bb3-4172-a156-c9926eacd294" />

<img width="736" height="927" alt="image" src="https://github.com/user-attachments/assets/4820c940-cd4e-4e9c-b77c-f421a9f166d6" />


---

### **3. Final CAD Prototype — Biomimetic Joint Geometry & Compact Palm**

The final design is the version used for 3D printing and testing.  
Major improvements include:


#### **a) Redesigned Joint Geometry for Natural Finger Motion**

- Replaced cube‑like joints with **curved hinge profiles** (≈10 mm radius).  
- Increased joint rotation from ~15° to ~**95° per joint**.  
- Achieved ~**270° total finger flexion**, enabling realistic curl and grasp shapes.  
- Introduced **6 mm clearance** between links to prevent self‑collision while maintaining stability.  
- Maintained ~**0.5 mm wall clearance** between inner and outer segments for smooth motion.

This redesign transformed the hand from “mechanically constrained” to more human-like.

---

#### **b) Torsion Springs for Passive Restoring Force**

To solve the “no return to neutral” problem:

- Integrated **180° torsion springs** at each joint.  
- Servos apply torque to pull tendons and flex the fingers.  
- When torque is released, torsion springs provide a **passive restoring torque**, returning fingers to neutral.  
- This reduces back‑driving loads on the servos by ~**65%** and improves energy efficiency.

This change made motion smoother, more natural, and more suitable for future closed‑loop control.

---

#### **c) Compact, Ergonomic Palm Redesign**

- Reduced palm size from **120 × 110 mm** to **90 × 85 mm** — over **30% volume reduction**.  
- Maintained full servo accessibility and tendon routing.  
- Improved weight distribution and overall ergonomics.  
- Result: a more **anthropomorphic** and realistic hand profile.

<img width="945" height="818" alt="image" src="https://github.com/user-attachments/assets/75ae6ec8-b643-43d7-8ce4-df8c23cdf157" />

<img width="1120" height="1036" alt="image" src="https://github.com/user-attachments/assets/7ed91bad-94c6-4666-bd65-819814da76ab" />

<img width="944" height="1019" alt="image" src="https://github.com/user-attachments/assets/e7e1d45a-e7b5-4191-a73d-96cdbc5fb724" />



---

### **4. Design for Assembly & Modularity**

The mechanical design also considers:

- **Per‑finger modularity** — each finger can be removed or replaced independently.  
- **Accessible tendon routing** — tunnels and caps to hide knots while keeping maintenance possible.  
- **Serviceability** — screws and covers designed so servos and tendons can be adjusted without disassembling the entire hand.

This makes HAND‑E not just a one‑off prototype, but maintainable and scalable for future experiments.

---

##  Electronics & PCB Design

The electronics for HAND‑E were designed to provide clean analogue sensing, stable PWM actuation, and a layout that supports the real‑time software pipeline.  
The system combines flex sensors, analogue filtering, an ESP32 microcontroller, a PCA9685 PWM driver, and a custom PCB engineered for low‑noise operation.

---

### **1. Flex Sensor Interface — Clean Analogue Sensing**

Each finger uses a resistive flex sensor embedded in a **potential divider**.  
To ensure stable ADC readings, the PCB integrates:

- **Local decoupling capacitors** from the sensor output to ground  
- **Short analogue traces** to minimise noise pickup  
- **Consistent routing** across all five channels  
- **Ground pour** beneath analogue paths to reduce impedance and interference  

These capacitors suppress **high‑frequency noise** generated by:
- servo switching currents  
- PWM edges  
- ESP32 Wi‑Fi/CPU activity  

This hardware filtering significantly improves the performance of the software’s moving‑average filter.


---

### **2. ESP32 Microcontroller — Real‑Time Processing Core**

The ESP32 handles:

- ADC sampling of all five flex sensors  
- Circular buffer updates  
- Hysteresis filtering  
- Per‑finger calibration  
- Angle → PWM mapping  
- I²C communication with the PCA9685  

The PCB routes all analogue inputs to **dedicated ADC‑capable pins**, ensuring:
- predictable sampling  
- minimal crosstalk  
- consistent voltage reference behaviour  

This microcontroller is the **software brain** of HAND‑E.

---

### **3. PCA9685 — Hardware PWM for Smooth Actuation**

Servo timing is offloaded to the PCA9685 to avoid jitter from ESP32 task scheduling.

Benefits:
- **16‑channel hardware PWM**  
- **Stable 50 Hz output**  
- **No CPU overhead**  
- **Scalable** (future wrist/abduction servos)  

The PCB places the PCA9685 close to the servo header block to minimise:
- voltage drop  
- ground bounce  
- PWM distortion  


---

### **4. Power Distribution — Servo Load Handling**

Servos introduce large transient currents.  
To handle this safely, the PCB includes:

- **Dedicated 5 V servo rail**  
- **Thickened copper traces** for high‑current paths  
- **Star‑ground topology** to prevent servo noise from entering analogue ground  
- **Bulk capacitor** near the servo rail to smooth voltage dips  

This ensures stable operation even during multi‑finger actuation.

---

### **5. PCB Layout Strategy — Noise‑Aware Design**

The PCB layout follows professional embedded design principles:

- **Analogue and digital separation**  
  - Flex sensor traces routed away from PWM and I²C lines  
  - Ground pour under analogue section for shielding  

- **Short, direct routing**  
  - Minimises parasitic capacitance and inductance  
  - Reduces ADC noise  

- **Consistent channel layout**  
  - All five flex channels follow similar routing patterns  
  - Ensures predictable calibration and behaviour  

- **Clear silkscreen labelling**  
  - Each sensor, resistor, capacitor, and servo header is labelled  
  - Simplifies debugging and assembly  


<img width="617" height="823" alt="image" src="https://github.com/user-attachments/assets/55e8cefb-21b0-43f2-b6bd-f4bfb027302c" />

<img width="422" height="354" alt="image" src="https://github.com/user-attachments/assets/ccb47cea-3633-4277-9065-bcebb21c7e6e" />




---

### **6. Integration with the Software Pipeline**

The electronics were designed specifically to support the real‑time control loop:

- **Decoupling capacitors** reduce high‑frequency noise → improves moving‑average stability  
- **Ground pour** reduces ADC jitter → improves hysteresis behaviour  
- **Short analogue paths** reduce drift → improves calibration accuracy  
- **PCA9685 hardware PWM** ensures smooth tendon actuation → improves biomimetic motion  

This tight coupling between hardware and software is what enables HAND‑E to behave like a **coherent robotic system**, not a collection of parts.

---

### **7. Modularity & Expandability**

The PCB is designed to be extended:

- Additional ADC channels can be added for force sensors  
- Extra PCA9685 channels allow wrist or thumb‑abduction servos  
- I²C breakout pads allow integration of IMUs or pressure sensors  
- The ESP32 can be upgraded to a dual‑core control architecture  


## Embedded Software & Control


The embedded software is the core of HAND‑E.  
Every motion, every response, and every behaviour is governed by a real‑time sensing → filtering → mapping → actuation pipeline running on the ESP32.  
The firmware is designed to be modular, deterministic, noise‑resilient, and feel natural.

---

### **1. Software Architecture Overview**

The control system is built around a 50 Hz real‑time loop:

1. Sample all flex sensors  
2. Update circular buffers  
3. Compute filtered values  
4. Apply hysteresis  
5. Map flex → angle  
6. Convert angle → PWM pulse  
7. Send commands to PCA9685  
8. Update last‑angle state  

This architecture ensures:
- low latency  
- smooth motion  
- stable behaviour even with noisy sensors  
- predictable timing for tendon actuation  

---

### **2. Finger Data Structure — Modular Per‑Finger Control**

Each finger is represented by a `Finger` struct containing:

- raw ADC value  
- circular buffer  
- filtered value  
- calibration range (`flexMin`, `flexMax`)  
- inversion flag  
- last servo angle  

This modular design allows:
- easy addition of new fingers  
- independent calibration  
- clean, readable code  
- scalable architecture for future sensors (e.g., force, IMU)  

---

### **3. Circular Buffer — Real‑Time Noise Smoothing**

Flex sensors produce noisy analogue signals.  
To stabilise readings without adding latency, HAND‑E uses a **5‑sample circular buffer** per finger.

Benefits:
- constant‑time updates  
- low memory footprint  
- smooth output without lag  
- ideal for embedded real‑time control  

This forms the first stage of the software filtering pipeline.

---

### **4. Hysteresis Filtering — Eliminating Micro‑Oscillations**

Even after smoothing, small fluctuations can cause servo jitter.  
To prevent this, HAND‑E applies a **3° hysteresis band**:

- If the new angle differs from the last angle by less than 3°, ignore it  
- If it exceeds 3°, update the servo  

This dramatically improves:
- stability  
- smoothness  
- servo lifespan  
- energy efficiency  

Hysteresis is essential for tendon‑driven systems where micro‑movements cause unnecessary tension changes.

---

### **5. Per‑Finger Calibration — Normalising Sensor Ranges**

Each flex sensor behaves differently due to:
- manufacturing tolerances  
- mounting position  
- tendon routing geometry  

To ensure consistent behaviour, each finger stores:
- `flexMin` (fully extended)  
- `flexMax` (fully flexed)  

The software maps these values to servo angles, ensuring:
- identical behaviour across fingers  
- predictable motion  
- accurate biomimetic control  

Calibration is performed once and stored in firmware.

---

### **6. Inversion Logic — Handling Mechanical Constraints**

Some tendons pull in the opposite direction due to servo placement.  
Instead of redesigning the mechanics, the firmware includes an **invert flag**:

- If `invert = true`, mapping is flipped  
- If `invert = false`, mapping is normal  

This allows:
- consistent code  
- flexible mechanical design  
- easy debugging  
- future expansion (e.g., mirrored hands)  

---

### **7. Mapping Layer — Human Motion → Robotic Motion**

After filtering, each flex value is mapped to a servo angle:

- `flexMin → MAX_SERVO_BEND`  
- `flexMax → MIN_SERVO_BEND`  

This creates a **biomimetic mapping** where human finger flexion directly controls tendon tension.

The mapping is:
- continuous  
- real‑time  
- calibrated  
- safe (angle limits enforced)  

---

### **8. PCA9685 Control — Smooth, Jitter‑Free Actuation**

The ESP32 sends PWM commands to the PCA9685 via I²C.  
Using hardware PWM ensures:

- stable 50 Hz servo timing  
- no jitter from ESP32 task scheduling  
- smooth tendon motion  
- scalable architecture (up to 16 channels)  

This is critical for natural, human‑like finger motion.

---

### **9. Real‑Time Loop — Deterministic Execution**

The entire pipeline runs at ~50 Hz, chosen because:

- it matches typical servo update rates  
- it provides smooth motion without overloading the ESP32  
- it aligns with human finger motion dynamics  

The loop is designed to be:
- deterministic  
- low‑latency  
- robust to noise  
- easy to extend  

---

### **10. Debugging & Telemetry**

The firmware includes optional serial debugging:

- raw ADC values  
- filtered values  
- mapped angles  
- hysteresis decisions  
- calibration ranges  

This makes tuning and troubleshooting straightforward.

---

### **11. Designed for Future AI/ML Integration**

The software architecture is intentionally modular so future upgrades can be added without rewriting the system:

- Replace mapping layer → ML‑based gesture recognition  
- Add force sensors → closed‑loop control  
- Add IMU → wrist stabilisation  
- Add camera → vision‑based grasping  

HAND‑E is built as a **software‑first robotics platform**, not a fixed prototype.

---

### **12. Known Limitations & Intentional Design Decisions**

One known limitation of the current firmware is that **occasional ADC samples may be lost** due to ESP32 task scheduling and peripheral timing. However, because the system samples at a high rate and applies both moving‑average smoothing and hysteresis, a single missed sample has **negligible impact** on the filtered output. For this reason, no explicit sample‑recovery mechanism was implemented — the control loop is intentionally designed to be robust to minor sampling irregularities.

Another intentional design choice is the use of a **fixed‑rate control loop** rather than a fully interrupt‑driven architecture. While interrupts could reduce latency further, they also introduce complexity and potential jitter when combined with I²C operations. The current design prioritises **deterministic behaviour, simplicity, and stability**, which are more important for tendon‑driven actuation than micro‑optimised timing.

---

## Signal Processing & Calibration

Signal processing is one of the most important parts of HAND‑E.  
Flex sensors are inherently noisy, non‑linear, and sensitive to mechanical mounting, so the system uses a multi‑stage filtering and calibration pipeline to produce stable, human-like motion.

The goal of this pipeline is simple:  
**convert noisy analogue flex readings into clean, predictable servo commands in real time.**

---

### **1. Hardware-Level Filtering — Decoupling Capacitors**

Before any software filtering occurs, each flex sensor output passes through a **local decoupling capacitor** to ground.  
This suppresses high‑frequency noise caused by:

- servo current spikes  
- PWM switching edges  
- ESP32 CPU/Wi‑Fi activity  
- long sensor leads acting as antennas  

This hardware filtering dramatically improves ADC stability and reduces the burden on the software filters.

---

### **2. Circular Buffer — Moving Average Smoothing**

Each finger maintains a **5‑sample circular buffer** that stores the most recent ADC readings.

Benefits:
- smooths out random noise  
- constant‑time updates  
- minimal memory footprint  
- no added latency  
- ideal for real‑time embedded systems  

This is the first software stage that stabilises the signal.

---

### **3. Hysteresis — Eliminating Micro‑Oscillations**

Even after smoothing, small fluctuations can cause servo jitter.  
To prevent this, HAND‑E applies a **3° hysteresis band**:

- If the new angle differs from the previous angle by less than 3°, ignore it  
- If it exceeds 3°, update the servo  

This prevents:
- micro‑movements  
- tendon flutter  
- unnecessary servo load  
- audible jitter  

Hysteresis is essential for tendon‑driven systems where tiny changes in angle cause noticeable tension differences.

---

### **4. Per‑Finger Calibration — Normalising Sensor Ranges**

Each flex sensor behaves differently due to:
- manufacturing tolerances  
- mounting position  
- tendon routing geometry  
- mechanical leverage differences  

To ensure consistent behaviour, each finger stores:

- **flexMin** → ADC value when the finger is fully extended  
- **flexMax** → ADC value when the finger is fully flexed  

The software maps these values to servo angles, ensuring:
- identical behaviour across fingers  
- predictable motion  
- accurate biomimetic control  

Calibration is performed once and stored in firmware.

---

### **5. Inversion Logic — Handling Mechanical Asymmetry**

Some tendons pull in the opposite direction due to servo placement.  
Instead of redesigning the mechanics, the firmware includes an **invert flag**:

- If `invert = true`, mapping is reversed  
- If `invert = false`, mapping is normal  

This keeps the software clean and the mechanical design flexible.

---

### **6. Mapping Function — Flex Value → Servo Angle**

After filtering and calibration, each flex value is mapped to a servo angle:

- `flexMin → MAX_SERVO_BEND`  
- `flexMax → MIN_SERVO_BEND`  

This mapping is:
- continuous  
- real‑time  
- biomimetic  
- safe (angle limits enforced)  

It ensures that human finger flexion directly controls tendon tension.

---

### **7. Combined Effect — Stable, Natural, Human‑Like Motion**

The full pipeline:

1. Hardware decoupling  
2. Circular buffer smoothing  
3. Hysteresis  
4. Calibration  
5. Inversion logic  
6. Angle mapping  

produces motion that is:
- smooth  
- stable  
- noise‑resilient  
- natural and human‑like  
- consistent across all fingers  

This is what makes HAND‑E feel like a **coherent robotic system**, not a set of independent servos.

---


## 🛠️ Build & Run


This section explains how to assemble the hardware, flash the firmware, and run the HAND‑E control system.  
The goal is to make the setup reproducible and easy to follow.

---

### **1. Hardware Assembly**

1. **Print all mechanical parts** (palm, finger segments, tendon caps, covers).  
2. **Install torsion springs** into each joint before inserting hinge pins.  
3. **Mount the servos** into the palm housing.  
4. **Route tendons** through the finger tunnels and tie them to the servo horns.  
5. **Install the PCB** into the palm and connect:
   - 5× flex sensors  
   - 5× servo connectors  
   - ESP32  
   - 5 V power input  

Ensure all grounds (ESP32, PCA9685, servos, sensors) are shared.

---

### **2. Wiring Overview**

- Flex sensors → PCB analogue inputs (ADC pins)  
- Decoupling capacitors → sensor output → GND  
- ESP32 → I²C → PCA9685 (SDA, SCL)  
- PCA9685 → servo headers  
- 5 V rail → servos + PCA9685  
- USB → ESP32 (power + flashing)

Refer to the PCB silkscreen for exact pin labels.

---

### **3. Firmware Setup**

1. Install **Arduino IDE** or **PlatformIO**.  
2. Install ESP32 board support.  
3. Clone this repository.  
4. Open the `firmware/` folder.  
5. Install required libraries:
   - `Adafruit_PWMServoDriver`  
   - `Wire`  
6. Select your ESP32 board and COM port.  
7. Flash the firmware.

---

### **4. Calibration Procedure**

Each finger must be calibrated once:

1. Open the serial monitor at **115200 baud**.  
2. Fully extend your finger → note the ADC value → set as `flexMin`.  
3. Fully flex your finger → note the ADC value → set as `flexMax`.  
4. Update the calibration values in `config.h`.  
5. Re‑flash the firmware.

This ensures consistent behaviour across all fingers.

---

### **5. Running the System**

Once flashed and calibrated:

1. Power the servos with a **stable 5 V supply** (≥ 2 A recommended).  
2. Connect the ESP32 via USB or external 5 V.  
3. The system automatically:
   - samples flex sensors  
   - filters the signals  
   - maps them to servo angles  
   - drives the tendons in real time  

No additional software is required.

---

### **6. Optional Debugging**

Enable debugging by setting:

```cpp
#define DEBUG true
```

This prints:
- raw ADC values  
- filtered values  
- mapped angles  
- hysteresis decisions  

Useful for tuning or diagnosing sensor issues.

---

### **7. Safety Notes**

- Do not power servos from USB.  
- Avoid over‑tightening tendons — this can stall servos.  
- Keep fingers clear of moving joints during testing.  
- Ensure the 5 V supply is regulated and noise‑free.

---

### **8. Troubleshooting**

**Finger moves erratically**  
- Check calibration values  
- Check decoupling capacitor orientation  
- Ensure tendons are not snagging  

**Servo jitter**  
- Check ground connections  
- Ensure 5 V supply is stable  
- Verify hysteresis is enabled  

**No movement**  
- Check I²C wiring (SDA/SCL)  
- Confirm PCA9685 is powered  
- Verify servo rail voltage
  
---
## 🎥 Demo & Media

<img width="825" height="548" alt="image" src="https://github.com/user-attachments/assets/9afa9f26-280b-4179-a990-e92235a2b30b" />

<img width="1000" height="1123" alt="image" src="https://github.com/user-attachments/assets/d21fc661-35d6-4c26-bf10-d0368fb1a970" />


---
##  Future Improvements

HAND‑E is designed as a modular platform, and several extensions can significantly enhance its capability, robustness, and research value.

### **1. Closed‑Loop Control with Force or Tension Sensors**
Adding force‑sensing resistors (FSRs) or tendon‑tension sensors would enable:
- adaptive grip strength  
- slip detection  
- compliant grasping  
- safer interaction with objects  

This would transform HAND‑E from open‑loop actuation to true closed‑loop manipulation.

### **2. Improved Flex Sensor Linearity**
Flex sensors are convenient but non‑linear. Potential upgrades include:
- hall‑effect joint angle sensors  
- optical bend sensors  
- IMU‑based finger tracking  

These would improve accuracy and reduce calibration drift.

### **3. Machine‑Learning‑Based Gesture Mapping**
Replacing the linear mapping layer with an ML model could enable:
- gesture classification  
- adaptive motion mapping  
- personalised control profiles  
- teleoperation with higher fidelity  

The current architecture already supports this swap.

### **4. Wrist and Thumb Abduction Degrees of Freedom**
The PCA9685 has spare channels, allowing:
- wrist flexion/extension  
- wrist rotation  
- thumb abduction/adduction  

This would significantly expand the hand’s manipulation capabilities.

### **5. Mechanical Refinements**
Future mechanical upgrades could include:
- ball‑bearing joints for reduced friction  
- improved tendon routing to reduce wear  
- modular servo cartridges for faster maintenance  
- lighter materials for improved responsiveness  

### **6. PCB Enhancements**
A future PCB revision could add:
- onboard voltage regulation  
- reverse‑polarity protection  
- integrated force‑sensor amplifiers  
- dedicated ADC reference filtering  

These would improve robustness and simplify wiring.

### **7. Wireless Telemetry & Logging**
Adding BLE/Wi‑Fi telemetry would allow:
- real‑time visualisation of sensor data  
- remote debugging  
- data collection for ML training  

The ESP32 already supports this natively.

---

##  Key Engineering Takeaways

Building HAND‑E required integrating mechanical design, embedded electronics, and real‑time software into a single coherent system.  
The following principles were critical to making the system stable, responsive, and biomimetic.

### **1. Hardware and Software Must Co‑Design Each Other**
The mechanical, electrical, and software layers were developed together.  
Examples:
- decoupling capacitors improved software filtering  
- torsion springs reduced servo load and improved control stability  
- joint geometry informed calibration ranges  

This co‑design approach is what makes HAND‑E feel like a unified system.

### **2. Noise‑Resilient Signal Processing Is Essential**
Flex sensors are noisy and inconsistent.  
The multi‑stage pipeline (hardware filtering → moving average → hysteresis → calibration) was crucial for:
- smooth motion  
- stable servo behaviour  
- predictable mapping  

Without this pipeline, the hand would jitter uncontrollably.

### **3. Deterministic Timing Beats Raw Speed**
A stable 50 Hz loop produced better results than a faster but inconsistent loop.  
Consistency matters more than speed in tendon‑driven systems.

### **4. Modularity Enables Scalability**
By structuring the system around:
- per‑finger structs  
- isolated sensing channels  
- a PCA9685 with spare PWM outputs  
- a clean PCB layout  

HAND‑E can grow into a more complex robotic system without redesigning the core.

### **5. Mechanical Iteration Is Non‑Negotiable**
The jump from CAD v1 to v2 (curved joints, torsion springs, compact palm) was transformative.  
Mechanical iteration is just as important as software iteration.

### **6. Simplicity Is a Feature**
Some limitations — like occasional lost ADC samples — were intentionally left unaddressed because:
- the filtering pipeline already absorbs them  
- added complexity would not improve performance  

This reflects mature engineering judgement: solve the problems that matter, not all possible problems.

### **7. A Robotics System Is More Than the Sum of Its Parts**
HAND‑E works because:
- the mechanics are biomimetic  
- the electronics are noise‑aware  
- the software is real‑time and stable  

None of these layers alone would produce the final behaviour — only their integration does.

