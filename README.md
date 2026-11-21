# CFD Validation of a Water-Droplet-Groove Microchannel Heat Sink (ANSYS ICEM CFD + Fluent)

Independent CFD validation of a published liquid-cooled microchannel heat sink study, done to learn the full simulation workflow end to end: geometry, meshing, solving, post-processing, validation against published data, and how much of that workflow can be automated.

**Reference paper:** Soo, Yu & Ismail (2025), *Heat transfer enhancement of microchannel heat sink with water-droplet grooves*, J. Braz. Soc. Mech. Sci. Eng. 47:568. DOI: [10.1007/s40430-025-05841-1](https://doi.org/10.1007/s40430-025-05841-1)

Validation work was guided by a PhD professor. The extensions beyond the paper (aspect ratio 4 and the entropy generation analysis) were my own and were done without his guidance.

---

## Why this problem matters for electronics cooling

Liquid-cooled microchannel heat sinks are the basic geometry behind cold plates for high-power chips. The central engineering trade-off in this study is the one a cold-plate designer faces: **thermal resistance versus pumping power**. Grooves improve heat transfer, but they also raise the pressure drop.

## Setup at a glance

| Item | Value |
|---|---|
| Geometry | Single periodic channel (unit cell): smooth, in-line grooved, staggered grooved |
| Groove pitch | 1250 µm (the pitch the paper found best; the pitch study was not repeated) |
| Flow | Steady, laminar, inlet velocity 0.2–1.0 m/s (Re ≈ 150–750) |
| Thermal load | Constant heat flux 650 kW/m², fluid inlet temperature 293 K |
| Meshing | ANSYS ICEM CFD, unstructured tetrahedral mesh |
| Solver | ANSYS Fluent, runs driven from ANSYS Workbench |
| Post-processing | CFD-Post, Excel, Python (Matplotlib) |

[[Confirm before pushing: coolant (water?), solid/fluid domains and boundary conditions, and whether the model is conjugate or fluid-only.]]

## Validation results

The paper's reference values (maximum temperature, thermal resistance, pumping power) were **digitised from the paper's Fig. 11**, not taken from a table, so the reference data itself has read-off uncertainty of roughly the size of the smallest errors below.

**Smooth channel (AR-2 validation case), 0.2 to 1.0 m/s**

| Quantity | Error vs paper |
|---|---|
| Maximum temperature | 0.01–0.09 % |
| Thermal resistance | 0.23–0.77 % |
| Pumping power | 0.005–0.09 % |

**Grooved channels (AR-2 validation case), 0.2 to 1.0 m/s**

| Geometry | Max temperature | Thermal resistance | Pumping power |
|---|---|---|---|
| In-line | 0.76–1.36 % | 4.9–21.1 % | 6.9–9.4 % |
| Staggered | 0.90–1.58 % | 5.9–19.7 % | 9.1–10.8 % |

Maximum temperature agrees well, but **thermal resistance error grows with velocity and reaches about 20 %** for the grooved channels. Thermal resistance is based on the temperature rise above the inlet (roughly 13–17 K in these runs), so a temperature difference of a few kelvin turns into a large percentage. See "Known limitations".

## Mesh study

Two studies are in the workbook.

1. **Grid study (smooth channel, GSF 1 to 5).** Used to pick the working mesh. The working mesh is **GSF 3 (163,237 elements)**, a compromise between accuracy and the compute available on a laptop.
2. **Mesh study (grooved channel, global element size 0.25 mm down to 0.05 mm, 96,533 to 11.3 M elements).**
   - Pumping power settles by about 0.43 M elements (4.59–4.66 × 10⁻⁵ W across the finer meshes).
   - Maximum temperature is still drifting down at the finest converged meshes (about 307.9 K at 0.86 M, 306.4 K at 2.8 M, 304.6 K at 6.6 M elements). It has **not reached a plateau**.
   - The 11.3 M element run was stopped before convergence because it was too large for my hardware, so it is excluded.

## Automation

- **ICEM replay scripts (`.rpl`).** I recorded one meshing session by hand, then used an AI assistant (ChatGPT) to extend the replay script so a new mesh can be generated for a different global element size or other mesh parameter. A new mesh takes under a minute this way.
- **Geometry scripting.** Scripted model clean-up and face creation for different groove sizes. This worked for the smooth and in-line cases.
- **Known script failure:** for the **staggered AR-3 case (100/300 µm radii)** the generated geometry/case file became corrupted and the solver would not converge. That case is excluded from the report, with a note.
- **Python (Matplotlib).** Plotting scripts for the results, written with AI assistance. [[Confirm file names in `scripts/`.]]

I am deliberately learning to write and debug these scripts myself rather than only prompting for them.

## Extensions beyond the paper (my own work)

- Groove aspect ratio 4 (not in the paper).
- Second-law (entropy generation) analysis. For the smooth channel and the staggered AR-4 case the Bejan number is about 0.996–1.000 over the velocity range, meaning thermal irreversibility dominates frictional. The interpretation is tentative and I am still working through the theory.
- Additional staggered cases with larger radii (200/400, 300/400, 400/400 µm): runs done, results not yet processed.

## Known limitations

- Thermal-resistance error reaches about 20 % for the grooved channels, and pumping power is 7–11 % off. I have not yet isolated the cause (candidates: working-mesh resolution, how thermal resistance is defined, and the digitised reference values).
- Mesh independence is not rigorous for temperature-based quantities (see Mesh study). A Grid Convergence Index calculation is still to be done.
- Staggered AR-3 is excluded because the case would not converge.
- A few workbook values need re-checking and may be re-run.

## Next steps

1. Grid Convergence Index on thermal resistance.
2. Make the replay scripts robust for the staggered cases.
3. Nanofluid versus water comparison.
4. Extend the same workflow to a cold-plate or electronics-cooling case.

## Repository contents

| File | What it is |
|---|---|
| `MCHS-Report.docx` | Full report |
| `Micro_Channel_Calculation_Sheet.xlsx` | Grid study, validation tables, entropy analysis |
| `Mesh_Independence_study_Calculation_Sheet.xlsx` | Mesh study on the grooved channel |
| `[[mesh_automation.rpl]]` | ICEM replay script |
| `[[scripts/]]` | Python plotting scripts |

## Contact

Kavan M S | kavan.ms98@gmail.com | [LinkedIn](https://linkedin.com/in/kavan-m-s-24a398208)
