# 🚤 BLACKMAMBA MicroBoat RC

### Compact Mechatronic Tow / Research Boat

> Embarcación eléctrica compacta, transportable sobre un automóvil, con radio control, sensores, telemetría y funciones de seguridad asistidas por microcontrolador.

**Proyecto:** BLACKMAMBA MicroBoat RC  
**Área:** Mecatrónica · Robótica · Sistemas Embebidos · Radio Control  
**Concepto:** Car → Beach → Water → Tow → Return

---

## 1. Visión

Diseñar una pequeña embarcación eléctrica capaz de transportarse en la parte superior de un automóvil y desplegarse rápidamente al llegar a playa, lago o zona acuática.

La plataforma busca integrar:

- radio control manual;
- propulsión eléctrica;
- microcontrolador principal;
- GPS/GNSS;
- IMU;
- telemetría;
- monitoreo de batería;
- detección de entrada de agua;
- medición de temperatura;
- medición de fuerza de remolque;
- retorno asistido al punto de lanzamiento;
- registro de datos para pruebas y optimización.

La idea no es construir solamente una lancha RC, sino una **plataforma mecatrónica acuática modular**.

---

## 2. Arquitectura general

```text
                  ┌─────────────────────┐
                  │   RADIO CONTROL     │
                  │ Joysticks / Switches│
                  └──────────┬──────────┘
                             │
                       RF / Telemetría
                             │
                             ▼
┌──────────────────────────────────────────────────┐
│              MICROCONTROLADOR PRINCIPAL          │
│                                                  │
│   ESP32-S3 / STM32                               │
│                                                  │
│   ┌─────────┐ ┌─────────┐ ┌─────────┐           │
│   │   GPS   │ │   IMU   │ │ Battery │           │
│   └────┬────┘ └────┬────┘ └────┬────┘           │
│        │            │           │                 │
│   ┌────▼────────────▼───────────▼────┐            │
│   │       CONTROL + SAFETY LOGIC     │            │
│   └──────────────┬───────────────────┘            │
└──────────────────┼────────────────────────────────┘
                   │
          ┌────────┴────────┐
          │                 │
          ▼                 ▼
      ESC MOTOR L       ESC MOTOR R
          │                 │
          ▼                 ▼
      PROPULSIÓN L      PROPULSIÓN R
```

---

## 3. Microcontrolador

### ESP32-S3

Primera opción para prototipo:

- doble núcleo;
- Wi-Fi y Bluetooth;
- PWM;
- ADC;
- UART;
- I²C;
- SPI;
- GPIO abundantes;
- ecosistema amplio.

Responsabilidades:

```text
Sensores
   ↓
ESP32-S3
   ├── lectura de radio
   ├── mezcla de motores
   ├── telemetría
   ├── alarmas
   ├── registro de datos
   └── lógica de failsafe
```

### Arquitectura avanzada

Para separar control crítico y comunicaciones:

```text
STM32
  ├── control de motores
  ├── sensores críticos
  ├── watchdog
  └── failsafe

ESP32
  ├── Wi-Fi
  ├── Bluetooth
  ├── interfaz
  └── telemetría
```

---

## 4. Radio control

El control principal debe funcionar sin depender de Internet.

Opciones:

- RC convencional 2.4 GHz;
- ExpressLRS;
- LoRa para telemetría de baja velocidad;
- Wi-Fi para configuración;
- Bluetooth para mantenimiento local.

```text
TRANSMISOR RC
     │
     ▼
RECEPTOR RC
     │
     ▼
MICROCONTROLADOR
     │
     ├── Throttle
     ├── Steering
     ├── Arm / Disarm
     ├── Mode
     ├── Lights
     └── Return Home
```

---

## 5. Propulsión

### Twin Motor

Configuración propuesta:

```text
      MOTOR L       MOTOR R
         ↘           ↙
          EMBARCACIÓN
```

Control diferencial:

```text
L = 70 %   R = 70 %   → RECTO
L = 30 %   R = 70 %   → GIRO IZQUIERDA
L = 70 %   R = 30 %   → GIRO DERECHA
```

Ventajas:

- elimina timón mecánico;
- permite giro diferencial;
- simplifica la mecánica;
- ofrece redundancia parcial;
- facilita control por software.

Cada motor BLDC utilizará su propio ESC.

---

## 6. Sensores

### IMU

Opciones:

- BNO085;
- ICM-20948;
- MPU6050 para prototipo económico.

Variables:

- roll;
- pitch;
- yaw;
- aceleración;
- velocidad angular.

### GPS / GNSS

Opción sugerida:

- u-blox M10.

Funciones:

- posición;
- velocidad;
- rumbo;
- distancia al punto de lanzamiento;
- geofence;
- retorno asistido.

### Batería

Sensores posibles:

- INA219;
- INA226;
- INA228.

Datos:

- voltaje;
- corriente;
- potencia;
- consumo acumulado;
- estimación de batería restante.

### Temperatura

Sensores tipo DS18B20 o equivalentes para:

- ESC izquierdo;
- ESC derecho;
- batería;
- compartimento electrónico.

### Detección de agua

Sensores internos en:

- proa;
- centro;
- popa.

Si se detecta agua, el sistema genera alerta y limita la misión.

### Celda de carga

Una celda de carga en el punto de remolque permite medir la tensión real del cabo.

```text
LANCHA
  │
LOAD CELL
  │
CUERDA
  │
FLOTADOR / CARGA
```

Esto permite desarrollar posteriormente control de fuerza de remolque.

---

## 7. Modos de operación

### DOCK

Velocidad limitada para maniobras cercanas.

### ECO

Prioriza autonomía.

### SPORT

Permite mayor potencia dentro de límites configurados.

### TOW

Modo de remolque con aceleración progresiva y límite de tensión.

---

## 8. Máquina de estados

```text
BOOT
  │
  ▼
SELF TEST
  │
  ▼
DISARMED
  │
  ▼
ARMED
  │
  ├───────────────┐
  ▼               ▼
MANUAL         ASSISTED
  │               │
  └──────┬────────┘
         ▼
   RETURN HOME
         │
         ▼
      DISARM
```

---

## 9. Self-test

Al encender:

```text
[✓] MCU
[✓] RADIO
[✓] IMU
[✓] GPS
[✓] BATTERY
[✓] WATER SENSOR
[✓] ESC LEFT
[✓] ESC RIGHT
```

Un fallo crítico debe impedir el armado.

---

## 10. Failsafe

Estados mínimos:

```text
NORMAL
WARNING
RETURN_HOME
EMERGENCY
DISARMED
```

Ejemplo de pérdida de radio:

```text
RADIO LOST
      │
      ▼
REDUCE SPEED
      │
      ▼
CHECK GPS
      │
      ├── GPS OK   → RETURN HOME
      └── GPS FAIL → SAFE STOP
```

Los motores deben arrancar siempre en estado seguro:

```text
DEFAULT MOTOR STATE = OFF
```

---

## 11. Watchdog

El firmware debe usar watchdog independiente.

Si el software deja de responder:

```text
WATCHDOG TIMEOUT
      ↓
RESET MCU
      ↓
MOTORS SAFE
```

---

## 12. Sistema eléctrico

```text
BATTERY
   │
MAIN FUSE
   │
MAIN SWITCH
   │
   ├──────── ESC LEFT ───── MOTOR LEFT
   ├──────── ESC RIGHT ──── MOTOR RIGHT
   │
   └──────── DC/DC
              │
          12V / 5V / 3V3
              │
          ELECTRONICS
```

Para primeras pruebas, LiFePO4 resulta atractiva por robustez y vida útil.

---

## 13. Telemetría

Datos sugeridos:

```text
BATTERY      82 %
GPS          OK
RADIO        96 %
SPEED        8.2 km/h
DIST HOME    74 m
ROLL         3°
PITCH        1°
MOTOR L      42 %
MOTOR R      44 %
TEMP ESC     38 °C
WATER        DRY
TOW FORCE    24 N
```

---

## 14. BlackMamba Smart Tow

Objetivo futuro: regular automáticamente la fuerza de remolque.

```text
TARGET FORCE
     │
     ▼
PID CONTROLLER
     │
     ▼
MOTOR POWER
     │
     ▼
LOAD CELL
     │
     └──────── feedback
```

Esto permite limitar aceleraciones bruscas y proteger estructura, propulsión y línea de remolque.

---

## 15. Geofence

El sistema podrá definir un radio máximo de operación.

```text
HOME
  │
  └──── MAX_RADIUS
```

Al superar el límite:

- impedir aumento de distancia;
- reducir potencia;
- iniciar retorno asistido cuando proceda.

---

## 16. Black Box

Cada navegación debe generar un registro:

```text
timestamp
GPS
speed
heading
battery_voltage
battery_current
motor_L
motor_R
roll
pitch
radio_quality
tow_force
water_alarm
temperature
```

Formatos iniciales:

- CSV;
- JSON;
- binario para registros de alta frecuencia.

---

## 17. Firmware

Estructura propuesta:

```text
firmware/
├── main/
│   └── main.cpp
├── control/
│   ├── propulsion.cpp
│   ├── mixer.cpp
│   └── pid.cpp
├── navigation/
│   ├── gps.cpp
│   ├── heading.cpp
│   ├── geofence.cpp
│   └── return_home.cpp
├── sensors/
│   ├── imu.cpp
│   ├── battery.cpp
│   ├── temperature.cpp
│   ├── water.cpp
│   └── loadcell.cpp
├── radio/
│   ├── receiver.cpp
│   └── telemetry.cpp
├── safety/
│   ├── failsafe.cpp
│   ├── watchdog.cpp
│   └── emergency.cpp
└── config/
    └── parameters.h
```

---

## 18. Casco

Una geometría tipo catamarán es una candidata interesante para el prototipo:

- mayor estabilidad;
- espacio para electrónica;
- motores separados;
- baja escora;
- buena plataforma para sensores.

```text
      PROA
       ▲

 ┌───────┐     ┌───────┐
 │ CASCO │─────│ CASCO │
 │   L   │     │   R   │
 └───────┘     └───────┘

 MOTOR L       MOTOR R
```

---

## 19. Transporte sobre automóvil

Concepto:

```text
CARRO
  │
ROOF RACK
  │
CRADLE
  │
MICROBOAT
```

Durante transporte, la potencia de los motores debe estar físicamente desconectada.

El diseño final deberá respetar los límites de carga dinámica y estática del vehículo y del portaequipajes empleado.

---

## 20. Seguridad de propulsión

Si la embarcación opera cerca de personas, se debe minimizar el riesgo de contacto con partes rotativas.

Opciones de estudio:

- propeller guard;
- ducted propeller;
- waterjet.

El sistema de propulsión para cualquier prueba cercana a nadadores requerirá validación física específica.

---

## 21. BOM inicial

### Control

- ESP32-S3;
- opcional STM32;
- receptor RC;
- transmisor RC;
- antenas.

### Navegación

- GNSS u-blox;
- IMU.

### Potencia

- batería;
- fusible;
- interruptor maestro;
- convertidor DC/DC;
- ESC ×2;
- motor BLDC ×2.

### Sensores

- corriente/voltaje;
- temperatura;
- entrada de agua;
- celda de carga;
- sonar opcional.

### Mecánica

- casco;
- soportes;
- propulsión protegida;
- punto estructural de remolque;
- caja estanca;
- prensaestopas y conectores apropiados.

---

## 22. Fases de desarrollo

### Phase 0 — Simulation

- masa;
- flotabilidad;
- centro de gravedad;
- empuje;
- resistencia;
- autonomía.

### Phase 1 — Hull

Validar:

- flotabilidad;
- estabilidad;
- peso;
- distribución de masas.

### Phase 2 — RC Propulsion

Instalar:

- radio;
- ESC;
- motores;
- batería.

Pruebas sin personas y sin remolque.

### Phase 3 — Sensors

Agregar:

- IMU;
- GPS;
- batería;
- temperatura;
- agua.

### Phase 4 — Telemetry

Transmitir variables en tiempo real.

### Phase 5 — Failsafe

Validar:

- radio loss;
- batería baja;
- sobretemperatura;
- entrada de agua;
- pérdida de GPS.

### Phase 6 — Return Home

Pruebas progresivas en entorno controlado.

### Phase 7 — Tow Instrumentation

Secuencia:

1. peso muerto ligero;
2. flotador sin persona;
3. carga instrumental;
4. validación de estructura, propulsión, control y límites.

---

## 23. Roadmap

```text
RC BOAT
   ↓
SENSOR BOAT
   ↓
TELEMETRY BOAT
   ↓
SMART TOW
   ↓
RETURN HOME
   ↓
MAPPING / RESEARCH PLATFORM
   ↓
ASSISTED AUTONOMY
```

---

## 24. Objetivo final

```text
MicroBoat
   │
   ├── RC Boat
   ├── Smart Tow Platform
   ├── Telemetry Boat
   ├── Sensor Platform
   ├── Mapping Boat
   ├── Research Platform
   └── Assisted Autonomous Boat
```

> Una embarcación suficientemente pequeña para viajar contigo, pero suficientemente instrumentada para comportarse como un vehículo robótico.

---

## BLACKMAMBA ENGINEERING

**Iyari Gomez Ing**

> Build it. Instrument it. Measure it. Improve it.
