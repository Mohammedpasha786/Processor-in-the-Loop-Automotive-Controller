# Processor-in-the-Loop Automotive Controller

## Overview

This project implements and verifies three automotive embedded controllers using a complete **Model-Based Design (MBD)** workflow aligned with the **ISO 26262** functional safety standard. Controllers are validated from requirements through PIL testing on a virtual **Arm Cortex-M7** processor using Arm Fast Models.

### Controllers Implemented

| Controller | Standard | Safety Goal |
|---|---|---|
| **Anti-Lock Braking System (ABS)** | ISO 26262 ASIL-D | Prevent wheel lockup under all conditions |
| **Tire Pressure Monitoring System (TPMS)** | ISO 26262 ASIL-B | Alert driver within 10s of pressure deviation |
| **Automatic Climate Control** | ISO 26262 ASIL-A | Maintain cabin temp within ±1°C of setpoint |

---

## V-Model Workflow

```
Requirements         ←→     Verification Report
    │                               │
System Design        ←→     PIL Tests (Cortex-M7)
    │                               │
Controller Design    ←→     MIL Tests (Simulink)
    │                               │
Code Generation      ←→     Static Analysis (Polyspace)
```

---

## Repository Structure

```
pil-automotive-project/
├── .github/
│   └── workflows/
│       ├── mil-tests.yml           # Model-in-the-Loop CI
│       ├── pil-tests.yml           # Processor-in-the-Loop CI
│       ├── coverage.yml            # Coverage merge & reporting
│       ├── polyspace.yml           # MISRA C static analysis
│       └── report-generation.yml   # Verification report export
│
├── src/
│   ├── abs_controller/
│   │   ├── ABS_Controller.slx      # Simulink model
│   │   ├── ABS_Controller.m        # Model parameters
│   │   └── codegen/                # Auto-generated C code (Embedded Coder)
│   ├── tpms_controller/
│   │   ├── TPMS_Controller.slx
│   │   ├── TPMS_Controller.m
│   │   └── codegen/
│   └── climate_controller/
│       ├── Climate_Controller.slx
│       ├── Climate_Controller.m
│       └── codegen/
│
├── tests/
│   ├── mil_tests/
│   │   ├── ABS_MIL_Tests.mldatx    # Simulink Test scenarios
│   │   ├── TPMS_MIL_Tests.mldatx
│   │   └── Climate_MIL_Tests.mldatx
│   └── pil_tests/
│       ├── ABS_PIL_Tests.mldatx
│       ├── TPMS_PIL_Tests.mldatx
│       └── Climate_PIL_Tests.mldatx
│
├── scripts/
│   ├── run_mil_tests.m             # MATLAB test runner (MIL)
│   ├── run_pil_tests.m             # MATLAB test runner (PIL)
│   ├── generate_code.m             # Code generation script
│   ├── merge_coverage.m            # Coverage merge script
│   └── generate_report.m           # Report generation script
│
├── config/
│   ├── cortex_m7_config.m          # Cortex-M7 target settings
│   ├── code_gen_config.m           # Embedded Coder config
│   └── polyspace_config.psprj      # Polyspace project config
│
├── reports/                        # Generated verification artifacts
│   └── .gitkeep
│
├── docs/
│   ├── requirements/
│   │   ├── ABS_Requirements.md
│   │   ├── TPMS_Requirements.md
│   │   └── Climate_Requirements.md
│   ├── architecture/
│   │   └── system_architecture.md
│   └── PIL_Setup_Guide.md
│
├── .gitignore
├── CHANGELOG.md
└── README.md
```

---

## Quick Start

### Prerequisites

| Tool | Version | Purpose |
|---|---|---|
| MATLAB | R2024a+ | Simulation environment |
| Simulink | R2024a+ | Model-based design |
| Embedded Coder | R2024a+ | C code generation |
| Simulink Test | R2024a+ | MIL/PIL test management |
| Simulink Coverage | R2024a+ | Code/model coverage |
| Requirements Toolbox | R2024a+ | Formal requirements |
| Arm Fast Models | 11.x+ | Virtual Cortex-M7 target |
| Polyspace Bug Finder | R2024a+ | Static analysis (optional) |

### 1. Clone & Setup

```bash
git clone https://github.com/your-org/pil-automotive.git
cd pil-automotive
```

```matlab
% In MATLAB
cd('pil-automotive-project')
addpath(genpath('.'))
run('scripts/setup_environment.m')
```

### 2. Run MIL Tests

```matlab
run('scripts/run_mil_tests.m')
```

### 3. Generate Code

```matlab
run('scripts/generate_code.m')
```

### 4. Run PIL Tests on Cortex-M7

```matlab
run('scripts/run_pil_tests.m')
```

### 5. Generate Verification Report

```matlab
run('scripts/generate_report.m')
```

---

## Requirements Summary

### ABS Controller
- **REQ-ABS-001**: Shall prevent wheel lockup during braking under all road conditions and vehicle speeds
- **REQ-ABS-002**: Shall maintain traction control response within 10 ms of slip detection
- **REQ-ABS-003**: Shall execute control loop within allocated 1 ms sample period on target hardware

### TPMS Controller
- **REQ-TPMS-001**: Shall detect tire pressure deviations > 10% from recommended levels
- **REQ-TPMS-002**: Shall alert driver within 10 seconds of pressure deviation occurrence
- **REQ-TPMS-003**: Shall operate across temperature range -40°C to +85°C

### Climate Controller
- **REQ-CLIM-001**: Shall maintain cabin temperature within ±1°C of driver setpoint
- **REQ-CLIM-002**: Shall achieve setpoint temperature within 5 minutes from cold start
- **REQ-CLIM-003**: Shall operate under all ambient conditions from -20°C to +45°C

---

## Coverage Targets

| Metric | Target | Tool |
|---|---|---|
| Model Decision Coverage | ≥ 90% | Simulink Coverage |
| Code Statement Coverage | ≥ 90% | Simulink Coverage |
| MC/DC Coverage | ≥ 90% | Simulink Coverage |
| Merged MIL + PIL Coverage | ≥ 95% | Simulink Coverage |

---

## PIL Execution on Cortex-M7

PIL runs C code generated by Embedded Coder on a **virtual Arm Cortex-M7** via Arm Fast Models. Simulink drives test inputs step-by-step and compares outputs against the MIL baseline.

**What PIL validates:**
- Numerical equivalence between model and generated code
- Real-time execution within the allocated sample period
- No floating-point precision issues in fixed-point conversion

See [`docs/PIL_Setup_Guide.md`](docs/PIL_Setup_Guide.md) for full Fast Model setup instructions.

---

## CI/CD Pipeline

| Workflow | Trigger | What It Does |
|---|---|---|
| `mil-tests.yml` | Push to `main`/`develop` | Runs all MIL test suites |
| `pil-tests.yml` | Push to `main`, PR | Runs PIL on virtual Cortex-M7 |
| `coverage.yml` | After tests pass | Merges MIL + PIL coverage |
| `polyspace.yml` | Weekly + PR | MISRA C static analysis |
| `report-generation.yml` | Tag `v*` | Generates & publishes verification report |

---

## Contributing

1. Branch from `develop`: `git checkout -b feature/your-feature`
2. Ensure MIL tests pass locally
3. Add/update requirements traceability in `docs/requirements/`
4. Open a PR — PIL tests run automatically in CI

---

## License

MIT License — see [LICENSE](LICENSE) for details.

---

*Industry Partner: Arm & MathWorks | Safety Standard: ISO 26262*
