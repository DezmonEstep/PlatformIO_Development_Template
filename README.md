# Arduino & ESP32 Embedded Development Environment

This repository contains a professional, automated embedded engineering environment optimized for **Ubuntu Linux**. It leverages **PlatformIO Core** for declarative dependency tracking, implements enterprise **Git** hygiene practices, and uses **GitHub Actions** for an automated CI/CD compilation and linting pipeline.

---

## 🛠️ System Prerequisites (Ubuntu Setup)

To avoid broken Python dependencies or permissions issues, follow these platform-specific setup steps.

### 1. Install Python 3, pip, and pipx

Do not use `sudo apt install platformio`, as the Debian upstream package is heavily outdated. Instead, isolate the core package globally using `pipx`:

```bash
# Update local package manager index
sudo apt update

# Install pipx
sudo apt install python3-pip pipx
pipx ensurepath
```

> **Note:** Restart your terminal window after running `ensurepath` to refresh your environment variables.

### 2. Install PlatformIO Core

```bash
pipx install platformio
```

### 3. Configure USB Port Permissions

By default, Ubuntu restricts raw access to serial interfaces (`/dev/ttyUSB*` and `/dev/ttyACM*`), causing upload failures. Grant your local user account permission to use these ports without requiring `sudo`:

```bash
# Add current user to the dialout group
sudo usermod -a -G dialout $USER

# Apply group changes dynamically without a reboot
newgrp dialout
```

---

## 📁 Repository Structure & Git Hygiene

To ensure clean commits, all hardware compilation binaries, cached indexers, and local IDE metadata are ignored via `.gitignore`.

### Layout Architecture

```text
├── .github/
│   └── workflows/
│       └── ci.yml          # Automated CI/CD Testing Pipeline
├── src/
│   └── main.cpp            # Application Entry Point
├── lib/
│   └── MyLocalLib/         # Project-specific custom classes and drivers
├── .gitignore              # Production-grade compilation exclusion file
└── platformio.ini          # Project Manifest (Boards, Libraries, Flags)
```

### Active `.gitignore` Configuration

```gitignore
# PlatformIO local build and toolchain caching directories
.pio/
.pioenvs/
.piolibdeps/
.clang_complete
.gcc-flags.json
.vscode/

# Python compiled environments
__pycache__/
*.pyc

# Operating system artifacts
.DS_Store
Thumbs.db
```

---

## ⚙️ Project Configuration (`platformio.ini`)

This single file declares our target boards and locks third-party library dependencies to specific versions. PlatformIO resolves these dependencies automatically during local builds and CI execution.

```ini
[env]
framework = arduino
monitor_speed = 115200

# Environment: Arduino Uno Target
[env:uno]
platform = atmelavr
board = uno
lib_deps =
    bblanchon/ArduinoJson @ ^6.21.0

# Environment: Heltec WiFi LoRa 32 V3 Target (ESP32-S3)
[env:heltec_wifi_lora_32_V3]
platform = espressif32
board = heltec_wifi_lora_32_V3
lib_deps =
    bblanchon/ArduinoJson @ ^6.21.0
    heltecautomation/Heltec ESP32 Dev-Boards @ ^2.0.2

# Environment: Heltec WiFi LoRa 32 V2 Target (Standard ESP32)
[env:heltec_wifi_lora_32_V2]
platform = espressif32
board = heltec_wifi_lora_32_V2
lib_deps =
    bblanchon/ArduinoJson @ ^6.21.0
    heltecautomation/Heltec ESP32 Series Dev-boards @ ^1.1.2
```

---

## 🧑‍💻 Code Separation Best Practices

### Application Logic Components (`src/`)

Put custom structural classes that are tied to your primary loop or program execution flow directly into the `src/` directory alongside `main.cpp`.

### Isolated Driver Modules (`lib/`)

Put reusable, hardware-specific wrappers (for example, custom sensor controllers or motor drivers) in their own subdirectories inside `lib/`. They will be modularly compiled automatically.

### C++ Declarations

Unlike the standard Arduino IDE, PlatformIO handles pure C++. Include:

```cpp
#include <Arduino.h>
```

at the top of your files, and forward-declare methods before calling them.

---

## 🚀 CI/CD Automation Pipeline

Place the following file at `.github/workflows/ci.yml`.

On every push or pull request, GitHub Actions provisions an isolated Ubuntu runner, installs dependencies, performs static analysis, and verifies compilation.

```yaml
name: Arduino CI Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Repository
        uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: "3.10"

      - name: Cache PlatformIO Dependencies
        uses: actions/cache@v4
        with:
          path: |
            ~/.platformio
            .pio
          key: ${{ runner.os }}-platformio-${{ hashFiles('**/platformio.ini') }}

      - name: Install PlatformIO Core
        run: |
          python -m pip install --upgrade pip
          pip install -U platformio

      - name: Run PlatformIO Static Code Linting
        run: pio check --fail-on-defect=high

      - name: Compile Code (Verify Build Multi-Target)
        run: pio run
```

---

## ⌨️ Execution and Deployment Commands

Run these commands from the project root directory.

### Compile Code for All Environments

```bash
pio run
```

### Compile and Upload to a Specific Board (Heltec V3)

```bash
pio run -e heltec_wifi_lora_32_V3 --target upload
```

### Execute Local Static Code Quality Analysis

```bash
pio check
```

### Launch Serial Port Monitor

```bash
pio device monitor
```
