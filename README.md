# Offline Broadcom BCM4360 WiFi Driver Installer (Linux Mint / Ubuntu 24.04 Kernel)

## 📌 Overview

This project provides a **fully offline installation bundle** for enabling WiFi on systems using the **Broadcom BCM4360 802.11ac wireless adapter**, particularly on:

* MacBook Pro (2013–2015 era)
* Linux Mint systems running newer Ubuntu-based kernels (e.g., 6.x series)
* Environments without immediate internet access

It solves a very specific but common problem:

> **No WiFi after installing Linux → cannot install WiFi drivers because there is no internet**

---

## 🚨 The Problem

When installing Linux Mint (or similar distributions) on hardware with Broadcom wireless chips:

* The required driver (`bcmwl-kernel-source`) is **proprietary**
* It is **not included by default**
* Installation requires:

  * DKMS
  * Kernel headers
  * Compiler toolchain (`gcc`, `make`)
  * Multiple system libraries

Without internet access, this creates a **dependency deadlock**:

* You need WiFi to install the driver
* You need the driver to get WiFi

---

## 💡 The Solution

This repository contains a **pre-downloaded set of `.deb` packages** that includes:

* Broadcom driver:

  * `bcmwl-kernel-source`
* Build system:

  * `dkms`
  * `build-essential` (and dependencies)
* Kernel compatibility:

  * `linux-headers-<matching version>`
* Required libraries:

  * `libc6-dev`, `linux-libc-dev`, etc.

All dependencies are resolved ahead of time using an Ubuntu 24.04 environment.

---

## 📦 Contents

```
wireless-driver/
├── bcmwl-kernel-source_*.deb
├── dkms_*.deb
├── gcc_*.deb
├── make_*.deb
├── linux-headers-6.14.0-37-*.deb
├── libc6-dev_*.deb
├── linux-libc-dev_*.deb
├── (additional dependency packages)
```

---

## ⚙️ Installation (Offline)

### 1. Copy to target machine

Move the `wireless-driver` folder onto the target system (via USB).

---

### 2. Install all packages

Open a terminal and run:

```bash
cd ~/wireless-driver
sudo dpkg -i *.deb
```

---

### 3. Fix any dependency issues

```bash
sudo apt --fix-broken install
```

---

### 4. Load the driver

```bash
sudo modprobe wl
```

---

### 5. Reboot

```bash
sudo reboot
```

---

## ✅ Expected Result

* WiFi interface appears
* Network manager detects available networks
* System connects normally

---

## 🧠 What I Learned

### 1. Dependency management matters more than the package itself

Installing a single driver actually required:

* Compiler toolchains
* Kernel-specific headers
* Matching system libraries

The *driver* wasn’t the hard part — the **ecosystem around it was**.

---

### 2. Kernel version compatibility is critical

* Newer kernels (6.x) introduce instability with proprietary drivers
* Matching headers exactly to `uname -r` is non-negotiable
* Even with correct installation, driver behavior can vary by kernel version

---

### 3. Offline Linux setup is a different skill tier

This process required:

* Understanding `.deb` package relationships
* Using tools like `apt-rdepends`
* Reconstructing an install environment manually

This is fundamentally different from typical package installs.

---

### 4. Environment matching is everything

Attempting to resolve Ubuntu dependencies from a Debian-based system caused failures.

The solution:

* Use a **matching Ubuntu environment (WSL 24.04)** to build the package set

---

### 5. There is almost always a faster workaround

In hindsight, the simplest solution would have been:

* USB tethering a phone → install driver in one command

But solving it offline provided deeper system-level understanding.

---

## 🚀 Use Cases

* Air-gapped systems
* Fresh Linux installs without network access
* MacBook Linux setups with Broadcom chips
* Learning Linux package/dependency internals

---

## ⚠️ Notes

* This bundle is **kernel-specific** (`6.14.0-37-generic`)
* May require regeneration for other kernel versions
* Broadcom drivers can break after kernel updates:

  ```bash
  sudo dkms autoinstall
  ```
  
---

## 🧩 Adapting This for Other Systems (WSL Method)

This bundle is **kernel-specific** (`6.14.0-37-generic`), which means it will not work universally across all systems.

However, you can **recreate a compatible offline bundle for *your own system*** using Windows Subsystem for Linux (WSL).

---

## 💻 Why Use WSL?

Windows Subsystem for Linux allows you to run a full Linux environment inside Windows.

This is useful because:

* You can match the exact Ubuntu version your target machine uses
* You can download packages *with all dependencies automatically*
* You avoid manual dependency hunting

---

## ⚙️ Step-by-Step: Rebuilding the Bundle for Your System

### 1. Install Ubuntu in WSL

Open PowerShell and run:

```bash
wsl --install -d Ubuntu-24.04
```

Launch it after installation completes.

---

### 2. Prepare a working directory

```bash
mkdir ~/wireless-driver
cd ~/wireless-driver
```

---

### 3. Update package lists

```bash
sudo apt update
```

---

### 4. Download required packages

Replace the kernel version with your own:

```bash
apt download bcmwl-kernel-source dkms build-essential linux-headers-$(uname -r)
```

---

### 5. Download all dependencies automatically

Install helper tool:

```bash
sudo apt install apt-rdepends -y
```

Then run:

```bash
apt-rdepends bcmwl-kernel-source dkms build-essential linux-headers-$(uname -r) \
| grep -v "^ " \
| sort -u \
| grep -vE "libc-dev|kldutils" \
| xargs -n 1 apt download
```

This will download:

* Compiler toolchain (`gcc`, `make`)
* Kernel headers
* All required libraries
* Every dependency needed for offline installation

---

### 6. Transfer to USB

Your files will be located at:

```
\\wsl$\Ubuntu-24.04\home\<your-username>\wireless-driver
```

Copy this folder to a USB drive.

---

### 7. Install on target Linux machine

On the offline system:

```bash
cd ~/wireless-driver
sudo dpkg -i *.deb
sudo apt --fix-broken install
```

Then load the driver:

```bash
sudo modprobe wl
```

Reboot:

```bash
sudo reboot
```

---

## 🧠 Key Takeaways

* The **kernel version determines everything**
* WSL lets you **mirror the exact environment** needed
* This approach scales to:

  * Different kernel versions
  * Different Ubuntu/Mint releases
  * Other offline driver or package installs

---

## 🚀 When to Use This Approach

Use this method when:

* You don’t have internet access on the target machine
* USB tethering isn’t available
* You want a **repeatable, portable install bundle**

---

## ⚠️ Notes

* Always verify kernel version with:

  ```bash
  uname -r
  ```
* Ensure WSL Ubuntu version matches your target system (e.g., 22.04 vs 24.04)
* Regenerate the bundle if your kernel changes

---

This turns a one-time fix into a **reusable offline installation workflow** for Linux systems.

---

## 🙌 Final Thoughts

This project turned a frustrating limitation into a deeper understanding of:

* Linux package systems
* Kernel-module relationships
* Real-world debugging workflows

It’s a reminder that sometimes the “hard way” teaches you what the “easy way” hides.

---
