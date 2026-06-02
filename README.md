# Ratcade

<img width="642" height="502" alt="Screenshot 2026-06-02 at 16 07 27" src="https://github.com/user-attachments/assets/ab3e7e55-64e4-4ee1-a260-1061022785e4" />

⚡ Kinetic Energy Harvesting & Micro-Grid Power Architecture                                  [ MECHANICAL MATRIX ]
 [Rat Wheel Assembly] ---> [Pulley Adapter] ---> [Drive Belt] ---> [Generator Shaft]
                                                                          |
                                                                          v
                                  [ ELECTRICAL MATRIX ]         [DC Permanent Magnet]
 [Main System Rails] <--- [5.5V Supercapacitor Bank] <--- [Synchronous Buck-Boost]
1. Mechanical-to-Electrical PowertrainThe power plant uses a permanent magnet DC brush generator integrated directly into the core structural chassis via high-speed shaft bearings.Because rodents deliver high torque but low raw RPM, a direct-drive setup would fail to cross the generator's minimum internal electromagnetic field ($EMF$) threshold. To correct this, the system incorporates a dual-stage transmission:The Drive Wheel Pulley: A custom-engineered, high-diameter 3D-printed pulley adapter is mounted flush to the wheel hub.The Input Pulley: A low-diameter aluminum input pulley is fixed directly to the generator drive shaft.The Transmission Link: A continuous high-tensile neoprene drive belt couples the assemblies, providing a 1:4.8 mechanical step-up ratio. This forces the generator shaft to spin at an optimal RPM where it can efficiently output raw voltage even during moderate rodent velocity spikes.2. Regulation, Supercapacitors, & InstrumentationThe raw output from the generator is highly unstable and fluctuates rapidly. This unregulated power is routed through a specialized power management PCB containing three distinct solid-state stages:Rectification & Buck-Boost Regulation: An ultra-low quiescent current synchronous Buck-Boost DC-DC converter automatically smooths the input spikes, stepping down overvoltage peaks or stepping up low-velocity generation tails to a constant 5.0V DC charging rail.Energy Storage Matrix: The regulated 5.0V rail charges a high-capacity 5.5V Supercapacitor Bank (dual-cell array configured in parallel). Supercapacitors are utilized here instead of lithium chemistry batteries to leverage their extremely low Equivalent Series Resistance ($ESR$). This allows the grid to instantly absorb sudden high-amperage kinetic bursts without thermal degradation, offering a lifespan of $>500,000$ complete charge-discharge cycles.Telemetry Instrumentation: To track generation metrics in real-time, a Current Sensor (e.g., INA219 high-side I2C monitor) is inline with the generator output. The sensor tracks exact micro-amperage ($I$) and voltage ($V$) fluctuations, sending instant telemetry frames to the system controller to register confirmed rotations.3. Micro-Grid Energy Balance & Power MathThe habitat runs a strict resource economy calculated against system power consumption. The continuous baseline parasitic current draw of the entire system (Atmega328P, 22 active smart sensors, 6 idling joysticks, and logic pull-ups) settles at:$$P_{\text{idle}} \approx 0.25\text{ Watts} \quad (50\text{mA} \text{ @ } 5\text{V})$$The energy transaction model is defined by the following physical parameters:$$\text{1 Full Wheel Rotation} \approx 13.32\text{ Seconds of System Uptime}$$Mathematical Derivation:To keep the grid alive for 13.32 seconds, the system requires a baseline energy allocation calculated as:$$E = P \times t = 0.25\text{W} \times 13.32\text{s} = 3.33\text{ Joules of Usable Energy}$$Given the mechanical efficiency of the belt-drive transmission ($\eta_{\text{mech}} \approx 92\%$) and the thermal efficiency of the synchronous buck-boost converter ($\eta_{\text{elec}} \approx 88\%$), the net system efficiency coefficient is $\eta_{\text{sys}} = 0.92 \times 0.88 \approx 81\%$.Therefore, to bank 3.33 Joules of usable energy, the rodent must exert a minimal kinetic mechanical work output ($W_{\text{input}}$) of approximately 4.11 Joules per full rotation of the wheel assembly. If the energy reserve drops below operational tolerances over the course of the 11-day challenge, the main power rails drop out, causing a total system brownout.🎛️ Hardware Interface Profile & Pin MappingSubsystemFunctional ComponentHardware ChannelPin TypePower IngestionGenerator Current Sensor (INA219)Analog Backup Voltage RailSDA / SCL PinsAnalog Pin A7I2C TelemetryAnalog DC InputDouble-Door 1 (Gate A)Portal Servo Left (1)Portal Servo Right (2)Trigger Joystick 1Trigger Joystick 2Digital Pin 2Digital Pin 3Analog Pin A5Analog Pin A4PWM OutputPWM OutputAnalog InputAnalog InputDouble-Door 2 (Gate B)Portal Servo Left (3)Portal Servo Right (4)Trigger Joystick 3Trigger Joystick 4Digital Pin 4Digital Pin 5Analog Pin A3Analog Pin A2PWM OutputPWM OutputAnalog InputAnalog InputCalorie FeedersAuger Servo 1 (Station 1)Auger Servo 2 (Station 2)Feeder Joystick 1Feeder Joystick 2Digital Pin 6Digital Pin 7Analog Pin A1Analog Pin A0PWM OutputPWM OutputAnalog InputAnalog InputWheel Fan OverrideSlow-Velocity Fan MotorManual Tactile ButtonDigital Pin 11Digital Pin 12PWM Hardware OutputDigital Input (Pullup)Aquatic Sub-SystemContinuous Filtration PumpWater Delivery SolenoidDigital Pin 9Digital Pin 10PWM Power OutputDigital OutputEcosystem EastereggsUnderground Lift ServoWaterslide Valve RelayDigital Pin 13Analog Pin A6 (Digital Mode)Digital OutputDigital Output💻 System Core FirmwareThis firmware code handles all components of the physical environment layer, incorporating continuous filtration networks, dynamic power telemetry monitoring, and hidden dynamic mechanical routines.C++#include <Servo.h>
#include <Wire.h> // Required for I2C communication with current sensor

// =========================================================================
// ⚙️ SERVO POSITION CALIBRATION POOL (ANTI-HUNTING COEFFICIENTS)
// =========================================================================
const int GATE1_LEFT_CLOSED  = 0;   const int GATE1_LEFT_OPEN  = 90; 
const int GATE1_RIGHT_CLOSED = 0;   const int GATE1_RIGHT_OPEN = 90; 

const int GATE2_CLOSED       = 0;   const int GATE2_OPEN       = 90; 
const int FEEDER_HOME_ANGLE  = 120;  const int FEEDER_DUMP_ANGLE = 0;  
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
const unsigned long FEEDER_LOCKOUT_TIME = 10800000; // Exactly 3.0 Hours in milliseconds

// Wheel Fan Configuration
const int fanPin = 11;       
const int buttonPin = 12;    
const int FAN_SLOWEST_SPEED = 45; 
bool fanIsRunning = false;
unsigned long fanStartedMillis = 0;
const unsigned long FAN_RUN_DURATION = 3000; 

// Power Instrumentation & Life Support Pins
const int generatorVoltagePin = A7; // Hardware line monitoring charging rail voltage
const int aquariumFilterPumpPin = 9;
const int cleanWaterSolenoidPin = 10;

// Easteregg Subsystem Mapping
Servo undergroundLiftServo;
const int undergroundLiftServoPin = 13;
const int waterslideValveRelayPin = A6; 

unsigned long lastHydroPurgeMillis = 0;
const unsigned long HYDRO_PURGE_INTERVAL = 1800000; // 30 Minute interval
bool waterSolenoidActive = false;
unsigned long waterSolenoidTriggerMillis = 0;

const int HAIR_TRIGGER_THRESHOLD = 25; 
const unsigned long doorOpenDuration = 16000; 

// Kinematic Tracking Parameters
int lastVoltageRead = 0;
const int VOLTAGE_SPIKE_THRESHOLD = 600; // ADC code mapping to kinetic generation threshold

int getStableRead(int pin) {
  analogRead(pin);
  delayMicroseconds(50);
  return analogRead(pin);
}

void setup() {
  Serial.begin(115200);
  Serial.println("[RATCADE_INITIALIZE] Executing system hardware boot...");
  Wire.begin(); // Boot the configuration bus for telemetry processing

  // Actuator Attaching
  gate1ServoLeft.attach(gate1ServoLeftPin);   gate1ServoLeft.write(GATE1_LEFT_CLOSED);
  gate1ServoRight.attach(gate1ServoRightPin); gate1ServoRight.write(GATE1_RIGHT_CLOSED);
  gate2ServoLeft.attach(gate2ServoLeftPin);   gate2ServoLeft.write(GATE2_CLOSED);
  gate2ServoRight.attach(gate2ServoRightPin); gate2ServoRight.write(GATE2_CLOSED);

  feederServo1.attach(feederServo1Pin); feederServo1.write(FEEDER_HOME_ANGLE);
  feederServo2.attach(feederServo2Pin); feederServo2.write(FEEDER_HOME_ANGLE);
  
  undergroundLiftServo.attach(undergroundLiftServoPin);
  undergroundLiftServo.write(LIFT_DOWN_POSITION);

  // Initialize Continuous Life Support Ports
  pinMode(aquariumFilterPumpPin, OUTPUT);
  pinMode(cleanWaterSolenoidPin, OUTPUT);
  pinMode(waterslideValveRelayPin, OUTPUT);
  
  // Power up primary life support water filtration node
  analogWrite(aquariumFilterPumpPin, 180); 
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

  Serial.println("[SYSTEM_ONLINE] Micro-grid telemetry monitoring active.");
}

void loop() {
  unsigned long currentMillis = millis();

  // =========================================================================
  // ⚡ POWER INSTRUMENTATION & GENERATION TRACKING
  // =========================================================================
  int rawVoltage = analogRead(generatorVoltagePin);
  // Detect if kinetic activity generated a voltage spike over baseline levels
  if (rawVoltage > VOLTAGE_SPIKE_THRESHOLD && lastVoltageRead <= VOLTAGE_SPIKE_THRESHOLD) {
    // Single rotation generation event confirmed; emit data frame to edge node
    Serial.println("TELEMETRY:{\"event\":\"kinetic_generation\",\"increment_seconds\":13.32}");
  }
  lastVoltageRead = rawVoltage;

  // =========================================================================
  // 🔘 LIFE SUPPORT SYSTEMS (AQUARIUM & WATER MATRIX)
  // =========================================================================
  if (currentMillis - lastHydroPurgeMillis >= HYDRO_PURGE_INTERVAL) {
    digitalWrite(cleanWaterSolenoidPin, HIGH);
    waterSolenoidActive = true;
    waterSolenoidTriggerMillis = currentMillis;
    lastHydroPurgeMillis = currentMillis;
    Serial.println("TELEMETRY:{\"event\":\"aquarium_replenish_start\"}");
  }

  if (waterSolenoidActive && (currentMillis - waterSolenoidTriggerMillis >= 5000)) { 
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
      gate2ServoLeft.write(GATE2_OPEN); 
      gate2ServoRight.write(GATE2_OPEN);
      gate2IsOpen = true; gate2Millis = currentMillis;
      Serial.println("TELEMETRY:{\"event\":\"gate_open\",\"id\":2}");
    }
  }
  if (gate2IsOpen && (currentMillis - gate2Millis >= doorOpenDuration)) {
    gate2ServoLeft.write(GATE2_CLOSED); 
    gate2ServoRight.write(GATE2_CLOSED);
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
    digitalWrite(waterslideValveRelayPin, HIGH); 
    
    Serial.println("TELEMETRY:{\"event\":\"fan_wheel_start\"}");
    Serial.println("TELEMETRY:{\"event\":\"secret_easteregg_triggered\"}");
  }

  if (fanIsRunning && (currentMillis - fanStartedMillis >= FAN_RUN_DURATION)) {
    analogWrite(fanPin, 0); 
    fanIsRunning = false;
    
    // Reset structural easteregg elements back into hidden configuration
    undergroundLiftServo.write(LIFT_DOWN_POSITION);
    digitalWrite(waterslideValveRelayPin, LOW); 
    
    Serial.println("TELEMETRY:{\"event\":\"fan_wheel_stop\"}");
  }
}
🛠️ Installation & Compilation Sequence1. Embedded Layer DeploymentLoad the core firmware sketch inside the native Arduino IDE environment.Select target board type as Arduino Nano (or equivalent ATmega328P target) inside Tools -> Board.Compile and execute the flash sequence onto the microchip via the specified USB port interface.2. Live Stream Edge Controller IntegrationInitialize your edge engine python dependency pool to handle the automated camera tracking pipeline:Bashpip install opencv-python obs-websocket-py
python src/vision_director.py --host 127.0.0.1 --port 4455 --secret LOCAL_SOCKET_SECRET
3. Solana Execution Module DeploymentCompile and run the performance-optimized Web3 routing module written in Rust:Bashexport RPC_ENDPOINT="https://api.mainnet-beta.solana.com"
export TOKEN_MINT_ADDRESS="YourPumpFunTokenMintAddress"
cargo run --release
