# Trabajo Práctico: Datos Personales

## Recolección de datos

Mis datos personales consisten de mi consumo de café y de la calidad de mi sueño.

### Café

Para recolectar mis datos de consumó de café simplemente anoté la fecha, la
cantidad de café que consumí y a qué hora. Generalmente consumo café de filtro
negro solo, pero de vez en cuando también tomo algún Mocha que contiene
espresso no café de filtro. Para esos casos investigué las cantidades de
cafeína que contiene el café de filtro y el espresso y normalicé todos los
resultados al equivalente en café de filtro. No es un valor exacto ya que eso
depende de exactamente el grano y el proceso que se haya utilizado, pero es un
aproximado que me parece bastante válido.

### Sueño

Para recolectar mis datos de sueño utilicé una aplicación para el celular que
se llama "Sleep as Android". Al momento de irme a dormir pongo la aplicación en
su modo de "Sleep Tracking" y dado que uso esta aplicación para mi despertador,
se corta solo en el momento que me despierto. Ya con eso puede calcular la
cantidad de tiempo que estuve durmiendo. Además, esta aplicación utiliza un
sistema sonar, emite un sonido a una frecuencia alta que no se escucha (para
los humanos) y después usa el micrófono del teléfono para recolectar el sonido
del ambiente, procesa esos datos y en base a eso presenta resultados de la
calidad de sueño. En particular: Hora en la que me acosté, Horas de sueño,
veces que me desperté y horas de sueño profundo.

## Procesamiento de datos personales

La mayoría de mis datos se pueden usar tal cual fueron recolectados, excepto por uno y ese es el déficit de sueño. El déficit de sueño lo largo de un período $T$, con intervalos discretos $t$, se define como:

Sean $K=7.5$ La cantidad de horas de sueño ideales y $h_i$ la cantidad de horas

$$
deficit_t=\sum_{i}^{t}{K - h_i}
$$

Para cada entrada calculé el déficit acumulado hasta ese punto. Lo hice
directamente con sheets y además lo corroboré de la siguiente manera.

```python
# Levanto el archivo csv a un dataframe
df = pd.read_csv("datos_personales/sleep.csv")

# Acumulo el deficit en una Serie
deficit_check = pd.Series(list(accumulate(df["Horas sueño"], lambda l1, l2: 7.5-l2 + l1, initial=0))[1:])

# Agrego la serie como una columna a mi dataframe
pd.concat([df["Déficit"], deficit_check], axis=1)
```

Al imprimir los resultados...

```
    Déficit      0
0     -0.42  -0.42
1     -1.00  -1.00
2     -0.35  -0.35
3      1.05   1.05
4      0.28   0.28
5      2.72   2.71
6      2.90   2.89
7      3.98   3.97
8      5.05   5.04
9      5.98   5.97
10     5.85   5.84
11     6.07   6.06
12     8.40   8.39
13    12.35  12.34
14    13.57  13.56
15    14.10  14.09
16    12.77  12.76
17    11.92  11.91
18     9.83   9.83
19    10.42  10.41
20    11.18  11.18
21    12.48  12.48
22    13.07  13.06
23    13.95  13.94
24    14.78  14.77
25    15.87  15.85
```

Podemos ver que son iguales (salvo algunas diferencias de redondeo)

## Armado de CSV

```python
# Armo dataframes con los archivos que descargue de sheets
df_sleep = pd.read_csv("datos_personales/sleep.csv")
df_coffee = pd.read_csv("datos_personales/coffee.csv")

# Paso las fechas de string a Date
df_sleep['Fecha'] = pd.to_datetime(df_sleep['Fecha'])
df_coffee['Fecha'] = pd.to_datetime(df_coffee['Fecha'])
```

Algunas de las herramientas que uso para visualizar no tienen la capacidad de
agregar datos en el momento, además para los gráficos exploratorios no estoy
usando todas las columnas de mi dataset, así que uso este paso para armar un
CSV simplificado que contenga tan solo 3 columnas: "Fecha", "Hora sueño",
"SUM(Consumo de cafe)".

```python
# Agrupo por fecha y sumo las cantidades de cafe
agg_coffee = pd.DataFrame(df_coffee.groupby(['Fecha'])['Cantidad (ml)'].sum())

# Hago un Join con el dataframe de horas de sueño en la Fecha y poniendo 0 como
# cantidad de cafe para aquellos dias que no tome cafe
df_simple = pd.merge(df_sleep[["Fecha", "Horas sueño"]], agg_coffee, on="Fecha", how="left").fillna(0)
```

Además, agrego minutos de sueño como una columna porque en algunos casos es más
fácil comparar números de magnitudes semejantes

```python
df_simple[Minutos sueño"] = df_simple["Horas sueño"].map(lambda x: round(x*60))"
```

Finalmente, lo escribo a un archivo csv.

```python
df_simple.to_csv("datos_personales/simple.csv", index=False)
```
