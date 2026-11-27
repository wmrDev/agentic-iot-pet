# Hoja de banco · Junta del 28 de noviembre de 2026

Hoja imprimible para una sola sesión. Sigue el [orden de la junta](2026-11-27-plan-junta-eva-02.md#orden-de-la-junta-del-28-de-noviembre): 1) ESP32-S3 sola, 2) banco por componente, 3) autonomía, 4) acuerdo de demo. No reemplaza a [H01_BANCO.md](../hardware/H01_BANCO.md), [T04_PLACA.md](../pruebas/T04_PLACA.md) ni a [TRASPASO.md](../equipo/TRASPASO.md): resume sus comandos en el orden de la sesión.

Cada paso termina con una línea **Resultado observado**. Al terminar el paso, pasar ese resultado a un registro propio en `docs/pruebas/resultados/` con la [plantilla](../pruebas/PLANTILLA_RESULTADO.md). Lo que no se haga queda `NO EJECUTADA` con el motivo. Una compilación no es una prueba en placa.

Los pasos que escriben en la placa (respaldo aparte), la cablean o borran la NVS se ejecutan con acuerdo de los presentes en la junta.

## 0. Antes de empezar

### Revisión que se lleva al banco

| Dato | Valor |
|---|---|
| Rama y commit | `main` en **`1623ada`** (`1623adaee34dfa4333795924d96cae17905eba52`, unión del PR #39) |
| Compilación en CI | Ejecución [46034564664](https://github.com/wmrDev/agentic-iot-pet/actions/runs/46034564664), resultado correcto. Compiló los tres proyectos de firmware (`firmware`, `firmware/test`, `firmware/diagnostics/h01`) en `espressif/idf:v5.5.6`, además del job `gradle` |
| Qué no prueba la CI | Nada en placa. Compila H01 con los valores por defecto (todos los GPIO en `-1`) |
| Compilaciones locales previas | `b951b8f` y `b503c73` (Windows), `17fb5c6` (H01). Ninguna es `1623ada`: la primera compilación local de hoy se anota en el registro |
| Cambios de firmware después de `1623ada` | Ninguno hasta `3cfeff9` (`git diff --stat 1623ada..3cfeff9 -- firmware` vacío; los PR #40 a #44 son documentación). Compilar `main` actual da las mismas fuentes; anotar igual el commit usado |

Las tres imágenes salen del mismo commit:

| Proyecto | Carpeta | Para qué | Consola |
|---|---|---|---|
| `prueba_usb` | `firmware` | Diagnóstico USB (latido `seq`) **y** lógica de la mascota de T04. Es la misma imagen que T04 llama «firmware de producto» | UART0 primaria, USB Serial/JTAG secundaria |
| `h01_banco` | `firmware/diagnostics/h01` | Un módulo por carga | Solo USB Serial/JTAG |
| `pet_state_test` | `firmware/test` | 61 tests Unity (borra la NVS) | Solo USB Serial/JTAG |

```powershell
git fetch
git switch --detach 1623ada     # o: git switch main; git pull, y comprobar que HEAD es 1623ada
git log --oneline -1
```

- [ ] Commit comprobado en el PC del banco: `__________`
- [ ] Responsable que ejecuta: `__________` · Revisor: `__________`

### Activar ESP-IDF v5.5.6 en Windows

En cada terminal PowerShell nueva, **una** de las dos opciones:

**A. Instalación con EIM** (verificada por Maximiliano el 26-11, [ENTORNOS.md](../equipo/ENTORNOS.md)):

```powershell
. C:\Espressif\tools\Microsoft.v5.5.6.PowerShell_profile.ps1
```

**B. Instalación en `C:\Espressif\frameworks`** (equipo de Walther, 27-11). El Python 3.11 de `C:\Espressif\tools\idf-python` debe quedar **primero en el PATH antes** de ejecutar `export.ps1`; si no, el script toma otro Python del sistema:

```powershell
$env:PATH = "C:\Espressif\tools\idf-python\3.11.2;" + $env:PATH
. C:\Espressif\frameworks\esp-idf-v5.5.6\export.ps1
```

La carpeta `3.11.2` es la del equipo de Walther; en otro equipo, comprobar el nombre con `Get-ChildItem C:\Espressif\tools\idf-python`.

Comprobar en ambos casos:

```powershell
idf.py --version            # esperado: ESP-IDF v5.5.6
python --version            # esperado: Python 3.11.x
python -m esptool version
```

Si el compilador Xtensa falla con un *segfault* intermitente (visto el 27-11, [#37](https://github.com/wmrDev/agentic-iot-pet/issues/37)), repetir con `$env:CMAKE_BUILD_PARALLEL_LEVEL = "1"`.

- [ ] Opción usada (A/B) y versiones: `____________________________`

**Resultado observado:** ______________________________________________

### Material a la mano

- [ ] Placa ESP32-S3 SuperMini sola, sin jumpers a ningún módulo.
- [ ] Cable USB de **datos** (no solo carga).
- [ ] Multímetro (continuidad y tensión). Instrumento de corriente: `__________` (si no hay, E10 queda `NO EJECUTADA`).
- [ ] Módulos presentes hoy (marcar): TTP223 ☐ · LED + 330 Ω ☐ · OLED ☐ · INMP441 ☐ · MAX98357A ☐ · parlante ☐. Según el [inventario](../hardware/INVENTARIO.md), la compra del MAX98357A está por confirmar y la OLED figuraba en carrito.
- [ ] Adaptador USB-UART de 3,3 V (solo para leer UART0 en el paso 1.5). **No figura en el inventario.**
- [ ] iPhone 14 para fotos y vídeo del montaje (sin rostros ni pantallas ajenas).

---

## 1. ESP32-S3 sola

### 1.1 Puerto

```powershell
Get-CimInstance Win32_SerialPort | Select-Object DeviceID, Description
Get-PnpDevice -PresentOnly -Class Ports | Select-Object FriendlyName, InstanceId
```

El `InstanceId` contiene el VID:PID (`USB\VID_xxxx&PID_xxxx\...`). Se espera el VID de Espressif `303A`; **confirmarlo, no suponerlo**. En los comandos siguientes, `COMx` es el puerto detectado (Winderson vio `COM12`; puede cambiar).

- Puerto: `COM____` · VID:PID: `______:______`

Si el puerto desaparece solo: modo programación (mantener BOOT, pulsar y soltar RESET, soltar BOOT) y volver a listar.

**Resultado observado:** ______________________________________________

### 1.2 Identificación previa (sin escribir)

```powershell
python -m esptool --chip esp32s3 -p COMx flash_id
```

Anotar revisión del chip, tamaño de flash y la línea de características (PSRAM). Sirve para la pregunta abierta de [#37](https://github.com/wmrDev/agentic-iot-pet/issues/37) sobre PSRAM octal (GPIO33–37).

El [registro del 25-11](../pruebas/resultados/2026-11-25-T00-prueba-usb.md) ya informa, por esptool, «PSRAM integrada de 2 MB (AP_3v3)». Una PSRAM integrada de 3,3 V y 2 MB corresponde a la variante cuádruple, que no usa GPIO33–37; la octal suele ser de 1,8 V. **Confirmarlo hoy** con la línea de esta lectura antes de liberar esos pines. Si esptool no informa el tipo, anotarlo como no determinado.

- Chip / revisión: `__________` · Flash: `__________` · PSRAM: `__________`

**Resultado observado:** ______________________________________________

### 1.3 Respaldo de la flash original (si sigue en la placa)

Fuera del repositorio; el archivo no se sube a Git. Dos lecturas y comparación ([TRASPASO.md, paso 5](../equipo/TRASPASO.md#5-respaldar-el-firmware-original-antes-de-escribir)):

```powershell
New-Item -ItemType Directory -Force "$HOME\Respaldos"
python -m esptool --chip esp32s3 -p COMx read_flash 0 0x400000 "$HOME\Respaldos\esp32s3-original.bin"
python -m esptool --chip esp32s3 -p COMx read_flash 0 0x400000 "$HOME\Respaldos\esp32s3-original-2.bin"
Get-Item "$HOME\Respaldos\esp32s3-original*.bin" | Select-Object Name, Length   # 4194304 cada uno
certutil -hashfile "$HOME\Respaldos\esp32s3-original.bin" SHA256
certutil -hashfile "$HOME\Respaldos\esp32s3-original-2.bin" SHA256
```

**Parar** si una lectura falla, si un tamaño no es 4194304 bytes o si los SHA-256 difieren.

Restauración (solo si hace falta y con acuerdo; sobrescribe toda la flash):

```powershell
python -m esptool --chip esp32s3 -p COMx write_flash 0x0 "$HOME\Respaldos\esp32s3-original.bin"
```

- [ ] Tamaños iguales a 4194304 · [ ] SHA-256 iguales: `________________________________`
- [ ] No había firmware original que respaldar (motivo): `__________`

**Resultado observado:** ______________________________________________

### 1.4 Cargar PRUEBA_USB y monitor sin reinicio

No usar `erase-flash`. El `sdkconfig.defaults` ya fija `esp32s3` y 4 MB.

```powershell
cd firmware
idf.py build
idf.py -p COMx flash
```

Si se entró en modo programación a mano, pulsar RESET sin mantener BOOT. Luego:

```powershell
idf.py -p COMx monitor --no-reset
```

Sin `--no-reset`, el propio monitor reinicia el chip y el primer `seq=0` no cuenta como falla (anotarlo). Salir con `Ctrl+]`. Ante un pánico la placa se detiene (`CONFIG_ESP_SYSTEM_PANIC_PRINT_HALT`): copiar el texto y pulsar RESET.

Con todos los pines `Pet hardware` en `-1`, se espera además `PET_SENSORS: touch=0 led=0 mic=0` o equivalente (sin periféricos).

Registrar durante `____` minutos:

| Dato | Valor |
|---|---|
| Línea de arranque (`ESP-IDF=… chip=… revision=…`) | |
| `Physical flash size` | |
| `reset_reason` / `reset_name` del arranque | |
| `boot_count` | |
| ¿`seq` sube de uno en uno cada 2 s? | |
| ¿`seq` vuelve a 0? Cuántas veces y con qué `reset_name` | |
| ¿El puerto desaparece? Cada cuánto (antes: cada 2,5 s) | |
| VID:PID antes / durante / después de la caída (repetir el comando de 1.1) | |

`boot_count` sube en cada reinicio sin corte de energía y vuelve a 1 tras un encendido. Tabla de `reset_reason` en [TRASPASO.md, paso 8](../equipo/TRASPASO.md#8-validar).

**Resultado observado:** ______________________________________________

### 1.5 Si el USB se cae: leer UART0

Solo la imagen de `firmware` escribe en UART0 (las de H01 y Unity usan solo USB). Requiere el adaptador USB-UART de 3,3 V, que **no está en el inventario**: si no hay, este paso queda `NO EJECUTADA`.

- Pines TX/RX de UART0 según la serigrafía de la placa: TX = `GPIO____`, RX = `GPIO____` (confirmar en la serigrafía; no conectar por suposición).
- Conexión: TX de la placa → RX del adaptador, RX → TX, GND común. **No** unir el 3V3 o 5 V del adaptador si la placa ya está alimentada por USB.

```powershell
idf.py -p COMy monitor --no-reset     # COMy = puerto del adaptador; 115200 baudios por defecto
```

Comparar: si UART0 sigue mostrando `seq` creciente mientras el USB cae, la placa no se reinicia y el fallo está en el enlace USB. Si `seq` vuelve a 0, anotar `reset_name` y `boot_count`.

**Resultado observado:** ______________________________________________

### Condición para seguir al paso 2

- [ ] Arranque estable varios minutos, sin reinicios ni caída del puerto. **Si el USB sigue cayéndose, registrar el síntoma y no añadir periféricos** (orden de la junta).

---

## 2. Banco por componente (H01)

Un módulo por carga, en este orden. Pines: **ninguno viene configurado y no se deciden por esta hoja**. Se eligen en la junta mirando la serigrafía y se anotan en los espacios. GPIO4 (TOUCH) y GPIO5 (LED) son solo una propuesta sin aprobar.

El firmware de banco rechaza GPIO19 y GPIO20 (USB), GPIO26 a GPIO32 (flash y PSRAM) y los que no existen en el chip; avisa en GPIO33–37 (PSRAM octal) y en los de arranque 0, 3, 45 y 46. Evitar 45 y 46.

### Antes de energizar cada módulo (placa desconectada)

1. Foto del módulo con la serigrafía legible.
2. Continuidad de cada jumper, de pin de placa a pin de módulo.
3. Sin continuidad entre 3V3 y GND, ni entre 5V y GND.
4. Solo ese módulo conectado.

### Alimentación

Los GPIO del ESP32-S3 no toleran 5 V.

| Módulo | Alimentación | Obligatorio |
|---|---|---|
| TTP223 | 3V3 de la placa, nunca 5 V | GND común; I/O directo al GPIO |
| LED | Desde el GPIO | 330 Ω en serie; cátodo a GND |
| OLED 128×64 I2C | 3V3 de la placa, nunca 5 V | GND común (con 5 V los pull-up llevarían SDA/SCL a 5 V) |
| INMP441 | 3V3 de la placa, nunca 5 V (máx. 3,6 V) | GND común; **L/R a GND** |
| MAX98357A | Según su módulo: 5 V del USB o 3V3 (anotar cuál y por qué) | GND común; parlante solo entre las salidas, ninguna a GND |

### Condiciones de parada (para todo el paso 2)

Desconectar el USB y anotar si ocurre cualquiera:

- módulo o placa calientes, olor o humo;
- el puerto desaparece al conectar el módulo y no aparecía sin él;
- reinicio con `reset_name=BROWNOUT` o `PWR_GLITCH`, o reinicios repetidos;
- el firmware rechaza el pin (`pin not configured`, `does not exist`, `used by the USB console`, `in-package flash/PSRAM`);
- continuidad entre 3V3 y GND o entre 5V y GND.

### Cómo cargar cada prueba

```powershell
cd firmware\diagnostics\h01
idf.py menuconfig
idf.py build
idf.py -p COMx flash monitor
```

En `menuconfig` → **H01 bench** → *Test to run*, elegir la prueba y escribir solo los pines de ese módulo. Los pines quedan en `firmware\diagnostics\h01\sdkconfig`, que Git ignora: anotarlos aquí. Al arrancar, copiar el encabezado `Pins: …` al registro. Nombre del registro: `AAAA-MM-DD-H01-<modulo>.md`.

`flash monitor` reinicia la placa al abrir el monitor; aquí es aceptable, pero ese reinicio no cuenta como falla.

### 2.1 TTP223 y LED

- *Test to run*: **TTP223 touch and LED**
- *Touch and LED* → `TTP223 I/O GPIO` = `GPIO____` · `LED GPIO (through resistor)` = `GPIO____`
- Esperado: un par `press`/`release` por toque con su duración en ms; el LED sigue al contacto; ningún pulso sin tocar durante 60 s.

| Gesto | Duraciones medidas (ms) |
|---|---|
| Toque breve (×5) | |
| Doble toque (×5, separación) | |
| Toque sostenido (×3) | |
| Pulsos sin tocar en 60 s | |

Estas duraciones calibran las ventanas provisionales 300/350/800 ms de `firmware/main/gestures.h` (T04).

**Resultado observado:** ______________________________________________

### 2.2 Escaneo I2C y OLED

1. *Test to run*: **I2C bus scan (OLED address)** · *OLED (I2C)* → `SDA GPIO` = `GPIO____` · `SCL GPIO` = `GPIO____` · `I2C SCL speed (Hz)` = 100000.
   Esperado: `ACK at 0x..`. Dirección real: `0x____`.
2. *Test to run*: **OLED border and two eyes** · `OLED 7-bit address` = la del escaneo · *Controller to try* = **SH1106 (132-column RAM, offset 2)**.
   Esperado: borde de 1 px en los cuatro lados y dos ojos.
3. Si falta o sobra una franja de dos columnas, o la pantalla queda negra aunque haya ACK: repetir con **SSD1306 (128-column RAM, offset 0)**.
4. Subir a 400000 Hz solo después de que funcione a 100 kHz, y anotarlo.

| Intento | Controlador | Velocidad | Bordes visibles | Observación |
|---|---|---|---|---|
| 1 | SH1106 | 100 kHz | | |
| 2 | SSD1306 | 100 kHz | | |

- Controlador confirmado: `__________` (no deducirlo del nombre comercial).

**Resultado observado:** ______________________________________________

### 2.3 INMP441

- *Test to run*: **INMP441 relative level**
- *Microphone INMP441 (I2S RX)* → `SCK / BCLK GPIO` = `GPIO____` · `WS GPIO` = `GPIO____` · `SD (data out of the mic) GPIO` = `GPIO____`
- Esperado: cinco líneas por segundo con `rms`, `peak`, `dc`, `zeros`. En silencio `rms` bajo y estable; con palmada a distancia fija `peak` sube. `zeros` igual al total = cableado o alimentación incorrectos. Son niveles relativos (0 a 1), **no dB SPL**. Las muestras no se guardan ni se transmiten.

| Condición | `rms` | `peak` | `dc` | `zeros` |
|---|---|---|---|---|
| Silencio 60 s | | | | |
| Palmada a ____ cm | | | | |

**Resultado observado:** ______________________________________________

### 2.4 MAX98357A con parlante

- *Test to run*: **MAX98357A test tone**
- *Amplifier MAX98357A (I2S TX)* → `BCLK GPIO` = `GPIO____` · `LRC GPIO` = `GPIO____` · `DIN GPIO` = `GPIO____` · `Tone amplitude, percent of full scale` = 10 (por defecto).
- Alimentación del módulo usada: `____ V` desde `__________`.
- Esperado: tono de 400 Hz, 1 s sí y 1 s no, limpio y sin reinicios. Si se reinicia al sonar, anotar `reset_reason` y parar.

**Resultado observado:** ______________________________________________

### Pines comprobados hoy

| Señal | GPIO | Continuidad medida | Prueba que lo usó |
|---|---|---|---|
| TTP223 I/O | | | |
| LED | | | |
| SDA | | | |
| SCL | | | |
| Mic SCK/BCLK | | | |
| Mic WS | | | |
| Mic SD | | | |
| Amp BCLK | | | |
| Amp LRC | | | |
| Amp DIN | | | |

Hasta que los cinco módulos tengan registro y el equipo los apruebe (`DECISIONES.md`, y `docs/hardware/PINOUT.md` que crea T03), estos pines son «comprobados en banco», no el pinout aprobado.

---

## 3. Autonomía (solo si el banco está estable)

### 3.1 Antes de borrar la NVS

Los tests Unity borran la NVS (`test_settings.c`) y cambian una hora la hora civil, que luego restauran. Respaldar la configuración de banco: copiar fuera del repositorio `firmware\sdkconfig` (si ya tiene los pines `Pet hardware`) y `firmware\diagnostics\h01\sdkconfig`. Git ignora ambos archivos.

```powershell
New-Item -ItemType Directory -Force "$HOME\Respaldos\sdkconfig-28-11"
Copy-Item firmware\diagnostics\h01\sdkconfig "$HOME\Respaldos\sdkconfig-28-11\h01.sdkconfig"
Copy-Item firmware\sdkconfig "$HOME\Respaldos\sdkconfig-28-11\firmware.sdkconfig"   # si existe
```

- [ ] Respaldo hecho · [ ] Acuerdo de borrar la NVS de esta placa

### 3.2 Tests Unity en la placa (T04)

```powershell
cd firmware
idf.py -C test set-target esp32s3
idf.py -C test -p COMx flash monitor
```

- Esperado: resumen final de Unity `61 Tests 0 Failures`. Copiar las líneas finales al registro `AAAA-MM-DD-T04-unity.md`.
- Si falla alguno: anotar cuál, el mensaje y la revisión. **No corregir el test para que pase.**
- `test_env_esp_timer_ignores_settimeofday` prueba el entorno, no la lógica de sesión.

- Tests / fallos / ignorados: `____ / ____ / ____`

**Resultado observado:** ______________________________________________

### 3.3 Volver a cargar el firmware de producto

```powershell
cd firmware
idf.py menuconfig
idf.py build
idf.py -p COMx flash monitor
```

En `menuconfig` → **Pet hardware**, escribir solo los pines comprobados en el paso 2: `TTP223 I/O GPIO`, `Listening LED GPIO`, `INMP441 SCK / BCLK GPIO`, `INMP441 WS GPIO`, `INMP441 SD GPIO`, `MAX98357A BCLK GPIO`, `MAX98357A LRC GPIO`, `MAX98357A DIN GPIO`. Dejar **Sound capture calibrated on this board** desactivado. Esta imagen no maneja la OLED.

Comprobar en el log (pendiente de [#37](https://github.com/wmrDev/agentic-iot-pet/issues/37)):

- [ ] `PET_SENSORS: touch=… led=… mic=…` coincide con lo conectado.
- [ ] Un pin repetido o reservado (19/20, 22–32) se rechaza al arrancar. Probarlo solo cambiando `menuconfig`, sin cablear nada.
- [ ] Un error del micrófono (por ejemplo, con el INMP441 desconectado) no reinicia la placa por el watchdog de tareas (`reset_name` distinto de `TASK_WDT`).

**Resultado observado:** ______________________________________________

### 3.4 Sesión sin red

El firmware de `1623ada` no incluye Wi-Fi: la comprobación «sin red» confirma que la sesión no depende de la app, no un corte de enlace (eso es E09). Procedimiento completo en [T04_PLACA.md, paso 3](../pruebas/T04_PLACA.md#3-gestos-y-sesión-p02-p12-parcial-p15-parcial).

| Caso | Acción | Esperado en el monitor | Observado |
|---|---|---|---|
| Inicio | Toque breve | `study … phase=1`, y un sonido corto si el MAX98357A está configurado en `Pet hardware` (sin amplificador, `Amplifier not configured: sounds disabled` al arrancar) | |
| Pausa | Doble toque | `phase=2` | |
| Reanudación | Doble toque otra vez | `phase=1` | |
| Mantener | > 0,8 s y soltar | `voice requested`, LED encendido; `voice released`, LED apagado; la sesión no cambia | |
| Toque intermedio | ~0,5 s | Nada | |
| Reinicio con sesión | Sesión iniciada y RESET | `Previous session was interrupted by a reboot`, `phase=4` | |
| Fin | 25 min, o 60 000 ms bajando temporalmente `settings_defaults()` (anotarlo) | `phase=3` y sonido de fin | |

Al terminar, la placa queda con el firmware de producto de `1623ada` y los pines anotados.

**Resultado observado:** ______________________________________________

La calibración del umbral de sonido y la prueba A01 (pasos 4 y 5 de T04_PLACA) quedan para otra sesión salvo que sobre tiempo.

---

## 4. Acuerdo de demo

### Qué fichas EVA_02 pueden ejecutarse mañana

Fichas en [FICHAS.md](../pruebas/eva02/FICHAS.md). Registro `AAAA-MM-DD-EVA02-Exx-<dispositivo>.md`.

| Ficha | ¿Mañana? | Motivo |
|---|---|---|
| E01 · Instalación y permisos | Sí | Solo Android y PC; misma APK con SHA-256 en INFINIX y Galaxy |
| E12 · Banco de la placa | Sí, si los pasos 1 y 2 salen estables | Placa sola. Pinout no verificado se escribe «pendiente». Con esta imagen no se puede comprobar «tiempo hasta la primera expresión» ni «OLED centrado»: anotar `NO EJECUTADA` en esa parte |
| E10 · Consumo | Parcial | C1 (reposo) solo si hay instrumento de corriente seguro; C5 (sonido) quizá con el amplificador en `Pet hardware` (por confirmar). C2, C3 (Wi-Fi y app), C4 (OLED en producto) y C6 no se pueden provocar con `1623ada` |
| E02–E09, E11 | No | Integradas con Android: necesitan el emparejamiento de T02 y el enlace HTTPS de T03 (y la telemetría para E03, E09 y E11) |

### Decisiones de la junta

- [ ] Repartir la explicación de la demo.
- [ ] Lo que se muestra en físico: `____________________________________________`
- [ ] Lo que queda pendiente y su dependencia: `____________________________________________`
- [ ] Evidencia del INFINIX y del Galaxy revisada (revisión, SHA-256 de la APK, versión de Android).
- [ ] Quién sube cada registro a `docs/pruebas/resultados/` y lo resume en [#29](https://github.com/wmrDev/agentic-iot-pet/issues/29).

**Resultado observado:** ______________________________________________

---

## Puntos inciertos de esta hoja

- El VID:PID esperado (`303A`) y que `flash_id` informe el tipo de PSRAM están por confirmar en la placa.
- Los pines de UART0 se confirman en la serigrafía; no hay adaptador USB-UART en el inventario.
- La alimentación del MAX98357A (5 V o 3V3) depende del módulo recibido.
- Que el amplificador suene con la imagen de producto para la condición C5 de E10 no está comprobado.
