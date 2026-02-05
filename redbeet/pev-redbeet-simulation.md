Between the EV and EVSE with an AC Type 2 cable (IEC 62196-2 + IEC 61851-1), the **physical** interfaces are:

- **Power conductors**  
  - `L1`, `L2`, `L3` (single- or three‑phase AC)  
  - `N` (neutral, if used by the grid topology)  
  - `PE` (protective earth)

- **Control Pilot (CP)**  
  - Single control wire between EV and EVSE.  
  - Carries the ±12 V PWM signal (IEC 61851) for state signaling and current limitation.  
  - In CCS/Type 2 Combo DC, the same CP/PE pair is also used as the **2‑wire PLC** medium (HomePlug Green PHY) – which you already mentioned.

- **Proximity Pilot (PP)**  
  - Wire between plug and EVSE (not the EV) via a resistor to `PE`.  
  - Encodes cable current rating and, in some designs, plug button / latch status.  
  - The EV doesn’t drive it, but it’s part of the physical interface set for Type 2.

- **Protective / sensing functions embedded in the plug** (not separate wires but still “interfaces”)  
  - Resistor networks and sometimes temperature sensors inside the plug and vehicle inlet, referenced to `PE`, used by EVSE for derating or shutdown.

So beyond the 2‑wire PLC path (CP–PE), the key physical interfaces are the **AC power contacts (L1/L2/L3, N, PE)** plus the **CP** and **PP** signal conductors, with CP carrying PWM and potentially PLC, and PP providing cable/plug information.

In the Type 2 system, “resistor networks embedded in the plug” usually refers to the **proximity pilot (PP) resistor(s)** inside the cable plug body.

- **PP–PE resistor (cable coding):**  
  Inside the plug, there is at least one fixed resistor between PP and PE. Its value is defined by IEC 62196 and encodes the **maximum current rating of the cable** (e.g. values corresponding to 13 A, 20 A, 32 A, 63 A, etc.), so the EVSE knows how far it may safely load that particular cable.

- **Optional switch / second resistor (latch / button):**  
  Some plugs add another resistor and a small switch to change the total PP–PE resistance when you press the release button. This lets the EVSE detect that you are about to unplug and stop power safely.

Separately (not in the plug but in the vehicle inlet), there is another resistor/diode network on **CP–PE** that signals the EV’s presence and charging state (states A/B/C/D) to the EVSE.

User: What are the physical interfaces are between EV and EVSE with AC Type2 charging cable? I know one - 2-wire PLC interface, what is else?

GitHub Copilot: Between the EV and EVSE with an AC Type 2 cable (IEC 62196-2 + IEC 61851-1), the **physical** interfaces are:

- **Power conductors**  
  - `L1`, `L2`, `L3` (single- or three‑phase AC)  
  - `N` (neutral, if used by the grid topology)  
  - `PE` (protective earth)

- **Control Pilot (CP)**  
  - Single control wire between EV and EVSE.  
  - Carries the ±12 V PWM signal (IEC 61851) for state signaling and current limitation.  
  - In CCS/Type 2 Combo DC, the same CP/PE pair is also used as the **2‑wire PLC** medium (HomePlug Green PHY) – which you already mentioned.

- **Proximity Pilot (PP)**  
  - Wire between plug and EVSE (not the EV) via a resistor to `PE`.  
  - Encodes cable current rating and, in some designs, plug button / latch status.  
  - The EV doesn’t drive it, but it’s part of the physical interface set for Type 2.

- **Protective / sensing functions embedded in the plug** (not separate wires but still “interfaces”)  
  - Resistor networks and sometimes temperature sensors inside the plug and vehicle inlet, referenced to `PE`, used by EVSE for derating or shutdown.

So beyond the 2‑wire PLC path (CP–PE), the key physical interfaces are the **AC power contacts (L1/L2/L3, N, PE)** plus the **CP** and **PP** signal conductors, with CP carrying PWM and potentially PLC, and PP providing cable/plug information.

User: What is PLC stands for?

GitHub Copilot: In this EV charging context, **PLC** stands for **Power Line Communication**.

User: What is the resistor networks which is embedded in the plug?

GitHub Copilot: In the Type 2 system, “resistor networks embedded in the plug” usually refers to the **proximity pilot (PP) resistor(s)** inside the cable plug body.

- **PP–PE resistor (cable coding):**  
  Inside the plug, there is at least one fixed resistor between PP and PE. Its value is defined by IEC 62196 and encodes the **maximum current rating of the cable** (e.g. values corresponding to 13 A, 20 A, 32 A, 63 A, etc.), so the EVSE knows how far it may safely load that particular cable.

- **Optional switch / second resistor (latch / button):**  
  Some plugs add another resistor and a small switch to change the total PP–PE resistance when you press the release button. This lets the EVSE detect that you are about to unplug and stop power safely.

Separately (not in the plug but in the vehicle inlet), there is another resistor/diode network on **CP–PE** that signals the EV’s presence and charging state (states A/B/C/D) to the EVSE.

User: Please tell me more about resistor/diode network on CP–PE that signals the EV’s presence and charging state (states A/B/C/D) to the EVSE.



GitHub Copilot: Here’s a concise picture of the CP–PE network and states for IEC 61851 AC charging (Type 2):

**1. Basic idea**

- The EVSE drives the Control Pilot (CP) with a ±12 V signal through a resistor.
- Inside the vehicle inlet, between CP and PE, there is a **diode + resistor(s)** network.
- By measuring the resulting CP voltage, the EVSE knows:
  - whether a vehicle is connected,
  - whether it’s ready to charge,
  - whether ventilation is required.

**2. Typical vehicle-side network**

- A **diode** (usually in series from CP to the rest of the circuit) ensures the EVSE can verify it’s talking to an actual EV, not just a short.
- **Resistor R1 (≈ 2.74 kΩ)** to PE: indicates “vehicle present”.
- **Additional resistor R2 (≈ 1.3 kΩ)** switched in parallel with R1 when the EV “closes the contactor request” (i.e. wants energy).
- Some variants add another path to signal ventilation needs, but that’s less common in modern EVs.

**3. CP voltage levels and states**

Rough DC voltage levels at CP (measured by the EVSE) map to standardized “states”:

- **State A – no EV**  
  - EV side: open circuit (no diode/resistors).  
  - CP ≈ +12 V.  
  - Interpretation: cable present at EVSE, but no vehicle.

- **State B – EV connected, not yet charging**  
  - EV side: diode + R1 (2.74 kΩ) to PE.  
  - CP ≈ +9 V.  
  - Interpretation: vehicle detected, can exchange PWM info, but no contactor request.

- **State C – EV ready, no ventilation required**  
  - EV side: diode + (R1 ∥ R2) (effective ≈ 882 Ω).  
  - CP ≈ +6 V.  
  - Interpretation: EV requests charging; EVSE may close power relays if all OK.

- **State D – EV ready, ventilation required**  
  - EV side: similar network but with different resistor value giving CP ≈ +3 V.  
  - Interpretation: EV can charge only if the area is ventilated (rare for modern passenger EVs).

If a fault occurs (shorts, wrong levels), the EVSE sees **State E/F** (error) and will disable power.

**4. PWM vs. DC level**

- The **DC level** (average of the CP waveform) via the resistor/diode network selects A/B/C/D.
- The **PWM duty cycle** on CP (still around ±12 V) tells the EV the **maximum current** it is allowed to draw. The EV reads the duty cycle; the EVSE reads the resulting DC level via the resistor network.
