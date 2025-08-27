# Siemens-TIA-Automated-Factory

# Automated Sorting, Machining, Assemblying and Packing Simulation

A fully simulated discrete manufacturing line built with Siemens TIA Portal (v19), PLCSIM, and Factory I/O.  
The virtual plant performs a complete mini production workflow:
1. Infeed & Identification (color / material based)
2. Sorting to dedicated buffers
3. Machining (e.g., drilling / surface treatment placeholder)
4. Assembly (pairing / combining parts logic)
5. Quality Gate (basic presence / type validation)
6. Packing / Palletizing (batch-based release)


![1](https://github.com/user-attachments/assets/7707c6c0-8167-4c50-aa64-389f60f18609)


![2](https://github.com/user-attachments/assets/8c1d7da5-2009-4099-b6bf-49c8d57da534)


![3](https://github.com/user-attachments/assets/f992d9c7-0730-40b0-8fa6-73b1c366a4b9)


![4](https://github.com/user-attachments/assets/8ff4890f-c850-40a0-af37-7eee2e13db37)


---

## 🎯 Project Objective

Demonstrate an educational yet industry-relevant control solution that:
* Integrates multiple functional cells under one PLC program architecture.
* Uses modular, reusable, and state-driven logic blocks.
* Simulates a part lifecycle from raw material arrival to finished packed unit.
* Provides a foundation for later optimization (OEE, traceability, recipe control).

---

## 🧱 System Overview

| Cell | Function | Key Actions | Core Signals (Examples) |
|------|----------|-------------|--------------------------|
| Infeed & Detection | Accept raw parts | Presence detect, color scan | Part_Present, Color_Blue/Green, Metal_Flag |
| Sorting Diverter | Route parts | Diverter left/center/right | Diverter_CMD, Jam_Sensor |
| Machining Station | Process parts | Clamp, spindle | Clamp_Cyl, Spindle_Run, Cycle_Done |
| Assembly Cell | Combine items | Pick/Place | Asm_Part_A_Ready, Asm_Part_B_Ready |
| Packing / Palletizing | Batch & release | Counter, eject | Pack_Count, Batch_Done | (To be continuned..)

---

## 🔄 Process Flow (High-Level)

1. Raw part enters via conveyor.
2. Sensor cluster classifies (color).
3. Send according to material type (Blue Green Red).
4. When machining station is free and buffer has stock → transfer part.
5. Machining cycle executes (simulate with timers / step sequence).
6. Machined part(product base) + complementary component (Product lid) assembled.
7. Assembled unit passes for packaging.

---

## 🧠 Control Architecture

Suggested program structuring (TIA Portal):

Program Blocks:
* OB1: Main cyclic orchestration calling modular FBs.
  
[Main(OB1).pdf](https://github.com/user-attachments/files/21991994/Main.OB1.pdf)

* Pick and Place Machine 1

[Pick and Place 1.pdf](https://github.com/user-attachments/files/21992008/Pick.and.Place.1.pdf)

* Pick and Place Machine 2
  
[Pick and Place 2.pdf](https://github.com/user-attachments/files/21992019/Pick.and.Place.2.pdf)

* Pick and Place Machine 3

[Pick and Place 3.pdf](https://github.com/user-attachments/files/22001818/Pick.and.Place.3.pdf)

Data Blocks:

[Programe Blocks.pdf](https://github.com/user-attachments/files/21992036/Programe.Blocks.pdf)

Tags:

[Tags.pdf](https://github.com/user-attachments/files/22001814/Tags.pdf)

---

## 🧬 Animation

Sorting:


https://github.com/user-attachments/assets/f82fc5e8-b0ca-4505-8fbb-c64fb0628032


Machining:


https://github.com/user-attachments/assets/afe20aac-1322-44ec-9088-3856ad0f5227


Assembling:


https://github.com/user-attachments/assets/24868bd7-c645-4bae-b6b9-e50c8c907e81


---

## 🧬 State Machine Example (Machining)

States (ENUM):
0 Idle  
1 WaitPart  
2 Clamp  
3 Process  
4 Unclamp  
5 Unload  
6 Complete (one-shot to upstream)  

Transition logic uses positive edges and condition guards (e.g., Part_Present AND Station_Free). Timers (TON) simulate cycle durations.

---

## ⚙️ Example Tag Naming Convention

Prefixing:
* I_: Inputs (I_Conv_Infeed_Sensor)
* Q_: Outputs (Q_Diverter_Left)
* M_: Internal memory flags (M_Part_Latched)
* DB/Struct-based recommended alternative: gIO.Infeed.Sensor, gCtrl.Sorter.Enable

Consistency improves readability and reuse.

---

## 🔌 I/O Mapping

Embed or recreate a table or screenshot mapping Factory I/O elements to PLC addresses.

Example (placeholder):

| Function | Factory I/O Element | PLC Address | Tag |
|----------|--------------------|-------------|-----|
| Infeed Photoeye | Sensor_1 | I0.0 | I_Infeed_PE |
| Color Sensor Blue | Sensor_2 | I0.1 | I_Color_Blue |
| Diverter Left Solenoid | Act_1 | Q0.0 | Q_Div_Left |
| Diverter Right Solenoid | Act_2 | Q0.1 | Q_Div_Right |
| Machining Clamp | Act_3 | Q0.2 | Q_Clamp |
| Spindle Motor | Act_4 | Q0.3 | Q_Spindle |
| QA Reject Pusher | Act_5 | Q0.4 | Q_Reject |

(Replace with actual tags; ensure alignment with the TIA project.)

![I/O Map](link_to_your_io_map_screenshot.jpg)

---

## 🧪 Quality Gate Logic (Sample Pseudocode)

IF (Assembled_Unit = TRUE) THEN  
  IF (ColorType = BLUE AND Machined = TRUE) OR (ColorType = METAL) THEN  
    QA_OK := TRUE;  
  ELSE  
    QA_Fail := TRUE;  
  END_IF;  
END_IF;

---

## 📦 Packing Logic

Batch_Size configurable (e.g., 10).  
On each QA_OK rising edge → Increment Pack_Count.  
IF Pack_Count >= Batch_Size THEN  
  Batch_Done := TRUE (one-shot)  
  Pack_Count := 0  
END_IF

Future extension: Pallet layers, recipe-driven batch sizes.

---

## 🛡️ Safety & Interlocks (Simulation-Oriented)

Basic (soft) checks:
* Jam detection (no movement while motor command active N seconds).  
* Diverter mutual exclusion (never energize Left & Right simultaneously).  
* Machining cannot start unless Clamp confirmed & Part_Present.  
* Assembly cannot proceed unless both required part flags set.  

Optional: Aggregate alarm bit to inhibit new cycle start.

---

## 🧾 KPIs (Extend Later)

Potential metrics to log in DB_Stats:
* Parts_Infeed_Total
* Machined_OK
* Assembled_OK
* QA_Fail_Count
* Packaged_Units
* MTBF / MTTR placeholders (manual resets)

---

## 🛠️ Toolkit

* PLC: Siemens S7-1200 (simulated with PLCSIM)
* IDE: Siemens TIA Portal v19
* Simulation: Factory I/O (Driver: Siemens S7-PLCSIM)
* (Optional) HMI: Basic panel or WinCC Comfort screen for status & KPIs

---

## 🚀 How to Run

1. Restore the provided .zap19 project in TIA Portal v19.
2. Open and compile; ensure hardware configuration matches S7-1200 (simulation ok).
3. Start PLCSIM and load the program.
4. Open the .factoryio scene file in Factory I/O.
5. Configure driver: Siemens S7-PLCSIM → Set IP (usually 127.0.0.1) & Rack/Slot (e.g., Rack 0 Slot 1 for S7-1200 CPU).
6. Start simulation; verify I/O bits toggle correctly (test a sensor).
7. Enable Master Start (e.g., HMI or M_Bit) → Observe flow.
8. Use Watch Table / Trace to monitor key tags: Part counters, State enums, Diverter commands.

---

## 🧪 Testing Checklist

| Test | Action | Expected |
|------|--------|----------|
| Infeed Detection | Place part at entry | I_Infeed_PE = 1 |
| Sorting | Cycle 3 different types | Diverter routes each correctly |
| Buffer Full | Simulate backlog | Buffer_Full bit prevents further route |
| Machining | Force Part Present | State machine steps through sequence |
| Assembly | Provide both components | Assembled flag pulses |
| QA Reject | Use wrong combo (simulate) | QA_Fail increments |
| Batch Complete | Run N good units | Batch_Done one-shot |

---

## 🧩 Troubleshooting

| Symptom | Possible Cause | Resolution |
|---------|----------------|-----------|
| No PLC connection | Driver misconfigured | Recheck IP / Rack / Slot |
| Diverter inactive | Interlock active | Check both sides not commanded simultaneously |
| Machining stuck in WaitPart | Buffer empty flag | Confirm buffer release logic |
| Assembly never starts | Component flags false | Trace upstream handshake bits |
| Batch never completes | Pack_Count not incrementing | Confirm QA_OK edge detection |

---

## 🔮 Future Enhancements

* Add RFID / ID-based serialization.
* Introduce recipe management (different machining times).
* Implement OEE dashboard (Availability, Performance, Quality).
* Add fault injection panel (simulate sensor failure).
* Use SCL for cleaner sequence code (currently LAD/FBD assumed).
* OPC UA link for external MES integration.

---

## 🗂️ Repository Structure (Proposed)

```
/project
  /tia_project/ (zap19 and exported blocks)
  /factory_io/  ( .factoryio scene file )
  /docs/
     io_map.png
     process_flow.svg
     state_diagrams/
  /hmi/ (screens or exported runtime)
  README.md
```

---

## 🖥️ Demonstration

(Replace placeholder with actual GIF/Video.)

![Simulation GIF](link_to_your_gif_or_video.gif)

---

## 📜 License

Specify your chosen license (e.g., MIT, Apache-2.0) if you intend to share publicly.

---

## 🙌 Contributions

Pull requests welcome:
1. Fork
2. Create feature branch
3. Commit & push
4. Open PR with clear description

---

## ❓ FAQ

Q: Can I use S7-1500 instead?  
A: Yes—adjust hardware configuration and (if needed) address changes.

Q: Can this run on a physical conveyor cell?  
A: Core logic yes; adapt I/O, safety circuits, and add hardware interlocks.

Q: Why are some steps timers instead of sensor-driven?  
A: In simulation we often abstract physical cycle completion; replace with actual sensors when migrating.

---

## 📎 Notes

The word "Assemblying" in the subtitle is retained for continuity, but “Assembling” is the standard spelling.

---

Feel free to replace all placeholder links, images, and tag examples with those from your actual implementation.
