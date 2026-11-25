# SC_Project

Design and coupled electromagnetic/thermal-hydraulic simulation of a 15 T REBCO superconducting solenoid wound with a 6-slot Cable-in-Conduit Conductor (CICC).

## Design Specifications

- Operating current (`I_op`): 50 kA
- Target peak magnetic field (`B_target`): 15 T
- Inner solenoid radius (`R_in`): 0.50 m
- Maximum solenoid height (`H_sol_max`): 1.50 m
- Current ramp rate (`dI/dt`): 200 A/s
- Operating inlet temperature (`T_op`): 20 K
- Required temperature margin (`dT_margin`): 5.0 K
- Coolant: supercritical helium at 5 bar (`mdot_max` = 15 g/s)

## Project Files

### Main Pipeline
- `setupProject_nuovissima.py`: Global design specifications, REBCO coated-conductor stratification, twist pitch, and thermal/electrical contact resistances.
- `dimensionamento_CICC_nuovissima.py`: CICC cross-section sizing (6 rectangular slots, copper former, central and slot helium channels, AISI 316LN jacket), twist coupling time constant, cross-section plot, and export to `CICC_Geometry_nuovissima.mat`.
- `Step2_MagneticDesign_nuovissima.py`: Axial/radial solenoid optimization minimizing total REBCO tape length, 3D Biot-Savart field evaluation (`segmentMagneticField1d`), double-pancake inductance matrix (`inductanceMatrix1d`), baseline plots, and export to `MagneticData_Final_nuovissima.mat`.
- `Step6_optimusPrime_nuovissima_c.py`: Baseline loss diagnostics at 20 K, bisection search for the minimum helium mass flow rate satisfying the 5 K margin, and 2-way coupled EM-TH transient simulation (3-equation BDF1 + upwind helium advection, NIST cryogenic solid properties, `CoolProp` helium properties, dynamic AC losses, and circuit network via `directSolve`).
- `main.py`: Runs the full workflow (`dimensionamento_CICC_nuovissima` -> `Step2_MagneticDesign_nuovissima` -> `Step6_optimusPrime_nuovissima_c`) in sequence. Supports `--no-plot` to run without opening figure windows.

### Supporting Modules
- Electromagnetic and data analysis: `Mu0.py`, `biot_savart_loops_vec.py`, `segmentMagneticField1d.py`, `inductanceMatrix1d.py`, `mutual_inductance_maxwell_grover.py`, `estrai_Ic.py`.
- Linear solver and utilities: `directSolve.py`, `factorizeMatrix.py`, `forwardBackwardSolve.py`, `checkMatrixStructure.py`, `assignFixValue.py`, `assembleSolution.py`, `iterativeRefinement.py`, `cond1est.py`, `cprintf.py`.

## Dependencies

Requires Python 3.8+ with:
- `numpy`
- `scipy`
- `matplotlib`
- `CoolProp`

## Usage

Run the full pipeline from the repository root (or from inside `SC_Project/`):

```bash
python SC_Project/main.py
```

Run without opening plot windows:

```bash
python SC_Project/main.py --no-plot
```

Or run each step individually in order:

```bash
python SC_Project/dimensionamento_CICC_nuovissima.py
python SC_Project/Step2_MagneticDesign_nuovissima.py
python SC_Project/Step6_optimusPrime_nuovissima_c.py
```

## Summary of Reference Results

- CICC layout: 348 tapes total (58 per slot), 4.00 x 7.25 mm slots, `R_former` = 20.22 mm, outer CICC side = 44.45 mm, `tau_coupling` = 2.387e-5 s.
- Magnet geometry: 30 pancakes (15 double pancakes), 16 radial turns per pancake, 905.8 km total REBCO length, `B_max` (3D) = 15.324 T, `L_DP_total` = 1.85 mH.
- Thermal-hydraulic transient: `mdot_min` = 9.695 g/s (64.6% of `mdot_max`), minimum temperature margin = 5.01 K, `max T_S2` = 22.23 K, `T_He` outlet = 22.22 K, total equatorial DP energy = 31.22 kJ (93.5% removed by helium).