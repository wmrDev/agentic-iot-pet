# Fichas de prueba EVA_02

Estado de todas las fichas: **NO EJECUTADA**. Índice y reglas en [README.md](README.md).

Cada ficha se ejecuta en **INFINIX ZERO 30 5G** y **Galaxy S24 Ultra** con la misma revisión, salvo que indique otra cosa. Antes de cada sesión, completar la cabecera de [DISPOSITIVOS.md](DISPOSITIVOS.md) y anotar en el registro el SHA-256 de la APK y el commit del firmware.

Las operaciones de umbral, aviso y confirmación las define T04 (#8). Los nombres que aparecen aquí (`config.set`, `alert.set`, `Confirmation`) vienen del plan aprobado. Antes de ejecutar, contrastarlos con el contrato vigente en `contracts/` y corregir la ficha si difieren.

## Preparación común

- Red local propia (router o hotspot) sin otros dispositivos ajenos; anotar banda y si es hotspot de qué teléfono.
- **Alimentación y log comparten el único USB-C de la SuperMini.** Ese conector lleva la alimentación y la consola USB Serial/JTAG, así que no es posible alimentar desde una fuente externa de 5 V y ver a la vez el log serie en el PC sin un adaptador UART en pines todavía no verificados. Por eso:
  - En las sesiones con log, alimentar desde el puerto del PC o desde un hub USB alimentado, opcionalmente con un medidor USB en línea que deje pasar datos.
  - Si se necesita alimentación externa (por ejemplo, la fuente USB-C 5 V / 3 A, que según el [inventario](../../hardware/INVENTARIO.md) está **por adquirir**), hacerlo en una sesión separada sin log serie y registrar qué evidencia sustituye al log.
  - Anotar siempre en el registro la fuente real usada (puerto del PC, hub alimentado con su modelo, o fuente externa) y si había log.
- Monitor serie del firmware abierto en el PC y guardado a archivo (`idf.py -p <puerto> monitor | tee <archivo>.log` o equivalente), para correlacionar con Android, en las sesiones con log.
- Reloj de referencia: anotar la hora del PC al iniciar el log y la hora mostrada por cada teléfono, para alinear eventos.
- Credenciales y certificados de prueba generados para el ensayo, fuera de Git (ver contrato, «Transporte e identidad»).
- Cada ficha declara su **entorno** (banco, simulado o integrado), que se copia al campo «Entorno» de la [plantilla](../PLANTILLA_RESULTADO.md). Banco e integrado son ejecuciones físicas; simulado usa el simulador o un proveedor simulado. Nunca se mezclan en un mismo registro.

---

## E01 · Identificación, instalación y permisos

- **Matriz / issue:** A04 parcial · #1, #18.
- **Objetivo:** demostrar que ambos Android ejecutan la misma APK y registrar su entorno.
- **Entorno:** físico, solo Android y PC; no usa la placa. Se registra aparte de los casos integrados.
- **Precondiciones:** APK generada desde un commit identificado; SHA-256 calculado en el PC (`certutil -hashfile <apk> SHA256` en Windows).
- **Pasos:**
  1. Registrar en [DISPOSITIVOS.md](DISPOSITIVOS.md) fabricante, modelo, versión de Android, nivel de API, arquitectura y parche de seguridad de cada teléfono.
  2. Copiar la APK al teléfono y comprobar nombre y tamaño en el teléfono.
  3. Instalar; anotar si Android pidió permitir orígenes desconocidos y qué advertencias mostró Play Protect.
  4. Abrir la app y anotar cada permiso solicitado (red local, notificaciones, dispositivos cercanos, etc.) y la respuesta dada.
  5. Desinstalar y reinstalar una vez; anotar si conserva o pierde datos.
- **Esperado:** instalación y arranque en ambos teléfonos con el mismo hash; lista de permisos idéntica o diferencias explicadas por versión de Android.
- **Negativo:** intentar instalar una APK de otro commit encima; registrar el comportamiento (actualización o rechazo por firma).
- **Evidencia:** hash, captura de «Acerca del teléfono» sin IMEI ni serie, captura de la app abierta.
- **No demuestra:** conexión con la placa ni funcionamiento de ninguna función.

## E02 · Acceso local (login)

- **Matriz / issue:** P04 parcial · #5, #9, #10.
- **Objetivo:** acceso con la credencial correcta y rechazo sin filtrar datos.
- **Entorno:** integrado (Android real con la ESP32-S3 real). Una repetición contra el simulador se registra aparte como simulada.
- **Pasos:**
  1. Abrir la app sin sesión y comprobar que no muestra estado ni lecturas de la placa.
  2. Introducir una contraseña incorrecta tres veces; anotar mensaje, tiempo de respuesta y si hay espera o bloqueo.
  3. Introducir la contraseña correcta; anotar tiempo hasta ver el estado.
  4. Cerrar sesión y comprobar que el estado deja de mostrarse.
- **Esperado:** solo el acceso correcto revela estado; el mensaje de error no indica si existe el usuario.
- **Negativo:** en el log serie, confirmar que la placa no aplica órdenes de una sesión cerrada (respuesta `UNAUTHENTICATED` o equivalente).
- **Evidencia:** capturas de cada estado (tapar la contraseña), extracto del log serie.
- **No demuestra:** robustez frente a fuerza bruta ni seguridad del almacenamiento de la credencial (eso es #16).

## E03 · Lectura actual y lectura guardada

- **Matriz / issue:** P02 parcial (estímulos de contacto y ruido controlado); la comparación de lectura actual y guardada no tiene ID propio en la [matriz](../MATRIZ.md) · #6, #9.
- **Objetivo:** la app muestra la lectura vigente y el historial guardado coincide con lo emitido por la placa.
- **Entorno:** integrado (Android real con la ESP32-S3 real). Una repetición contra el simulador se registra aparte como simulada.
- **Pasos:**
  1. Con sesión iniciada, generar estímulos conocidos: tocar el TTP223 cinco veces y producir ruido controlado (palmada a distancia fija, anotada) tres veces.
  2. Anotar hora y valor mostrados en la app para cada estímulo.
  3. Abrir el historial y comparar con el log serie: cantidad de eventos, orden y marcas de tiempo.
  4. Cerrar y reabrir la app; comprobar que el historial persiste.
- **Esperado:** cada evento del log aparece una vez en el historial; la lectura actual se actualiza a la frecuencia de telemetría (1 muestra/s).
- **Negativo:** sin estímulos durante 60 s, el historial no inventa eventos.
- **Evidencia:** tabla estímulo → log serie → app; capturas.
- **No demuestra:** recuperación tras corte (E09).

## E04 · Ajuste de umbral

- **Matriz / issue:** P03 · #6, #8.
- **Objetivo:** cambiar el umbral con validación de rango y ver el estado aplicado.
- **Entorno:** integrado (Android real con la ESP32-S3 real). Una repetición contra el simulador se registra aparte como simulada.
- **Rango de referencia (T04):** intensidad relativa normalizada; umbral válido `(0, 1]`, histéresis inicial 0,05. No etiquetar valores como dB.
- **Pasos:**
  1. Anotar el umbral actual mostrado por la app y el que reporta la placa en el log.
  2. Cambiar a 0,50; si el cambio exige confirmación física, ejecutar E05.
  3. Verificar que la app muestra el valor solo después de la respuesta aplicada.
  4. Producir ruido por debajo y por encima del umbral nuevo; anotar reacción.
- **Esperado:** la placa reacciona según el nuevo umbral; la app nunca muestra éxito antes de la confirmación.
- **Negativos:** intentar 0, 1,5, un valor negativo y un texto. Todos rechazados (`INVALID_MESSAGE` o validación en la app) sin cambiar el umbral vigente.
- **Evidencia:** capturas antes y después, extracto del log con la solicitud y la respuesta.
- **No demuestra:** calibración acústica absoluta.

## E05 · Toque de confirmación

- **Matriz / issue:** P12 parcial · #8.
- **Objetivo:** solo la confirmación vigente de la operación actual produce un efecto.
- **Entorno:** integrado (Android real con la ESP32-S3 real). Una repetición contra el simulador se registra aparte como simulada.
- **Referencia (T04):** `Confirmation` dura 15 000 ms; estados `pending/applied/cancelled`.
- **Pasos:**
  1. Solicitar desde la app un cambio protegido (umbral o aviso).
  2. **Caso A:** tocar el TTP223 dentro del plazo. Esperado: `applied`, un solo efecto.
  3. **Caso B:** no tocar; cronometrar. Esperado: `cancelled` al vencer, sin efecto.
  4. **Caso C:** con una sesión de estudio en curso, tocar dos veces seguidas dentro del plazo. Esperado: un solo efecto y la sesión de estudio **no** queda pausada, aunque el doble toque sea el gesto de pausa. Anotar el estado de la sesión antes y después.
  5. **Caso D:** mantener el toque desde antes de solicitar. Esperado: no confirma hasta soltar y tocar de nuevo.
- **Esperado:** cada caso con su resultado; tiempo de vencimiento observado cercano a 15 s.
- **Evidencia:** vídeo del iPhone 14 mostrando placa y teléfono a la vez; log serie.
- **No demuestra:** comportamiento durante relevo Android/Windows (P12 completo, T06).

## E06 · Aviso manual y retorno al modo automático

- **Matriz / issue:** P03 parcial · #8, #10.
- **Entorno:** integrado (Android real con la ESP32-S3 real). Una repetición contra el simulador se registra aparte como simulada.
- **Pasos:**
  1. Con la placa en modo automático, activar el aviso manual desde la app (`alert.set` o nombre vigente).
  2. Observar LED, OLED y sonido; anotar tiempo entre la acción y el efecto.
  3. Producir ruido por encima del umbral; comprobar que el modo manual no se superpone ni duplica el aviso.
  4. Volver al modo automático desde la app y repetir el ruido.
- **Esperado:** el aviso manual se aplica una vez; al volver, la reacción automática funciona con el umbral vigente.
- **Negativo:** enviar «volver a automático» dos veces; un solo cambio de estado.
- **Evidencia:** vídeo, log serie, capturas de la app en cada modo.

## E07 · Correlación de solicitudes y respuestas

- **Matriz / issue:** P05 parcial · #6.
- **Objetivo:** cada solicitud de Android tiene una única respuesta identificable en la placa.
- **Entorno:** integrado (Android real con la ESP32-S3 real). Una repetición contra el simulador se registra aparte como simulada.
- **Precondiciones:**
  - La app adquiere el control con `POST /v1/control/acquire` y lo renueva cada 2 000 ms con `/v1/control/renew` (el permiso dura 6 000 ms). Anotar el `lease_epoch` recibido y cualquier cambio durante la prueba.
  - **Dependencia de #6:** la app todavía no tiene un registro de diagnóstico que muestre `request_id`, `sequence`, `lease_epoch` y código de respuesta. Si al ejecutar no existe, los pasos que lo usan quedan `NO EJECUTADA` con ese motivo; no se reconstruyen los valores a mano.
- **Pasos:**
  1. Activar en la app el registro de diagnóstico que muestre `request_id`, `sequence`, `lease_epoch` y código de respuesta (sin el encabezado Authorization).
  2. Enviar diez órdenes `expression.set` con duraciones distintas entre 100 y 5 000 ms.
  3. Emparejar en una tabla `request_id` del teléfono con la línea del log serie y la respuesta recibida.
  4. Medir tiempo entre envío y respuesta aplicada con el enlace ya establecido; medir aparte la primera conexión TLS.
- **Esperado:** diez pares completos, sin huecos ni duplicados; meta inferior a 1 s con enlace establecido.
- **Evidencia:** tabla de correlación, extractos de ambos logs.
- **No demuestra:** comportamiento con pérdida de paquetes (E09).

## E08 · Rechazos de seguridad sin activar el aviso

- **Matriz / issue:** P04, P05, A02 · #5, #6, #16.
- **Objetivo:** ninguna solicitud no autorizada o inválida produce efecto ni revela estado.
- **Entorno:** integrado (cliente HTTPS del PC contra la ESP32-S3 real). La misma batería contra el simulador (A02) se registra aparte como simulada.
- **Herramienta:** cliente HTTPS del PC (por ejemplo `curl` con el certificado de prueba) en la misma red. No usar opciones que desactiven la verificación TLS en los casos que deben pasar.
- **Precondiciones:**
  - Para los casos que necesitan permiso vigente, el cliente de prueba adquiere el control con `POST /v1/control/acquire` y lo renueva cada 2 000 ms con `/v1/control/renew`. Anotar el `lease_epoch` recibido. Si no hay forma de renovar cada 2 s desde el PC, enviar cada solicitud dentro de los 6 000 ms siguientes a la adquisición o renovación y anotar el tiempo transcurrido.
  - Para ver `request_id`, `sequence` y `lease_epoch` del lado de la app se depende del registro de diagnóstico de #6; si no existe, esos datos se toman solo del cliente del PC y del log serie, y la parte de la app queda `NO EJECUTADA`.
- **Casos:** una fila por caso y un único resultado esperado por fila.

| ID | Solicitud | Respuesta esperada (contrato) |
|---|---|---|
| E08a | Sin `Authorization` | 401 `UNAUTHENTICATED`, sin datos de estado |
| E08b | Credencial revocada | 401 `UNAUTHENTICATED`, sin datos de estado ni resultados anteriores |
| E08c | Credencial válida del cliente B con el `client_id` del cliente A en el cuerpo | 403 `CLIENT_MISMATCH`, aunque el cuerpo traiga una época válida |
| E08d | Cliente que verifica contra un certificado distinto al emparejado | La conexión no se establece; la app no envía la orden |
| E08e | JSON con clave repetida | 400 `INVALID_MESSAGE` |
| E08f | JSON con propiedad desconocida | 400 `INVALID_MESSAGE` |
| E08g | JSON con tipo incorrecto (por ejemplo, `duration_ms` como cadena) | 400 `INVALID_MESSAGE` |
| E08h | Petición mayor de 1 024 bytes | 413 `MESSAGE_TOO_LARGE` |
| E08i | `boot_id` anterior tras reiniciar la placa | 409 `STALE_BOOT` |
| E08j | Reintento exacto (mismo `request_id` y mismo contenido) de una orden aplicada, con la época vigente y el permiso sin vencer | Resultado original (`applied`, mismo `request_id`); contador de efectos igual a 1 |
| E08k | `request_id` nuevo con una `sequence` ya consumida | 409 `STALE_SEQUENCE` |
| E08l | Mismo `request_id` con otro contenido | 409 `ID_CONFLICT` |
| E08m | Orden de un cliente autenticado mientras otro cliente tiene el control | 403 `NOT_OWNER` |
| E08n | Orden con un `lease_epoch` anterior al vigente (tras liberar y volver a adquirir) | 409 `STALE_EPOCH` |
| E08o | Orden con la época vigente después de dejar vencer el permiso sin renovar (más de 6 000 ms) | 409 `LEASE_EXPIRED` |
| E08p | *Opcional, capa de transporte (T02/T03):* cuerpo con `Content-Type` distinto de `application/json` | 415 `UNSUPPORTED_MEDIA_TYPE`. Si la capa de transporte aún no lo implementa, `NO EJECUTADA` con ese motivo |

- **Esperado:** en los rechazos el OLED, el LED y el parlante no cambian; en E08j no se produce un segundo efecto. Comprobarlo mirando la placa, no solo el código HTTP.
- **Evidencia:** comando usado (sin credenciales), código y cuerpo de respuesta, línea del log serie, foto o vídeo de la placa sin cambio.
- **No demuestra:** auditoría completa de seguridad (#16).

## E09 · Corte de enlace, autonomía, reconexión y recuperación

- **Matriz / issue:** P16 · #8, #9, #28. Solo cuenta además como P15 parcial si el corte deja sin internet (por ejemplo, apagando el router) y se anota que la IA quedó deshabilitada; apagar solo el Wi-Fi del teléfono no cubre P15.
- **Objetivo:** con la placa alimentada, la sesión sigue sin enlace y al volver se recuperan las lecturas una sola vez.
- **Entorno:** integrado (Android real con la ESP32-S3 real). Una repetición contra el simulador se registra aparte como simulada.
- **Pasos:**
  1. Iniciar una sesión de estudio desde la app; anotar duración objetivo y hora.
  2. Cortar el enlace sin apagar la placa: apagar Wi-Fi del teléfono o del router (anotar cuál). Cronometrar cinco minutos.
  3. Durante el corte: producir estímulos conocidos (toques y ruido) y anotar hora; observar que la placa sigue reaccionando y el temporizador avanza.
  4. Comprobar que la app indica la desconexión y no muestra lecturas nuevas como recibidas.
  5. Restaurar el enlace; anotar tiempo hasta la reconexión automática.
  6. Comparar historial de la app con el log serie: muestras recuperadas, duplicados y brechas.
  7. Repetir aparte con la placa reiniciada durante el corte, para registrar la brecha real (escenario distinto, no se suma al anterior).
- **Esperado:** al menos 300 muestras recuperadas sin duplicados; estímulos del corte presentes una vez; brechas reales visibles; la sesión no se alarga ni se reinicia.
- **Negativo:** una orden pendiente de antes del corte no se aplica al reconectar si su permiso venció.
- **Evidencia:** log serie completo, capturas de la app antes, durante y después, tabla de conteo.
- **No demuestra:** sincronización con el destino externo local (parte de #28, registrada por Walther).

## E10 · Consumo de la placa

- **Matriz / issue:** P18 parcial · #7, #8.
- **Entorno:** banco (placa con sus periféricos; el teléfono solo provoca los estados C3 y C6).
- Procedimiento completo en [CONSUMO.md](CONSUMO.md#placa). Se ejecuta una vez por revisión de firmware, no por teléfono.

## E11 · Teléfono bloqueado 30 minutos

- **Matriz / issue:** P18 · #11, #28.
- **Entorno:** integrado (Android real con la ESP32-S3 real). Una repetición contra el simulador se registra aparte como simulada.
- **Pasos:**
  1. Batería entre 50 % y 90 %, sin cargar; ahorro de batería activado; anotar el modo exacto.
  2. Con la app conectada, bloquear el teléfono 30 min sin tocarlo.
  3. Producir tres estímulos conocidos en la placa a los 10, 20 y 29 min.
  4. Desbloquear; comprobar estado de conexión, estímulos en el historial y si la app se reinició.
  5. Registrar batería inicial y final según [CONSUMO.md](CONSUMO.md#telefono).
- **Esperado:** servicio activo o recuperación documentada; los tres estímulos presentes una vez; sin reinicios de la placa.
- **Evidencia:** capturas antes y después, log serie, datos de batería.

## E12 · Banco de la placa

- **Matriz / issue:** P01 parcial · #7.
- **Objetivo:** dejar identificado el prototipo que usan E01–E11. Esta ficha es de banco; no se mezcla con resultados integrados.
- **Entorno:** banco (placa sola, alimentada y con log por el puerto USB del PC).
- **Pasos:**
  1. Fotografiar el montaje completo y cada módulo con su serigrafía visible.
  2. Registrar la identificación del diagnóstico USB (chip, revisión y flash) y el commit del firmware cargado. El diagnóstico no informa la PSRAM, porque se compila con SPIRAM desactivada: tomarla de `python -m esptool --chip esp32s3 -p COMx flash_id` (paso 1.2 de la [hoja de banco](../../entrega/2026-11-28-hoja-banco.md)).
  3. Registrar el pinout verificado por H01. Si un pin no está verificado, escribirlo como pendiente; no completarlo con el previsto.
  4. Reiniciar la placa tres veces; anotar `reset_reason`, tiempo hasta la primera expresión y si el OLED se ve centrado.
- **Esperado:** arranque estable en los tres reinicios; identificación coincide con la de H01.
- **Evidencia:** fotos, log serie, enlace al registro de H01.
- **No demuestra:** consumo (E10) ni enlace con Android.
