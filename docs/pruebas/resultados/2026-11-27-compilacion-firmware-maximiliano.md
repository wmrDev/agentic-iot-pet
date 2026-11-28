# Registro de prueba: compilación local de los tres proyectos de firmware

- ID de prueba / tarea: preparación de la junta del 28-11 ([guía](../../entrega/2026-11-28-preparacion-equipo.md#maximiliano--banco-pruebas-físicas-y-consumo), tarea «Entorno listo en tu PC»). Parte de H01 ([#7](https://github.com/wmrDev/agentic-iot-pet/issues/7)).
- Fecha, hora y responsable: 2026-11-27; PC de Maximiliano, ejecutado por su agente (Claude).
- Revisor: pendiente.
- Commit o revisión exacta: `main` en `3cfeff9` (unión del PR #44). Las fuentes de `firmware/` son idénticas a `1623ada`, la revisión de la [hoja de banco](../../entrega/2026-11-28-hoja-banco.md): `git diff --stat 1623ada..3cfeff9 -- firmware` sale vacío.
- Entorno: compilación en computador; nada se cargó en la placa.
- Hardware, sistema operativo y versiones: Windows 11 Home Single Language 26H2, compilación 26300.10075. ESP-IDF v5.5.6 instalado con EIM y activado con `. C:\Espressif\tools\Microsoft.v5.5.6.PowerShell_profile.ps1` (opción A de la hoja). Python 3.11.9 del entorno de ESP-IDF, Xtensa GCC 14.2.0 (`esp-14.2.0_20260121`), CMake 3.30.2 y Ninja 1.12.1.
- Condiciones iniciales, red, alimentación y permisos: clon limpio en `main`; carpetas de compilación y `sdkconfig` nuevas, fuera del repositorio, con los valores por defecto (todos los GPIO en `-1`; H01 con la prueba *TTP223 touch and LED*).
- Comando o procedimiento reproducible, desde cada carpeta del proyecto:

  ```powershell
  idf.py -B <carpeta de salida> -D SDKCONFIG=<archivo sdkconfig nuevo> build
  ```

- Resultado esperado: los tres proyectos compilan para `esp32s3` sin errores.
- Resultado observado:

  | Proyecto | Carpeta | Código | Avisos | Imagen | Tamaño | Libre | Tiempo |
  |---|---|---|---|---|---|---|---|
  | Producto (diagnóstico USB + T04) | `firmware` | 0 | 0 | `prueba_usb.bin` | 0x4a9f0 (305 648 B) | 71 % | 4 min |
  | Tests Unity (61) | `firmware/test` | 0 | 0 | `pet_state_test.bin` | 0x35420 (218 144 B) | 79 % | 4 min |
  | Banco H01 | `firmware/diagnostics/h01` | 0 | 0 | `h01_banco.bin` | 0x2da30 (186 928 B) | 82 % | 7 min |

  SHA-256 de las imágenes generadas en este PC (sirven para identificar lo que se cargue el sábado; no se espera que coincidan con otra compilación):

  | Imagen | SHA-256 |
  |---|---|
  | `prueba_usb.bin` | `8F6B5956A78C0EEC47089A4FF180318084D6BF2BC550ADF8E9768D1008A56428` |
  | `pet_state_test.bin` | `865840D96C9F1577E92E5F036C26A30EE931BDCF50781E71B0E1B655C1B2C0F0` |
  | `h01_banco.bin` | `E30C3384FD1DFD4CE66E68EA70EFF01F9CC7BAE94DFCC7C152E6A58DB6887C25` |

  No apareció el *segfault* intermitente de GCC descrito en [#37](https://github.com/wmrDev/agentic-iot-pet/issues/37); no hizo falta `CMAKE_BUILD_PARALLEL_LEVEL=1`.

- Estado: NO EJECUTADA en hardware. Compilación correcta; no equivale a una prueba física.
- Evidencia: este registro. Los registros completos de compilación quedaron en el PC y no se suben.
- Defecto asociado o ajuste acordado: ninguno.
- Qué no demuestra esta prueba: carga en la placa, arranque, que los 61 tests pasen, funcionamiento de periféricos, las otras cuatro configuraciones de H01 (Walther las compiló en `17fb5c6`; la CI solo compila la configuración por defecto) ni compilaciones con pines configurados.
