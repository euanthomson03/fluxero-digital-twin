Fluxero Digital Twin (MVP)

A lightweight digital twin pipeline for simulating second-life solar → DC-DC boost converter → electrolyser behaviour.
This MVP connects ngspice, a C shim, and a Go API that Unity can call in real time.

This lets Unity request a simulation (e.g., number of panels), ngspice runs the electrical model, and the Go service returns hydrogen output and system behaviour.

🌍 Current MVP Overview

The system currently includes:

✓ SPICE Electrical Model

Located in netlists/dcdc_boost_basic.cir

Simulates DC-DC boost converter + electrolyser stub

Supports adjustable parameters (Ton, Tper, Iset, Vmin, etc.)

✓ C Shim (sim_shim)

Wraps the ngspice runner

Provides a callable function run_simulation()

Used by Go service to trigger simulation and read results

✓ Go Simulation Service

Exposes a simple HTTP API

Runs the C shim with chosen parameters

Returns clean JSON output for Unity

Listens on localhost:8080

This is the foundation needed for Unity to animate hydrogen tank filling, show system performance, and eventually integrate real-world data.

📁 Folder Structure
/netlists
    dcdc_boost_basic.cir

/build
    spice_runner        # compiled C++ runner that calls ngspice

/go_service
    main.go             # Go web API
    sim_shim.c          # copied C shim
    sim_shim.h

/src
    test_shim.c         # basic C shim test program

UNITY_DATA.csv           # legacy CSV output (not needed for API)

🚀 Running the Simulation API
1. Go to your project folder:
cd /Users/euanthomson/Documents/packages/electronics/spice

2. Start the Go service:
go run ./go_service


You should see:

Go sim service listening on :8080

🧪 Testing the API (CURL or Postman)
From another terminal:
curl "http://localhost:8080/simulate"


Expected JSON result:

{
  "vout_avg_V": 71.222,
  "vout_pp_V": 0.468,
  "Pin_avg_W": 128.909,
  "Pout_avg_W": 101.453,
  "eff_pct": 78.7,
  "H2_kg_window": 1.08e-7,
  "uptime_pct": 100,
  "sim_duration_s": 0.005
}

Testing with Postman (recommended)

Open Postman

Create a new GET request

URL: http://localhost:8080/simulate

Press Send

JSON response appears instantly

This is how Shiv will interact with the API.

📡 Unity Integration (for Shiv)

Unity should:

Send an HTTP GET request to:

http://localhost:8080/simulate


Receive the JSON response

Update Unity UI or animations:

tank fill level

hydrogen produced

electrolyser performance

system efficiency

Optional: Unity can poll the API repeatedly to simulate “live” behaviour.

🔧 Parameters (configurable inputs)

Right now the service uses defaults in the netlist:

Ton (μs)

Tper (μs)

Iset (A)

Vmin (V)

Rs_el (Ω)

Cbus (μF)

n_panels

These can easily be exposed as API inputs once Unity needs them.

📈 JSON Output (current)
Field	Meaning
vout_avg_V	Average converter output voltage
vout_pp_V	Ripple peak-to-peak voltage
Pin_avg_W	Power from PV panels
Pout_avg_W	Power into electrolyser
eff_pct	Converter + electrolyser combined efficiency
H2_kg_window	Hydrogen produced over measurement window
uptime_pct	% time system operated above minimum voltage
sim_duration_s	Simulation duration

This is enough for Unity to visualise basic hydrogen output.

🛠 Next Steps (Future Work)
Developer tasks:

Allow Unity to send panel count as a query parameter

Add weather/irradiance input

Add panel degradation curves

Add more detailed electrolyser model

Unity tasks:

Build front-end UI for simulation results

Animate hydrogen tank filling

Connect settings sliders to API parameters

✔ Status

The digital twin MVP backend is fully running:
Go service → C shim → ngspice → JSON → ready for Unity integration.
