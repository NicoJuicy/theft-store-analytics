# Project Threshold: Strategic Intent & Development Roadmap (v2.0)

## 1. Strategic Intent

**The Why:** Retailers lose billions to "shrinkage" because current security is either reactive or computationally noisy. We are building **Project Threshold** to transform disparate RSTP video feeds into a single, deterministic state-machine.

**The Problem:** The "Dark Transition." A physical item moves from a shelf to a person and then crosses a door without a digital "Commit" command (Payment). Traditional systems fail to link these events across cameras, leading to missed detections or false alarms.

**The Solution (Geometric Invariant):** We treat the store as a **Unified Coordinate Canvas**. Every person is a "Hull" moving in a 2D floor-plan space.
* **The Invariant:** A Hull crossing the `Exit_Polygon` while `Is_Carrying == True` and `Has_Paid == False` triggers an automated evidence capture.

---

## 2. Feature Set

### Feature 1: Global Canvas Engine (GCE)
**Description:** Collapses multiple 3D camera perspectives into a single 2D "Global Coordinate Space."
* **Mechanism:** Uses homography matrices to map pixel coordinates $(u, v)$ to floor plan coordinates $(x, y)$.
* **Result:** A single source of truth for all movement, eliminating "camera-by-camera" logic.

### Feature 2: Autogenous Spatial Mapping (ASM) - *Neural Auto-Calibration*
**Description:** A neural-network-driven layer that automatically detects camera overlaps and aligns the ground plane.
* **Mechanism:** Uses a lightweight CNN (e.g., SuperPoint + LightGlue) to identify shared visual anchors (floor patterns, corners) across feeds. It calculates the relative transformation between cameras automatically.
* **Result:** Self-healing topology. If a camera is bumped or moved, the system re-calibrates the floor map without human intervention.


### Feature 3: Hull Persistence & Re-ID (HPR)
**Description:** Maintains shopper identity across non-overlapping views using visual embeddings.
* **Mechanism:** Generates a 128-D vector for each person. The system "stitches" identities based on vector similarity and spatiotemporal proximity.
* **Result:** Status bits (`Is_Carrying`, `Has_Paid`) stay "stuck" to the person throughout their journey.

### Feature 4: The Invariant Gate (TIG)
**Description:** A logic layer monitoring interactions between Hulls and defined Polygons.
* **Mechanism:** 1. **Shelf Interaction:** Hull + Shelf overlap > 2s $\rightarrow$ `Is_Carrying = True`.
    2. **Transaction Zone:** Hull + Register overlap $\rightarrow$ `Has_Paid = True`.
    3. **Exit Breach:** Hull + Exit overlap while `(Carrying && !Paid)` $\rightarrow$ Alert.


---

## 3. User Stories

### **Story 1: The Unified Journey (Platform Engineer)**
> **As a Developer**, I want to project bounding boxes onto a single floor plan, **so that** I can track a shopper's trajectory as one continuous line rather than disconnected clips.
* **Acceptance Criteria:** * Real-time $(x, y)$ coordinate generation.
    * Latency $< 100ms$.

### **Story 2: Self-Healing Topology (SRE)**
> **As an SRE**, I want the system to auto-detect camera overlaps and calibrate the map, **so that** manual re-mapping is not required after hardware maintenance.
* **Acceptance Criteria:**
    * System identifies overlaps within 60s of camera startup.
    * Ground-plane stitch error $< 15cm$.

### **Story 3: The "Sticky" Status (ML Engineer)**
> **As an ML Engineer**, I want to use Re-ID embeddings to maintain state across "blind spots," **so that** status is not lost during camera handoffs.
* **Acceptance Criteria:**
    * Re-ID accuracy $> 95\%$ across gaps.
    * Persistence of bits through 5s of total occlusion.

### **Story 4: The Evidence Receipt (Loss Prevention)**
> **As a Security Officer**, I want a notification with a snapshot of the item pickup and the person’s face, **so that** I have proof to intercept at the door.
* **Acceptance Criteria:**
    * Alert triggers within 2s of exit.
    * Evidence package contains exactly 3 frames: Interaction, Identity, and Violation.

---

## 4. Minimalist Manifesto (What Was Removed)

To ensure performance and reliability, we have explicitly **deleted** these complexities:
* **Behavioral Psychology:** No "suspicion" scores or "nervousness" detection.
* **Pose Estimation:** We don't track elbows or knees; we track the $(x, y)$ centroid of the Hull.
* **Continuous Cloud Ingest:** Only the 3-frame Evidence Package is sent to the cloud upon violation; all other video is transient and deleted locally.
* **Manual Calibration UI:** Replaced by the **ASM Neural Layer** to handle geometric alignment automatically.

---

**Next Step:** Would you like me to define the **API specification** for the Evidence Package JSON or the **Homography Matrix structure** for the ASM layer?