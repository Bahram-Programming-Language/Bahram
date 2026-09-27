<p align="center">
  <img src="assets/bahram-logo.png" alt="Bahram Logo" width="220"/>
</p>

<p align="center">
  <b>Bahram</b><br/>
  Aerospace &amp; embedded domain-specific language — direct run + console simulator + Ada/SPARK
</p>

<p align="center">
  <code>Bahram hello.bz run</code>
  &nbsp;·&nbsp;
  <code>bahramsim run adcs.bz</code>
</p>

<p align="center">
  <a href="README.md">نسخه فارسی (Persian)</a>
</p>

---

# Bahram

**Bahram** is a domain-specific language (DSL) for **safety-critical aerospace and embedded software**: flight control, satellite ADCS, FADEC, sensors, and real-time tasks.

| Feature | Description |
|--------|-------------|
| Visual symbol syntax | `@ # ? ~ ! ^ $ > < ..` |
| Run without GNAT | Built-in interpreter for `.bz` files |
| Console simulator | Virtual hardware + sensors + **RK4** physics |
| Ada/SPARK output | `pre`/`post` contracts, DO-178C-oriented path |
| Physical units | e.g. `Angle<rad>`, `Torque<N_m>` |
| Error model | `Result` types — no exceptions |

---

## 1. About the project

### Goals

Write control and avionics logic in a **simple, contract-friendly** language, then:

1. **Run immediately** on your machine (Windows/Linux) without installing Ada  
2. **Simulate** sensors and dynamics in the terminal  
3. Optionally **emit Ada/SPARK** for formal toolchains (GNAT / GNATprove)

### Repository layout

```text
bahram/
├── Bahram.py / Bahram.cmd      # Program runner
├── bahram_sim.py / bahramsim   # Console simulator
├── compiler/                   # Parser, types, Ada generator
├── interpreter/                # Interpreter
├── simulator/                  # HW, sensors, RK4
├── examples/                   # Sample .bz programs
├── stdlib/std/                 # Standard library
├── grammar/Bahram.g4
├── docs/                       # Spec & tutorials
├── playground/                 # Web editor
├── vscode-bahram/              # VS Code highlighting
├── assets/bahram-logo.png
└── dist/                       # Prebuilt binaries (if built)
```

### Example program

```bahram
@spark_level(Gold)

# factorial(n :: Int) -> Int
pre: n >= 0
post: result >= 1
? n <= 1
! 1;
..
! n * $factorial(n - 1);
..

# main() -> Int
@ r :: Int := $factorial(6);
$print(r);
! r;
..
```

---

## 2. From GitHub to Windows

### Prerequisites

- Windows 10 or 11  
- **Mode A (developers):** Python 3.11+ from [python.org](https://www.python.org/downloads/) with **Add to PATH** checked  
- **Mode B (end users):** only `Bahram.exe` and/or `bahramsim.exe` — **no Python required**

### Step 1 — Get the code

```bat
git clone https://github.com/USER/bahram.git
cd bahram
```

Or download `Full-Bz.zip` from Releases, extract it, then:

```bat
cd bahram
```

### Step 2 — Install dependencies (source mode only)

```bat
python -m pip install -r requirements.txt
```

Skip this if you only use prebuilt executables.

### Step 3 — First run

**From source:**

```bat
python Bahram.py examples\hello.bz
python Bahram.py examples\hello.bz run
python Bahram.py run examples\factorial.bz
```

**Without Python (after you have the exe):**

```bat
Bahram.exe hello.bz
Bahram.exe hello.bz run
Bahram.exe run factorial.bz
```

Expected factorial output: `720`

### Step 4 — Build Windows executables (once, for distribution)

```bat
build_windows.bat
```

→ `dist\Bahram.exe`

```bat
build_bahramsim.bat
```

→ `dist\bahramsim.exe`

End users then only need:

```bat
Bahram.exe myprog.bz run
bahramsim.exe run examples\adcs_3axis.bz
```

---

## 3. Compiler and day-to-day coding

### Runner commands

| Command | Action |
|---------|--------|
| `Bahram file.bz` | Run |
| `Bahram file.bz run` | Run |
| `Bahram run file.bz` | Run |
| `Bahram check file.bz` | Static analysis |
| `Bahram help` | Help |

From source, use the same arguments with `python Bahram.py ...`.

### Generate Ada/SPARK

```bat
python bahramc.py examples\fadec.bz -c -o fadec.adb
```

Output includes `SPARK_Mode` and contracts. Formal proof still requires GNAT/GNATprove separately.

### Examples

| File | Topic |
|------|--------|
| `examples\hello.bz` | Getting started |
| `examples\factorial.bz` | Contracts + recursion |
| `examples\flybywire.bz` | Flight control |
| `examples\fadec.bz` | Engine fuel limiting |
| `examples\sensor_reader.bz` | Sensor + task |
| `examples\adcs_3axis.bz` | ADCS |
| `examples\control_flow.bz` | else / for / enum / match |

### VS Code

Copy the `vscode-bahram` folder to:

```text
%USERPROFILE%\.vscode\extensions\bahram-2.0.0
```

Restart VS Code. `.bz` files get syntax highlighting and snippets.

### Web playground (optional)

```bat
python -m pip install fastapi uvicorn
python -m uvicorn api.main:app --host 127.0.0.1 --port 8000
```

Then open `playground\index.html` in a browser (API must be running).

---

## 4. Console simulator (`bahramsim`)

Interprets Bahram code, drives virtual sensors/actuators, advances time with **RK4**, and prints results in the terminal.

### Windows

```bat
python bahram_sim.py run examples\adcs_3axis.bz --duration 2 --step 0.05 --plot torque
```

Or:

```bat
bahramsim.exe run examples\adcs_3axis.bz --duration 2 --step 0.05 --plot torque
```

### Linux

```bash
./bahramsim run examples/adcs_3axis.bz --duration 2 --step 0.05 --plot torque
```

### Options

| Option | Meaning |
|--------|---------|
| `--duration 5` | Simulation length (seconds) |
| `--step 0.01` | Time step |
| `--real-time` | Sync to wall clock |
| `--log out.csv` | File log |
| `--plot torque` | ASCII plot |
| `--target "STM32F4"` | Banner label |

### Debug REPL

```bat
python bahram_sim.py debug examples\hello.bz
```

Commands: `break` · `print` · `watch` · `run` · `stack` · `quit`

### What is simulated?

- **Hardware:** GPIO, ADC, UART, Timer, Watchdog  
- **Sensors:** Gyro, Accel, Mag, Baro, Temp, GPS  
- **Physics:** rigid-body angular dynamics + **RK4** + approximate LEO orbit  

---

## 5. Suggested learning path (Windows)

1. `Bahram.exe run examples\hello.bz`  
2. Edit `examples\factorial.bz` and run again  
3. `Bahram check examples\fadec.bz`  
4. `bahramsim run examples\adcs_3axis.bz --duration 1 --plot torque`  
5. Write your own `myctrl.bz` → `Bahram myctrl.bz run`  

---

## 6. Language symbols (cheat sheet)

| Symbol | Role |
|--------|------|
| `@` `@!` | Variable / constant |
| `#` | Function |
| `?` `: else` | If / else |
| `~` | While / for |
| `!` | Return |
| `^` | Task |
| `$` | Call |
| `>` `<` | Task messaging |
| `..` | End block |
| `pre:` `post:` | SPARK contracts |

More detail: `docs/LANGUAGE_SPEC.md` · `docs/TUTORIAL.md` · `docs/WINDOWS.md` · `docs/END_USER.md`

---

## 7. Windows troubleshooting

| Issue | What to try |
|-------|-------------|
| `python` not found | Install with PATH, or use only `.exe` builds |
| File not found | Run from inside the `bahram` folder |
| Garbled Unicode | In CMD: `chcp 65001` |
| Hide Python entirely | Build once; ship only `dist\*.exe` |

---

## 8. License

Educational / research use with attribution is welcome.

---

<p align="center"><b>Bahram</b> — from GitHub clone to run &amp; simulate on Windows</p>
