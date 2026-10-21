# Catálogo de películas con listas de selección

**Autor:** Paula Rossi. Programación Orientada a Datos, Lic. en Ciencia de Datos, UCA Rosario, 2026.

## Problema

Para armar una lista de películas recomendadas hay que buscar por título, género o año. Además hay que tener en cuenta cuántas opiniones respaldan cada valoración. Si se ordena solo por la media, una película con una única valoración de 5 queda arriba de otras con cientos de valoraciones buenas.

## Usuario

Cualquier persona que arme listas de películas, como un cineclub, un docente, alguien que escribe un blog o un grupo de amigos. El resultado le sirve para elegir títulos bien valorados y con respaldo suficiente.

## Solución propuesta

Armé un programa que:

1. lee un catálogo de películas y sus valoraciones, en **CSV o JSON**;
2. valida y limpia los datos, y registra cada fila rechazada con su motivo;
3. filtra por **texto del título**, **género** y **año**;
4. exige un **mínimo configurable de valoraciones**;
5. ordena y exporta la lista resultante.

**Alcance de la versión 1:** búsqueda y filtros simples (un género por consulta) y media aritmética.
**Queda fuera:** recomendaciones personalizadas, interfaz gráfica, uso del `timestamp` y la base de etiquetas (`tags.csv`).

## Instalación

Requiere **Python 3.10 o superior**. No usa paquetes externos.

```
git clone https://github.com/USUARIO/tp2-Paula-Rossi.git
cd tp2-Paula-Rossi
```

## Uso

Todos los comandos se ejecutan desde la raíz del repositorio.

```
# Lista general con la muestra (CSV)
python -m src.main

# Comedias con al menos 3 valoraciones, top 5
python -m src.main --genero Comedy --min-valoraciones 3 --top 5

# La misma consulta leyendo JSON
python -m src.main --peliculas data/peliculas_muestra.json --ratings data/ratings_muestra.json --genero Comedy --min-valoraciones 3

# Buscar por texto y año
python -m src.main --texto "star" --anio 1977
```

| Parámetro | Descripción | Por defecto |
|---|---|---|
| `--peliculas` | archivo de películas (.csv/.json) | `data/peliculas_muestra.csv` |
| `--ratings` | archivo de valoraciones (.csv/.json) | `data/ratings_muestra.csv` |
| `--texto` | fragmento del título (no distingue mayúsculas) | sin filtro |
| `--genero` | género exacto (`Comedy`, `Drama`, ...) | sin filtro |
| `--anio` | año de estreno | sin filtro |
| `--min-valoraciones` | mínimo de valoraciones | 1 |
| `--top` | cantidad de resultados | 10 |
| `--salida` | CSV de la lista | `outputs/lista.csv` |
| `--rechazos` | CSV de filas rechazadas | `outputs/rechazos.csv` |

## Pruebas

```
python -m unittest -v
```

Se prueban un **caso normal** (con valoraciones 3 y 5 la media da 4 y el conteo 2), **entradas inválidas y casos límite** (rating fuera de escala, película inexistente, id duplicado, búsqueda sin resultados, JSON mal formado) y **propiedades del resultado** (CSV y JSON dan lo mismo, aceptadas + rechazadas = filas leídas, la lista está ordenada).

## Organización del código

```
src/
  fuentes.py   FuenteDatos (abstracta) -> FuenteCSV, FuenteJSON. Todas devuelven una lista de diccionarios.
  modelo.py    ConsultaCatalogo: guarda y valida los filtros; coincide(pelicula).
  pipeline.py  validar_esquema, extraer_anio (regex), construir_catalogo,
               acumular_valoraciones, buscar_titulos, generar_lista, verificar_resultado.
  main.py      argumentos de consola, coordina el flujo y guarda las salidas.
tests/         pruebas con unittest.
scripts/       generar_muestra.py: crea la muestra CSV/JSON reproducible.
data/          muestra y documentación de los datos (ver data/README.md).
```

Flujo: `cargar → validar_esquema → construir_catalogo / acumular_valoraciones (limpieza) → generar_lista → verificar_resultado → guardar_csv`.

## Expresión regular

`\((\d{4})\)\s*$` extrae el año que aparece **entre paréntesis al final** del título (por ejemplo, `Toy Story (1995)` da 1995).

- La regex **solo valida el formato**. Después se comprueba que el año esté entre 1874 y el año actual.
- Si no hay año (`Babylon 5`), o el año es un rango o está fuera de rango, el año queda como **"sin año"**. No se inventa un valor.

## Decisiones de limpieza

| Situación | Decisión |
|---|---|
| `movieId` no entero o ≤ 0 | se rechaza la fila |
| título vacío | se rechaza |
| `movieId` repetido en el catálogo | se conserva la **primera** aparición |
| `(no genres listed)` | lista de géneros vacía ("sin género") |
| rating no numérico o fuera de 0.5–5.0, o que no es múltiplo de 0.5 | se rechaza |
| valoración de una película que no está en el catálogo | se rechaza ("película inexistente") |
| mismo usuario valora dos veces la misma película | se conserva la primera (**no se duplican opiniones**) |

Todas las filas rechazadas se guardan en `outputs/rechazos.csv` con el archivo, el número de fila, el motivo y el contenido original.

**Regla de orden:** media descendente; a igual media, más valoraciones primero; luego título alfabético.

## Ejemplo de resultado

```
$ python -m src.main --min-valoraciones 3 --top 5
 #  Media     N      Año  Título
 1   4.20     5     1993  Schindler's List
 2   4.00     5     1995  Jumanji
 3   4.00     5     1991  Terminator 2: Judgment Day
 4   4.00     3     1993  Jurassic Park
 5   3.83     6     1977  Star Wars: Episode IV - A New Hope
Filas rechazadas: 7
```

## Interpretación y limitaciones

- Con `--min-valoraciones 1`, *Grumpier Old Men* aparece primera con una sola valoración de 5.0. Con un mínimo de 3 desaparece de la lista. **Este es el problema que la aplicación resuelve.**
- Las valoraciones de la muestra son **sintéticas**. Para conclusiones reales hay que usar MovieLens (ver `data/README.md`).
- La media no corrige el sesgo de los usuarios de MovieLens, y el mínimo elegido es una decisión de quien usa la aplicación.

## Fuente de datos

MovieLens latest-small, de GroupLens Research. Licencia de uso no comercial. Detalle en [`data/README.md`](data/README.md).
