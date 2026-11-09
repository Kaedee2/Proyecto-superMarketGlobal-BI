# SuperMarketMundial

Proyecto semestral de Inteligencia de Negocios (UACM, semestre 2026-2).

**Equipo BI SuperMarket:** Francisco Aramis Alcantara Vivas · Brayan Perez Nava · Joseline Ariana Marcial Espinoza

**Pregunta de negocio.** ¿A partir de que nivel de descuento la venta deja de ser rentable, y ese punto es el mismo en todas las categorias, subcategorias y mercados?

**Decision que apoya.** Fijar un descuento maximo por subcategoria y mercado para dejar de vender con perdida.

## Estructura del repositorio

```
├── Avance 1 de ProyectoBI/    Documento del avance 1 y sus correcciones (PDF)
├── README.md                  Este archivo
├── requirements.txt           Librerias necesarias
├── data/
│   ├── raw/                   Excel original, sin modificaciones
│   └── processed/             Salidas del ETL (se regeneran al ejecutar el notebook)
│       ├── ventas_preparadas.csv
│       ├── registros_excluidos.csv
│       └── registros_pendientes.csv
├── notebooks/
│   ├── avance2_preparacion_datos.ipynb   Lectura y perfilamiento (avance 2)
│   └── avance3_etl.ipynb                 ETL, conciliacion y estadistica (avance 3)
├── bitacora/
│   ├── bitacora.csv           Bitacora automatica del ETL (la genera anotar())
│   └── bitacora_equipo.md     Tareas, responsables y retroalimentacion
├── docs/
│   └── informe_avance3.md     Informe del avance 3
└── presentacion/              PDF de la exposicion del avance 3
```

## Que hay de cada avance

| Avance | Donde esta |
|---|---|
| Avance 1. Propuesta, pregunta de negocio y fuentes | `Avance 1 de ProyectoBI/` (documento original, correcciones y version corregida) |
| Avance 2. Lectura, perfilamiento y reglas de preparacion | `notebooks/avance2_preparacion_datos.ipynb` |
| Avance 3. Integracion, ETL, estadistica y bitacora | `notebooks/avance3_etl.ipynb`, `data/processed/`, `bitacora/`, `docs/informe_avance3.md` y `presentacion/` |

## Fuente de datos

| Elemento | Detalle |
|---|---|
| Archivo | `data/raw/Ventas Supermercado- Rodrigo Ayala.xlsx` (20.2 MB, seis hojas) |
| Origen | Publicado por Rodrigo Ayala en GitHub: https://github.com/RodrigoAyalaDA/Analisis-Ventas-Excel |
| Version | Huella SHA-256 `a599dc2cf5d00385b9fae82204c373353d22140a26762e2acf36a35beb6c42e3` |
| Hojas usadas | *Ventas Supermercado* (51,290 filas y 35 columnas). *Regiones* y *Productos* solo para revisar |
| Cobertura | Ventas del 1 de enero de 2011 al 31 de diciembre de 2014, 7 mercados y 147 paises |

El Excel no se modifica. Si falta, se puede descargar del enlace de origen y guardarse en `data/raw/`. El notebook vuelve a calcular la huella y avisa en la bitacora si el archivo no es la misma version.

## Como ejecutar

**En la computadora (VS Code o Jupyter)**

1. Clonar o descargar este repositorio.
2. Instalar las librerias con `pip install -r requirements.txt`.
3. Abrir `notebooks/avance3_etl.ipynb` y ejecutar todas las celdas (**Run All**).

**En Google Colab**

1. Abrir `notebooks/avance3_etl.ipynb` en Colab (**Archivo → Abrir cuaderno → GitHub**).
2. Ejecutar todo con **Entorno de ejecucion → Ejecutar todo**.
3. Cuando lo pida, subir el Excel de `data/raw/`. Las salidas quedan en la carpeta `data/processed/` de Colab.

**Orden de ejecucion.** `avance2_preparacion_datos.ipynb` contiene el perfilamiento. Se abre en Colab porque pide subir el Excel, y no es necesario para generar las salidas. `avance3_etl.ipynb` hace todo el proceso desde el Excel original y genera los CSV y la bitacora. Con la misma entrada siempre produce la misma tabla preparada (lo comprueba la seccion 3.7 del notebook).

**Versiones probadas.** Google Colab y Python 3.13 en la computadora, los dos con pandas 2.2 y openpyxl 3.1. Tarda menos de un minuto.

## Salidas

| Archivo | Que contiene | Filas |
|---|---|---|
| `data/processed/ventas_preparadas.csv` | Ventas con descuento de US, LATAM, EU y APAC, listas para el analisis. Una fila por pedido + producto, con 23 columnas | 19,141 |
| `data/processed/registros_excluidos.csv` | Filas que no entran al analisis, con la regla (`Regla`) y el motivo (`Motivo`) | 32,114 |
| `data/processed/registros_pendientes.csv` | Filas que no pasaron algun control, con la regla que fallo. En esta ejecucion esta vacio | 0 |
| `bitacora/bitacora.csv` | Un renglon por paso del ETL, con filas antes y despues, regla, resultado y observaciones | 11 |

Conciliacion: 51,290 filas originales = 6 (R2) + 10,271 (R4) + 21,861 (R5) + 0 pendientes + 19,152 preparadas. Las 19,152 quedan en 19,141 filas porque 11 parejas se unieron con la regla R3.

## Reglas de negocio

| Regla | Que hace |
|---|---|
| R1 | Redondea a dos decimales los 629 descuentos con tres decimales (Copiers de LATAM) y guarda el original en *Descuento original* |
| R2 | Elimina las 3 parejas con el mismo pedido y producto pero con cliente distinto (6 filas) |
| R3 | Une las 35 parejas que son un pedido enviado en dos partes, sumando cantidad, ventas, utilidad y costo de envio |
| R4 | Deja solo US, LATAM, EU y APAC |
| R5 | Deja solo ventas con descuento mayor a 0 |
| R6 | No cuenta el costo de envio en la utilidad, porque lo paga el cliente. El margen sale solo de Utilidad y Ventas |

## Donde esta cada evidencia (guia del avance 3)

| Seccion de la guia | Donde |
|---|---|
| 3.1 Continuidad | `notebooks/avance3_etl.ipynb`, seccion 3.1 |
| 3.2 Integracion | Notebook, seccion 3.2 |
| 3.3 Contrato y ETL | Notebook, seccion 3.3 (funciones `extraer`, `normalizar`, `r1` a `r3`, `separar`, `derivar`, `revisar`, `preparar` y `guardar`) |
| 3.4 Conciliacion y validacion | Notebook, seccion 3.4 |
| 3.5 Estadistica descriptiva | Notebook, seccion 3.5 (medidas M1 a M5) |
| 3.6 Bitacora automatica | `bitacora/bitacora.csv` y notebook, seccion 3.6 |
| 3.7 Reproducibilidad y limitaciones | Notebook, seccion 3.7 |
| Informe | `docs/informe_avance3.md` |
| Bitacora del equipo | `bitacora/bitacora_equipo.md` |
