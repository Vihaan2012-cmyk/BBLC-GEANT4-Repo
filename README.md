# Proton-Beam Water-Treatment Simulation

Geant4 (QGSP_BIC_EMZ) simulation of proton energy deposition in water, plus a
unified Python analysis script. Used to validate dose delivery (Bragg peaks vs.
analytical range) and to compute the beam time needed for kGy radiolysis doses.

## What's here
| File | What it is |
|------|------------|
| `proton_water.cc`   | Bare water slab. For the Bragg sweep + convergence study. |
| `proton_dish.cc`    | Petri-dish geometry (PS walls + 5 mm water layer). |
| `proton_cuvette.cc` | Sealed cuvette; wall material/thickness + water depth settable from the macro. |
| `*.mac`             | Geant4 macros that drive each program. |
| `analysis.py`       | One script that reads the output and makes every plot/table. |
| `CMakeLists.txt`    | Builds all three executables. |

## Requirements
- Geant4 11.x (with Qt/vis support for the viewer)
- A C++ compiler and CMake
  - Windows: Visual Studio 2022 Community with the "Desktop development with C++" workload, plus CMake
  - Linux: gcc/g++, make, cmake
- Python 3 with numpy + matplotlib (`pip install numpy matplotlib`)

---

## Build

### Windows (Visual Studio + CMake)
Open the **"Developer PowerShell for VS 2022"** (Start menu) so the compiler is on PATH, then:
```powershell
# from this folder
mkdir build
cd build

# point CMake at your Geant4 install (adjust the path to yours) and generate a VS solution
cmake -G "Visual Studio 17 2022" -A x64 -DGeant4_DIR="C:/path/to/geant4/lib/cmake/Geant4" ..

# compile (Release build)
cmake --build . --config Release
```
This produces `proton_water.exe`, `proton_dish.exe`, `proton_cuvette.exe` inside
`build/Release/`, with the macros copied alongside.

If Geant4 set up an environment script, run it first so the DLLs and data files
are found, e.g.:
```powershell
& "C:/path/to/geant4/bin/geant4.bat"
```

### Linux (make)
```bash
# from this folder
mkdir build && cd build
cmake ..
make -j$(nproc)
```
Produces `proton_water`, `proton_dish`, `proton_cuvette` in `build/`.

---

## Run the simulation (creates the .txt data files)

### Windows (from build/Release, in the Developer PowerShell)
```powershell
cd Release

# Bragg curves at several energies        -> edep_*MeV.txt
.\proton_water.exe run_all.mac

# Full sweep (energies x particle counts) -> edep_E*MeV_N*.txt
.\proton_water.exe sweep.mac

# Dose in the 5 mm water layer (dish)     -> edepW_E*MeV.txt
.\proton_dish.exe dish.mac

# Same, but cuvette geometry              -> edepW_E*MeV.txt
#   edit cuvette.mac to set wall material/thickness, then:
.\proton_cuvette.exe cuvette.mac

# Visual check (opens the Qt viewer)
.\proton_water.exe            # no macro arg -> interactive, runs vis.mac
```

### Linux (from build/)
```bash
./proton_water run_all.mac
./proton_water sweep.mac
./proton_dish dish.mac
./proton_cuvette cuvette.mac
./proton_water                # interactive viewer
```

---

## Analyse / plot (reads the .txt files made above)
Run the script in the folder where the `.txt` files were written.

### Windows
```powershell
# from build/Release (where the .txt files are)
python ..\..\analysis.py            # interactive menu
python ..\..\analysis.py dose       # just the feasibility table
python ..\..\analysis.py all        # every analysis
```

### Linux
```bash
python3 ../analysis.py
python3 ../analysis.py dose
python3 ../analysis.py all
```

The menu asks which analysis, whether to draw the image or print numbers only,
and lets you override the facility assumptions (spot area, current, depth,
target doses) at runtime. Outputs are PNGs prefixed `fug_`.

---

## Setting cuvette wall material/thickness (no recompile)
Edit the top of `cuvette.mac`:
```
/cuvette/wallMaterial  G4_POLYSTYRENE     # or G4_Pyrex_Glass, G4_PLEXIGLASS ...
/cuvette/wallThickness 1.0 mm
/cuvette/waterDepth    5.0 mm
```
If you change `wallThickness` or `waterDepth`, update the scoring-mesh
`translate`/`boxSize` in the same macro (the program prints the correct
`waterCentreZ` at startup).
