# Entrega práctica 3 — Bases de datos NoSQL

- **Nombre:** Mena, Jeremías
- **student_id:** jmena
- **Escala:** small

Respondé cada pregunta en una a tres oraciones. Cuando se pide un dato, copiá el valor que te dio el notebook.

## CAP y PACELC

**1. Con la red partida, ¿qué respondió la réplica C en modo CP y qué respondió en modo AP? ¿Qué propiedad de CAP sacrifica cada modo?**

En modo CP la réplica C dio error tanto al leer como al escribir el saldo 50. En modo AP, frente a la lectura del saldo devolvió 100 y no dio error al escribir el saldo 50.
En modo CP sacrifica disponibilidad y en modo AP sacrifica consistencia.

**2. En modo AP, ¿qué saldo quedó en las tres réplicas después de la reparación y qué escritura se perdió?**

El saldo vigente luego de la reparación fue 50. Se perdió la escritura del saldo de la réplica A (80) y B (80).

**3. Según el gráfico de latencia, ¿cuál es la p99 con W=1 y con W=3? ¿Qué se gana a cambio de esperar más réplicas? (PACELC)**

La p99 con W=1 es 9.6 ms y la p99 con W=3 es 55.6 ms. A cambio de esperar más réplicas de gana mayor latencia.

**4. Con cinco réplicas partidas en ABC | DE, ¿qué lado pudo seguir escribiendo en modo CP y por qué?**

Sólo el lado ABC pudo seguir escribiendo en modo CP ya que cumplen con el quorum.

## Clave-valor

**5. ¿Cuántas veces más rápido fue el GET en memoria que el GET con Spark? ¿Por qué Redis guarda los datos en memoria?**

El GET en memoria fue ~2,848,294 veces más rápido que el GET con Spark. Redis gaurda los datos en memoria porque su objetivo principal es  la velocidad.

**6. ¿Por qué para contar los clientes de AR hubo que leer y parsear todos los valores? Mencioná una ventaja y una desventaja del modelo clave-valor.**

Para poder contar los clientes de AR fue necesario leer y parsear todos los valores porque dada una clave exacta, la base devuelve su valor y la misma desconoce la estructura interna del valor. Una ventaja de este modelo es su velocidad y simplicidad a la hora de hacer una consulta, sin embargo, como desventaja, se observa que estas consultas son limitadas ya que para consultar sobre el contenido implica recorrer todo.

**7. Al pasar de 4 a 5 nodos, ¿qué porcentaje de claves se movió con hash mod N y con hashing consistente? ¿Por qué conviene el segundo?**

El 79.6% de las claves se movió con hash mod N y el 21.2% fue movido con hashing consistente. Conviene el segundo porque en lugar de mover todos los datos por la red, al agregar un nodo al anillo sólo se mueven las claves del tramo que ocupa.

## Documental

**8. ¿Cuántas filas devolvió la consulta relacional del cliente 42 y cuántos documentos la documental? ¿Qué datos quedaron embebidos dentro del documento?**

Las cantidad de filas que devolvió la consulta relacional fueron once y la documental devolvió un solo documento. Dentro del documento quedaron datos asociados al perfil del cliente, estadísticos sobre las transacciones y el número de transacciones

**9. Al leer documentos con campos distintos, ¿qué hizo Spark con los campos que faltaban en algunos? ¿Por qué se dice que el modelo documental tiene esquema flexible?**

Al existir campos faltantes en algunos documentos, Spark completó con `null`. El modelo documental tiene un esquema flexible porque no obliga a que todos los documentos tengan la misma estructura.

**10. ¿Cuánto pesa el documento más grande? ¿Qué problema aparece si un documento crece sin límite?**

El documento más grande pesa 5.2 kb. Si un documento crece sin límite tanto la lectura como la escritura serían más costosas, además que la primera podría comenzar a fallar si se alcanza el tope establecido por el motor por ej.: MongoDB que soporta documentos de hasta 16 mb.

**11. Para calcular el monto por categoría hubo que usar explode. ¿Qué tipo de consultas resuelve bien el modelo documental y cuáles le cuestan más?**

El modelo documental resuelve mejor las consultas que están asociadas a un sólo documento ya que favorece el acceso por entidad, en cambio, las consultas que involucran el cruce con otros documentos cuestan más ya que es necesario volver al esquema por filas

## Grafos

**12. ¿Cuántos nodos Customer, nodos Device y aristas USES tiene el grafo?**

5000 nodos Customer, 2361 nodos Device y 5400 aristas USES.

**13. ¿Cuántos joins necesitó SQL para el patrón de 2 saltos y para el de 4? ¿Por qué Neo4j recorre relaciones más eficientemente que una base relacional?**

Para el patrón de dos saltos necesitó tres joins y para el de cuatro precisó cinco joins. Neo4j recorre las relaciones más eficientemente porque cada nodo guarda punteros directos a sus relaciones y a sus vecinos. 

**14. ¿Cuántas iteraciones tardó en converger la búsqueda de componentes conexos y cuántos nodos tiene el componente más grande?**

Tardó 14 iteraciones en coverger y el componente más grande tiene 23 nodos.

**15. ¿Qué porcentaje de aristas quedó entre máquinas distintas al repartir los nodos con hash(id) mod 4? ¿Por qué es difícil distribuir una base de grafos?**

El 74.4% de aristas quedó entre máquinas distintas al repartir los nodos con hash(id) mod 4. Es difícil distribuir una base de grafos porque los datos están definidos por sus conexiones, y no hay una forma de repartirlos entre máquinas que no corte alguna de ellas.

## Vectorial

**16. ¿Qué representa el vector de cada cliente y qué mide la similitud coseno?**

El vector de cada cliente representa las columnas disponibles en la tabla original. La similitud coseno mide la cercanía que tiene dos vectores.

**17. ¿Qué porcentaje de clientes marcados hay entre los vecinos de clientes marcados y cuál es la tasa base? ¿Para qué sirve buscar "vecinos parecidos"?**

Existe un 31.6% de clientes marcados entre los vecinos de clientes marcados y la tasa base es del 14.8%. Buscar vecinos parecidos sirve para encontrar, los elementos cuyo significado o características se parecen más a una consulta en particular, sin depender de coincidencias exactas de palabras. 

**18. En la curva del índice IVF, ¿qué recall y qué porcentaje de vectores escaneados se obtienen con nprobe=4? ¿Qué se gana y qué se pierde con la búsqueda aproximada (ANN)?**

Con nprobe=4 se obtiene un recall de 0.872 y un 5.985% de los vectores escaneados. Con la búsqueda aproximada se gana velocidad, escalabilidad y un menor costo de cómputo, mientras que se pierde recall, memoria y tiempo de construcción.

## Columnar

**19. ¿Cuánto ocupan los datos en CSV, JSON y Parquet? ¿Qué columnas leyó Spark para la consulta de amount sobre Parquet (ReadSchema)? ¿Cuántos archivos se pueden saltear con los datos ordenados?**

En CSV 50 mb, en JSON 121.1 mb y en Parquet 5.3 mb. Para la consulta de amount, Parquet leyó las columnas: transaction_id, customer_id, product_id, event_ts, amount, payment_channel, device_id, is_fraud, source_batch_id y updated_at. Con los datos ordenados se pueden saltear siete archivos. 

**20. ¿Por qué el formato columnar es bueno para analítica y poco conveniente para modificar una fila por vez?**

El formato columnar es bueno para analítica porque lee sólo lo necesario y comprime mucho mejor los datos, pero es poco conveniente para modificar una fila por vez porque los archivos Parquet suelen ser inmutables, además de que los los datos comprimidos y ordenados son difíciles de alterar.