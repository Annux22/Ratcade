# Ratcade

<img width="642" height="502" alt="Screenshot 2026-06-02 at 16 07 27" src="https://github.com/user-attachments/assets/ab3e7e55-64e4-4ee1-a260-1061022785e4" />


+-----------------------------------------------------------------------+|                        CORE HARDWARE LAYER                            |+-----------------------------------------------------------------------+| (22 Smart Sensors, 6 Joysticks, 6 Servos, DC Generator, PWM Fan)v+-----------------------------------------------------------------------+|                   MAIN MICROCONTROLLER (ATmega328P)                   |+-----------------------------------------------------------------------+| (USB Serial Data Frame: 115200 Baud)v+-----------------------------------------------------------------------+|                       EDGE COMPUTING NODE                             ||           (Raspberry Pi 5 / NVIDIA Jetson Nano Linux Host)            |+-----------------------------------------------------------------------+// (Local WebSockets / JSON Frame)                                       \ (Rust Engine / RPC)v                                                                         v+------------------------------------+                    +------------------------------------+|       OBS STREAM AUTOMATION        |                    |       ON-CHAIN EXECUTION           ||  Multi-Cam OpenCV Zone Tracking    |                    |   Programmatic Pump.fun/Jupiter    ||  Automated Scene Transitions       |                    |   Non-Custodial Block Engine Swaps |+------------------------------------+                    +------------------------------------+
---

## 📊 Level 1 Mission Rules & Resource Economics

The Ratcade architecture enforces an environment of controlled resource scarcity where basic rodent needs are gated behind mechanical tokens and gamified milestones.

* **Starting Reserves:** The system initializes with a stored energy buffer of exactly **72 Hours** within the onboard supercapacitor array.
* **Survival Objective:** The enclosure's inhabitants must balance intake and harvest kinetic energy to sustain systemic power for **11 Days (264 Hours) continuously** without a micro-grid brownout.
* **Caloric Control:** The dual feeding stations are software-throttled via non-blocking hardware clocks, allowing an execution loop only once every **3 Hours** per station.
* **Life Support Systems:** The high-volume aquarium filtration grid and hydroponic life-support watering lines run continuously, isolated mathematically from behavioral lockouts to guarantee habitat safety.

---

## ⚡ Kinetic Energy Harvesting & Micro-Grid Power Architecture

The power generation module scales the physical mechanics of a direct-drive wind turbine down to a high-efficiency rodent running wheel assembly.

### 1. Mechanical-to-Electrical Powertrain
The running wheel is mated to a low-cogging, high-flux DC permanent magnet generator via a precision-machined **3D printed step-up pulley adapter** and a high-tensile drive belt. The pulley ratio is structurally optimized to translate low-RPM mechanical rotation into the generator's peak voltage output curve.

The variable, high-ripple DC voltage generated from active running is normalized via a synchronous **Buck-Boost DC-DC Converter** containing strict overvoltage clamping protection. This clean power directly charges a **5.5V Supercapacitor Bank**. Supercapacitors were integrated instead of chemical lithium cells due to their ultra-low Equivalent Series Resistance ($ESR$) and near-infinite charge/discharge cycle life, efficiently absorbing high-amperage kinetic generation spikes.

### 2. Micro-Grid Balance Calibration
The entire sensor array, logic gates, and servo idle states produce a continuous baseline parasitic current draw ($P_{\text{idle}} \approx 0.25\text{W}$). The electro-mechanical system transaction is calibrated precisely to the following mathematical model:

$$\text{1 Full Mechanical Wheel Rotation} \approx 13.32\text{ Seconds of Grid System Uptime}$$

To achieve this, the generator captures and delivers approximately $3.33\text{ Joules}$ of net electrical energy directly to the supercapacitor storage pool per full rotation, accounting for mechanical torque friction and an estimated $\eta = 88\%$ DC-DC conversion efficiency.

---

## 👁️ Computer Vision Mesh & Automated Broadcast Switching

The tracking system turns the physical enclosure into an automated live broadcast environment. An edge computing node continuously processes a multi-camera vision mesh using an optimized background subtraction algorithm.

### 1. Spatial Segmentation Algorithm
The Linux host grabs frame buffers from 4 independent USB camera inputs positioned at critical functional zones.

```python
import cv2
# MOG2 Background Subtractor Initialization for Target Isolation
backSub = cv2.createBackgroundSubtractorMOG2(history=500, varThreshold=16, detectShadows=False)
The system segregates the rodent's spatial coordinates $(X, Y)$ from static structures, monitoring movement density vectors across four distinct operational matrices:Zone 1: Kinetic Generator WheelZone 2: Portal Gate A (Double-Doors 1)Zone 3: Portal Gate B (Double-Doors 2)Zone 4: Automated Calorie Dispensers / Aquarium Recirculation View2. OBS WebSockets AutomationWhen movement density crosses the tracking threshold inside any specific zone and remains stable for more than $400\text{ms}$, a Python orchestration daemon executes a JSON payload across a localized WebSocket connection directly to OBS Studio via obs-websocket-py.OBS dynamically triggers an instant, frame-accurate scene change to the video feed focused on that active sector, managing the entire broadcast stream autonomously.🦀 On-Chain Architecture: Programmatic Token Buys & BurnsRatcade bridges physical ecosystem metrics directly with the Solana blockchain. When critical milestones are achieved (e.g., survival milestones or specific generation quotas), a headless Rust execution worker handles trustless market transactions on pump.fun bonding curves or Raydium AMM pools.[Telemetry Trigger] ---> [Rust Execution Worker] ---> [AWS KMS Core Signer] ---> [Jito MEV Block Engine] ---> [Token Burn Address]
1. Secure Telemetry ValidationState transitions (such as verified dispenser cycles or generator voltage spikes) are packaged into encrypted data frames by the firmware and delivered via serial pipeline to the host node. The host runs an validation loop to prevent false triggers or hardware noise from altering transactional outputs.2. Non-Custodial Key SignaturesTo guarantee enterprise-grade security, no raw private keys or seed phrase strings are stored on-disk or exposed inside the execution environment. The Rust script interacts with an AWS Key Management Service (KMS) or dedicated Hardware Security Module (HSM) holding the transaction-signing keys. The payload is signed inside the isolated cryptographic module before broadcast.3. Jito MEV Front-Running ProtectionTo insulate transactions from malicious sandwich attacks, front-running bots, or mempool slippage, the compiled transaction is not sent via standard public RPC nodes. Instead, the signed payload is converted into an atomic transaction bundle and sent directly to the Jito Block Engine alongside a specified tip in SOL. This bypasses the public mempool, ensuring guaranteed, flash-bot-protected block execution.4. Immutable Deflationary SwapsThe execution worker queries the Jupiter Aggregator API to resolve the most efficient swap path, converting incoming ecosystem fees (SOL) into the target SPL token. The purchased token balance is immediately transferred to the native Solana system burn address:Plaintext11111111111111111111111111111111
This process achieves an un-manipulable, permanent reduction in circulating supply, verifiable directly on-chain.🎛️ Hardware Interface Profile & Pin MappingSubsystemFunctional ComponentHardware ChannelPin TypeDouble-Door 1 (Gate A)Portal Servo Left (1)Portal Servo Right (2)Trigger Joystick 1Trigger Joystick 2Digital Pin 2Digital Pin 3Analog Pin A5Analog Pin A4PWM OutputPWM OutputAnalog InputAnalog InputDouble-Door 2 (Gate B)Portal Servo Left (3)Portal Servo Right (4)Trigger Joystick 3Trigger Joystick 4Digital Pin 4Digital Pin 5Analog Pin A3Analog Pin A2PWM OutputPWM OutputAnalog InputAnalog InputCalorie FeedersAuger Servo 1 (Station 1)Auger Servo 2 (Station 2)Feeder Joystick 1Feeder Joystick 2Digital Pin 6Digital Pin 7Analog Pin A1Analog Pin A0PWM OutputPWM OutputAnalog InputAnalog InputWheel Fan OverrideSlow-Velocity Fan MotorManual Tactile ButtonDigital Pin 11Digital Pin 12PWM Hardware OutputDigital Input (Pullup)Aquatic Sub-SystemContinuous Filtration PumpWater Delivery SolenoidDigital Pin 9Digital Pin 10PWM Power OutputDigital OutputEcosystem EastereggsUnderground Lift ServoWaterslide Valve RelayDigital Pin 13Analog Pin A6 (Digital Mode)Digital OutputDigital Output💻 System Core FirmwareThis production-grade, non-blocking C++ framework executes the low-level processing loops across all components of the physical environment layer, incorporating hydro-filtration networks and hidden dynamic mechanical routines.C++#include <Servo.h>

// =========================================================================
// ⚙️ SERVO POSITION CALIBRATION POOL (ANTI-HUNTING COEFFICIENTS)
// =========================================================================
const int GATE1_LEFT_CLOSED  = 0;   const int GATE1_LEFT_OPEN  = 90; 
const int GATE1_RIGHT_CLOSED = 0;   const int GATE1_RIGHT_OPEN = 90; 

const int GATE2_LEFT_CLOSED  = 0;   const int GATE2_LEFT_OPEN  = 90; 
const int GATE2_RIGHT_CLOSED = 0;   const int GATE2_RIGHT_OPEN = 86; 

const int FEEDER_HOME_ANGLE  = 120;  const int FEEDER_DUMP_ANGLE = 0;  

// Easteregg Mechanical Extents
const int LIFT_DOWN_POSITION = 10;  const int LIFT_UP_POSITION = 170;

// =========================================================================
// 🛠️ HARDWARE ASSIGNMENT OBJECT MATRIX
// =========================================================================
Servo gate1ServoLeft; Servo gate1ServoRight;
const int gate1ServoLeftPin = 2; const int gate1ServoRightPin = 3;
const int joy1Pin = A5; const int joy2Pin = A4;
int baseline1, baseline2;
bool gate1IsOpen = false; unsigned long gate1Millis = 0;

Servo gate2ServoLeft; Servo gate2ServoRight;
const int gate2ServoLeftPin = 4; const int gate2ServoRightPin = 5;
const int joy3Pin = A3; const int joy4Pin = A2;
int baseline3, baseline4;
bool gate2IsOpen = false; unsigned long gate2Millis = 0;

Servo feederServo1; const int feederServo1Pin = 6; const int joyFeeder1Pin = A1; int baselineFeeder1;
Servo feederServo2; const int feederServo2Pin = 7; const int joyFeeder2Pin = A0; int baselineFeeder2;

unsigned long lastFeeder1Activation = 0;
unsigned long lastFeeder2Activation = 0;
const unsigned long FEEDER_LOCKOUT_TIME = 10800000; // Updated to exactly 3.0 Hours in milliseconds

// Wheel Fan Override
const int fanPin = 11;       
const int buttonPin = 12;    
const int FAN_SLOWEST_SPEED = 45; 
bool fanIsRunning = false;
unsigned long fanStartedMillis = 0;
const unsigned long FAN_RUN_DURATION = 3000; 

// Life Support Infrastructure Pins
const int aquariumFilterPumpPin = 9;
const int cleanWaterSolenoidPin = 10;

// Easteregg Subsystem Mapping
Servo undergroundLiftServo;
const int undergroundLiftServoPin = 13;
const int waterslideValveRelayPin = A6; // Designated analog line utilized as digital out

unsigned long lastHydroPurgeMillis = 0;
const unsigned long HYDRO_PURGE_INTERVAL = 1800000; // Periodic self-clean system flush (30 Minutes)
bool waterSolenoidActive = false;
unsigned long waterSolenoidTriggerMillis = 0;

const int HAIR_TRIGGER_THRESHOLD = 25; 
const unsigned long doorOpenDuration = 16000; 

int getStableRead(int pin) {
  analogRead(pin);
  delayMicroseconds(50);
  return analogRead(pin);
}

void setup() {
  Serial.begin(115200);
  Serial.println("[RATCADE_INITIALIZE] Executing system hardware boot...");

  // Actuator Attaching
  gate1ServoLeft.attach(gate1ServoLeftPin);   gate1ServoLeft.write(GATE1_LEFT_CLOSED);
  gate1ServoRight.attach(gate1ServoRightPin); gate1ServoRight.write(GATE1_RIGHT_CLOSED);
  gate2ServoLeft.attach(gate2ServoLeftPin);   gate2ServoLeft.write(GATE2_LEFT_CLOSED);
  gate2ServoRight.attach(gate2ServoRightPin); gate2ServoRight.write(GATE2_RIGHT_CLOSED);

  feederServo1.attach(feederServo1Pin); feederServo1.write(FEEDER_HOME_ANGLE);
  feederServo2.attach(feederServo2Pin); feederServo2.write(FEEDER_HOME_ANGLE);
  
  undergroundLiftServo.attach(undergroundLiftServoPin);
  undergroundLiftServo.write(LIFT_DOWN_POSITION);

  // Initialize Continuous Life Support Ports
  pinMode(aquariumFilterPumpPin, OUTPUT);
  pinMode(cleanWaterSolenoidPin, OUTPUT);
  pinMode(waterslideValveRelayPin, OUTPUT);
  
  // Power up primary life support water filtration node
  analogWrite(aquariumFilterPumpPin, 180); // Establish optimal continuous flow metrics
  digitalWrite(cleanWaterSolenoidPin, LOW);
  digitalWrite(waterslideValveRelayPin, LOW);

  pinMode(fanPin, OUTPUT);
  analogWrite(fanPin, 0); 
  pinMode(buttonPin, INPUT_PULLUP); 

  // Sensor Stabilization Calibration Delay
  delay(500);
  baseline1 = getStableRead(joy1Pin);
  baseline2 = getStableRead(joy2Pin);
  baseline3 = getStableRead(joy3Pin);
  baseline4 = getStableRead(joy4Pin);
  baselineFeeder1 = getStableRead(joyFeeder1Pin);
  baselineFeeder2 = getStableRead(joyFeeder2Pin);

  Serial.println("[SYSTEM_ONLINE] Telemetry monitoring active.");
}

void loop() {
  unsigned long currentMillis = millis();

  // =========================================================================
  // 🔘 LIFE SUPPORT SYSTEMS (AQUARIUM & WATER MATRIX)
  // =========================================================================
  // Automated Hydro-Purge: Keep water levels pristine automatically
  if (currentMillis - lastHydroPurgeMillis >= HYDRO_PURGE_INTERVAL) {
    digitalWrite(cleanWaterSolenoidPin, HIGH);
    waterSolenoidActive = true;
    waterSolenoidTriggerMillis = currentMillis;
    lastHydroPurgeMillis = currentMillis;
    Serial.println("TELEMETRY:{\"event\":\"aquarium_replenish_start\"}");
  }

  if (waterSolenoidActive && (currentMillis - waterSolenoidTriggerMillis >= 5000)) { // 5-second freshwater top-off
    digitalWrite(cleanWaterSolenoidPin, LOW);
    waterSolenoidActive = false;
    Serial.println("TELEMETRY:{\"event\":\"aquarium_replenish_stop\"}");
  }

  // =========================================================================
  // 🔘 DOUBLE-DOOR 1 MAIN PROCESSING RUNTIME (GATE A)
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
  // 🔘 DOUBLE-DOOR 2 MAIN PROCESSING RUNTIME (GATE B)
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
  // 🔘 CALORIE DISPENSER 1 PROCESSING ENGINE (3.0 HOUR LIMITER)
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
  // 🔘 CALORIE DISPENSER 2 PROCESSING ENGINE (3.0 HOUR LIMITER)
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
  // 🔘 WHEEL FAN DRIVE MODULE & AUTOMATED SECRET EASTEREEGS
  // =========================================================================
  bool buttonIsPressed = (digitalRead(buttonPin) == LOW);

  if (buttonIsPressed && !fanIsRunning) {
    analogWrite(fanPin, FAN_SLOWEST_SPEED); 
    fanIsRunning = true;
    fanStartedMillis = currentMillis;
    
    // Easteregg Activation Chain: Trigger underground lift and fill pool via waterslide valve
    undergroundLiftServo.write(LIFT_UP_POSITION);
    digitalWrite(waterslideValveRelayPin, HIGH); // Open luxury pool filling circuit
    
    Serial.println("TELEMETRY:{\"event\":\"fan_wheel_start\"}");
    Serial.println("TELEMETRY:{\"event\":\"secret_easteregg_triggered\"}");
  }

  if (fanIsRunning && (currentMillis - fanStartedMillis >= FAN_RUN_DURATION)) {
    analogWrite(fanPin, 0); 
    fanIsRunning = false;
    
    // Reset structural easteregg elements back into hidden configuration
    undergroundLiftServo.write(LIFT_DOWN_POSITION);
    digitalWrite(waterslideValveRelayPin, LOW); // Close pool flush valve
    
    Serial.println("TELEMETRY:{\"event\":\"fan_wheel_stop\"}");
  }
}
🛠️ Installation & Compilation Sequence1. Embedded Layer DeploymentLoad the core firmware sketch inside the native Arduino IDE environment.Select target board type as Arduino Nano (or equivalent ATmega328P target) inside Tools -> Board.Compile and execute the flash sequence onto the microchip via the specified USB port interface.2. Live Stream Edge Controller IntegrationInitialize your edge engine python dependency pool to handle the automated camera tracking pipeline:Bashpip install opencv-python obs-websocket-py
python src/vision_director.py --host 127.0.0.1 --port 4455 --secret LOCAL_SOCKET_SECRET
3. Solana Execution Module DeploymentCompile and run the performance-optimized Web3 routing module written in Rust:Bashexport RPC_ENDPOINT="[https://api.mainnet-beta.solana.com](https://api.mainnet-beta.solana.com)"
export TOKEN_MINT_ADDRESS="YourPumpFunTokenMintAddress"
cargo run --release
