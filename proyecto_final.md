PROYECTO FINAL: SISTEMA DE CONTROL DE SALA

Materia: Estructuras Computacionales

Docente: Samuel Andrés Cifuentes Muñoz

Integrantes: María de los Ángeles Prieto Ortega - Mariana Zuluaga Yepes 

Diciembre de 2025

MATERIALES

Microcontrolador STM32L476RG

Pantalla OLED SSD1306 (128×64)

Teclado matricial 4×4

Módulo WiFi ESP-01

Sensor de temperatura DHT11

Resistencias

Jumpers

OBJETIVO: 

Diseñar un sistema capaz de controlar el acceso a una habitación mediante contraseña, visualizar información en pantalla, supervisar la temperatura y activar un ventilador de forma automática o manual. Además, se incluye un modo de emergencia y envío de alertas mediante el módulo ESP-01.

PROCEDIMIENTO:

1. Inicialización general

En la función room_control_init() se configuran todos los módulos necesarios:

Estado inicial del sistema: LOCKED (bloqueado).

Contraseña por defecto: 2222.

Se limpia el buffer donde se guarda la contraseña que ingresa el usuario.

Se configura el pin encargado de la puerta.

Se inicia el PWM para controlar el ventilador con un duty cycle inicial de 0%.


2. Funcionamiento del sistema por estados

El sistema trabaja como una máquina de estados, lo que organiza el comportamiento en etapas claras:

Estado 1: ROOM_STATE_LOCKED (Bloqueado)

Se muestra en pantalla “SISTEMA BLOQUEADO”.

La puerta permanece cerrada.

Cuando el usuario presiona cualquier número, el sistema pasa al estado de ingresar contraseña.


Estado 2: ROOM_STATE_INPUT_PASSWORD (Ingreso de contraseña)

Se muestra la palabra “CLAVE” y debajo los asteriscos de la contraseña digitada.

El usuario puede borrar con la tecla C.

Si ingresa 4 dígitos:

Si coincide con la contraseña → pasa a UNLOCKED.

Si no coincide → pasa a ACCESS_DENIED.

Si pasan 10 segundos sin escribir → vuelve a LOCKED.


Estado 3: ROOM_STATE_UNLOCKED (Acceso concedido)

La pantalla muestra:
"ACCESO OK"

Temperatura actual
Nivel del ventilador (OFF, BAJO, MEDIO, ALTO)
Modo: MANUAL o AUTO

Se permite:

Cambiar el modo del ventilador
Control manual con teclas 0-3
Volver a modo automático con A
Bloquear el sistema con B
Entrar en emergencia con D


La puerta permanece abierta.


Estado 4: ROOM_STATE_ACCESS_DENIED (Acceso denegado)

En pantalla aparece “ACCESO DENEGADO”.

Se envía un mensaje al ESP-01:
 "ALERTA: Acceso denegado"

Pasados 3 segundos, vuelve al estado LOCKED.


Estado 5: ROOM_STATE_EMERGENCY (Emergencia)

La pantalla muestra:

“EMERGENCIA”
“SALGA”

La puerta se abre sola.

El ventilador se pone al 100%.

Se envía mensaje al ESP-01:
 "ALERTA: Emergencia activada"

Para salir de emergencia se presiona la tecla #.

3. Partes principales del código

room_control_init()

Configura los valores iniciales.

Deja el sistema en estado bloqueado.

Enciende PWM con valor inicial 0%.

room_control_update()

Revisa el estado actual y ejecuta acciones:
 bloquear puerta, mostrar mensajes, manejar tiempo de espera.

Luego actualiza el display, el ventilador y la puerta.





room_control_process_key()

Reacciona inmediatamente cuando se presiona una tecla.

Según el estado ejecuta acciones específicas:
Agregar dígitos, borrar, cambiar modo ventilador, activar emergencia.

room_control_update_fan()

Traduce el nivel del ventilador a un valor PWM real.

room_control_update_door()

Activa o desactiva un pin para abrir/cerrar.

room_control_update_display()

Muestra el estado correspondiente en la pantalla OLED.

room_control_calculate_fan_level()

Niveles automáticos según temperatura:
OFF,  BAJO, MEDIO, ALTO.

4. Funcionamiento del teclado

El teclado matricial envía un carácter por cada tecla presionada, y según el estado en el que se encuentre el sistema, se toman diferentes acciones:



Tabla 1. Funcionamiento del teclado matricial 4x4.

Se usan los pines 

Filas (Outputs del STM32): PC0 – ROW0, PC1 – ROW1, PC2 – ROW2 y PC3 – ROW3.

Columnas (Inputs con Pull-up): PC4 – COL0, PC5 – COL1, PC6 – COL2 y PC7 – COL3.

5. Control del ventilador 

Automático: 
Se calcula el nivel según la temperatura:

< 25°C → OFF
25–28°C → BAJO
28–31°C → MEDIO
> 31°C → ALTO

Manual: 

0 → OFF
1 → BAJO
2 → MEDIO
3 → ALTO

Se usa el pin PA6 – TIM3_CH1

Duty cycle usado: 0–99

6. Control de la puerta:

Los estados que abren la puerta son:

UNLOCKED
EMERGENCY

Los que la cierran:

LOCKED

ACCESS_DENIED

Se usa el pin PA4 – GPIO Output 

Nivel lógico LOW → Puerta cerrada, HIGH → Puerta abierta



7. Pantalla OLED SSD1306

La pantalla se actualiza únicamente cuando display_update_needed = true. 

Esto reduce los parpadeos y hace el sistema más eficiente.

Se usan los pines PB6 – I2C1_SCL (señal del reloj del bus) y PB7 – I2C1_SDA (línea de datos del bus)

Comunicación con ESP-01

El módulo ESP-01 recibe mensajes simples por UART.
 Ejemplos enviados:

"ALERTA: Acceso denegado\r\n"
"ALERTA: Emergencia activada\r\n"

Estos mensajes permiten que el sistema pueda interactuar con un servidor o una aplicación externa.

Se usan los pines PB10 – USART3_TX (enviar) y PB11 – USART3_RX (recibir)

Optimizaciones aplicadas 

Uso de PWM ya inicializado: El PWM se configura una vez y luego solo se actualiza el valor.

Actualización de pantalla solo cuando cambia algo: Evita parpadeos y libera CPU.

Temperatura con control automático eficiente: Solo recalcula el nivel del ventilador cuando cambia realmente.

Switch organizado por estados: Eficiente y fácil de extender.









Resultados: 

Figura 1. Circuito montado. 

Figura 2. Diagrama 

Figura 3. Control de sala funcionando. 

Observaciones: 

La arquitectura modular facilitó la organización del código y permitió encontrar errores más rápido, ya que cada parte del sistema se pudo probar por separado. Esto hizo el proceso de desarrollo más claro y manejable.

La máquina de estados mostró ser fundamental para mantener el sistema estable, y el uso de display_update_needed ayudó a optimizar el refresco de la pantalla, evitando operaciones innecesarias.

El teclado 4×4 presentó sensibilidad al ruido eléctrico, generando pulsaciones repetidas ocasionales. La implementación de una rutina adecuada de debounce redujo en gran parte este problema.

El sistema del ventilador funcionó bien en modo manual y automático, pero se observó que el usuario puede activar el modo manual sin querer. Sería útil agregar una indicación visual más evidente en futuras mejoras.

El modo de emergencia respondió correctamente, aunque sería conveniente añadir una confirmación adicional para evitar activaciones por error. Esto incrementaría la seguridad del usuario.

Finalmente, se evidenció que el teclado es un componente crítico: cualquier falla en su lectura afecta directamente el funcionamiento general del sistema.

Conclusiones:

El microcontrolador STM32 permitió controlar la pantalla OLED, el teclado, el ventilador y la puerta sin conflictos, logrando una interacción clara y confiable entre el usuario y el sistema.

La gestión del ventilador en modo automático y manual respondió correctamente a los cambios de temperatura, y la visualización en pantalla facilita al usuario entender cada estado del sistema.

El modo de emergencia mostró la importancia de prever situaciones críticas, permitiendo activar acciones inmediatas como abrir la puerta y forzar el ventilador al máximo.

En general, el proyecto cumplió los objetivos planteados y evidenció la importancia del diseño modular, logrando un sistema funcional, seguro y fácil de usar, con posibilidad de ampliarse en el futuro.







# Imágenes añadidas

![WhatsApp Image 2025-12-11 at 11.47.22 AM.jpeg](/mnt/data/WhatsApp Image 2025-12-11 at 11.47.22 AM.jpeg)

![WhatsApp Image 2025-12-11 at 11.16.13 AM.jpeg](/mnt/data/WhatsApp Image 2025-12-11 at 11.16.13 AM.jpeg)

![WhatsApp Image 2025-12-11 at 9.57.18 AM.jpeg](/mnt/data/WhatsApp Image 2025-12-11 at 9.57.18 AM.jpeg)

![WhatsApp Image 2025-12-11 at 9.55.34 AM.jpeg](/mnt/data/WhatsApp Image 2025-12-11 at 9.55.34 AM.jpeg)
