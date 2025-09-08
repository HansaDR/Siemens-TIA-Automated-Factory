# Siemens-TIA-Automated-Factory

# Automated Sorting, Machining, Assembling and Packing Simulation

A fully simulated discrete manufacturing line built with Siemens TIA Portal (v19), PLCSIM, and Factory I/O.  
This project now includes completed palletizer and overhead crane subsystems (see demo link below). The virtual plant performs a complete mini production workflow:
1. Infeed & Identification (color / material based)
2. Sorting to dedicated buffers
3. Machining (e.g., drilling / lid production)
4. Pick & Place / Assembly (pairing product base with product lid)
5. Quality Gate (presence / type validation)
6. Packing / Palletizing (batch-based release)
7. Pallet handling with Palletizer and Overhead Crane (pallet transport / staging)

Demo (YouTube — full system including Palletizer & Crane): https://youtu.be/OTtNMz4cj60

---

## 📸 Layout & Images

(Consider mirroring these images into `docs/images/` for repo permanence.)

![Layout 1](https://github.com/user-attachments/assets/7707c6c0-8167-4c50-aa64-389f60f18609)

![Layout 2](https://github.com/user-attachments/assets/8c1d7da5-2009-4099-b6bf-49c8d57da534)

![Palletizer](https://github.com/user-attachments/assets/8878b386-3330-4e31-9886-498a8339153e)

![Crane](https://github.com/user-attachments/assets/be5425c2-5c86-4801-99ac-891e44bb19c9)

![I/O Mapping View A](https://github.com/user-attachments/assets/f992d9c7-0730-40b0-8fa6-73b1c366a4b9)

![I/O Mapping View B](https://github.com/user-attachments/assets/8ff4890f-c850-40a0-af37-7eee2e13db37)

![Siemens portal View](https://github.com/user-attachments/assets/936d9914-f021-4e46-8a9a-2890ef23163e)

![Full Layout](https://github.com/user-attachments/assets/63dd3062-6885-4eab-84bc-48e1a5f31dc1)


---

## 🎯 Project Objective

Demonstrate an educational yet industry-relevant control solution that:
* Integrates multiple functional cells (sorting, machining, pick & place, assembly, inspection, palletizing, crane handling) under one PLC program architecture.
* Uses modular, reusable, and state-driven logic blocks.
* Simulates a product lifecycle from raw material arrival to packaged and palletized units.
* Provides a base for OEE, traceability, recipe control, and integration with MES.

---

## 🧱 System Overview (Updated)

| Cell | Function | Key Actions | Example Signals |
|------|----------|-------------|-----------------|
| Infeed & Detection | Accept parts & classify | Presence detect, color/metal scan | I_Infeed_PE, I_Color_Blue, I_Color_Green, I_Metal |
| Sorting Diverter | Route parts to buffers or machines | Pivot arms, stoppers | Q_Pivot_Blue, Q_Stopper_A |
| Buffer Queues | Accumulate WIP per type | FIFO control, release handshake | BufferCount_Blue, Buffer_Release_Blue |
| Machining Centers | Produce base or lid | Clamp, spindle/process, release | MC[n]_Clamp, MC[n]_Progress, MC[n]_Done |
| Pick & Place (PP1..PP3) | Transfer & position parts | X/Z motion, rotate, grip | PPn_X, PPn_Z, PPn_Grab |
| Assembly | Combine base + lid | Positioning, clamp, confirm | ASM_BaseReady, ASM_LidReady, ASM_Done |
| Quality Gate | Validate assembly | Type & presence checks | QA_OK, QA_Fail |
| Packing / Palletizer | Batch, form cartons, pick/place on pallet | BatchCount, Pallet_Clamp, Pallet_Rise | PACK_Count, PKG_Batch_Done, PAL_Clamp |
| Overhead Crane | Move loaded pallets between stations | Travel X/Y, lift/lower, clamp | CRN_XPos, CRN_YPos, CRN_Clamp |
| Safety & Diagnostics | Start/Stop, doors, alarms | EStop, Door_OK, AlarmBits | I_Stop, SafetyDoor_OK, Alarm_Aggregate |

---

## 🔄 Process Flow (High-Level, updated)

1. Raw parts enter via infeed conveyor and are classified (Blue / Green / Metal).
2. Parts are sorted to dedicated buffers or routed to machining centers.
3. Machining centers produce machined bases and/or lids.
4. Pick & Place robots collect components and deliver them to assembly stations.
5. Assembly combines base + lid; QA gate validates the unit.
6. Valid units accumulate into batches; packing forms boxes or pallet loads.
7. Palletizer places packages on pallets and secures (clamp/strap simulation).
8. Overhead crane moves completed pallets to storage or shipping staging area.
9. System logs KPI events and errors to DB_Stats for later analysis.


https://github.com/user-attachments/assets/df8f3cd5-ff5d-4ef1-b110-b7a586000ebb



https://github.com/user-attachments/assets/bc6bd6c0-6cc8-4c47-9b1a-6a771f753608




https://github.com/user-attachments/assets/d9c5d33d-6e95-437a-aa43-d561a95f411f



https://github.com/user-attachments/assets/8f1e0e19-c0f8-4095-960b-fefa080bbcad



https://github.com/user-attachments/assets/47565103-dc18-467a-962b-02ae6d3db955


---

## 🧠 Control Architecture (Recommended)

Program Blocks:
* OB1 – Main cyclic orchestration
* OB100 – Startup / reset
* FB_Sorter – classification & diverter control
* FB_Buffer – FIFO queue logic (multi-instance)
* FB_Machining – per machine (state machine: Idle→Load→Clamp→Process→Unload)
* FB_PickPlace – per PP robot (motion & grip sequencing)
* FB_Assembly – assembly state & handshake
* FB_QA – validation logic & result pulses
* FB_Packing – batch forming & pack release
* FB_Palletizer – pallet placement sequencing, clamp & eject
* FB_Crane – crane motion & pallet handover sequencing
* FB_Safety – interlocks & aggregated inhibits
* FC_Util_* – edge detectors, timers, helpers

Data Blocks:
* DB_Global – global flags, master start, modes
* DB_IO / gIO – structured I/O snapshot (recommended)
* DB_Instance_* – instance DBs for each FB (FB_Machining_DB0, DB_PP1, etc.)
* DB_Stats – KPI counters and timestamps
* DB_Recipes – batch sizes, pallet patterns, machine recipes

PDF references included in repo:
* Main control organization: [Main(OB1).pdf](https://github.com/user-attachments/files/22209186/Main.OB1.pdf)
* Pick & Place FBs: [Pick and Place 1.pdf](https://github.com/user-attachments/files/21992008/Pick.and.Place.1.pdf), [Pick and Place 2.pdf](https://github.com/user-attachments/files/21992019/Pick.and.Place.2.pdf), [Pick and Place 3.pdf](https://github.com/user-attachments/files/22001818/Pick.and.Place.3.pdf)
* Palletizer FBs: [Palletizer.pdf](https://github.com/user-attachments/files/22209213/Palletizer.pdf)
* Crane FBs: [Crane.pdf](https://github.com/user-attachments/files/22209227/Crane.pdf)
* Program summary:[program-blocks.pdf](https://github.com/user-attachments/files/22209290/program-blocks.pdf)
* Tags / I/O list: [Tags.pdf](https://github.com/user-attachments/files/22209248/Tags.pdf)

---

## 🧬 Palletizer & Crane: Functional Summary

Palletizer:
* Accepts boxes from packing conveyor.
* Accumulates boxes according to pallet pattern (rows / columns).
* Controls pallet magazine (present / clamp / raise / lower).
* Outputs: Pallet_Ready, Pallet_Full, Pallet_Clamp, Pallet_Raise
* Handshake with Crane: Pallet_ReadyForPick → Crane picks up pallet.

Overhead Crane:
* Receives pallet pickup requests from Palletizer or operator.
* Moves along X/Y rails to pick & place pallet positions.
* Includes lift/lower and clamp/unclamp actions.
* Interlocks: speed limits, no-motion during adjacent cell active if configured.
* Outputs: Crane_Position, Crane_Busy, Crane_Error

Suggested tags (examples — map to actual %I/%Q addresses in Tags.pdf):
* PAL_Present, PAL_ClampCmd, PAL_RaiseCmd, PAL_PalletID
* CRN_XCmd, CRN_YCmd, CRN_LiftCmd, CRN_ClampCmd, CRN_Status

---

## 🧬 State Machine Example (Palletizer)

States:
0 Idle  
1 WaitForBox  
2 PlaceBoxRow  
3 AdvancePattern  
4 FullPalletPrepare  
5 SignalCranePickup  
6 Complete / Reset

Transitions triggered by box arrival, pattern index, pallet sensors and crane handshake.

---

## 🔌 I/O Mapping

Be sure to update the I/O mapping PDF/files to include:
* Palletizer sensors & actuators
* Crane travel axes, lift, clamp feedback
* Any new safety edges (light curtains / crane zone sensors)

Example mapping should appear in `docs/pdf/tags-io-mapping.pdf` (suggested) or the existing Tags.pdf.

---

## 🧪 Quality Gate & Packing Logic (Reminder)

Quality gate and packing logic remain the same, with the addition that Packed units are forwarded to the Palletizer. The packing system should emit a Palletizer_Request when enough packages are ready to start a pallet pattern.

---

## 🛠️ Toolkit

* PLC: Siemens S7-1200 (PLCSIM) or S7-1500 if you migrate
* IDE: Siemens TIA Portal v19
* Simulation: Factory I/O (Siemens S7-PLCSIM driver)
* Optional: WinCC HMI for operational control & KPI screens

---

## 🚀 How to Run (Updated)

1. Restore the `.zap19` project in TIA Portal v19 (ensure all FBs/DBs imported).
2. Compile & load to PLCSIM (check CPU selection).
3. Open the `.factoryio` scene and start Factory I/O.
4. Set driver: Siemens S7-PLCSIM (127.0.0.1), verify Rack/Slot.
5. Confirm new I/O bits for palletizer & crane respond in the driver panel.
6. Start simulation and enable master start.
7. Watch tags: PACK_Count, PAL_State, CRN_Status, QA_OK, MC[n].State.
8. Use the demo YouTube link above to compare expected behavior.

---

## 🧪 Testing Checklist (Added Palletizer & Crane tests)

| Test | Action | Expected |
|------|--------|----------|
| Palletizer Start | Send N boxes according to pattern | Palletizer places boxes row-by-row |
| Pallet Full | Fill pallet pattern | Pallet_Full = TRUE; Palletizer signals Crane |
| Crane Pickup | Palletizer signals pickup | Crane moves to pallet, clamps & transports |
| Crane Place | Crane places pallet in staging | Pallet staged; Crane returns to home |
| Palletizer Error | Pause conveyor mid-pattern | Palletizer stops & raises alarm |
| Emergency Stop | Press E-Stop | Palletizer & Crane motion stop immediately |

---

## 🔮 Future Enhancements

* Pallet pattern editor in HMI
* Pallet labeling & serialization (RFID)
* Crane collision zones & multi-crane support
* OPC UA telemetry for MES / Warehouse Management
* Enhanced alarm history & CSV export
* Optimize FBs to SCL for maintainability

---

## 🗂️ Suggested Repository Structure

```
/tia_project/            (zap19 + exported blocks)
/factory_io/             (.factoryio scene)
/docs/
  images/
    layout_01.png
    layout_02.png
  pdf/
    ob1-main.pdf
    fb-pick-place-1.pdf
    program-blocks.pdf
    tags-io-mapping.pdf
  io-mapping.md
  state-machines.md
/hmi/
/animations/
  sorting.mp4
  machining.mp4
  assembly.mp4
README.md
LICENSE
```

---

## 📜 License & Contribution

Add a license of choice (MIT, Apache-2.0, etc.) to clarify reuse. Contributions welcome — fork, branch, and open a PR with changes and test notes.

---

## ❓ Notes & Corrections Applied
* Corrected inconsistent color reference (removed stray "Red" mention; product types are Blue, Green, Metal).
* Corrected spelling “Assemblying” → “Assembling”.
* Added Palletizer and Crane functional descriptions, tags, state machines, and tests.
* Included the provided YouTube demo link at the top (your request).
* Recommended renaming and mirroring PDFs/images into `docs/` for permanence.

---

## 📎 Adobe / External Assets

If you host assets in Adobe Creative Cloud or another external storage, add the shared link here. Replace the placeholder with your real shared URL:

Adobe Creative Cloud (Shared assets): https://adobe.com/your-shared-link

---

Happy Automating — congratulations on adding the palletizer and crane!  
(If you'd like, I can prepare the commit/PR with this README and move/rename the assets into the suggested `docs/` layout — tell me the filenames and the Adobe shared URL and I'll prepare the PR content for you.)
