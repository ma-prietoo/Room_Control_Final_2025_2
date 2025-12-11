PROYECTO FINAL: SISTEMA DE CONTROL DE SALA

Materia: Estructuras Computacionales

Docente: Samuel Andrés Cifuentes Muñoz

Integrantes: María de los Ángeles Prieto Ortega - Mariana Zuluaga Yepes 

Diciembre de 2025

## MATERIALES

- Microcontrolador STM32L476RG
- Pantalla OLED SSD1306 (128×64)
- Teclado matricial 4×4
- Módulo WiFi ESP-01
- Sensor de temperatura DHT11
- Resistencias
- Jumpers

## OBJETIVO

Diseñar un sistema capaz de controlar el acceso a una habitación mediante contraseña, visualizar información en pantalla, supervisar la temperatura y activar un ventilador de forma automática o manual. Además, se incluye un modo de emergencia y envío de alertas mediante el módulo ESP-01.

## PROCEDIMIENTO

### 1. Inicialización general

En la función `room_control_init()` se configuran todos los módulos necesarios:

- Estado inicial del sistema: **LOCKED** (bloqueado).
- Contraseña por defecto: **2222**
- Se limpia el buffer donde se guarda la contraseña ingresada.
- Se configura el pin encargado de la puerta.
- Se inicia el PWM del ventilador (duty cycle 0%).

---

### 2. Funcionamiento del sistema por estados

El sistema funciona como una máquina de estados:

---

#### **Estado 1: ROOM_STATE_LOCKED (Bloqueado)**

- Se muestra “SISTEMA BLOQUEADO”.
- La puerta permanece cerrada.
- Con cualquier número presionado → pasa a ingreso de contraseña.

---

#### **Estado 2: ROOM_STATE_INPUT_PASSWORD (Ingreso de contraseña)**

- Se muestra “CLAVE:” con asteriscos.
- Tecla C → borrar.
- Si ingresa 4 dígitos:
  - Correcta → pasa a **UNLOCKED**
  - Incorrecta → pasa a **ACCESS_DENIED**
- Inactividad 10 segundos → vuelve a **LOCKED**

---

#### **Estado 3: ROOM_STATE_UNLOCKED (Acceso concedido)**

La pantalla muestra:

- ACCESO OK
- Temperatura actual
- Estado del ventilador: OFF / BAJO / MEDIO / ALTO
- Modo: MANUAL o AUTO

Permite:

- Cambiar modo del ventilador  
- Control manual con 0-3  
- Auto con A  
- Bloqueo con B  
- Emergencia con D  

Puerta abierta.

---

#### **Estado 4: ROOM_STATE_ACCESS_DENIED (Acceso denegado)**

Pantalla: “ACCESO DENEGADO”

Se envía por ESP-01:

`ALERTA: Acceso denegado`

Después de 3 segundos vuelve a LOCKED.

---

#### **Estado 5: ROOM_STATE_EMERGENCY (Emergencia)**

Pantalla:

- EMERGENCIA  
- SALGA  

Acciones automáticas:

- Puerta abierta  
- Ventilador al 100%  
- Mensaje al ESP-01: `ALERTA: Emergencia activada`  

Salir con tecla `#`

---

### 3. Partes principales del código

- `room_control_init()`
- `room_control_update()`
- `room_control_process_key()`
- `room_control_update_fan()`
- `room_control_update_door()`
- `room_control_update_display()`
- `room_control_calculate_fan_level()`

---

### 4. Funcionamiento del teclado

| Tecla | Acción |
|-------|--------|
| 0–9 | Digitar clave / controlar ventilador |
| A | Activar modo automático |
| B | Bloquear sistema |
| C | Borrar |
| D | Emergencia |
| 0–3 | Niveles manuales del ventilador |
| # | Salir de emergencia |

Pines:

- Filas (STM32): PC0–PC3  
- Columnas: PC4–PC7

---

### 5. Control del ventilador 

**Modo automático:**

- < 25°C → OFF  
- 25–28°C → BAJO  
- 28–31°C → MEDIO  
- > 31°C → ALTO  

**Modo manual:**

- 0 → OFF  
- 1 → BAJO  
- 2 → MEDIO  
- 3 → ALTO  

PWM: PA6 – TIM3_CH1 (duty cycle 0–99)

---

### 6. Control de la puerta:

Abre en:

- UNLOCKED  
- EMERGENCY  

Cierra en:

- LOCKED  
- ACCESS_DENIED  

Pin: PA4 (GPIO Output)

---

### 7. Pantalla OLED SSD1306

Se actualiza solo cuando `display_update_needed = true`.

I2C:

- PB6 – SCL  
- PB7 – SDA  

---

### 8. Comunicación con ESP-01

Por UART.

Ejemplos enviados:


Pines:

- PB10 – TX  
- PB11 – RX  

---

## Optimizaciones aplicadas

- PWM inicializado solo una vez.  
- Actualización de pantalla eficiente.  
- Control automático del ventilador optimizado.  
- Máquina de estados modular.

---

## Resultados
![WhatsApp Image 2025-12-11 at 11.16.13 AM](WhatsApp%20Image%202025-12-11%20at%2011.16.13%20AM.jpeg)
![WhatsApp Image 2025-12-11 at 9.57.18 AM](WhatsApp%20Image%202025-12-11%20at%209.57.18%20AM.jpeg)

![WhatsApp Image 2025-12-11 at 9.55.34 AM](WhatsApp%20Image%202025-12-11%20at%209.55.34%20AM.jpeg)

![WhatsApp Image 2025-12-11 at 11.47.22 AM](WhatsApp%20Image%202025-12-11%20at%2011.47.22%20AM.jpeg)


---

## Observaciones

- El diseño modular facilitó la depuración.  
- La máquina de estados aportó estabilidad.  
- Se mitigó el ruido del teclado con debounce.  
- El modo manual puede activarse accidentalmente.  
- El modo emergencia funciona bien pero podría requerir confirmación adicional.  
- El teclado es crítico para el funcionamiento.

---

## Conclusiones

- El STM32 permitió integrar todos los periféricos eficientemente.
- El ventilador respondió correctamente en ambos modos.
- El modo emergencia es fundamental para situaciones críticas.
- El sistema es funcional, seguro y escalable.



