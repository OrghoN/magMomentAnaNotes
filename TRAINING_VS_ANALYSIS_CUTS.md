# Training cuts vs. final analysis cuts

This note compares two event selections:

* the **final analysis cutflow** of the νMM tech note, Table 12 (pp. 16–17),
  with the details in Tables 6–11 and Sections 3.1, 4 and 5;
* the **training selection** in the CSV extraction macros in
  `nova_data_extractor/mag_moment/`:
  `enhanced_nu_on_e_extractor.C` (signal), `nd_bg_extractor.C` (nominal ND background) and
  `mec_extractor.C` (νe CC MEC background).

The three macros have **identical reco selections** (`kTrainingSelection`). They differ only in their truth requirement and input dataset. So one
comparison covers all three.

References written "tech note §N" are to sections of the tech note. A bare "§N" is a section
of this document.

Library values are quoted from `CAFAna/Cuts/NuoneCuts.h` in the `development` release on
CVMFS. These are the ND ν-on-e group's cuts. The tech note's "originally" values are the same
numbers.

---

## 1. Cut-by-cut comparison

| # | Cut | ND group (`NuoneCuts.h`) | **Analysis** (tech note Table 12) | **Training** (macros) | Training vs. analysis |
|---|-----|------------------|---------------------------|-----------------------|---------------|
| 1 | slicing | – | – | – (standard slices, `kStandardSpillCuts`) | same |
| 2 | vtxIsValid | – | > 0 | `vtx.elastic.IsValid` | same |
| 3 | nPng | – | > 0 | `knpng > 0` | same |
| 4 | longestProng | < 800 cm | **< 600 cm** | < 800 cm | looser (back to ND value) |
| 5 | nPlane | < 120 | **< 95** | < 120 | looser (back to ND value) |
| 6 | nHits (nCell) | < 600 | **< 200** | < 600 | looser (back to ND value) |
| 7 | closestSlc | `ktimeCut` | −25 < Δt < 25 ns rejected | `nuone::ktimeCut` | same |
| 8 | fiducial | Table 7 box | min (−185, −175, 95), max (175, 175, 1095) cm | `nuone::knuoneFiducial` | same (values checked) |
| 9 | containment | Table 7 box | min (−190, −180, 105), max (180, 190, 1275) cm | `nuone::kcontainment` | same (values checked) |
| 10 | showerEFrac | > 0.8 | **> 0.8** | > 0.5 | looser (lower edge of the tech note §4.2 validation region) |
| 11 | vtxE | < 0.02 GeV | **< 0.016 GeV** | < 0.05 GeV | looser, and also looser than the ND value |
| 12 | gap | < 20 cm | **< 10 cm** | < 20 cm | looser (back to ND value) |
| 13 | showerE | 0.5–5 GeV | **0.3–5 GeV** | 0.1–10 GeV | looser at both ends |
| 14 | nuoneid (CNN) | > 0.73 | **> 0.73** | none | dropped (stored as a column) |
| 15 | epi0id (CNN) | > 0.92 | **> 0.92** | none | dropped (stored as a column) |
| 16 | eth2 (Eθ²) | < 0.005 | **< 0.005 GeV·rad²** | none | dropped (stored as a column) |

In short:

* **Kept as in the analysis (cuts 1–3 and 7–9):** the basic quality cuts, the timing
  isolation cut, the fiducial cut and the containment cut.
* **Loosened (cuts 4–6 and 10–13):** the activity cuts, the single-shower cuts and the shower
  energy window. Four of them go back to the ND-group values that the tech note started from.
  showerEFrac, vtxE and showerE go further than that.
* **Dropped (cuts 14–16):** both CNN classifiers and the forward-going cut Eθ².

Every dropped or loosened cut variable is written to the CSV.The macros
also write two pass bits:

* `passAnaPresel` = Table 12 cuts 1–13;
* `passAnaFull` = all of Table 12.

These bits are built from Table 12's numbers by hand, not from the library `nuone::kPreselection`,
which still has the ND-group values. With them the exact analysis selection can be
recovered from the training CSV when the model is evaluated.

The training selection is a strict **superset** of both the analysis selection and the
ND-group selection: every event that passes either one also passes the training cuts.

---

## 2. Why loosening the cuts for training makes sense

The analysis cutflow and the training selection are built for different jobs.

* **The analysis cutflow** is a classifier. Each threshold was chosen to maximise a figure of
  merit, S/√(S+B) or S/√(S+B+δB²) (tech note §3, Figs. 4–8), so that the νMM signal stands
  out from background in the final Eθ² spectrum.
* **The transformer** is a regressor. From the reconstructed prongs and slice quantities, it
  estimates `trueE` (the neutrino energy) and `trueLepE` (the recoil-electron energy). It needs
  to learn the mapping from reconstructed to true quantities over the region where it will be
  used, and it should not be thrown off at the edges of that region.

A cut that maximises sensitivity is not automatically a good definition of a training
domain. The arguments below explain why loosening helps.

### 2.1 Avoid truncation bias at the cut edges, especially the energy threshold

The `showerE` cut is placed on a **reconstructed** energy (the calE of the leading shower). If
training uses the same 0.3 GeV threshold, then near threshold the sample contains only events
whose reconstructed energy fluctuated *up*. Events with the same true energy that fluctuated
down are missing. A regressor trained on this sample learns a mapping from reconstructed to
true energy that is biased exactly where the cut sits. The model also sees no neighbouring
events below the edge, so its predictions just inside the edge rest on one-sided information.

Lowering the training threshold to 0.1 GeV (and raising the upper edge from 5 to 10 GeV)
gives a margin on both sides. The analysis window, 0.3–5 GeV, then lies **inside** the
training domain rather than at its boundary. The model sees the full migration population for
every event that ends up in the analysis window.

The low-energy edge matters most for this analysis. The magnetic-moment cross section grows as
the recoil energy falls  and the threshold was lowered from
0.5 to 0.3 GeV exactly to gain that region. The most valuable part of the signal sits right at
the threshold, so this is where a truncation bias in the energy estimator would cost the most.
With `FlatSpectraWeights` and a percentage-error (MAPE) loss in the training config, low-energy
events already get a lot of weight in training. Having real data below the edge stops the model
from fitting a distorted edge.

The same reasoning applies, less strongly, to the other loosened cuts. Each of them is
correlated with how the energy is deposited:

* **showerEFrac** is the fraction of slice energy in the leading shower;
* **vtxE** is the energy near the vertex outside the leading shower;
* **gap** affects which hits are associated with the shower.

A hard cut on any of these removes one tail of the reco-to-true relationship. Loosening them
lets the model learn how that relationship changes as each variable approaches its analysis
value.

### 2.2 More training statistics where the analysis has few events

The full analysis cutflow is very tight. Table 19 of the tech note shows how few signal
events survive. A transformer with 12 layers and 16 heads (the `input_nova_magMomentAna*.json`
configs) needs far more examples than the final selection keeps.

The loosened events are not a different physics process. They are the same ν-on-e (or
background) interactions with slightly more activity, a slightly less dominant leading shower,
or slightly more or less energy. They teach the same reco-to-true mapping. A larger sample
reduces variance and overfitting, and it gives the model more prong topologies to learn from.
How much larger the sample becomes has not been measured here. The cutflow tables written by
the macros would give it.

### 2.3 Classifier cuts are not energy-quality cuts (nuoneid, epi0id, eth2)

The CNN cuts and the Eθ² cut answer the question "is this a ν-on-e event?". They do not
answer "is the energy of this event well measured?". Dropping them for training is justified
for four reasons.

* **Eθ² depends on the reconstructed energy.** Eth2 = E_reco·θ². Cutting on it couples the
  allowed energy range to the reconstructed angle: at high E only very small θ survive. That
  carves a non-trivial shape into the joint distribution of the regression inputs. Training on
  a sculpted sample would build that correlation into the energy estimator.
* **It keeps the regressor independent of the CNNs.** NuoneID and Epi0ID were trained by the
  ND group for the Standard Model ν-on-e signal. The tech note names retraining them for νMM
  as a possible improvement (lines 275–278). If the training sample does not depend on the CNN
  scores, the energy model stays valid when the CNNs change.
* **The tech note's own control regions drop these cuts.**
  * The low-energy data/MC validation (tech note §4.2) inverts the cut to 0.5 < showerEFrac < 0.8, and
    drops the CNN and eth2 cuts.
  * The Eθ² sideband study (tech note §5, lines 411–415) drops the CNN cuts.

  An energy estimator that will be checked or used in these regions has to be trained on
  events that populate them. The training cut showerEFrac > 0.5 is set to the lower edge of
  the tech note §4.2 validation region for exactly this reason. So the model can be checked against
  data in a region that has no signal, before anyone looks at the signal region.
* **Background events have to be in training.** Every event that passes the final selection
  gets an energy estimate, backgrounds included. The training sample must contain the
  backgrounds that look like the signal (NC π⁰, νμ CC with a π⁰, νe CC, νe CC MEC). If the
  CNNs were applied, almost all of those backgrounds would be removed, which is the job the
  CNNs were trained for. The model would then never see them.

### 2.4 Going back to ND-group values gives a validated, well-defined superset

For longestProng, nPlane, nHits and gap, the training value is exactly the ND ν-on-e group's
value (`NuoneCuts.h`). The tech note calls these the "original" values and only tightened them
on FOM grounds (Figs. 4–6 and 8). These values belong to an established selection that the ND
and LDM groups use and have validated against data. They are not arbitrary. The tech note
itself says the tightening was a sensitivity optimisation, and that the sideband and low-energy
studies show no new data/MC problems from the changes (tech note §5.3, lines 434–445).

Training on the ND-group preselection therefore gives one model that can be used with either
cutflow. It also means that any future re-optimisation of these four FOM-driven thresholds
stays inside the training domain, so the model does not have to be retrained.

### 2.5 Why some cuts are *not* loosened

The cuts kept at analysis values are the ones that decide whether the reconstructed quantities
contain the information the model needs:

* **vtxIsValid and nPng > 0.** The input features (prong directions and starts, the
  leading-prong calE, the leading-prong truth match) do not exist without a valid vertex and
  at least one prong.
* **Fiducial and containment.** The tech note applies these because the detector edges are not
  well understood in terms of energy deposition (lines 234–242). If energy leaks out of the
  detector, the true energy cannot be recovered from the hits, and the regression target is no
  longer a function of the inputs. Training on uncontained events would only add irreducible
  noise. The signal truth definition (`kNuoneSigFid`) also uses the fiducial volume, so reco
  and truth stay consistent.
* **closestSlc (timing isolation).** Slices close in time to another slice are likely to share
  hits from pile-up. Their reconstructed energy then contains activity that the truth label of
  `mc.nu[0]` does not describe, which is label noise.

So the rule is: **loosen cuts that select signal over background, and keep cuts that ensure the
event is measurable.**

---

## 3. Consequences and things to check

1. **Report performance on `passAnaFull == 1`.** Resolution and bias numbers for the analysis
   must come from the analysis selection, not from the training selection. `passAnaPresel` and
   the eth2 sideband (0.005 ≤ `etheta2` < 0.04) give more evaluation regions.
2. **The training prior is not the analysis prior.** A regressor learns roughly
   E[true | reco] under the training population. The loose sample has a different energy
   spectrum and background mix from the final selection. `FlatSpectraWeights` reduces how much
   the result depends on the true-energy spectrum, but it does not remove the dependence.
   Check the bias in the `passAnaFull` subset as a function of true energy, especially in
   0.3–0.5 GeV. If the bias there is significant, consider reweighting or fine-tuning on the
   analysis-like subset.
3. **The three macros must keep identical reco cuts.** Their CSVs are merged by column
   position, and the background and signal samples have to share one selection. Change any
   threshold in all three macros at once.

