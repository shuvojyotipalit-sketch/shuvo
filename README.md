⚡ EV Powertrain Simulation with Regenerative Braking
Modeling and Simulation of an Electric Vehicle Powertrain with Regenerative Braking and Industrial-Grade Mechatronic Integration

Python Platform License Domain Drive Cycle

Table of Contents
Overview
Background & Motivation
System Architecture
Subsystem Models
Project Structure
Getting Started
Simulation Results
Key Performance Indicators
Mathematical Formulations
Configuration & Parameters
Workflow
Contributing
License
Overview
This project delivers a physics-based, closed-loop simulation of a complete electric vehicle powertrain. It models the full energy chain — from battery pack through power electronics and motor to the wheels — and the reverse path through regenerative braking. The simulation is built for educational, research, and pre-validation engineering purposes.

The system integrates:

A battery equivalent circuit model with SoC tracking and fault protection
A PMSM motor model with efficiency mapping and field-weakening
A bidirectional power electronics chain (inverter + DC-DC converter)
A PID torque controller with anti-windup
A regenerative braking decision controller with speed-blending
A vehicle dynamics model with aerodynamic drag and rolling resistance
A standard drive cycle runner (UDDS urban profile)
Background & Motivation
The transition to electric mobility demands engineering tools that go beyond basic EV models. Most academic simulations omit critical industrial-grade constraints — thermal derating, SoC protection, PWM switching dynamics, and fault logic — making them unsuitable for validating real control firmware.

This project bridges that gap by implementing:

Gap in existing models	This project's solution
No regenerative braking	Bidirectional DC-DC + regen controller
Ideal motor (constant η)	Efficiency map over torque-speed space
No SoC limits	Coulomb counting + protection thresholds
No fault handling	Safe-state shutdown logic
No drive cycle	Full UDDS profile with velocity tracking
No power electronics	Inverter + DC-DC with efficiency losses
In urban driving cycles, regenerative braking can recover 15–35% of energy that would otherwise be lost as heat in friction brakes. This simulation quantifies that recovery accurately.

System Architecture
┌─────────────────────────────────────────────────────────────┐
│                    DRIVER INPUT LAYER                        │
│         Throttle / Brake Pedal →  Drive Cycle Reference      │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────┐
│                  PID TORQUE CONTROLLER                       │
│           Speed Error → Force Demand → Torque Demand         │
└────────────┬───────────────────────────────┬────────────────┘
             │ Motoring                       │ Braking
             ▼                               ▼
┌────────────────────┐           ┌──────────────────────────┐
│  INVERTER (PWM)    │           │  REGEN BRAKING CTRL      │
│  DC → 3-phase AC   │           │  T_regen + T_mech blend  │
└────────┬───────────┘           └────────────┬─────────────┘
         │                                    │
         ▼                                    ▼
┌────────────────────┐           ┌──────────────────────────┐
│  PMSM MOTOR        │           │  DC-DC CONVERTER         │
│  Torque-speed map  │           │  Bidirectional energy     │
└────────┬───────────┘           └────────────┬─────────────┘
         │                                    │
         ▼                                    ▼
┌─────────────────────────────────────────────────────────────┐
│                  BATTERY PACK (ECM)                          │
│    OCV – I·R_int  |  Coulomb counting SoC  |  Fault logic   │
└─────────────────────────────────────────────────────────────┘
         │
         ▼
┌─────────────────────────────────────────────────────────────┐
│               VEHICLE DYNAMICS MODEL                         │
│    F_aero + F_roll + F_inertia  →  v_actual [m/s]           │
└─────────────────────────────────────────────────────────────┘
Subsystem Models
1. Battery — Equivalent Circuit Model (ECM)
The battery is modelled as an ideal voltage source (open-circuit voltage, OCV) in series with an internal resistance R_int:

OCV varies linearly with SoC: V_oc = V_nom × (0.85 + 0.15 × SoC)
SoC updated via Coulomb counting: ΔSoC = -(I × Δt) / (Q_cap × 3600)
Protection: SoC clamped to [SoC_min, SoC_max]; fault flag raised at low SoC
2. Motor — PMSM with Efficiency Map
Torque limited in field-weakening region above base speed: T_max(ω) = T_max × ω_base / ω
Efficiency modelled as Gaussian peak at 70% load: η(T,ω) = η_motor × exp(−0.3 × (T/T_rated − 0.7)²)
Generator efficiency (regen) applied separately: η_gen = 0.88
3. Regenerative Braking Controller
Activated when: braking demand detected AND speed > 5 rad/s AND SoC < SoC_max
Speed-blending factor prevents instability at low speed: blend = clip(ω / 50, 0, 1)
Energy recovered: P_recovered = T_regen × ω × η_gen × η_inv × η_dcdc
4. PID Torque Controller
Standard PID with integral anti-windup clamping
Output clipped to [−T_max, T_max]
Tuned for UDDS cycle: Kp=300, Ki=20, Kd=5
5. Vehicle Dynamics
F_aero  = 0.5 × ρ × C_d × A × v²
F_roll  = C_rr × m × g
F_wheel = T_motor × gear_ratio × η_trans / r_wheel
a       = (F_wheel − F_road) / m_veh
Project Structure
ev-powertrain-simulation/
│
├── EV_Powertrain_Simulation.py   # Main simulation (paste into Colab)
├── README.md                     # This file
├── LICENSE                       # MIT License
│
├── docs/
│   └── system_architecture.png   # Architecture diagram
│
└── results/
    └── ev_simulation_results.png # Auto-generated on run

The simulation will:
- Generate the UDDS drive cycle profile
- Run the closed-loop powertrain simulation
- Print a metrics summary to the console
- Display 7 result plots
- Save `ev_simulation_results.png` to the working directory

---

## Simulation Results

Running the simulation produces seven plots:

| Plot | Description |
|---|---|
| Velocity tracking | Reference vs actual vehicle speed over UDDS cycle |
| State of Charge | Battery SoC (%) with protection thresholds shown |
| Power flow | Battery discharge (orange) and regen charge (green) power |
| Regen power | Instantaneous power recovered during braking events |
| Motor torque | Output torque with rated torque reference lines |
| Efficiency map | Motor operating points colored by efficiency % |
| Energy summary | Bar chart: consumed / recovered / net energy with regen % |

**Example console output:**

======================================================= EV POWERTRAIN SIMULATION — RESULTS SUMMARY
Drive cycle distance : 11.94 km Total energy consumed : 1842.3 Wh Energy recovered (regen) : 412.7 Wh Regen recovery rate : 22.4% Net energy used : 1429.6 Wh Consumption : 119.7 Wh/km SoC change : 2.38% Average motor efficiency : 92.1% Fault events : 0

---

## Key Performance Indicators

| KPI | Description | Typical range |
|---|---|---|
| Regen recovery rate | % of consumed energy recovered | 15–35% (urban) |
| Consumption [Wh/km] | Net energy per kilometre | 100–180 Wh/km |
| Motor efficiency | Avg η at operating points | 88–95% |
| SoC swing | Δ SoC over drive cycle | 1–5% |
| Fault events | Protection trigger count | 0 (nominal) |

---

## Mathematical Formulations

### Battery terminal voltage

V_terminal = V_oc − I_batt × R_int

where I_batt = (V_oc − √(V_oc² − 4·R_int·P_demand)) / (2·R_int)


### SoC update (Coulomb counting)

SoC[k+1] = SoC[k] − (I_batt · Δt) / (Q_cap · 3600)


### Motor power chain (motoring)

P_mech = T_demand × ω P_elec = P_mech / (η_motor × η_inv) P_batt = P_elec [discharge]


### Motor power chain (regenerating)

P_mech_in = T_regen × ω P_recovered = P_mech_in × η_gen × η_inv × η_dcdc P_batt = −P_recovered [charge]


### Aerodynamic drag

F_aero = ½ · ρ_air · C_d · A_front · v²


---

## Configuration & Parameters

All system parameters are centralised in the `EVParameters` class at the top of `EV_Powertrain_Simulation.py`. Key parameters:

| Parameter | Symbol | Default | Unit | Description |
|---|---|---|---|---|
| `V_nom` | V_nom | 400 | V | Nominal pack voltage |
| `Q_cap` | Q | 60 | Ah | Battery capacity |
| `R_int` | R_int | 0.05 | Ω | Internal resistance |
| `SoC_init` | SoC₀ | 0.90 | — | Initial state of charge |
| `T_rated` | T_rated | 250 | Nm | Motor rated torque |
| `T_max` | T_max | 400 | Nm | Motor peak torque |
| `omega_base` | ω_base | 314 | rad/s | Base speed (~3000 RPM) |
| `m_veh` | m | 1800 | kg | Vehicle mass |
| `C_d` | C_d | 0.28 | — | Aerodynamic drag coeff. |
| `gear_ratio` | γ | 8.0 | — | Single-speed gearbox |
| `Kp / Ki / Kd` | — | 300/20/5 | — | PID gains |

To modify parameters, edit the `EVParameters` class directly before running.

---

## Workflow

Define EVParameters ↓
Generate UDDS drive cycle profile ↓
Initialize Battery, PID controller, state arrays ↓
For each timestep: ├── PID → force demand → torque demand ├── If braking → Regen controller → DC-DC → charge battery ├── If motoring → Motor → Inverter → discharge battery ├── Battery ECM update (SoC, V_terminal, fault check) └── Vehicle dynamics integration (Euler forward) ↓
Post-process: plot 7 result charts ↓
Print metrics summary

---

## Contributing

Contributions are welcome. Suggested extensions:

- **Thermal model** — Motor and battery temperature dynamics with derating curves
- **Multi-speed gearbox** — Gear shift logic and efficiency map per gear
- **WLTP drive cycle** — Replace or extend beyond UDDS
- **Hardware-in-the-loop** — Export control logic to C for embedded validation
- **Battery aging** — Capacity fade model over charge/discharge cycles
- **Supercapacitor hybrid** — Add ultracap buffer for peak power

To contribute:

```bash
git clone https://github.com/your-username/ev-powertrain-simulation.git
cd ev-powertrain-simulation
# Create a feature branch
git checkout -b feature/thermal-model
# Make changes, then open a pull request
License
This project is licensed under the MIT License — see the LICENSE file for details.

Acknowledgements
Drive cycle data structure based on the EPA UDDS (Urban Dynamometer Driving Schedule)
Motor efficiency modelling informed by standard PMSM loss characterisation methods
Battery ECM approach follows Randles circuit conventions used in automotive BMS design
Built for engineers, researchers, and students working at the intersection of power electronics, control systems, and electric vehicle technology.
