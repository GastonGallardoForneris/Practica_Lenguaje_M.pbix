# Práctica: Lenguaje M en Power Query

**Autor:** Gastón Gallardo Forneris

## Dataset utilizado

Archivo: ventas.csv.

Fuente pública:
https://github.com/Ricojacob01/Latam_resources_spanish/blob/main/Data/retail_ventas/ventas.csv

Elegí este dataset porque está en español, tiene 700 registros y 8 columnas, e incluye textos, números y fechas. Su tamaño permite practicar y revisar las transformaciones fácilmente.

Contiene identificadores técnicos, como venta_id y producto_id, que permiten trabajar con nombres y tipos de datos. En la revisión no se encontraron celdas vacías ni filas completamente duplicadas.

El archivo está publicado en un repositorio público con contenido accesible para su revisión. No pude verificar el método de generación de los datos, por lo que lo utilizo con finalidad educativa y no como evidencia de ventas reales.

## Transformaciones realizadas

1. Cambié venta_id de número entero a texto porque representa un identificador, no una cantidad.
2. Renombré venta_id como id_ventas para identificar la columna en español.
3. Filtré ciudad para conservar únicamente las ventas de Buenos Aires.
4. Corregí monto_total a número decimal y utilicé la configuración regional en-US porque el CSV usa punto como separador decimal.
5. En el Editor Avanzado renombré el paso de cambio de nombre como "Renombrar identificador de venta".
6. Actualicé la referencia a ese paso dentro de Table.SelectRows y agregué comentarios técnicos con //.

El filtro delimita el análisis a Buenos Aires; no elimina registros del archivo original.

## Código M completo

La ruta de File.Contents corresponde a mi computadora.
Para ejecutar la consulta en otro equipo, debe ajustarse a la ubicación del CSV.

```powerquery
let
    Origen = Csv.Document(
        File.Contents("C:\Users\gasst\OneDrive\Documents\Curso Analista de Datos\Analisis de datos\Power BI\ventas.csv"),
        [
            Delimiter = ",",
            Columns = 8,
            Encoding = 65001,
            QuoteStyle = QuoteStyle.None
        ]
    ),

    #"Encabezados promovidos" = Table.PromoteHeaders(
        Origen,
        [PromoteAllScalars = true]
    ),

    // El CSV usa punto decimal: en-US permite interpretar los importes correctamente.
    #"Tipo de columna cambiado" = Table.TransformColumnTypes(
        #"Encabezados promovidos",
        {
            {"venta_id", Int64.Type},
            {"fecha", type date},
            {"tienda", type text},
            {"ciudad", type text},
            {"producto_id", Int64.Type},
            {"cantidad", Int64.Type},
            {"monto_total", type number},
            {"medio_pago", type text}
        },
        "en-US"
    ),

    // El identificador es un código, no una cantidad para sumar.
    #"Tipo de columna cambiado 1" = Table.TransformColumnTypes(
        #"Tipo de columna cambiado",
        {{"venta_id", type text}}
    ),

    // Usamos un nombre descriptivo para identificar la clave de cada venta.
    #"Renombrar identificador de venta" = Table.RenameColumns(
        #"Tipo de columna cambiado 1",
        {{"venta_id", "id_ventas"}}
    ),

    // Delimitamos el análisis a Buenos Aires sin modificar el archivo original.
    #"Filas filtradas" = Table.SelectRows(
        #"Renombrar identificador de venta",
        each [ciudad] = "Buenos Aires"
    )
in
    #"Filas filtradas"
```

## ¿Por qué es útil entender let … in?

El bloque let permite definir pasos con nombres y utilizar sus resultados en otros pasos. El bloque in indica qué resultado devuelve la consulta.

Entender esta estructura permite seguir las dependencias, localizar errores y modificar transformaciones sin romper las referencias.

En este ejercicio, el paso "Filas filtradas" utiliza el resultado de "Renombrar identificador de venta". Por eso, al cambiar el nombre del paso, también actualicé su referencia dentro del filtro.

## ¿Qué significa que M sea Case Sensitive?

Significa que distingue mayúsculas y minúsculas.

Por ejemplo, Table.SelectRows y table.selectrows son nombres diferentes. La segunda forma no identifica la función correcta y genera un error.

También deben respetarse las mayúsculas y minúsculas de los nombres de pasos y columnas. Cambiar un nombre sin actualizar sus referencias puede impedir que la consulta se ejecute.

## Resultado

La consulta devuelve las filas de Buenos Aires, conserva los importes decimales y utiliza id_ventas como identificador de texto.

El proyecto se guardó como Practica_Lenguaje_M.pbix.
