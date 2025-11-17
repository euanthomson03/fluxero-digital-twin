🚀 Fluxero Digital Twin Engine

Second-Life Solar → Boost Converter → Electrolyser → Hydrogen KPIs (for Unity Visualisation)

This repository contains the backend simulation engine for Fluxero’s digital twin.
It models how second-life solar panels feed a DC/DC boost converter and a simple electrolyser load to estimate hydrogen production.

The simulation outputs clean, time-averaged KPIs into a CSV file (UNITY_DATA.csv) that an external Unity project reads to animate hydrogen tanks, gauges, and dashboards.

⚠️ Important: This repo does not contain Unity.
Unity is a separate front-end project that simply reads the CSV exported by this backend.

🌞 What This Engine Does

Full simulation chain:

PV Source → Boost Converter → DC Bus → Electrolyser Load → KPIs → (Unity Front-End)

It models:

Second-life PV voltage variation (PWL source)

Switching boost converter (inductor, MOSFET, diode)

PWM control (Ton/Tper)

Output capacitor + DC bus dynamics

Simple electrolyser load (threshold + current draw)

Instantaneous power and efficiency

Simplified hydrogen production estimate

It outputs:

sim.csv — raw ngspice waveforms

UNITY_DATA.csv — clean KPIs for the Unity visualisation frontend

📂 Repository Structure
spice/
│
├── netlists/
│   └── dcdc_boost_basic.cir       # SPICE boost converter + electrolyser model
│
├── src/
│   └── main.cpp                   # C++ runner using libngspice
│
├── build/
│   ├── spice_runner               # Compiled ngspice runner
│   ├── export_unity.py            # Converts sim.csv → UNITY_DATA.csv
│   ├── USER_INPUT.csv             # User-defined simulation parameters
│   └── UNITY_DATA.csv             # Output consumed by Unity (not stored in repo)
│
├── CMakeLists.txt                 # Build config for C++ runner
└── README.md

🔧 Build Instructions (C++ Runner)

From inside build:

cd build
cmake ..
cmake --build .


This produces the spice_runner executable.

▶️ How to Run the Simulation
1. Configure the input parameters

Edit:

nano USER_INPUT.csv


Example:

n_panels,1
Ton_us,12
Tper_us,20
Iset_A,5
Vmin_V,55
Rs_el_ohm,0.5
Cbus_uF,470

2. Run the full pipeline
cd build
python3 export_unity.py


This will:

run ngspice

generate sim.csv

compute KPIs

output UNITY_DATA.csv

3. View the output
cat UNITY_DATA.csv

🔄 Live Refresh Mode (for Unity Front-End)

To continuously simulate and refresh KPIs (Unity polls the file):

while true; do python3 export_unity.py; sleep 5; done


Unity can then read the updated CSV every few seconds to animate live hydrogen production.

📊 What Unity Reads (Format of UNITY_DATA.csv)

Unity consumes a single row with:

Column	Meaning
n_panels	Number of PV modules
vout_avg_V	DC bus voltage (average)
vout_pp_V	Voltage ripple
Pin_avg_W	Input power
Pout_avg_W	Power delivered to electrolyser
eff_pct	Converter efficiency
H2_kg_window	Hydrogen produced in last time window
uptime_pct	% of time electrolyser was active
sim_duration_s	Length of the simulation

Unity uses these values to update UI elements like:

hydrogen tank fill

voltage/power gauges

efficiency display

hydrogen production rate

Again: Unity is not included in this repo.
It is a separate project that simply reads this CSV.

🧱 Project Status
Completed:

Fully working ngspice boost converter + electrolyser model

C++ runner executing simulations automatically

Python script converting raw waveforms into clean KPIs

Clean CSV pipeline for Unity integration

Suitable for MVP demos

Next Steps:

Add weather/irradiance input

Add realistic PV degradation curves

Upgrade electrolyser model to full I–V curve

Move from CSV polling → WebSockets

Add hydrogen storage and fuel cell modules

🧪 Requirements

macOS/Linux

ngspice with libngspice

C++17 compiler

Python 3 (pandas, numpy)

Unity (external, not included here)

🤝 Contributing

Contributions improving PV models, electrolyser behaviour, or Unity integration are welcome.

📄 License

Add MIT, Apache 2.0, or any license you prefer.
