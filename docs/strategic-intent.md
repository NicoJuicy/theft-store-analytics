# Project Threshold: Strategic Intent & Development Roadmap

## 1. Strategic Intent

**The Why:** Retailers lose billions to "shrinkage" because current security is either reactive (recording crimes to watch later) or noisy (AI that guesses "suspicious behavior" based on body language). We are building **Project Threshold** to transform RTSP video feeds into a real-time, deterministic state-machine.

**The Problem:** The "Dark Transition." A physical item moves from a shelf (Inventory) to a person (Transport) and then crosses a door (Exit) without a digital "Commit" command (Payment). Traditional systems fail to link these events across multiple camera feeds accurately.

**The Solution (Geometric Invariant):** Instead of detecting "theft" (a complex human behavior), we detect **"Unclosed Spatial Circuits."** We project all camera feeds onto a unified 2D floor plan. Every person is a "Hull" with two boolean bits: `Is_Carrying` and `Has_Paid`.
* **The Invariant:** A Hull cannot intersect the `Exit_Polygon` if `Is_Carrying == True` AND `Has_Paid == False`.

---

## 2. Feature Set

### Feature 1: Global Canvas Engine (GCE)
**Description:** Collapses multiple 3D camera perspectives into a single 2D "Global Coordinate Space."
* **Mechanism:** Uses homography matrices to map pixel coordinates $(u, v)$ to floor plan coordinates $(x, y)$.
* **Result:** A single "source of truth" for all movement, eliminating the need to process 10 separate video streams as independent problems.

### Feature 2: Hull Persistence & Re-ID (HPR)
**Description:** Maintains the identity of a shopper across non-overlapping camera views using visual embeddings.
* **Mechanism:** Generates a 128-D vector for each detected person. When a person leaves Camera A and enters Camera B, the system "stitches" the identity based on vector similarity and spatiotemporal logic.
* **Result:** The `Is_Carrying` and `Has_Paid` status bits stay "stuck" to the person throughout their entire store journey.

### Feature 3: The Invariant Gate (TIG)
**Description:** A logic layer that monitors interactions between Hulls and defined Polygons.
* **Mechanism:** 1.  **Shelf Polygon:** If `Hull` overlaps with `Shelf` for $> 2s$, set `Is_Carrying = True`.
    2.  **Payment Polygon:** If `Hull` overlaps with `Register`, set `Has_Paid = True`.
    3.  **Exit Polygon:** If `Hull` overlaps with `Exit` while `Is_Carrying && !Has_Paid`, trigger an alert.
* **Result:** Deterministic theft detection with zero reliance on "mood" or "suspicion."

---

## 3. User Stories

### **Story 1: The Unified Journey (Platform Engineer)**
> **As a Developer**, I want to project bounding boxes from multiple camera feeds onto a single floor plan, **so that** I can track a shopper's trajectory as a single continuous line rather than disconnected clips.
* **Acceptance Criteria:**
    * System accepts a configuration file defining `Shelf`, `Payment`, and `Exit` polygons.
    * Real-time $(x, y)$ coordinates are generated for all active Hulls.
    * Latency between physical movement and coordinate update is $< 100ms$.

### **Story 2: The "Sticky" Status (ML Engineer)**
> **As an ML Engineer**, I want to use Re-ID embeddings to maintain a Hull's state across "blind spots," **so that** a shopper who paid in Camera 3 isn't flagged as a thief when they walk out of Camera 1.
* **Acceptance Criteria:**
    * Re-ID matching accuracy of $> 95\%$ for handovers occurring within 10 seconds.
    * The `Has_Paid` bit persists even if the person is occluded for up to 5 seconds.

### **Story 3: The Evidence Package (Loss Prevention)**
> **As a Security Officer**, I want a notification containing a snapshot of the person taking the item and a snapshot of them leaving, **so that** I have immediate proof to stop them.
* **Acceptance Criteria:**
    * Alert triggers immediately upon `Exit_Polygon` intersection.
    * Evidence includes: `Event_ID`, `Identity_Crop`, `Interaction_Crop` (the moment they touched the shelf), and `Violation_Crop` (the moment they left).

---

## 4. Minimalist Manifesto (What Was Removed)

To ensure maximum performance and lower false-positive rates, we have explicitly **removed** the following complexities:
* **Behavioral Analysis:** No detection of "looking around," "pacing," or "concealment gestures."
* **Pose Estimation:** We do not track limbs or skeletal joints; the "Hull" is a single coordinate point.
* **Facial Recognition:** We do not store biometric templates; we use temporary "Appearance Embeddings" that expire when the Hull leaves the store.
* **Continuous Recording:** We do not save 24/7 video; we only save the 3-frame "Evidence Package" for violations.