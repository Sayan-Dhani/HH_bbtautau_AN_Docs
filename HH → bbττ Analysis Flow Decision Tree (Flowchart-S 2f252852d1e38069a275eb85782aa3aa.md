# HH → bbττ Analysis Flow: Decision Tree (Flowchart-Style)

## A) ONLINE SELECTION (Triggers)

**Start: event accepted if it passes OR of any relevant triggers**

### A1) Resolved τ channels (τμτh / τeτh / τhτh)

- **If event passes single-μ OR μ+τh cross triggers → keep**
- **If event passes single-e OR e+τh cross triggers → keep**
- **If event passes di-τh triggers → keep**
- **If event passes DiTau+Jet trigger (2 τh + central jet) → keep**
    
    *(notable Run-3 efficiency improvement for τhτh)*
    

### A2) Boosted τhτh trigger regions (orthogonal)

- **If event passes METno-μ trigger (PFMETNoMu + PFMHTNoMu) → keep**
- **Else if event passes AK8 + ParticleNet ττ trigger → keep**

### A3) VBF parking triggers (2023)

- **If event passes any VBF parking trigger → keep**
    - VBF inclusive (2 jets with large mjj, Δηjj)
    - VBF + μ
    - VBF + e
    - VBF + τh / di-τh
    - etc.

➡️ **If none fired → event not in dataset**

---

## B) OFFLINE OBJECT BUILDING (baseline object collections)

**Build the physics objects that the analysis trusts. Apply quality IDs + kinematic thresholds + cleaning.**

### B1) Leptons and taus

- Build **tight muons** (PV cuts, iso, trigger-consistent pT)
- Build **tight electrons** (BDT ID, PV cuts, iso, trigger-consistent pT)
- Build **resolved τh** (HPS + DeepTau WPs; trigger-consistent pT)
- Build **boosted τ candidates** (DeepBoostedTau WP; OS requirement; cleaning vs leptons/fatjet)

### B2) MET and event cleaning

- Apply MET filters (beam halo / HCAL noise / bad PF muon, etc.)
- Keep MET for later (not necessarily tight-cut here)

### B3) Jets

- Build AK4 jets (apply JES/JER)
- Build AK8 jets (apply grooming variables where needed)

---

## C) CHANNEL ASSIGNMENT (ττ final state, orthogonal by construction)

**Assign event to exactly one ττ channel using priority rules.**

### C1) Primary channel logic (resolved channels first)

1. **If ≥1 tight muon exists → channel = τμτh**
2. **Else if ≥1 tight electron exists → channel = τeτh**
3. **Else if ≥2 resolved τh exist → channel = τhτh**
4. **Else → go to boosted τhτh logic**

### C2) Boosted τhτh channel

- **If ≥2 boosted τ candidates exist (OS + cleaning) → channel = boosted τhτh**
- **Else → reject event (no valid ττ candidate)**

### C3) Orthogonality enforcement (third-lepton veto)

- **If extra isolated e or μ present beyond the selected pair → reject**
    - Ensures channels don’t overlap and suppresses DY/tt-like contamination.

---

## D) ττ CANDIDATE CONSTRUCTION (choose one ττ pair)

**If multiple candidate pairs exist, keep the “best” one.**

### D1) Candidate list creation

- Construct all valid ττ pairs compatible with the channel:
    - τμτh: (μ, τh)
    - τeτh: (e, τh)
    - τhτh: (τh1, τh2)
    - boosted τhτh: (bτ1, bτ2)

### D2) Baseline ττ pair requirements

- **Opposite-sign (OS)**
- **ΔR(legs) > 0.5** (avoid overlaps/double counting)
- Both legs pass channel-specific IDs and pT/η cuts

### D3) Best-pair sorting (ambiguity resolution)

Sort candidate pairs by:

1. **leading-leg isolation** (best first)
2. tie → **leading-leg pT** (higher first)
3. then repeat for **subleading isolation**, tie → **subleading pT**
    
    ➡️ **Select the first pair passing all baseline requirements**
    

---

## E) RECONSTRUCT (m_{H\tau\tau})

**Because neutrinos prevent direct ττ mass reconstruction.**

### E1) Signal channels (τμτh / τeτh / τhτh and boosted τhτh)

- Apply **parameterized NN** neutrino regression:
    - outputs 2 neutrino 3-momenta (6 values)
    - yields reconstructed (m_{H\tau\tau}) (“regression mass”)

### E2) Control channels (μμ / ee if used in specific CRs)

- Use **FASTMTT** likelihood-based reconstruction

➡️ Save: **ττ 4-vector**, (m_{H\tau\tau}), and other ττ kinematics for later.

---

## F) bb CANDIDATE CONSTRUCTION (H→bb)

**Decide boosted vs resolved bb topology and choose the Higgs bb candidate.**

### F1) Build b-tag scores

- AK4 jets: ParticleNet **BvAll**
- AK8 jets: ParticleNet **bb-score** + soft-drop mass

### F2) Choose bb topology (boosted gets priority inside ggF-like events)

1. **If ≥1 AK8 jet satisfies boosted-H(bb) criteria**
    - e.g. pT > 300 GeV, |η|<2.5
    - groomed mass window ~80–170 GeV
    - bb-score passes chosen threshold
        
        → **bb topology = Boosted**
        
        → choose AK8 with **highest bb-score** as H→bb candidate
        
2. **Else (no valid AK8 H(bb)) → bb topology = Resolved**
    - Use AK4 jets and choose the best pair using **HHBTag**
    - Select **two AK4 jets with highest HHBTag scores** as H→bb candidate

➡️ Save: **bb candidate**, (m_{Hbb}), bb kinematics.

---

## G) APPLY b-TAGGING SCALE FACTORS (shape reweighting)

For MC:

- Apply per-jet weights (\omega(D,p_T,\eta)) to reshape ParticleNet distributions
- Multiply by residual normalization factor **r** if used in the analysis implementation

➡️ Produces final per-event weight for b-tag systematics modeling.

---

## H) VBF JET IDENTIFICATION (VBFJTag)

**Check if the event is compatible with VBF production.**

### H1) Build VBF jet candidates

- From jets not used in the bb candidate (and after cleaning rules)
- Run **VBFJTag** to pick the best dijet pair

### H2) Basic VBF kinematic requirements

- **mjj > 500 GeV**
- **Δηjj > 3.0**
    
    ➡️ If satisfied → event enters “VBF-loose” region, else “non-VBF”.
    

---

## I) CATEGORY ASSIGNMENT (VBF precedence, then ggF categories)

**Priority rule: VBF first (small cross section → preserve it).**

### I1) VBF category decision

**If (VBF-loose kinematics satisfied):**

- Evaluate **VBF DNN** (multi-class: VBF HH, ggF HH, DY, tt)
- **If VBF-DNN(VBF node) > 0.5 → Category = VBF**
- Else → not considered VBF (treated as ggF-like)

### I2) ggF-like categories (if not VBF)

Assign based on bb topology and resolved b-tag content:

- **If bb topology = Boosted → Category = Boosted**
- **Else if bb topology = Resolved:**
    - Check ParticleNet medium WP on the two HHBTag jets:
        - **If both pass → Category = res2b**
        - **If only one passes → Category = res1b**
        - (otherwise usually fails baseline / not part of SR definitions)

---

## J) MASS-BASED BACKGROUND REDUCTION (ellipse cut)

Applied only where statistics support it.

### J1) Elliptical mass cut (resolved ggF only)

- **Apply only if Category ∈ {res1b, res2b}**
- Use ellipse in ((m_{H\tau\tau}, m_{Hbb})) tuned per channel:
    - keep ~99% of signal
    - remove far-off background tails
- **Not applied** to:
    - **Boosted** category
    - **VBF** category

➡️ After this, the event is in a **final SR-like region**.

---

## K) FINAL DISCRIMINANT (DNN output)

### K1) If Category = VBF

- Use **VBF DNN output** as final discriminant (or the dedicated VBF node / scoring used by the fit)

### K2) If Category ∈ {res1b, res2b, boosted} (ggF-like)

- Use **ggF multi-class DNN output** as final discriminant

---

## L) HISTOGRAMMING AND BINNING (fit-ready shapes)

### L1) Fill histograms

For each (Era × Channel × Category):

- Fill histogram of the chosen DNN output (with event weights)
- Include backgrounds, signal, data, and systematic variations

### L2) Signal flattening / bin optimization

- Merge/choose bin edges so signal is ~flat per bin
- Apply the same binning consistently to all templates used in datacards

➡️ Output: **final templates (DNN-shape histograms) for statistical inference.**

---