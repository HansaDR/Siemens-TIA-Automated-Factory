# Siemens-TIA-Automated-Factory

# Automated Sorting, Machining, Assemblying and Packing Simulation

A fully simulated discrete manufacturing line built with Siemens TIA Portal (v19), PLCSIM, and Factory I/O.  
The virtual plant performs the complete workflow: automated identification, sorting, machining / lid production, robotic pick & place, assembly, quality validation, and final batching (packing) of good units.

---

## 📸 Layout & System Visuals

| Ref | Image | Description |
|-----|-------|-------------|
| ![Layout Overview with Robotic Cells](image1) | image1 | Global layout: dual machining / robotic cells (front-left), elevated transfer conveyors, central sorting & distribution, safeguarded zones, multi-level flow. |
| ![Sorting & Diverter Network Close-up](image2) | image2 | Close-up on high-density conveyor matrix: pivot arms, stoppers, sensors guiding parts to processing lanes & lid production. |
| ![I/O Tag Mapping – Extended View A](image3) | image3 | Factory I/O driver panel snapshot: sensors (left) and actuators (right) with address correlation (early view). |
| ![I/O Tag Mapping – Extended View B](image4) | image4 | Operational driver mapping (Siemens S7-PLCSIM) showing consolidated addresses used in PLC tag table. |

(When committing images to the repository, place them under docs/images/ and replace the inline references `image1`..`image4` with relative paths, e.g. `![Layout Overview](docs/images/layout_overview.png)`.)

---

## 🎯 Project Objective

Demonstrate an industry-relevant yet educational control solution that:
* Integrates multiple functional cells under one PLC.
* Utilizes modular state-driven sequence blocks.
* Simulates a part (and lid) lifecycle from raw material feeding to packaged unit.
* Enables extension toward OEE, traceability, and recipe control.

---

## 🧱 Functional Cells

| Cell | Purpose | Core Functions | Notable Devices (Examples) |
|------|---------|----------------|-----------------------------|
| Infeed & Vision / Color Check | Part arrival + classification | Detect presence, color/metallic type, part vs. lid check | Diffuse Blue/Green, Capacitive Metal, Vision Check bits |
| Sorting / Distribution | Route parts to correct processes | Pivot arms, stoppers, selective release | Pivot Arm (Blue/Green/Lid), Stoppers, Diverters |
| Machining Centers (0–2) | Produce lids / machine parts | Clamp → Process → Release | Machining Center (Progress, Busy, Error, Start/Stop) |
| Robotic Pick & Place (PP1 / PP2) | Transfer / assembly support | X/Z axis move, rotate, grab, clamp positioner | PP1 X/Z, PP1 Rotate, PP1 Clamp DS; PP2 equivalents |
| Lid Production (Embedded in Machining) | Generate lids for assembly | Produce Lids commands per color | MC Blue/Green/Metal Start/Stop |
| Assembly Cell | Merge base part + lid | Positioning, clamp verification | Clamp Positioner, Pivot Arm Lid Turn |
| Quality Gate | Validate correct pairing | Type + presence logic | Vision Blue/Green/Metal Check |
| Packing / Batching | Accumulate valid units | Count to batch size, signal pack complete | Batch counter tags |
| Safety Layer | Controlled operation | Start/Stop, doors, resets, camera | Safety Door 0/1, Start/Stop Buttons, Factory I/O Run/Reset |

---

## 🔄 Process Flow (High-Level)

1. Raw part enters via infeed conveyor (presence + diffuse sensors).
2. Classification: Blue / Green / Metal (or lid via capacitive / diffuse + logic).
3. Sorting logic actuates pivot arms & stoppers → route to:
   * Machining center (if raw blank requiring lid production).
   * Assembly buffer (if lid already produced / available).
4. Machining center cycles (clamp, process, produce lid if configured).
5. Pick & Place units (PP1/PP2) transfer machined part + select lid.
6. Assembly clamp holds components; pivot arm positions lid.
7. Vision / sensor validation (type + lid presence).
8. If OK → Counting toward batch; if fail → reject / rework path (future).
9. On reaching Batch_Size → Batch_Done one-shot → simulated “packed”.

---

## 🧠 Control Architecture (Suggested TIA Structure)

Program Blocks:
* OB1 Main Cycle
* OB100 Initialization
* (Optional) OB35 Time-Critical (if implementing high-speed timestamping)
* FB_Sorter
* FB_Buffer (multi-instance for each type)
* FB_Machining (per machine instance: MC0, MC1, MC2)
* FB_PickPlace (PP1, PP2)
* FB_LidProduction (could be integrated inside Machining or standalone)
* FB_Assembly
* FB_QA
* FB_Packing
* FB_Safety
* FB_Alarms
* FC_Util_Edge, FC_Util_Timer, FC_Util_State (helpers)

Data Blocks:
* DB_Global (modes, master start, interlocks)
* DB_IO (structured I/O copy)
* Instance DBs per FB
* DB_Stats (throughput, rejects, MTBF placeholders)
* DB_Recipes (future extension: cycle times, batch size per product type)

---

## 🧬 Example State Machine (Machining Center)

States (ENUM):
0 Idle  
1 WaitForPart  
2 Clamp  
3 Process (Timer / progress %)  
4 Unclamp  
5 Unload / Transfer Request  
6 Complete Pulse → Return to Idle  

Transition Guards:
* PartPresent & MachineFree → WaitForPart
* ClampClosed Confirm within Timeout else Error
* ProcessComplete (TON Done or %Progress >= 100)
* Unload handshake with Pick & Place (PP Ready)

Error Handling:
* Timeouts raise MachineError bit; FB_Alarms aggregates then groups by severity.

---

## 📦 Packing Logic

Configurable: Batch_Size (e.g., 12 units).  
On QA_OK rising edge: Pack_Count++  
If Pack_Count >= Batch_Size:
* Batch_Done one-shot
* Pack_Count reset
* Stats: Packed_Batches++

(Optional) Add Pallet_Layer counter & Multi-layer palletization in future.

---

## 🧪 Quality Validation (Sample Pseudocode / SCL Sketch)

```
IF AssemblyDone AND RisingEdge(AssemblyDone) THEN
   ValidType := ( (PartType = BLUE OR PartType = GREEN) AND LidPresent )
                 OR (PartType = METAL AND (NOT RequiresLid));
   IF ValidType THEN
      QA_OK := TRUE;
   ELSE
      QA_Fail := TRUE;
   END_IF;
END_IF;
```

Extend with: combination matrix, torque feedback, vision mismatch classification codes.

---

## 🔌 I/O Mapping

Images ![I/O Tag Mapping A](image3) and ![I/O Tag Mapping B](image4) illustrate the raw tag list from Factory I/O’s Siemens S7-PLCSIM driver.

Below is a distilled logical mapping example (replace with your actual addresses):

| Logical Group | Raw Tag (Example) | Description |
|---------------|-------------------|-------------|
| Sensors.Infeed.Blue | %I0.1 | Diffuse Blue |
| Sensors.Infeed.Green | %I0.3 | Diffuse Green |
| Sensors.Infeed.Metal | %I0.6 | MC Metal Capacitive |
| Sensors.MC[0].Progress | %I0.9 | Machining Center 0 Progress (Int / % value) |
| Actuators.Conv.Main0 | %Q0.0 | Belt Conveyor (2m) 0 |
| Actuators.Conv.SortA | %Q0.7 | Belt Conveyor PP1 A |
| Actuators.Pivot.ArmBlue | %Q1.0 | Pivot Arm Blue |
| Actuators.PP1.Grab | %Q1.1 | PP1 Grab |
| Actuators.MC[1].Start | %Q1.4 | Machining Center 1 Start |
| Actuators.Lid.ProduceBlue | %Q2.? | MC Blue Start (lid production) |
| System.FactoryRun | %Q? | Factory I/O (Run) command |
| Safety.StopButton | %I1.1 | Stop Button 0 |

Recommendation:
1. Create structured tags in a global DB: gIO.Sensors.*, gIO.Actuators.*
2. OB1 copies physical I/O → gIO for decoupling.
3. Internal logic references structured namespace (improves portability).
4. Edge detect only on gIO snapshot.

---

## ⚙️ Naming Conventions

Pattern: <Domain>_<Location/Unit>_<Element>  
Examples:
* SRT_Pivot_BlueCmd
* MC1_ClampClosed
* PP2_XBusy
* QA_LastResult
* PKG_BatchDone

Alternatively adopt CamelCase inside structs:
gCtrl.Sorter.PivotBlueCmd, gStat.QA.LastResult

---

## 🛡️ Safety & Interlocks

Soft (Simulation) Interlocks:
* Start requires: AllSafetyDoorsClosed AND NoActiveAlarms.
* Diverter mutual exclusion: Prevent simultaneous contradictory coil energizing.
* Machining start inhibited if any MachineError or ClampNotConfirmed.
* Pick & Place motion inhibited unless Z axis safe height before X translate.
* Emergency / Stop Button sets MasterInhibit until Reset.

Future Hardening:
* Add safe torque off, door zones, e-stop network simulation (if migrating physical).

---

## 🧾 KPIs & Data Capture (DB_Stats)

| Metric | Source | Update Event |
|--------|--------|--------------|
| PartsInfeedTotal | Infeed sensor rising | Each new part |
| MachinedCount | MC Complete | State 6 → Idle |
| AssembledOK | Assembly + QA_OK | QA validation |
| QA_FailCount | QA_Fail pulse | Validation fail |
| PackagedUnits | Batch counter | Each QA_OK |
| BatchesCompleted | Batch_Done | Packaging cycle |
| AvgCycleTime_MC[n] | Timestamp diff | End vs. Start |
| Utilization_MC[n] | Busy time / elapsed | Periodic roll-up |

Add retention or CSV export via OPC UA / external script later.

---

## 🛠️ Toolkit

* PLC: Siemens S7-1200 (PLCSIM)
* IDE: Siemens TIA Portal v19
* Simulation: Factory I/O v2.5.x (Siemens S7-PLCSIM driver)
* (Optional) HMI: WinCC Comfort/Advanced panel for KPIs & diagnostics
* Version Control: Git (this repository)

---

## 🚀 Getting Started

1. Clone repository & restore TIA `.zap19` project.
2. Open Factory I/O scene (`.factoryio`).
3. In TIA: Compile hardware + load to PLCSIM.
4. In Factory I/O:
   * Driver: Siemens S7-PLCSIM
   * Map addresses (verify against images ![I/O Tag Mapping B](image4))
   * Connect.
5. Use Watch Table for: MasterStart, QA_OK, Pack_Count, MC[n].State.
6. Press Start Button (physical tag or HMI) → Sequence begins.
7. Observe sorting & machining (refer to layout ![Layout Overview](image1)).

---

## 🧪 Commissioning / Test Matrix

| Test ID | Scenario | Procedure | Expected |
|---------|----------|-----------|----------|
| T01 | Infeed Classification | Feed Blue, Green, Metal parts | Correct type flags set |
| T02 | Sorting Path | Introduce sequential multi-types | Pivot arms actuate appropriate lanes |
| T03 | Machining Cycle | Feed raw part to MC0 | States progress 0→6; Progress% increments |
| T04 | Lid Production | Trigger lid production cycle | Lid availability flag after process |
| T05 | Assembly | Provide part + lid | AssemblyDone + QA_OK |
| T06 | QA Fail (No Lid) | Disable lid feed | QA_Fail pulse; no Pack_Count increment |
| T07 | Batch Completion | Run N=Batch_Size good units | Batch_Done one-shot; Pack_Count reset |
| T08 | Safety Stop | Press Stop Button mid-cycle | Motions halt; Resume after Reset |
| T09 | Machine Error Timeout | Block Clamp feedback | MachineError bit set; cycle inhibited |

---

## 🧩 Troubleshooting Guide

| Symptom | Likely Cause | Action |
|---------|--------------|-------|
| No motion after Start | MasterInhibit or SafetyDoor open | Check safety bits; reset |
| Parts bypass sorting | Sensor misaddress | Verify %I mapping vs. driver |
| Machine stuck in Clamp | Clamp feedback missing | Check clamp sensor tag & TON timeout |
| QA always failing | Type logic mismatch | Trace PartType & LidPresent |
| Batch never completes | QA_OK edge not detected | Confirm rising edge logic |
| Robot idle while machine finished | Handshake bit mismatched | Ensure MC.UnloadRequest & PP.Ready alignment |

---

## 🔮 Future Enhancements

* Recipe / Product family manager (dynamic cycle times, required lid color).
* OPC UA / MQTT telemetry for dashboards.
* Advanced vision emulation (geometry / defect codes).
* Multi-layer palletization algorithm.
* Alarm classification with timestamps + CSV export.
* Add SCL refactor for all sequence FBs.
* Energy usage simulation (motor run time accumulation).

---

## 🗂️ Suggested Repository Structure

```
/tia_project/            (Zap + exported blocks)
 /factory_io/            (Scene .factoryio)
 /docs/
   images/
     layout_overview.png
     conveyor_network.png
     io_mapping_a.png
     io_mapping_b.png
   state_diagrams/
 /hmi/
README.md
LICENSE
```

---

## 🖥️ Demonstration

(Embed GIF or video of full cycle—sorting → machining → assembly → batch complete.)

`![Simulation GIF](docs/images/demo_cycle.gif)`

---

## 📜 License

Specify (e.g., MIT). Without a license the project is effectively “all rights reserved”.

---

## 🙌 Contributions

Pull requests welcome:
1. Fork repository
2. Create feature branch (feat/…)
3. Commit descriptive changes
4. Open PR with test notes / screenshots

---

## ❓ FAQ

Q: Why retain the term “Assemblying”?  
A: Maintained for continuity with original project subtitle (standard spelling is “Assembling”).

Q: Can an S7-1500 be substituted?  
A: Yes; update hardware config, adjust addresses if required.

Q: Can physical hardware be integrated?  
A: Logic architecture is portable; you must implement real safety, E-stops, and rated IO modules.

Q: How are progress percentages generated?  
A: Simulated via TON elapsed / preset or incremental counter within the Machining FB.

---

## 📝 Notes

Replace placeholder tag rows and image references with your actual project resources.  
Ensure consistent tag mapping between Factory I/O driver (as seen in images) and TIA DB structures to avoid misalignment in future enhancements (like OEE or traceability).

---

Happy Automating! 🚀
