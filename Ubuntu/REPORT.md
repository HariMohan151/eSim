# eSim – Ubuntu 25.04 Support Report

This report documents the issues faced while installing **eSim** and its dependency **nghdl** on Ubuntu 25.04, along with the fixes implemented.

---

## 🔧 nghdl

### 1. LLVM

* **Issue:**
  GHDL does not support the latest version of LLVM (`llvm-20.1`) available in Ubuntu 25.04.
* **Fix:**
  Modified the installer script to install `llvm-18.1`, which is supported.

---

### 2. InstallGHDL

* **Issue:**
  The installer script uses `/usr/bin/llvm-config` assuming `llvm-20.1`.
  Example command:

  ```bash
  ./configure --with-llvm-config=/usr/bin/llvm-config
  ```
* **Fix:**
  Replaced with the correct path for LLVM 18.1:

  ```bash
  ./configure --with-llvm-config=/usr/bin/llvm-config-18
  ```

---

### 3. Gtk Canberra

* **Issue:**
  `libcanberra-gtk-module` package is no longer available in Ubuntu 25.04.
* **Fix:**
  Modified installer to use only `libcanberra-gtk3-module`.

---

## 🔧 eSim

### 1. install-eSim.sh

* **Issue:**
  Ubuntu 25.04 was not supported in the main installer script.
* **Fix:**
  Added Ubuntu 25.04 support.

---

### 2. New Installer Script

* **Added:**
  `install-eSim-scripts/install-eSim-25.04.sh`

* **Differences from `install-eSim-24.04.sh`:**

  #### installKicad function:

  * Defines PPA for Ubuntu 25.04:

    ```bash
    ppa:kicad/kicad-8.0-releases
    ```

    (same as Ubuntu 24.04).
  * Installed specific version:

    ```bash
    kicad=8.0.8+dfsg-1
    ```

    instead of the latest version (latest causes dependency issues with `libgit2-1.8`).

---

3. Syntax & Fallback Handling in `install-eSim.sh` (`run_version_script`)
- Issue: Inside `run_version_script()`, the `if [[ -f "$SCRIPT" ]]; then` block was missing its closing `fi` keyword (with leftover `config_file` lines inside the block), causing `./install-eSim.sh: line 67: syntax error near unexpected token '}'`. Additionally, if `install-eSim-25.04.sh` is missing from `install-eSim-scripts/`, no fallback script was triggered.
- Fix: Closed the conditional block properly with `fi` and added a fallback to `install-eSim-24.04.sh` when `install-eSim-25.04.sh` is not found.

4. Ubuntu Version Detection Regex (`get_ubuntu_version`)
- Issue: `lsb_release -d | grep -oP '\d+\.\d+\.\d+'` expected a three-part version string (such as `24.04.1`), returning an empty string on Ubuntu `25.04`.
- Fix: Updated the regex to `grep -oP '\d+\.\d+(\.\d+)?'` so two-part version numbers (`25.04`) are detected properly.

5. APT Repository Mirrors & Python `distutils` Removal (`installDependency`)
- Issue: Ubuntu 25.04 (`plucky`) repositories moved to `old-releases.ubuntu.com` (causing `404 Not Found` during `apt-get update`), and `python3-distutils` is removed in Python 3.13 (`E: Unable to locate package python3-distutils`).
- Fix: Added a `VERSION_ID="25.04"` check to switch `archive.ubuntu.com` and `security.ubuntu.com` to `old-releases.ubuntu.com` before `apt-get update`, and replaced `python3-distutils` with `python3-setuptools`.
