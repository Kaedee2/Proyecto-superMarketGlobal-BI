# Informe del avance 3. Integracion, analisis descriptivo y ETL reproducible

**Proyecto:** SuperMarketMundial · **Equipo:** Equipo BI SuperMarket
**Integrantes:** Francisco Aramis Alcantara Vivas · Brayan Perez Nava · Joseline Ariana Marcial Espinoza

En este informe resumimos lo que hicimos en el avance 3 y decimos donde esta cada cosa. El codigo, las tablas completas y los resultados estan en `notebooks/avance3_etl.ipynb`, que sigue la misma numeracion de la guia.

## 1. Pregunta y cambios por la retroalimentacion

**Pregunta.** ¿A partir de que nivel de descuento la venta deja de ser rentable, y ese punto es el mismo en todas las categorias, subcategorias y mercados?

**Decision.** Queremos ayudar a la Gerencia de Marketing a poner un descuento maximo por subcategoria y mercado, para que dejen de vender con perdida.

**Con que datos trabajamos ahora.** Ventas con descuento de US, LATAM, EU y APAC, de enero de 2011 a diciembre de 2014.

El 3 de noviembre la profesora nos indico redondear los descuentos con tres decimales y juntar las parejas de envios. Despues, en la revision del avance 2, nos pidio dejar solo las reglas de negocio, quitar las 3 parejas con cliente distinto en lugar de dejarlas pendientes, trabajar solo con ventas que tienen descuento y tomar el costo de envio como un pago del cliente. Tambien nos pidio agregar media, mediana y moda para responder algo concreto. La tabla completa de cambios esta en la seccion 3.1 del notebook.

## 2. Fuentes e integracion

Tenemos una sola fuente, el Excel *Ventas Supermercado* (su huella SHA-256 empieza con `a599dc2cf5d00385`), y lo guardamos sin cambios en `data/raw/`. Cada fila es un producto dentro de un pedido, y la identificamos con `ID Venta` + `ID Producto`.

No juntamos tablas. La hoja de ventas ya trae la categoria, la subcategoria, el pais y el mercado. Ademas, los catalogos repiten claves (Regiones repite 5 paises y Productos 476 codigos), asi que si los cruzaramos las ventas pasarian de 51,290 a 81,640 filas con Regiones y a 55,996 con Productos. Los catalogos solo los usamos para revisar que cada pais y producto exista, y todos existen (notebook, seccion 3.2).

Despues de limpiar, cada fila sigue siendo un producto dentro de un pedido, pero ahora la clave ya no se repite, porque las parejas se quitaron (R2) o se juntaron (R3).

## 3. Contrato del ETL y reglas

| Pregunta | Lo que acordamos |
|---|---|
| ¿Que entra? | El Excel de `data/raw/` con la hoja *Ventas Supermercado* y sus 35 columnas |
| ¿Que funciones hay? | `extraer`, `normalizar`, `r1_redondear_descuentos`, `r2_eliminar_parejas_inconsistentes`, `r3_unir_envios`, `separar` (para R4 y R5), `derivar`, `revisar`, `preparar` y `guardar` |
| ¿Que sale? | `ventas_preparadas.csv` con 23 columnas y sin claves repetidas, y tambien `registros_excluidos.csv`, `registros_pendientes.csv` y `bitacora.csv` |
| ¿Que pasa si algo viene mal? | Si falta o sobra una columna, el proceso se detiene. Si un valor no se puede convertir, la fila no pasa la revision y se va a pendientes con la regla que fallo |

| Regla | ¿Que hacemos? | ¿Por que? | ¿Que paso? |
|---|---|---|---|
| R1 | Redondeamos a dos decimales los descuentos con tres decimales | Indicacion de la profesora (3 de noviembre) | Cambiaron 629 valores, todos de Copiers en LATAM. 461 quedaron en 0, 41 en 0.2, 104 en 0.4 y 23 en 0.6 |
| R2 | Quitamos las parejas con el mismo pedido y producto pero con cliente distinto | Indicacion de la profesora (revision del avance 2) | Salieron 6 filas de 3 parejas |
| R3 | Juntamos las parejas que solo cambian en cantidad, ventas, utilidad y envio, sumando esas columnas | Indicacion de la profesora (3 de noviembre) | Juntamos 35 parejas. Antes revisamos que las demas columnas fueran iguales |
| R4 | Nos quedamos solo con US, LATAM, EU y APAC | Retroalimentacion del avance 1 | Quedaron fuera 10,271 filas del Excel |
| R5 | Nos quedamos solo con ventas con descuento mayor a 0 | Indicacion de la profesora (revision del avance 2) | Quedaron fuera 21,861 filas del Excel |
| R6 | El costo de envio lo paga el cliente, asi que no lo restamos. La Utilidad es la utilidad neta y el margen sale solo de Utilidad y Ventas | Indicacion de la profesora (revision del avance 2) | En 7,377 grupos de ventas iguales con distinto envio la utilidad nunca cambia, asi que no trae restado el envio |

Lo tecnico (quitar espacios, pasar codigos a mayusculas y asegurar tipos) corrigio 139 codigos de cliente y 16 nombres de producto, sin juntar clientes distintos y sin dejar valores vacios. Las columnas nuevas son *Mes*, *Tramo de descuento* y *Margen* (notebook, seccion 3.3, paso 4).

## 4. Validacion y conciliacion

`revisar()` le hace 14 preguntas de si o no a cada fila, por ejemplo si la clave se repite, si el descuento esta en rango, si las fechas tienen sentido, si el pais y el producto existen en los catalogos y si *Ganancia* coincide con la utilidad. Las 19,141 filas pasaron todas, asi que no hay pendientes. El contrato de salida (11 preguntas a la tabla completa) tambien se cumple (notebook, seccion 3.4).

| ¿A donde fueron? | Filas en la tabla | Filas del Excel que representan |
|---|---|---|
| Fuera por R2 | 6 | 6 |
| Fuera por R4 | 10,264 | 10,271 |
| Fuera por R5 | 21,844 | 21,861 |
| Pendientes | 0 | 0 |
| Limpias | 19,141 | 19,152 |
| **Total** | | **51,290** |

Contamos en filas del Excel porque cada fila guarda en *Número de envíos* cuantas filas del Excel representa. Los totales de cantidad, ventas, utilidad y costo de envio del Excel tambien son iguales a la suma de todos los destinos.

## 5. Estadistica descriptiva

Calculamos cinco medidas sobre las 19,141 ventas limpias (notebook, seccion 3.5).

| Medida | Resultado principal |
|---|---|
| M1. Porcentaje de ventas con perdida por tramo | 21.1 % (1 a 10 %), 23.6 % (11 a 20 %), 62.2 % (21 a 30 %), 80.5 % (31 a 40 %), 96.4 % (41 a 50 %) y 100 % (mas de 50 %) |
| M2. Margen agregado por tramo (suma de utilidad ÷ suma de ventas) | 16.5 %, 9.7 %, −5.6 %, −24.2 %, −46.1 % y −107.8 % |
| M3. Mediana y RIC del margen de cada venta | La mediana es de 16.6 % y 12.5 % en los dos primeros tramos y de −4.3 % en el de 21 a 30 %. El RIC crece de 28.9 a 80 puntos |
| M4. Correlacion entre descuento y margen | Spearman −0.73 y Pearson −0.81, con 19,141 ventas |
| M5. Media, mediana y moda por categoria y mercado | Lo mas vendido es Chairs, Binders y Phones, pero ningun producto concentra las ventas. En LATAM el descuento que mas se usa es 40 %, muy arriba del limite |

**¿Que nos dicen?** El margen deja de ser positivo en el tramo de 21 a 30 %. Las medidas M1 a M4 coinciden en ese punto, y pasa igual en las tres categorias y en APAC, EU y US. En LATAM no hay ventas con descuentos de 1 a 10 % ni de 21 a 30 %, entonces solo sabemos que su margen pasa de 6.3 % con 11 a 20 % a −27.6 % con 31 a 40 %.

Por subcategoria el limite no es igual. Supplies ya pierde desde 11 a 20 %, y Chairs, Bookcases, Art, Tables y Machines desde 21 a 30 %. Las otras 10 subcategorias no tienen ventas con descuentos de 21 a 30 %. Para la decision, esto apunta a un descuento maximo de alrededor de 20 %, y mas bajo en Supplies, pero hay que confirmarlo en los grupos que tienen pocas ventas.

Tambien comparamos con los descuentos como venian. Con la regla R1 el margen de cada tramo cambia menos de un punto, y el tramo donde aparece la perdida es el mismo.

## 6. Bitacora y reproducibilidad

`anotar()` va escribiendo cada paso en `bitacora/bitacora.csv` con la fecha, el archivo, el paso, las filas antes y despues, el detalle, el responsable, la regla, el resultado y las observaciones (11 pasos en cada corrida). En la seccion 3.7 del notebook volvemos a correr todo desde el Excel y comprobamos que la tabla limpia sale identica a la que guardamos, byte por byte, y que el Excel no cambio.

## 7. Limitaciones y siguientes pasos

Lo que concluimos vale para el 37 % de las filas del Excel, que son las ventas con descuento de cuatro mercados. No comparamos contra ventas sin descuento, aunque esas ventas estan guardadas en los excluidos. Por la regla R6 tomamos el envio como un pago del cliente, y comprobamos que la Utilidad no lo trae restado. Si lo pagara la tienda, el margen bajaria casi 11 puntos y la perdida empezaria desde 11 a 20 %. Tampoco sabemos en que moneda estan los montos. Las reglas R1 y R3 suponen un error de captura y envios en dos partes. Algunos grupos tienen pocas ventas, y que el descuento y el margen se muevan juntos no prueba que uno cause el otro.

Lo que sigue es cargar `ventas_preparadas.csv` en PostgreSQL (unidad 3), sacar el limite por subcategoria y mercado juntando los grupos chicos, y revisar como cambia todo mes a mes con la columna *Mes*.
