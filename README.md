# Ratcade: The Autonomous Gamified Rodent Enclosure & On-Chain Eco-System

<img width="642" height="502" alt="Screenshot 2026-06-02 at 16 07 27" src="https://github.com/user-attachments/assets/ab3e7e55-64e4-4ee1-a260-1061022785e4" />


## 📌 Project Overview
Ratcade is an open-source, fully autonomous gamified rodent habitat fusing precision electromechanical engineering, real-time edge computer vision (CV), and programmatic decentralized finance (DeFi). The system provides structured physical enrichment for rodents via automated puzzle-gates, dual-station calorie dispensing, and a kinetic energy capture wheel acting as an active DC generator. 

The environment dynamically manages live-stream broadcasts based on kinetic triggers and translates physical rodent activity into real-time on-chain token functions (automated buybacks and burns) on the Solana network via custom automated market maker (AMM) hooks.

---

## 🛠 System Architecture

+-------------------------------------------------+
           |              Ratcade Core Hardware              |
           +-------------------------------------------------+
             /          |                    |             \
            /           |                    |              \
[6x Analog Joysticks] [5x Micro Servos]  [DC Generator Wheel] [PWM Fan/Button]
\           |                    |              /
\          v                    v             /
+-------------------------------------------------+
|          Main System Microcontroller            |
+-------------------------------------------------+
| (Serial telemetry via USB)
v
+-------------------------------------------------+
|             Edge Edge Compute Node              |
|       (Raspberry Pi 5 / Jetson Nano)            |
+-------------------------------------------------+
/

/ (OpenCV Scene Directing)                \ (Rust/Web3 Data Feed)
v                                           v
+-----------------------------+             +-----------------------------+
|    OBS Studio Broadcast     |             |      Solana Engine /        |
| (RTMP/WebRTC Automated Cam) |             |   Pump.fun Smart Engine     |
+-----------------------------+             +-----------------------------+


### 1. Mechanical & Hardware Layer
* **Double-Door Assembly (Gates A & B):** Two dual-servo automated safety doors activated by hair-trigger analog inputs mapping user configuration or rodent target tracking.
* **Dual Feeding Stations:** High-torque micro-servo distribution augers featuring mechanical lockout limits to precisely control food allocation.
* **Kinetic Wheel Array:** A low-friction, high-durability running wheel connected via a continuous drive belt to a small 5V DC Generator, managed by an integrated power tracking unit with supercapacitors for energy storage buffering.

### 2. Edge Computer Vision & Automated Scene Transitions
The system features a multi-camera vision mesh driven by a localized edge processing node (e.g., Raspberry Pi 5 or NVIDIA Jetson Nano) running a lightweight spatial tracking algorithm via OpenCV.

* **Tracking Engine:** A Gaussian Mixture-based Background/Foreground Segmentation model (`cv2.createBackgroundSubtractorMOG2()`) isolates the rodent's pixel coordinate space $(X, Y)$ within localized bounding boxes.
* **Scene Directing Protocol:** Instead of a static security view, a dedicated Python engine monitors movement metrics across defined Zones (Zone 1: Wheel, Zone 2: Gate A, Zone 3: Gate B, Zone 4: Feeders). 
* **OBS Studio Integration:** The director script exposes a local WebSocket server connected directly to **OBS Studio via obs-websocket-py**. When pixel deviation thresholds are sustained inside a specific zone for $>400\text{ms}$, the node triggers an instantaneous, smooth scene transition to the camera capturing that sector, ensuring live streams are always fixed on active zones.

### 3. Programmatic Tokenomics: Automated Buys & Burns
Activity in the Ratcade ecosystem is directly tokenized on-chain on Solana (`pump.fun` bonding curves migrating to Raydium pools). To achieve true, trustless execution without exposing private keys in vulnerable browser extensions, the project deploys a secure, head-mounted **Rust-based execution engine**.

[Kinetic Generation Matrix] ---> [Validated Telemetry] ---> [AWS KMS Signer] ---> [Jito Block Engine (MEV Tip)] ---> [Solana Mainnet Swap/Burn]


#### The Tech Stack:
* **Telemetry Verification:** Hardware state changes (e.g., total wheel rotations measured by voltage spikes on the current sensor, completed feeding loops) are securely signed by the local firmware and piped to an ingestion API.
* **Cryptographic Key Management:** Swaps are routed through **AWS Key Management Service (KMS)** or an isolated hardware security module (HSM) holding the deployer/marketing operational wallet keypair securely, ensuring zero raw private key strings exist on disk.
* **Execution Wrapper:** A custom Rust worker leverages the `@solana/web3.js` interface and the **Jito Block Engine** to bundle purchase transactions. This guarantees atomic transaction delivery and bypasses public mempool front-running.
* **The Deflationary Loop:**
    1.  **Buy Back:** When target milestones are completed by the rodent, a configured percentage of accumulated ecosystem fees (SOL) is dynamically extracted via a Jupiter Aggregator API swap route to buy the target token.
    2.  **Burn:** The acquired SPL-token balance is systematically directed to the native Solana system burn address (`11111111111111111111111111111111`), permanently shrinking the circulating token supply directly verifiable on the Explorer.

---

## 💻 Firmware Repository

Paste this complete, non-blocking firmware stack into your IDE. This controls the 4 door servos, 2 independent feeder servos with 2.5-hour safe lockouts, and the PWM wheel fan.

```cpp
#include <Servo.h>

// =========================================================================
// ⚙️ SERVO ANGLE CONFIGURATION TUNING POOL
// =========================================================================
const int GATE1_LEFT_CLOSED  = 0;   const int GATE1_LEFT_OPEN  = 90; // Servo 1
const int GATE1_RIGHT_CLOSED = 0;   const int GATE1_RIGHT_OPEN = 90; // Servo 2

const int GATE2_LEFT_CLOSED  = 0;   const int GATE2_LEFT_OPEN  = 90; // Servo 3
const int GATE2_RIGHT_CLOSED = 0;   const int GATE2_RIGHT_OPEN = 86; // Servo 4 

const int FEEDER_HOME_ANGLE  = 120;  const int FEEDER_DUMP_ANGLE = 0;  

// =========================================================================
// 🛠️ HARDWARE PROFILE: DOUBLE-DOOR 1 (GATE A)
// =========================================================================
Servo gate1ServoLeft; Servo gate1ServoRight;
const int gate1ServoLeftPin = 2; const int gate1ServoRightPin = 3;
const int joy1Pin = A5; const int joy2Pin = A4;
int baseline1, baseline2;
bool gate1IsOpen = false; unsigned long gate1Millis = 0;

// =========================================================================
// 🛠️ HARDWARE PROFILE: DOUBLE-DOOR 2 (GATE B)
// =========================================================================
Servo gate2ServoLeft; Servo gate2ServoRight;
const int gate2ServoLeftPin = 4; const int gate2ServoRightPin = 5;
const int joy3Pin = A3; const int joy4Pin = A2;
int baseline3, baseline4;
bool gate2IsOpen = false; unsigned long gate2Millis = 0;

// =========================================================================
// 🛠️ HARDWARE PROFILE: AUTO FEEDERS (WITH 2.5 HOUR LOCKOUT)
// =========================================================================
Servo feederServo1; const int feederServo1Pin = 6; const int joyFeeder1Pin = A1; int baselineFeeder1;
Servo feederServo2; const int feederServo2Pin = 7; const int joyFeeder2Pin = A0; int baselineFeeder2;

unsigned long lastFeeder1Activation = 0;
unsigned long lastFeeder2Activation = 0;
const unsigned long FEEDER_LOCKOUT_TIME = 9000000; // 2.5 Hours in milliseconds

// =========================================================================
// 🛠️ HARDWARE PROFILE: WHEEL FAN & BUTTON (ANYTIME RUNTIME)
// =========================================================================
const int fanPin = 11;       
const int buttonPin = 12;    

const int FAN_SLOWEST_SPEED = 45; 
bool fanIsRunning = false;
unsigned long fanStartedMillis = 0;
const unsigned long FAN_RUN_DURATION = 3000; 

// =========================================================================
// ⚙️ SHARED SYSTEM RULES
// =========================================================================
const int HAIR_TRIGGER_THRESHOLD = 25; 
const unsigned long doorOpenDuration = 16000; // 16 Seconds Hold Time

int getStableRead(int pin) {
  analogRead(pin);
  delayMicroseconds(50);
  return analogRead(pin);
}

void setup() {
  Serial.begin(115200);
  Serial.println("[RATCADE_CORE] Initializing Firmware Modules...");

  // Register Actuators
  gate1ServoLeft.attach(gate1ServoLeftPin);   gate1ServoLeft.write(GATE1_LEFT_CLOSED);
  gate1ServoRight.attach(gate1ServoRightPin); gate1ServoRight.write(GATE1_RIGHT_CLOSED);
  gate2ServoLeft.attach(gate2ServoLeftPin);   gate2ServoLeft.write(GATE2_LEFT_CLOSED);
  gate2ServoRight.attach(gate2ServoRightPin); gate2ServoRight.write(GATE2_RIGHT_CLOSED);

  feederServo1.attach(feederServo1Pin); feederServo1.write(FEEDER_HOME_ANGLE);
  feederServo2.attach(feederServo2Pin); feederServo2.write(FEEDER_HOME_ANGLE);

  pinMode(fanPin, OUTPUT);
  analogWrite(fanPin, 0); 
  pinMode(buttonPin, INPUT_PULLUP); 

  // Fast Hardware Calibration Sequence
  delay(500);
  baseline1 = getStableRead(joy1Pin);
  baseline2 = getStableRead(joy2Pin);
  baseline3 = getStableRead(joy3Pin);
  baseline4 = getStableRead(joy4Pin);
  baselineFeeder1 = getStableRead(joyFeeder1Pin);
  baselineFeeder2 = getStableRead(joyFeeder2Pin);

  Serial.println("[SYSTEM_READY] All subsystems operational. Monitoring telemetry lines.");
}

void loop() {
  unsigned long currentMillis = millis();

  // =========================================================================
  // 🔘 DOUBLE-DOOR 1 RUNTIME ENGINE
  // =========================================================================
  int dev1 = abs(getStableRead(joy1Pin) - baseline1);
  int dev2 = abs(getStableRead(joy2Pin) - baseline2);

  if (dev1 > HAIR_TRIGGER_THRESHOLD || dev2 > HAIR_TRIGGER_THRESHOLD) {
    if (!gate1IsOpen) {
      gate1ServoLeft.write(GATE1_LEFT_OPEN); 
      gate1ServoRight.write(GATE1_RIGHT_OPEN);
      gate1IsOpen = true; gate1Millis = currentMillis;
      Serial.println("TELEMETRY:{\"event\":\"gate_open\",\"id\":1}");
    }
  }
  if (gate1IsOpen && (currentMillis - gate1Millis >= doorOpenDuration)) {
    gate1ServoLeft.write(GATE1_LEFT_CLOSED); 
    gate1ServoRight.write(GATE1_RIGHT_CLOSED);
    gate1IsOpen = false;
    Serial.println("TELEMETRY:{\"event\":\"gate_close\",\"id\":1}");
  }

  // =========================================================================
  // 🔘 DOUBLE-DOOR 2 RUNTIME ENGINE
  // =========================================================================
  int dev3 = abs(getStableRead(joy3Pin) - baseline3);
  int dev4 = abs(getStableRead(joy4Pin) - baseline4);

  if (dev3 > HAIR_TRIGGER_THRESHOLD || dev4 > HAIR_TRIGGER_THRESHOLD) {
    if (!gate2IsOpen) {
      gate2ServoLeft.write(GATE2_LEFT_OPEN); 
      gate2ServoRight.write(GATE2_RIGHT_OPEN);
      gate2IsOpen = true; gate2Millis = currentMillis;
      Serial.println("TELEMETRY:{\"event\":\"gate_open\",\"id\":2}");
    }
  }
  if (gate2IsOpen && (currentMillis - gate2Millis >= doorOpenDuration)) {
    gate2ServoLeft.write(GATE2_LEFT_CLOSED); 
    gate2ServoRight.write(GATE2_RIGHT_CLOSED);
    gate2IsOpen = false;
    Serial.println("TELEMETRY:{\"event\":\"gate_close\",\"id\":2}");
  }

  // =========================================================================
  // 🔘 AUTO-FEEDER 1 ENGINE (2.5 HOUR LOCKED)
  // =========================================================================
  int devFeeder1 = abs(getStableRead(joyFeeder1Pin) - baselineFeeder1);
  if (devFeeder1 > HAIR_TRIGGER_THRESHOLD) {
    if (lastFeeder1Activation == 0 || (currentMillis - lastFeeder1Activation >= FEEDER_LOCKOUT_TIME)) {
      feederServo1.write(FEEDER_DUMP_ANGLE); delay(180);                           
      feederServo1.write(FEEDER_HOME_ANGLE); delay(180);  
      lastFeeder1Activation = currentMillis; 
      Serial.println("TELEMETRY:{\"event\":\"feeder_dispense\",\"id\":1}");
    }
  }

  // =========================================================================
  // 🔘 AUTO-FEEDER 2 ENGINE (2.5 HOUR LOCKED)
  // =========================================================================
  int devFeeder2 = abs(getStableRead(joyFeeder2Pin) - baselineFeeder2);
  if (devFeeder2 > HAIR_TRIGGER_THRESHOLD) {
    if (lastFeeder2Activation == 0 || (currentMillis - lastFeeder2Activation >= FEEDER_LOCKOUT_TIME)) {
      feederServo2.write(FEEDER_DUMP_ANGLE); delay(180);                            
      feederServo2.write(FEEDER_HOME_ANGLE); delay(180);    
      lastFeeder2Activation = currentMillis; 
      Serial.println("TELEMETRY:{\"event\":\"feeder_dispense\",\"id\":2}");
    }
  }

  // =========================================================================
  // 🔘 WHEEL FAN ENGINE (ANYTIME OVERRIDE)
  // =========================================================================
  bool buttonIsPressed = (digitalRead(buttonPin) == LOW);

  if (buttonIsPressed && !fanIsRunning) {
    analogWrite(fanPin, FAN_SLOWEST_SPEED); 
    fanIsRunning = true;
    fanStartedMillis = currentMillis;
    Serial.println("TELEMETRY:{\"event\":\"fan_wheel_start\"}");
  }

  if (fanIsRunning && (currentMillis - fanStartedMillis >= FAN_RUN_DURATION)) {
    analogWrite(fanPin, 0); 
    fanIsRunning = false;
    Serial.println("TELEMETRY:{\"event\":\"fan_wheel_stop\"}");
  }
}
🚀 Setup & Installation
Hardware Deployment
Connect micro servos to Digital PWM Pins 2, 3, 4, 5, 6, and 7.

Connect Analog Joysticks to Pins A0 through A5.

Wire the activation button to Digital Pin 12 and a common GND rail.

Route Pin 11 to the base of your Fan Wheel drive mosfet circuit.

Broadcast Director Implementation
Ensure your compute engine has python packages initialized:

Bash
pip install opencv-python obs-websocket-py
python src/vision_director.py --host 127.0.0.1 --port 4455 --secret YOUR_OBS_PASSWORD
On-Chain Broker Deployment
Configure the Rust environment variables inside your production module (/src/solana_engine/):

Bash
export SOLANA_RPC_URL="[https://api.mainnet-beta.solana.com](https://api.mainnet-beta.solana.com)"
export TARGET_TOKEN_MINT="YourPumpFunTokenMintAddressHere"
cargo run --release
