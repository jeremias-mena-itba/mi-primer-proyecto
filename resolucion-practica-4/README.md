# Entrega práctica 4 — Streaming

- **Nombre:** Mena, Jeremías
- **student_id:** jmena
- **Escala:** small

### Log y offsets (`01_log_y_offsets`)

**1. En el ejemplo en Python, ¿cuántos mensajes quedaron en la cola y cuántos en el log después de leer tres? ¿Qué diferencia hay entre una cola tradicional y un log como Kafka?**

En la cola quedaron dos mensajes: "pago_4" y "pago_5", mientras que en el log siguen existiendo cinco mensajes. La diferencia principal es que una cola tradicional elimina el mensaje una vez consumido, en cambio un log, lo conserva según una política definida.

**2. Después de la llegada 2, ¿cuántos mensajes había leído el consumidor `antifraude` y cuántos `reportes`? ¿El atraso de `reportes` afectó a `antifraude`? ¿Por qué?**

Luego de la llegada 2, el consumidor antifraude leyó 100 mensajes y el consumidor reportes leyó 280. El atraso de reportes no afectó a antifraude porque las lecturas son independientes, es decir que si uno de los consumidores se atrasa, retoma su lectura desde el checkpoint

**3. ¿Qué es un offset y dónde lo guarda cada consumidor en esta práctica?**

El offset es una versión del topic que leyó cada consumidor y se guarda en cada checkpoint.

**4. ¿Cuántos mensajes leyó el replay? ¿Qué propone la arquitectura Kappa para reprocesar datos y en qué se diferencia de Lambda?**

El replay leyó 280 mensajes. Para reprocesar los datos, Kappa propone lanzar una consulta nueva que lee el log desde el principio y la diferencia con Lambda es que hay sólo código de streaming.

### Ingesta y enriquecimiento (`02_ingesta_y_enriquecimiento`)

**5. ¿Cuántas filas nuevas procesó Bronze en cada llegada y cuántas al reejecutar sin archivos nuevos? ¿Qué componente evita leer dos veces el mismo archivo?**

|llegada   |filas nuevas en Bronze|
|----------|----------------------|
|stream_001|                    60|
|stream_002|                   120|
|stream_003|                    40|
|stream_004|                    40|
|stream_005|                    20|

Al reejecutar sin archivos nuevos, no hay filas nuevas.

El componente que evita leer dos veces el mismo archivo es el checkpoint.

**6. ¿Qué motivos de rechazo aparecieron en la cuarentena y en qué llegada? ¿Por qué Bronze guarda el texto original en lugar de descartar los registros con errores?**

Los motivos de rechazo que aparecieron en cuarentena fueron `INVALID_AMOUNT` y `UNKNOWN_CUSTOMER` en la llegada dos (stream_002). Bronze guarda el texto original ya que la capa encargada de descartarlos o modificarlos es Silver.

**7. ¿Qué columnas agregó el join con `silver_customers` y `silver_products`? ¿Qué tipo de join es (stream-stream, stream-tabla o tabla-tabla) según la teoría?**

El join agregó las columnas `country` y `category`. El tipo de join utilizado es el join stream-tabla.

### Tiempo y ventanas (`03_tiempo_y_ventanas`)

**8. ¿Qué diferencia hay entre event time y processing time? Usá `late_ok` como ejemplo.**

Event time es cuándo ocurrió el evento en el mundo real, mientras que processing time es cuándo llegó al sistema. El evento etiquetado como late_ok, ocurrió a las 12:03 entre e002 y e003 y su llegada fue para el segundo lote, junto con e004 y e005.

**9. ¿Cuánto valía el watermark al terminar cada llegada? ¿Cómo se calcula a partir del máximo event time visto?**

|llegada   |watermark al terminar |
|----------|----------------------|
|stream_001|                 11:54|
|stream_002|                 11:58|
|stream_003|                 12:16|
|stream_004|                 12:17|
|stream_005|                 12:45|

El watermark se calcula quitándole 10 minutos al máximo event time visto. Esto nos indica el retraso máximo tolerado.

**10. ¿En qué llegada aparecieron las primeras ventanas tumbling en modo append y por qué no antes?**

Las primeras ventanas tumbling en modo append aparecen en la llegada 3 ya que en las llegada 1 y 2 el max event time visto todavía era bajo.

**11. ¿Qué pasó con `late_bad`? ¿Por qué se aceptó `late_ok` y se descartó `late_bad`?**

`late_bad` se descartó. `late_ok` se aceptó porque llegó dentro del margen del watermark, mientras que `late_bad` quedó más viejo.

**12. Mirando el gráfico de ventanas, ¿en qué se diferencian tumbling, hopping y session? Da un ejemplo de negocio para cada una.**

Tumbling tiene ventanas con tamaño fijo, una pegada a la otra,  sin solaparse. Un ejemplo de negocio podría ser un tablero de ventas por canal cada 5 minutos. Cada compra se cuenta en un solo período. 

Hopping tiene ventanas más anchas que su salto. Por eso se solapan, y cada evento cuenta en dos ventanas. Un ejemplo de negocio: una alerta antifraude del tipo "más de N compras en los últimos 10 minutos", evaluada cada 5 minutos.

Session por su lado, tiene ventanas de tamaño dinámico, además de que se arman por cliente y se cierran cuando pasa un período de inactividad. Un ejemplo de negocio: medir la visita de compra de un cliente en un e-commerce: cuántas compras hizo por sesión, cuánto gastó y cuánto duró. 

**13. Para la ventana [12:00, 12:05) del canal card, ¿cuántas veces y con qué valores se emitió en modo update y en modo append? ¿Qué ventaja y qué desventaja tiene cada modo?**

**Valores que se emitieron en modo update y append**
|modo        |compras      |monto  |
|------------|-------------|-------|
|update      |           40|6000.00|
|update      |           60|8400.00|
|append      |           60|8400.00|

**Modo update**
* Ventaja: baja latencia. La ventana se emite apenas tiene datos y se vuelve a emitir cada vez que cambia.
* Desventaja: la misma ventana puede llegar varias veces, con valores distintos.

**Modo append**
* Ventaja: cada ventana se emite una sola vez, con su valor final.
* Desventaja: más demora. Hay que esperar a que el watermark supere el fin de la ventana.

**14. ¿Qué se gana y qué se pierde si el watermark fuera de 1 minuto en lugar de 10?**

Si el watermark fuera de 1 minuto en lugar de 10, se gana rapidez y menos estado en memoria, y se pierde completitud ya que los eventos que llegan con retraso se descartan.

### Garantías y fallas (`04_garantias_y_fallas`)

**15. ¿Cuántas filas quedaron en el destino sin deduplicar y cuántas deduplicando por `event_id`? ¿Por qué el productor puede enviar el mismo evento dos veces?**

En el destino quedaron 20 filas sin deduplicar y 200 deduplicando por cada `event_id`. El productor puede enviar el mismo eventos dos veces porque es una garantía de entrega. El sistema asegura que cada evento llegará al consumidor una o más veces.

**16. ¿Qué pasó al reiniciar con el mismo checkpoint y qué pasó al perderlo? ¿Qué garantía de entrega se observa en cada caso?**

Al reiniciar con el mismo checkpoint se mantuvieron la cantidad de filas, mientras que al perder este checkpoint todo se escribe otra vez, de forma que se duplican las filas. Al reiniciar el checkpoint, tenemos la garantía exactly-once ya que no reprocesa, mientras que cuando se pierde tenemos la garantía at-least-once ya que un destino append recibe todo otra vez.

**17. ¿Por qué con `MERGE` el resultado no cambió aunque se perdiera el checkpoint? ¿Qué significa que una escritura sea idempotente?**

A pesar de haber perdido el checkpoint, el resultado no cambió porque `MERGE` tiene una escritura idempotente, lo que significa que hace que reprocesar no cambie el resultado.

### CDC (`05_cdc`)

**18. ¿Cuántos eventos de cambio generó el log y de qué tipos? ¿Por qué un `UPDATE` genera dos eventos?**

Se generaron 13 eventos de cambio (6 insert, 3 upadte_preimage, 3 update_postimage y 1 delete). `UPDATE` genera dos eventos ya que guarda el estado de la fila antes y después.

**19. ¿Se pudo reconstruir la tabla a partir del log? ¿Qué información tiene el log que la tabla actual no tiene?**

Sí, se pudo reconstruir la tabla a partir del log. El log conserva información acerca de la mudanza del cliente 2, es decir, conserva los estados intermedios de cada registro.