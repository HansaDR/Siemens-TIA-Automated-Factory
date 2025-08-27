# Siemens-TIA-Automated-Factory V1

# Automated Sorting, Machining, Assemblying and Packing Simulation

A fully simulated discrete manufacturing line built with Siemens TIA Portal (v19), PLCSIM, and Factory I/O.  
The virtual plant performs a complete mini production workflow:
1. Infeed & Identification (color / material based)
2. Sorting to dedicated buffers
3. Machining (e.g., drilling / surface treatment / lid production placeholder)
4. Assembly (pairing product base with product lid)
5. Quality Gate (presence / type validation)
6. Packing / Palletizing (batch-based release)


---

## 📸 Layout Images


![Layout 1](https://github.com/user-attachments/assets/7707c6c0-8167-4c50-aa64-389f60f18609)

![Layout 2](https://github.com/user-attachments/assets/8c1d7da5-2009-4099-b6bf-49c8d57da534)

![I/O Mapping View A](https://github.com/user-attachments/assets/f992d9c7-0730-40b0-8fa6-73b1c366a4b9)

![I/O Mapping View B](https://github.com/user-attachments/assets/8ff4890f-c850-40a0-af37-7eee2e13db37)


---

## 🎯 Project Objective

Demonstrate an educational yet industry-relevant control solution that:
* Integrates multiple functional cells under one PLC program architecture.
* Uses modular, reusable, and state-driven logic blocks.
* Simulates a part lifecycle from raw material arrival to finished packed unit.
* Provides a foundation for optimization (OEE, traceability, recipe control).

---

## 🧱 System Overview

| Cell | Function | Key Actions | Core Signals (Examples) |
|------|----------|-------------|--------------------------|
| Infeed & Detection | Accept raw parts & classify | Presence detect, color / metal scan | Part_Present, Type_Blue, Type_Green, Type_Metal |
| Sorting Diverter / Distribution | Route parts to buffers or machines | Diverters, stoppers, pivot arms | Diverter_CMD, Stopper_Release, Jam_Sensor |
| Buffer Queues (per type) | Accumulate WIP | FIFO queue control, release | Buffer_Count_X, Buffer_Release_X |
| Machining Station(s) | Process / produce base or lid | Clamp, process cycle, release | Clamp_Closed, Cycle_Progress, Cycle_Done |
| Pick & Place Units (PP1..PP3) | Transfer & position parts | X/Z motion, rotation, grip | PPn_XBusy, PPn_ClampDS, PPn_Grip |
| Assembly Cell | Combine base + lid | Position, clamp, confirm | Asm_BaseReady, Asm_LidReady, Asm_Done |
| Quality Gate | Validate assembly | Sensor / logic checks | QA_OK, QA_Fail, Part_Type |
| Packing / Palletizing | Batch & release | Count, batch complete pulse | Pack_Count, Batch_Done |
| Safety & System Control | Safe operation | Start/Stop, door interlocks | Start_Button, Stop_Button, SafetyDoor_OK |
| Diagnostics & Stats | KPIs & logging | Count, timing, errors | Stats.*, Alarm_Active |

---

## 🔄 Process Flow (High-Level)

1. Raw part enters via infeed conveyor (presence sensor latches a record).
2. Classification sensors identify type (Blue / Green / Metal).
3. Sorting logic actuates diverters / stoppers to queue or send part to a machining center.
4. When a machining station is free and buffer has stock → transfer part.
5. Machining cycle executes (timed or progress-based) producing a base or lid component.
6. Pick & Place units collect required base + lid and deliver to assembly station.
7. Assembly clamps, positions, and completes mechanical join (simulated).
8. Quality Gate validates correct pairing and component presence.
9. Accepted units increment packing counter; upon reaching Batch_Size a Batch_Done pulse signals a virtual packing event.

---

## 🧠 Control Architecture

Suggested program structuring (TIA Portal):

Program Blocks (OB / FB / FC):
* OB1 – Main cyclic orchestration calling modular FBs.
* Additional FBs (Sorter, Buffer, Machining, Pick & Place, Assembly, QA, Packing, Safety, Alarms, Utility).

### PDF Documentation (Uploaded)

* Main Organization Block: [Main(OB1).pdf](https://github.com/user-attachments/files/21991994/Main.OB1.pdf)
  
* Pick & Place Machine 1: [Pick and Place 1.pdf](https://github.com/user-attachments/files/21992008/Pick.and.Place.1.pdf)
  
* Pick & Place Machine 2: [Pick and Place 2.pdf](https://github.com/user-attachments/files/21992019/Pick.and.Place.2.pdf)
  
* Pick & Place Machine 3: [Pick and Place 3.pdf](https://github.com/user-attachments/files/22001818/Pick.and.Place.3.pdf)


### Data / Program Blocks Overview PDFs
* Program Blocks Summary: [Programe Blocks.pdf](https://github.com/user-attachments/files/21992036/Programe.Blocks.pdf) (Typo in filename: should be “Program”)
* Tags / Global Variables: [Tags.pdf](https://github.com/user-attachments/files/22001814/Tags.pdf)


---

## 🧬 Animation (Process Clips)

Sorting:  

https://github.com/user-attachments/assets/aebd0dca-e3bc-4d08-803b-5e45f96bc2d8


Machining:  


https://github.com/user-attachments/assets/11333f61-7730-4964-84fb-701ae40516f8


Assembly:  


https://github.com/user-attachments/assets/c5af7db6-ae08-4349-af2a-88a7930d3e28


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


---

## ⚙️ Tag Naming Convention (Examples)

Prefix style (legacy):
* I_ (Inputs) – e.g., I_Infeed_PE
* Q_ (Outputs) – e.g., Q_Div_Left
* M_ (Memory) – e.g., M_Part_Latched

Structured style (recommended):
* gIO.Sensors.Infeed.Present
* gCtrl.Machining[1].State
* gStat.Packing.BatchCount


---

## 🔌 I/O Mapping (Example Skeleton)

| Function | Factory I/O Element | PLC Address | Tag |
|----------|--------------------|-------------|-----|
| Infeed Photoeye | Sensor_1 | I0.0 | I_Infeed_PE |
| Color Sensor Blue | Diffuse_Blue | I0.1 | I_Color_Blue |
| Color Sensor Green | Diffuse_Green | I0.2 | I_Color_Green |
| Metal Capacitive | Cap_Sensor | I0.3 | I_Metal |
| Diverter Left Solenoid | Diverter_L | Q0.0 | Q_Div_Left |
| Diverter Right Solenoid | Diverter_R | Q0.1 | Q_Div_Right |
| Machining Clamp | ClampAct | Q0.2 | Q_Clamp |
| Spindle Motor | Spindle | Q0.3 | Q_Spindle |
| QA Reject Pusher | RejectPusher | Q0.4 | Q_Reject |


---

## 🧪 Quality Gate Logic (Sample Pseudocode)

```
IF RisingEdge(Assembled_Unit) THEN
  IF (PartType = BLUE AND Machined = TRUE AND LidPresent = TRUE)
     OR (PartType = GREEN AND Machined = TRUE AND LidPresent = TRUE)
     OR (PartType = METAL AND Machined = TRUE AND (NOT RequiresLid OR LidPresent))
  THEN
     QA_OK := TRUE;
  ELSE
     QA_Fail := TRUE;
  END_IF;
END_IF;
```

---

## 📦 Packing Logic

* Parameter: Batch_Size (e.g., 10)
* On QA_OK rising edge → Pack_Count := Pack_Count + 1
* IF Pack_Count >= Batch_Size THEN
  * Batch_Done (one-shot pulse)
  * Pack_Count := 0

Future: Multi-layer palletization, dynamic batch sizing per recipe.

---

## 🛡️ Safety & Interlocks (Simulation-Oriented)

Soft Checks:
* Jam detection: Part presence not changing while conveyor commanded for > timeout.
* Diverter mutual exclusion: Prevent simultaneous left/right coil outputs.
* Machining start requires clamp feedback & Part_Present.
* Assembly requires base + lid ready flags.
* E-Stop / Stop button sets MasterInhibit until Reset.

Optional:
* Central Alarm Aggregator (Bitwise OR of faults).
* Controlled restart logic (wait for all moving axes to reach safe position).

---

## 🧾 KPIs (Planned)

* Parts_Infeed_Total
* Machined_OK
* Assembled_OK
* QA_Fail_Count
* Packaged_Units
* Batches_Completed
* Avg_Machining_Cycle_Time
* Machine_Utilization[%]
* MTBF / MTTR placeholders (manual capture initially)

---

## 🛠️ Toolkit

* PLC: Siemens S7-1200 (PLCSIM)
* IDE: Siemens TIA Portal v19
* Simulation: Factory I/O (Siemens S7-PLCSIM driver)
* (Optional) HMI: WinCC panel / PC runtime

---

## 🚀 How to Run

1. Restore the `.zap19` project in TIA Portal v19.
2. Compile & load to PLCSIM (check correct CPU type).
3. Open the `.factoryio` scene in Factory I/O.
4. Configure Driver: Siemens S7-PLCSIM (IP 127.0.0.1), verify Rack/Slot (e.g., 0/1).
5. Connect – confirm sensor bits change when parts are inserted.
6. Enable master start (HMI / Watch table).
7. Monitor key tags: gCtrl.Machining[*].State, gStat.Packing.Pack_Count, QA_OK, Diverter_CMD.
8. Run animations to validate flow.

---

## 🧪 Testing Checklist

| Test | Action | Expected |
|------|--------|----------|
| Infeed Detection | Place part at entry | I_Infeed_PE = 1 |
| Sorting | Introduce Blue/Green/Metal sequence | Correct diverter activation |
| Buffer Full | Fill buffer beyond threshold | New routing pauses for that type |
| Machining | Feed & start cycle | State transitions 0→6 |
| Assembly | Provide base + lid | Asm_Done + QA_OK |
| QA Reject | Disable lid for lid-required type | QA_Fail increments |
| Packing | Run Batch_Size good units | Batch_Done pulse |
| Stop Button | Press mid-process | All motion outputs drop |
| Restart | Reset & Start | Cycle resumes from safe Idle |

---

## 🧩 Troubleshooting

| Symptom | Possible Cause | Resolution |
|---------|----------------|-----------|
| No PLC connection | Wrong driver IP / slot | Reconfigure driver |
| Diverter inactive | Interlock or no part request | Check Diverter_CMD & part classification |
| Machining stuck in WaitPart | Buffer empty flag true | Inspect Buffer_Release logic |
| Assembly never starts | Missing lid or base ready | Validate readiness bits |
| Batch never completes | QA_OK edge missing | Review edge detection logic |
| Continuous QA fails | Classification mismatch | Trace PartType & LidPresent sensors |

---

## 🔮 Future Enhancements

* RFID / serialized tracking
* Recipe management (per-type cycle times, required lid)
* OEE dashboard (Availability / Performance / Quality)
* Fault injection (toggle simulated sensor errors)
* Migration of sequences to SCL for maintainability
* OPC UA / MQTT integration for MES / SCADA layer
* Alarm history ring buffer with timestamping

---

## 🗂️ Repository Structure (Proposed)

```
/tia_project/            (zap19 + exported blocks)
/factory_io/             (.factoryio scene)
/docs/
  images/
  pdf/
    ob1-main.pdf
    fb-pick-place-1.pdf
    program-blocks.pdf
    data-blocks.pdf
  io-mapping.md
  state-machines.md
/hmi/                    (HMI runtime or screens)
README.md
LICENSE
```

---


## 🙌 Contributions

1. Fork repository  
2. Create feature branch (`feat/...`)  
3. Commit with descriptive messages  
4. Open Pull Request with summary + test evidence  

---

## ❓ FAQ

Q: Can I use S7-1500 instead?  
A: Yes—adjust hardware configuration and confirm address integrity.

Q: Can this run on a physical cell?  
A: Core logic can, but implement real safety (E-Stop circuits, guard monitoring).

Q: Why are timers used for some steps?  
A: Simulation abstracts physical confirmations; replace timers with sensor feedback in real hardware.

---
