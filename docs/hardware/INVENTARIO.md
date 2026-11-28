# Inventario y validación física

Estado basado en lo confirmado por Walther; **pedido, carrito y recepción física son estados distintos**. No incluye precios.

| Elemento | Estado conocido | Uso |
|---|---|---|
| ESP32-S3 SuperMini | Adquirida | Firmware, WiFi, sensores y salidas |
| INMP441 | Adquirido | Intensidad relativa del sonido por I2S |
| Parlante 3 W / 4 Ω | Adquirido | Sonidos mediante amplificador |
| SX1308 | Adquirido; sin uso previsto | Reservado; alimentación actual por USB |
| MAX98357A | Compra separada pendiente de confirmar | Amplificador I2S para el parlante |
| OLED blanca 1,3 pulgadas, 128×64 | Seleccionada en carrito Afel | Expresiones; verificar controlador SH1106 |
| Protoboard 400 puntos, LED rojo, tira de pines | Carrito Afel preparado | Montaje y señalización |
| Jumpers macho–macho 20 cm | Carrito Afel preparado | Conexiones del prototipo |
| Alicate cortante, fuente USB-C 5 V / 3 A | Carrito Afel preparado | Montaje y alimentación |
| TTP223, jumpers macho–hembra y resistencias | Pedido MechatronicStore confirmado | Contacto, señales y LED/micrófono |
| Capacitores, cable paralelo y cable USB de datos | Pedido MechatronicStore confirmado | Alimentación, audio y programación |
| Caja, multímetro, pila 9 V y kit de soldadura | Pedido MechatronicStore confirmado | Protección, medición y montaje |

La caja recomendada fue 158×90×60 mm; confirmar la variante efectivamente recibida. No hay motor de vibración, batería interna ni zumbador en el diseño vigente.

## Registro de recepción

Por completar al montar: fotografía, marca/serigrafía, variante, fecha de recepción, responsable y prueba individual. Si la lista de pedido real difiere, actualizarla con evidencia y revisar el software afectado.

Avance al 25-11 (parcial): ESP32-S3 SuperMini identificada (revisión v0.2, flash 4 MB, PSRAM 2 MB) y soldada; TTP223 soldado y medido; LED con 330 Ω probado con alimentación directa. Ver [mediciones](../pruebas/resultados/2026-11-25-H01-preliminar.md) y [placa](../pruebas/resultados/2026-11-25-T00-prueba-usb.md). Faltan fotografías, serigrafía y el resto de componentes. Pinout previsto, sin confirmar: TOUCH en GPIO4, LED en GPIO5.

## Confirmación de recepción · junta del 28-11

Estado al 27-11: **el material lo tiene Winderson** y Maximiliano no lo ha visto. Solo se marca «confirmado» lo que tiene evidencia; el resto se completa en la junta, mirando cada pieza. Lo que no aparezca queda «no recibido», no «adquirido».

| Elemento | Evidencia antes de la junta | ¿Presente el 28-11? | Variante o serigrafía | Consecuencia si falta |
|---|---|---|---|---|
| ESP32-S3 SuperMini | Confirmada: identificada con esptool y soldada (25-11, Winderson) | | | Sin banco ni pruebas en placa |
| Cable USB-C de datos | Confirmado: se usó para leer la placa el 25-11 | | | Sin carga ni monitor |
| TTP223 | Confirmado: soldado y medido (25-11) | | | Sin gestos (H01 2.1, T04) |
| LED + resistencia de 330 Ω | Confirmado: probado con alimentación directa (25-11) | | | Sin indicador de escucha |
| OLED 1,3" 128×64 | Por confirmar (figuraba en carrito) | | Controlador: SH1106 / SSD1306 / no determinado | Sin H01 2.2; la cara queda en simulación |
| INMP441 | Por confirmar (figuraba como adquirido) | | | Sin H01 2.3 ni reacción acústica |
| MAX98357A | Por confirmar (compra separada sin confirmar) | | | Sin H01 2.4, sonidos ni A01 |
| Parlante 3 W / 4 Ω | Por confirmar (figuraba como adquirido) | | | Igual que el MAX98357A |
| Protoboard y jumpers | Por confirmar (carrito y pedido) | | | Montaje solo soldado |
| Fuente USB-C 5 V / 3 A | Por confirmar (carrito) | | | Alimentar desde el PC; anotarlo en E10 |
| Caja | Por confirmar (pedido; recomendada 158×90×60 mm) | | Medidas reales | Sin efecto en el banco |

### Instrumentos

| Instrumento | Evidencia antes de la junta | ¿Presente el 28-11? | Marca y modelo | Consecuencia si falta |
|---|---|---|---|---|
| Multímetro | Hay al menos uno: Winderson midió tensiones el 25-11 (marca y modelo sin registrar) | | | Sin continuidad ni tensiones: no se energiza ningún módulo |
| Escala de corriente del multímetro (10 A) | Por confirmar | | | E10 de corriente queda `NO EJECUTADA` |
| Medidor USB en línea | No figura en ningún pedido | | | E10 solo con el multímetro en serie, o `NO EJECUTADA` |
| Adaptador USB-UART de 3,3 V | No figura en ningún pedido | | | Paso 1.5 de la hoja de banco `NO EJECUTADA` |

Al terminar la junta: completar las columnas vacías, pasar las fotos de serigrafía al registro de H01 correspondiente y corregir la tabla inicial de este archivo con el estado real.

## Antes de fijar el pinout

- Identificar memoria y pines de la placa real; conservar USB y condiciones de arranque.
- Verificar dirección I2C y controlador de la pantalla con una prueba gráfica.
- TTP223 en modo momentáneo para medir duración del contacto.
- INMP441 alimentado a 3,3 V y configuración I2S validada.
- MAX98357A con alimentación adecuada, masas comunes y parlante entre sus salidas; no unir una salida de parlante a GND.
- Comprobar polaridad de capacitores, resistencia de LED y continuidad antes de energizar.
- Registrar consumo y estabilidad con WiFi, pantalla y audio simultáneos.

No se publica un mapa de pines supuesto. T03 lo documentará después de verificar el lote. El simulador permite avanzar en software, pero no valida ninguna conexión física.
