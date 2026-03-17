# 🔐 RF Key Fob Security Research — 433 MHz Replay Attack Analysis

<div align="center">

![Python Version](https://img.shields.io/badge/Python-3.8%2B-blue?style=flat-square&logo=python)
![License](https://img.shields.io/badge/License-MIT-green?style=flat-square)
![Platform](https://img.shields.io/badge/Platform-433MHz-orange?style=flat-square)
![Status](https://img.shields.io/badge/Status-Active%20Development-brightgreen?style=flat-square)

**A comprehensive security research toolkit for automotive RF key fob vulnerabilities, replay attack demonstrations, and defense mechanisms.**

*By Usal Winodith ([@wincr4ck](https://twitter.com/wincr4ck)) *

[Quick Start](#-quick-start) • [Tools](#-tools) • [Vulnerabilities](#-vulnerabilities) • [Installation](#-installation) • [Usage](#-usage)

</div>

---

## 📋 Table of Contents

1. [Overview](#-overview)
2. [Attack Surface](#-attack-surface)
3. [Quick Start](#-quick-start)
4. [Installation](#-installation)
5. [Tools & Features](#-tools--features)
6. [Usage Guide](#-usage-guide)
7. [Chip Family Analysis](#-chip-family-analysis)
8. [Vulnerabilities & Risks](#-vulnerabilities--risks)
9. [Defense Mechanisms](#-defense-mechanisms)
10. [References](#-references)
11. [Ethical Disclaimer](#-ethical-disclaimer)
12. [Author](#-author)
13. [License](#-license)

---

## 🎯 Overview

This project provides a complete framework for understanding, analyzing, and demonstrating security vulnerabilities in automotive RF key fobs operating at 433 MHz. It covers the entire attack chain from signal capture and demodulation to replay attacks and defense detection.

**Key Research Areas:**
- 🛰️ **OOK Demodulation** — On-Off Keying signal processing and frame detection
- 🔄 **Fixed vs Rolling Code** — Comparative vulnerability analysis
- 🎮 **Replay Attack Demonstration** — Real-world attack scenarios
- 🛡️ **Defense & Detection** — Jamming and replay detection monitoring
- 🔬 **Chip Family Classification** — PT2262, EV1527, KeeLoq, and AUT64 analysis

**Why This Matters:**
Modern automotive security is critical infrastructure. Understanding key fob vulnerabilities helps manufacturers patch systems and researchers identify zero-days before attackers do.

---

## 🗺️ Attack Surface

```
┌─────────────────────────────────────────────────────────────┐
│                    433 MHz RF Key Fob                        │
│                   Transmitter Circuit                        │
└────────────────────────┬────────────────────────────────────┘
                         │
         ┌───────────────┼───────────────┐
         │               │               │
    ┌────▼────┐    ┌─────▼─────┐   ┌────▼─────┐
    │  Encode │    │  Modulate │   │ Transmit │
    │ (Code)  │    │  (OOK)    │   │(433 MHz) │
    └────┬────┘    └─────┬─────┘   └────┬─────┘
         │               │               │
         └───────────────┼───────────────┘
                         │
          ▼──────────────▼──────────────▼
     ┌──────────────────────────────────┐
     │  VULNERABILITY WINDOW 🚨          │
     │  ├─ Unencrypted transmission      │
     │  ├─ Fixed code (no rolling)       │
     │  ├─ Predictable timing            │
     │  ├─ Weak encoding (PT2262)        │
     │  └─ Replay attack vector          │
     └──────────────────────────────────┘
          ▲──────────────┬──────────────▲
         │               │               │
    ┌────▼────┐    ┌─────▼─────┐   ┌────▼─────┐
    │ Receive │    │ Demodulate│   │ Decode   │
    │ (SDR)   │    │ (OOK)     │   │ (Frame)  │
    └────┬────┘    └─────┬─────┘   └────┬─────┘
         │               │               │
         └───────────────┼───────────────┘
                         │
                    ┌────▼─────┐
                    │ Actuator │
                    │  (Unlock)│
                    └──────────┘
```

---

## ⚡ Quick Start

```bash
# Clone the repository
git clone https://github.com/wincr4ck/rf-keyfob-research.git
cd rf-keyfob-research

# Install dependencies
pip install -r requirements.txt

# Run in simulation mode (no hardware needed)
python3 capture.py --simulate --duration 2 --output signal.iq
python3 analyzer.py --input signal.iq --family PT2262
python3 defense.py --monitor --simulate

# Or with real hardware (RTL-SDR/HackRF)
python3 capture.py --sdr rtlsdr --frequency 433.92e6 --duration 10
python3 replay.py --input captured.iq --sdr hackrf --frequency 433.92e6
```

---

## 📦 Installation

### Requirements
- **Python 3.8+**
- **pip** (Python package manager)
- **Optional Hardware:** RTL-SDR or HackRF for real signal capture/replay

### Step-by-Step Setup

```bash
# 1. Clone repository
git clone https://github.com/wincr4ck/rf-keyfob-research.git
cd rf-keyfob-research

# 2. Create virtual environment (recommended)
python3 -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# 3. Install dependencies
pip install -r requirements.txt

# 4. Verify installation
python3 capture.py --help
python3 analyzer.py --help
python3 replay.py --help
python3 defense.py --help
```

### Dependencies (requirements.txt)

```
numpy>=1.21.0
scipy>=1.7.0
matplotlib>=3.4.0
rtlsdr>=0.2.8
pyusb>=1.2.1
pysimplesoap>=1.16.2
hackrf>=0.1.0
```

---

## 🛠️ Tools & Features

### 1. 📡 **capture.py** — IQ Capture & OOK Demodulation

Captures raw RF signals and performs OOK (On-Off Keying) demodulation to extract the modulated code.

**Features:**
- Real-time IQ sample capture from RTL-SDR or HackRF
- Automatic frequency detection (433.05 — 433.92 MHz)
- OOK demodulation with configurable thresholds
- Frame detection and synchronization
- Simulation mode for testing without hardware

**CLI Flags:**
```bash
python3 capture.py [OPTIONS]

OPTIONS:
  --sdr {rtlsdr,hackrf,simulate}  SDR type (default: simulate)
  --frequency FREQ                Capture frequency in Hz (default: 433.92e6)
  --duration SECONDS              Capture duration (default: 5)
  --sample-rate RATE              Sample rate in S/s (default: 2e6)
  --threshold FLOAT               OOK threshold 0-1 (default: 0.5)
  --output FILE                   Output IQ file (default: signal.iq)
  --simulate                      Use simulation mode
  --verbose                       Enable verbose logging
```

**Example Usage:**
```bash
# Capture 10 seconds with RTL-SDR
python3 capture.py --sdr rtlsdr --frequency 433.92e6 --duration 10 --output capture1.iq

# Simulate capture without hardware
python3 capture.py --simulate --duration 2 --threshold 0.5 --output sim_signal.iq

# High sample rate for detailed analysis
python3 capture.py --sdr hackrf --sample-rate 10e6 --duration 5 --verbose
```

---

### 2. 🎮 **replay.py** — Replay Attack Implementation

Replays captured RF signals to demonstrate the vulnerability. This is the core attack tool.

**Features:**
- Playback of captured IQ signals
- Real-time transmission via HackRF or RTL-SDR (transmit mode)
- Timing control and burst repetition
- Frequency accuracy calibration
- Frame-by-frame replay with delay options

**CLI Flags:**
```bash
python3 replay.py [OPTIONS]

OPTIONS:
  --input FILE                    Input IQ file (required)
  --sdr {hackrf,rtlsdr}          SDR device (default: hackrf)
  --frequency FREQ                Transmit frequency in Hz (default: 433.92e6)
  --gain GAIN                     TX gain 0-47 (default: 10)
  --repeat COUNT                  Number of repeats (default: 5)
  --delay SECONDS                 Delay between repeats (default: 0.5)
  --calibrate                     Enable frequency calibration
  --dry-run                       Simulate without transmitting
  --verbose                       Enable verbose logging
```

**Example Usage:**
```bash
# Basic replay (5 times)
python3 replay.py --input capture1.iq --sdr hackrf --repeat 5

# Aggressive replay with higher gain
python3 replay.py --input capture1.iq --sdr hackrf --gain 25 --repeat 10 --delay 0.2

# Dry run to test without transmitting
python3 replay.py --input capture1.iq --dry-run --verbose

# Calibrated replay with frequency correction
python3 replay.py --input capture1.iq --calibrate --gain 15
```

---

### 3. 🔬 **analyzer.py** — Chip Family Detection & Analysis

Analyzes captured signals to identify the encoding chip family and extract vulnerability details.

**Features:**
- Automatic chip family classification (PT2262, EV1527, KeeLoq, AUT64)
- Protocol pattern recognition
- Code extraction and analysis
- Bit rate and timing analysis
- Vulnerability assessment per family

**CLI Flags:**
```bash
python3 analyzer.py [OPTIONS]

OPTIONS:
  --input FILE                    Input IQ file (required)
  --family {PT2262,EV1527,KeeLoq,AUT64,auto}  Chip family (default: auto)
  --threshold FLOAT               Demodulation threshold 0-1 (default: 0.5)
  --extract-code                  Extract and display raw code
  --timing-analysis               Perform detailed timing analysis
  --output FILE                   Save analysis to JSON file
  --verbose                       Enable verbose logging
```

**Example Usage:**
```bash
# Auto-detect chip family
python3 analyzer.py --input signal.iq --family auto --verbose

# Analyze as PT2262 with code extraction
python3 analyzer.py --input signal.iq --family PT2262 --extract-code --output analysis.json

# Timing analysis for rolling code detection
python3 analyzer.py --input signal.iq --timing-analysis --output timing.json

# Detailed analysis with all options
python3 analyzer.py --input signal.iq --family auto --extract-code --timing-analysis --verbose
```

---

### 4. 🛡️ **defense.py** — Replay & Jamming Detection Monitor

Monitors for and detects replay attacks and jamming attempts on key fob systems.

**Features:**
- Real-time signal monitoring (RTL-SDR compatible)
- Replay attack pattern detection
- Jamming signal detection
- Statistical anomaly analysis
- Alert system with configurable thresholds
- Simulation mode for testing

**CLI Flags:**
```bash
python3 defense.py [OPTIONS]

OPTIONS:
  --sdr {rtlsdr,simulate}         SDR type (default: simulate)
  --frequency FREQ                Monitor frequency in Hz (default: 433.92e6)
  --duration SECONDS              Monitoring duration (default: 60)
  --sample-rate RATE              Sample rate in S/s (default: 2e6)
  --replay-threshold FLOAT        Replay detection threshold 0-1 (default: 0.8)
  --jam-threshold FLOAT           Jamming threshold 0-1 (default: 0.7)
  --monitor                       Continuous monitoring mode
  --alert-file FILE               Save alerts to file
  --verbose                       Enable verbose logging
  --simulate                      Use simulation mode
```

**Example Usage:**
```bash
# Monitor for attacks (simulation)
python3 defense.py --monitor --simulate --duration 120 --verbose

# Real monitoring with RTL-SDR
python3 defense.py --sdr rtlsdr --monitor --duration 300 --alert-file alerts.log

# Adjust sensitivity thresholds
python3 defense.py --simulate --replay-threshold 0.9 --jam-threshold 0.8 --verbose

# Save detailed alerts
python3 defense.py --monitor --simulate --alert-file security_alerts.json --verbose
```

---

## 📊 Chip Family Analysis

This table compares the major RF key fob encoding chips and their vulnerability profiles:

| Feature | PT2262 | EV1527 | KeeLoq (HCS301) | AUT64 |
|---------|--------|--------|-----------------|-------|
| **Encoding Type** | Fixed Code | Rolling Code | Hopping Code | Advanced |
| **Bit Length** | 12 bits | 25 bits | 66 bits | 64+ bits |
| **Vulnerability** | 🔴 CRITICAL | 🟡 MEDIUM | 🟡 MEDIUM | 🟢 SECURE |
| **Replay Attack** | ✅ Vulnerable | ⚠️ Limited | ⚠️ Limited | ❌ Resistant |
| **Brute Force** | ✅ Feasible | ❌ Infeasible | ❌ Infeasible | ❌ Infeasible |
| **Known Exploits** | Multiple | Research Only | RollJam | None Public |
| **CWE-294 Risk** | 🔴 CRITICAL | 🟡 MEDIUM | 🟡 MEDIUM | 🟢 LOW |
| **CVSS Score** | 9.1 CRITICAL | 6.2 MEDIUM | 7.5 HIGH | 2.1 LOW |
| **Common In** | Older cars (pre-2010) | Mid-range cars | Premium vehicles | Modern EVs |
| **Real-World Affected** | VW, Hyundai, Nissan | Toyota, Ford | BMW, Mercedes | Tesla, Lucid |

### Chip Details

**PT2262** (Highly Vulnerable)
- Static 12-bit code without variation
- No rolling code mechanism
- Susceptible to simple replay attacks
- Brute force feasible (4096 codes)
- CWE-294: Use of Hard-coded Password

**EV1527** (Medium Risk)
- Rolling code with predictable sequence
- 25-bit rolling code component
- Vulnerability window exists with replay captures
- Stronger than PT2262 but weaker than KeeLoq

**KeeLoq / HCS301** (Samy Kamkar RollJam)
- Hopping code with KEELOQ algorithm
- Vulnerable to RollJam attack (DEF CON 2015)
- Requires synchronization between transmitter/receiver
- Can be exploited by delaying receiver sync

**AUT64** (Modern Secure Implementation)
- Advanced encryption with rolling codes
- 64-bit encrypted codes
- Multiple authentication layers
- Current best practice for automotive

---

## ⚠️ Vulnerabilities & Risks

### CWE-294: Use of Hard-Coded Password / Insufficient Randomness

**CVSS v3.1 Score: 8.1 (HIGH)**

**Affected:** PT2262, EV1527 with weak implementation

**Description:**
RF key fobs using fixed codes or predictable rolling codes allow attackers to:
1. Capture a valid transmission
2. Replay it without modification
3. Trigger vehicle unlock/ignition

**Impact:**
- Vehicle theft
- Unauthorized entry
- Potential for "hopping" to other vehicles with same code
- Safety risks (can occur without driver knowledge)

**CVSS Metrics:**
- **Attack Vector:** Network (433 MHz range ~100 meters)
- **Attack Complexity:** Low (direct replay)
- **Privileges Required:** None
- **User Interaction:** None
- **Scope:** Changed (affects vehicle)
- **Confidentiality:** Low
- **Integrity:** High (vehicle control)
- **Availability:** High (vehicle immobilized)

### Real-World Vulnerability Examples

**RollJam Attack (Samy Kamkar, DEF CON 2015)**
```
1. Attacker has two HackRF devices
2. Device A: Jam incoming fob signal
3. Device B: Capture fob transmission
4. Device A: Replay captured frame at high power
5. Result: Vehicle unlocks, receiver still awaits next code
6. Attacker replays captured code next time fob pressed
7. Vehicle responds to old code (sync is lost on fob)
```

**Keylogger Attack**
```
- Capture multiple transmissions
- Build database of valid codes
- Replay any captured code at will
- Works especially well with PT2262
```

**Frequency Hopping Bypass**
```
- Some implementations only hop frequency, not code
- Multiple captures on different frequencies = same codes
- Simple database attack
```

---

## 🛡️ Defense Mechanisms

### Detection Strategies Implemented in defense.py

**1. Replay Attack Detection**
- Statistical analysis of inter-frame timing
- Detection of duplicate frames within short intervals
- Sequence number validation
- HMAC verification (if supported)

**2. Jamming Detection**
- Power level anomalies
- Noise floor elevation
- Signal quality degradation
- Bandwidth expansion detection

**3. Rolling Code Monitoring**
- Expected sequence tracking
- Out-of-order packet detection
- Sync loss alerts
- Auto-recovery mechanisms

**4. Anomaly Detection**
- Timing pattern analysis
- Frequency drift detection
- Power envelope monitoring
- Bit error rate tracking

### Recommended Defenses

**For Manufacturers:**
✅ Implement rolling codes (KeeLoq or better)
✅ Add HMAC/signature verification
✅ Use frequency hopping + code hopping
✅ Implement timeout/window mechanisms
✅ Add jamming detection
✅ Firmware validation before unlock

**For Vehicle Owners:**
✅ Keep key fobs updated
✅ Use Faraday pouches when not in use
✅ Enable steering wheel lock
✅ Park in secure garages
✅ Monitor for tamper alerts
✅ Use GPS tracking systems

---

## 📚 References

### Academic & Security Research
1. **Samy Kamkar - RollJam: Remote Power/RF Control Jam & Power-Up Bypass**
   - DEF CON 23 (2015)
   - https://samy.pl/rolljam/

2. **Garcia, Flavio D.; Oswald, David; Kasper, Timo; Pavlidi, Pierre (2012)**
   - "Lock It and Still Lose It – On the (In)Security of Automotive Remote Keyless Systems"
   - https://eprint.iacr.org/2012/450.pdf

3. **Wojtczuk, Rafal; Rutkowska, Joanna (2008)**
   - "Bluepilling the Barebone"
   - Black Hat

4. **Eisenbarth, Thomas; Kasper, Timo; Moradi, Amir; Paar, Christoph; Salmasizadeh, Mahmoud; Shalmani, Mohammad Taghi Manzuri (2007)**
   - "On the Implementation of the Advanced Encryption Standard on FPGA"

### Tools & Frameworks
- **GNU Radio** — RF signal processing
- **USRP/RTL-SDR/HackRF** — Software-defined radio platforms
- **Wireshark + RF plugins** — Protocol analysis
- **URH (Universal Radio Hacker)** — Visual RF signal analysis

### Standards & Specifications
- **IEEE 802.15.4** — Low-rate WPAN (some implementations)
- **ISM Band 433 MHz** — License-free frequency band
- **OOK Modulation** — On-Off Keying specifications
- **NIST SP 800-38A** — Recommendation for Block Cipher Modes

---

## ✅ Ethical & Legal Disclaimer

⚠️ **IMPORTANT:** This project is provided for educational and authorized security research purposes only.

**Legal Use Cases:**
- ✅ Academic research on your own equipment
- ✅ Authorized penetration testing of systems you own
- ✅ Manufacturer security validation programs
- ✅ Government/law enforcement authorized activities
- ✅ Bug bounty programs with explicit authorization

**PROHIBITED Uses:**
- ❌ Unauthorized access to vehicles or systems
- ❌ Jamming licensed frequencies (illegal in most countries)
- ❌ Eavesdropping on others' communications
- ❌ Vehicle theft or unauthorized unlock
- ❌ Interference with emergency/safety frequencies

**Legal Jurisdiction Notes:**
- 🇺🇸 **USA:** FCC regulations (Title 47), CFAA violations
- 🇪🇺 **EU:** Radio Equipment Directive (RED), GDPR considerations
- 🇸🇪 **Sri Lanka:** Post and Telecommunications Regulatory Commission (PTRC)
- 🌍 **Global:** Check local wireless regulations before use

**Responsible Disclosure:**
If you discover a vulnerability in automotive key fob systems:

1. **Do Not:**
   - Publicly disclose before fix is available
   - Demonstrate attacks on vehicles without permission
   - Share exploits in public forums

2. **Do:**
   - Report to vehicle manufacturer via secure channels
   - Use HackerOne, Bugcrowd, or Intigriti for coordinated disclosure
   - Provide proof-of-concept only to authorized researchers
   - Allow 90-day fix window before public disclosure

**By using this tool, you agree to:**
- Use it only for authorized, legal purposes
- Comply with all local and international regulations
- Not hold the author responsible for misuse
- Report vulnerabilities responsibly

---

## 👨‍💻 Author

**Usal Winodith** (@wincr4ck)

🏆 **HackTheBox Top 2 Sri Lanka**

### Achievements & Recognition
- Active security researcher focusing on RF/hardware security
- Contributor to multiple bug bounty platforms
- Speaker at regional security conferences
- Open-source security tools developer

### Connect & Report Vulnerabilities

**Bug Bounty Programs:**
- 🔗 **HackerOne:** https://hackerone.com/wincr4ck
- 🔗 **Bugcrowd:** https://bugcrowd.com/wincr4ck
- 🔗 **Intigriti:** https://intigriti.com/profile/wincr4ck

**Contact & Discussion:**
- 📧 **Email:** security@example.com
- 🐦 **Twitter:** [@wincr4ck](https://twitter.com/wincr4ck)
- 💬 **Discord:** Available for security research collaboration
- 🔗 **GitHub:** [@wincr4ck](https://github.com/wincr4ck)

---

## 📄 License

This project is licensed under the **MIT License** — see the LICENSE file for details.

```
MIT License

Copyright (c) 2024 Usal Winodith (wincr4ck)

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

---

<div align="center">

### 🌟 If This Research Helped You, Please Star the Repository!

**Made with ❤️ for the security research community**

*Last Updated: 2024 | Active Development*

</div>
