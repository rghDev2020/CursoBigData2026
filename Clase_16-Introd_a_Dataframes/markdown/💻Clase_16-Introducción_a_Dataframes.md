# 💻Clase 16 - Introducción a Dataframes

---

# Agenda:

<aside>
💡

#### 9:00 - 9:50    →  Sesión 1  - Repaso CombinebyKey

#### 9:50 - 11:20   → Ejercicios y caso de uso

#### **11:20 - 11:40 → Descanso**

#### 11:40 - 12:40  → Sesión 2 - Introducción a Dataframes

#### 12:40 - 14:00  → Ejercicios y caso de uso

</aside>

# Sesión 1 :

---

# 1.1 - Pasos para resolver los problemas con la compatibilidad de java 17 y con la librería kryo en los notebooks de Jupyter:

### 1.  Cierra VS Code por completo y todas las terminanes que tengas abiertas.

### 2. Abre PowerShell de nuevo.

Ejecuta esto:

```python
$env:JAVA_HOME="C:\java\jdk-17.0.18.8-hotspot"
$env:Path="$env:JAVA_HOME\bin;$env:Path"
$env:JAVA_TOOL_OPTIONS="--add-opens=java.base/sun.nio.ch=ALL-UNNAMED --add-opens=java.base/java.nio=ALL-UNNAMED --add-opens=java.base/java.lang=ALL-UNNAMED --add-opens=java.base/java.lang.invoke=ALL-UNNAMED --add-opens=java.base/java.util=ALL-UNNAMED"
java -version
code .

```

Lo importante aquí es:

- forzar **JDK 17**
- añadir los `-add-opens`
- abrir **VS Code desde esa misma consola**

### 3. Abre tu notebook y reinicia el kernel.

Dentro de VS Code:

- abre el `.ipynb`
- usa **Restart Kernel**
- vuelve a crear `SparkSession` desde cero.

### 4) Comprueba **dentro del notebook** que ya estás en Java 17.

Primero ejecuta esto en una celda de jupyter:

```scala
System.getProperty("java.version")
```

Debería devolverte algo tipo:

```scala
"17.0.18"
```

Luego crea Spark otra vez:

```scala
import $ivy.`org.apache.spark::spark-sql:4.1.1`
import org.apache.spark.sql.SparkSession

val spark = SparkSession.builder()
  .appName("Spark411-Almond")
  .master("local[*]")
  .getOrCreate()

val sc = spark.sparkContext

println(s"Java: ${System.getProperty("java.version")}")
println(s"Spark: ${spark.version}")
println(s"Scala: ${scala.util.Properties.versionNumberString}")
```

### 5) Prueba con un wordcount:

```scala
val lineas = sc.parallelize(List(
  "apache spark es un motor de procesamiento distribuido",
  "spark procesa datos en memoria de forma muy eficiente",
  "scala es el lenguaje nativo de apache spark",
  "spark tiene apis en scala python java y r"
))

val palabras = lineas.flatMap(linea => linea.split(" "))
val pares = palabras.map(palabra => (palabra, 1))
val conteo = pares.reduceByKey((a, b) => a + b)
val ordenado = conteo.sortBy({ case (_, count) => count }, ascending = false)

val top10 = ordenado.take(10)

println("Top 10 palabras más frecuentes:")
println("─" * 35)
top10.foreach { case (palabra, count) =>
  println(f"  $palabra%-30s → $count veces")
}
```

 Si esta opción falla , se debe a que el kernel Almond no ha cargado los `--add-opens` necesarios.

### 6.  Localiza la carpeta del kernel.

Dependiendo del equipo, tu kernel podría estar en una ruta similar a esta:

```scala
C:\Users\Imp_06 - Mañana\AppData\Roaming\jupyter\kernels\scala21318
```

<aside>

Si no la encuentras escribe esto en un nuevo terminal de powershell:

```scala
explorer "$env:APPDATA\jupyter\kernels\
```

 Buscas una carpeta que diga scala 2.13 o 2.13.18 , entras a la carpeta.

</aside>

Ahí debe haber un archivo llamado:

```scala
kernel.json
```

### 7.  Abre `kernel.json`

Ábrelo con VS Code o con el Bloc de notas. Veras algo parecido a esto:

```json
{
  "argv": [
    "java",
    "-cp",
    "C:\\Users\\Imp_06 - Mañana\\AppData\\Roaming\\jupyter\\kernels\\scala21318\\launcher.jar",
    "coursier.bootstrap.launcher.Launcher",
    "--id",
    "scala21318",
    "--display-name",
    "Scala 2.13.18 (Almond)",
    "--connection-file",
    "{connection_file}"
  ],
  "display_name": "Scala 2.13.18 (Almond)",
  "language": "scala"
}
```

### 8. Añade los `--add-opens` dentro de `argv` , entre `java` y `-cp`:

```json
--add-opens=java.base/java.nio=ALL-UNNAMED
--add-opens=java.base/sun.nio.ch=ALL-UNNAMED
--add-opens=java.base/java.lang=ALL-UNNAMED
--add-opens=java.base/java.lang.invoke=ALL-UNNAMED
--add-opens=java.base/java.util=ALL-UNNAMED
```

Debería quedar en el JSON de esta forma:

```json
{
  "argv": [
    "java",
    "--add-opens=java.base/java.nio=ALL-UNNAMED",
    "--add-opens=java.base/sun.nio.ch=ALL-UNNAMED",
    "--add-opens=java.base/java.lang=ALL-UNNAMED",
    "--add-opens=java.base/java.lang.invoke=ALL-UNNAMED",
    "--add-opens=java.base/java.util=ALL-UNNAMED",
    "-cp",
    "C:\\Users\\Imp_06 - Mañana\\AppData\\Roaming\\jupyter\\kernels\\scala21318\\launcher.jar", 
    "coursier.bootstrap.launcher.Launcher",
    "--id",
    "scala21318",
    "--display-name",
    "Scala 2.13.18 (Almond)",
    "--connection-file",
    "{connection_file}"
  ],
  "display_name": "Scala 2.13.18 (Almond)",
  "language": "scala"
}
```

<aside>

Tener en cuenta que si copias y pegas todo este JSON puede darte errores por que la ruta C:\\Users\\Imp_06 - Mañana\\AppData\… cambia en cada equipo. Solo tienes que añadir a tu archivo JSON entre:

```scala
{
  "argv": [
    "java",
    "aqui van los add-opens", 
    "-cp",
    "C:\\Users\\Imp_06 - Mañana\\AppData\\Roaming\\jupyter\\kernels\\scala21318\\launcher.jar", 
    "coursier.bootstrap.launcher.Launcher",
    "--id",
    "scala21318",
    "--display-name",
    "Scala 2.13.18 (Almond)",
    "--connection-file",
    "{connection_file}"
  ],
  "display_name": "Scala 2.13.18 (Almond)",
  "language": "scala"
}
```

</aside>

### 9. Ahora haz esto exactamente:

- guarda el `kernel.json`
- cierra **todo** VS Code
- vuelve a abrir VS Code
- abre el notebook
- reinicia el kernel
- vuelve a crear `SparkSession`

Después prueba primero esta celda:

```json
System.getProperty("java.version")
```

Y luego esta:

```scala
import $ivy.`org.apache.spark::spark-sql:4.1.1`
import org.apache.spark.sql.SparkSession

val spark = SparkSession.builder()
  .appName("Spark411-Almond")
  .master("local[*]")
  .getOrCreate()

val sc = spark.sparkContext

println(s"Java: ${System.getProperty("java.version")}")
println(s"Spark: ${spark.version}")
println(s"Scala: ${scala.util.Properties.versionNumberString}")
```

Finalmente repite el ejercicio que te había dado problemas o prueba con un wordcount:

```scala
val lineas = sc.parallelize(List(
  "apache spark es un motor de procesamiento distribuido",
  "spark procesa datos en memoria de forma muy eficiente",
  "scala es el lenguaje nativo de apache spark",
  "spark tiene apis en scala python java y r"
))

val palabras = lineas.flatMap(linea => linea.split(" "))
val pares = palabras.map(palabra => (palabra, 1))
val conteo = pares.reduceByKey((a, b) => a + b)
val ordenado = conteo.sortBy({ case (_, count) => count }, ascending = false)

val top10 = ordenado.take(10)

println("Top 10 palabras más frecuentes:")
println("─" * 35)
top10.foreach { case (palabra, count) =>
  println(f"  $palabra%-30s → $count veces")
}
```

## 1.2 - Ejemplo ampliado de:  `combineByKey`

```scala
import $ivy.`org.apache.spark::spark-sql:4.1.1`
import org.apache.spark.sql.SparkSession

val spark = SparkSession.builder()
  .appName("PairRDDs")
  .master("local[*]")
  .getOrCreate()

val sc = spark.sparkContext

// RDD de ventas como cadenas: "producto,región,importe"
val ventas = sc.parallelize(List(
  "Laptop,Norte,1200",
  "Teclado,Sur,45",
  "Monitor,Norte,350",
  "Laptop,Sur,1200",
  "Ratón,Norte,25",
  "Monitor,Sur,350",
  "Teclado,Norte,45"
))

// Convertimos a Pair RDD: (región, importe)
val ventasPorRegion = ventas.map { linea =>
  val campos = linea.split(",")
  val region  = campos(1)
  val importe = campos(2).toDouble
  (region, importe)   // ← tupla (clave, valor)
}

ventasPorRegion.collect().foreach(println)
// (Norte,1200.0)
// (Sur,45.0)
// (Norte,350.0)
// (Sur,1200.0)
// (Norte,25.0)
// (Sur,350.0)
// (Norte,45.0)

// Objetivo: para cada región → List con todos los importes
val importesPorRegion = ventasPorRegion.combineByKey(
  (v: Double) => List(v),                       // createCombiner: primer valor → List
  (acc: List[Double], v: Double) => acc :+ v,   // mergeValue: añadir al List
  (acc1: List[Double], acc2: List[Double]) => acc1 ++ acc2 // mergeCombiners: unir Lists 
)

importesPorRegion.collect().foreach { case (region, lista) =>
  println(s"$region → $lista")
}
// Norte → List(1200.0, 350.0, 25.0, 45.0)
// Sur   → List(45.0, 1200.0, 350.0)
```

---

Dado un RDD con estructura:

```scala
(region, importe)
```

Queremos obtener:

```scala
region -> List(importes)
```

Ejemplo:

```scala
(Norte,1200.0)
(Sur,45.0)
(Norte,350.0)
(Sur,1200.0)
(Norte,25.0)
(Sur,350.0)
(Norte,45.0)
```

Resultado esperado:

```scala
Norte -> List(1200.0, 350.0, 25.0, 45.0)
Sur   -> List(45.0, 1200.0, 350.0)
```

---

## 🧠 Idea Clave

`combineByKey` trabaja en **tres fases**:

1. Crear acumulador (por primera vez que aparece una clave)
2. Añadir valores al acumulador
3. Unir acumuladores entre particiones

---

## 🧱 Sintaxis General

```scala
combineByKey(
  createCombiner,
  mergeValue,
  mergeCombiners
)
```

---

# 🚀 Paso 1: Datos de entrada

```scala
val ventasPorRegion = ventas.map { linea =>
  val campos = linea.split(",")
  val region = campos(1)
  val importe = campos(2).toDouble
  (region, importe)
}
```

Salida:

```scala
(Norte,1200.0)
(Sur,45.0)
(Norte,350.0)
(Sur,1200.0)
(Norte,25.0)
(Sur,350.0)
(Norte,45.0)
```

---

# 🔹 Paso 2: createCombiner

```scala
(v: Double) => List(v)
```

## 📌 ¿Qué hace?

Se ejecuta cuando **una clave aparece por primera vez**.

## 🔍 Ejemplo

```scala
(Norte, 1200.0)
```

Resultado:

```scala
Norte -> List(1200.0)
```

---

# 🔹 Paso 3: mergeValue

```scala
(acc: List[Double], v: Double) => acc :+ v
```

## 📌 ¿Qué hace?

Se ejecuta cuando **la clave ya existe** y llega un nuevo valor.

## 🔍 Ejemplo

Estado actual:

```scala
Norte -> List(1200.0)
```

Llega:

```scala
(Norte, 350.0)
```

Resultado:

```scala
Norte -> List(1200.0, 350.0)
```

---

# 🔹 Paso 4: mergeCombiners

```scala
(acc1: List[Double], acc2: List[Double]) => acc1 ++ acc2
```

## 📌 ¿Qué hace?

Se ejecuta cuando Spark **une resultados de distintas particiones**.

## 🔍 Ejemplo

Partición 1:

```scala
Norte -> List(1200.0, 350.0)
```

Partición 2:

```scala
Norte -> List(25.0, 45.0)
```

Resultado final:

```scala
Norte -> List(1200.0, 350.0, 25.0, 45.0)
```

---

# 🔁 Paso 5: Ejecución completa

```scala
val importesPorRegion = ventasPorRegion.combineByKey(
  (v: Double) => List(v),
  (acc: List[Double], v: Double) => acc :+ v,
  (acc1: List[Double], acc2: List[Double]) => acc1 ++ acc2
)
```

---

# 🖥️ Paso 6: Versión con prints

```scala
val importesPorRegion = ventasPorRegion.combineByKey(

  (v: Double) => {
    println(s"[CREATE] Nuevo acumulador con: $v")
    List(v)
  },

  (acc: List[Double], v: Double) => {
    println(s"[MERGE VALUE] Lista actual: $acc | Añadiendo: $v")
    acc :+ v
  },

  (acc1: List[Double], acc2: List[Double]) => {
    println(s"[MERGE COMBINERS] Uniendo: $acc1 y $acc2")
    acc1 ++ acc2
  }

)

importesPorRegion.collect().foreach(println)
```

---

# 📦 Visualización de Particiones + Ejercicios `combineByKey`

---

# 🔍 Función para ver datos por partición

```scala
def verParticiones[T](rdd: org.apache.spark.rdd.RDD[T]): Unit = {
  rdd.mapPartitionsWithIndex { (index, iter) =>
    Iterator(s"📦 Partición $index -> ${iter.toList.mkString(", ")}")
  }.collect().foreach(println)
}
```

---

# 🧪 Ejercicio 1: Temperaturas por ciudad

## 📥 Datos

```scala
val temperaturas = sc.parallelize(List(
  ("Madrid", 18.5),
  ("Barcelona", 20.0),
  ("Madrid", 21.0),
  ("Valencia", 22.5),
  ("Sevilla", 25.0),
  ("Barcelona", 19.5),
  ("Madrid", 17.0),
  ("Valencia", 23.0),
  ("Sevilla", 26.5),
  ("Barcelona", 18.0),
  ("Madrid", 20.5),
  ("Valencia", 21.5)
), 3)

verParticiones(temperaturas)
```

## 🎯 Objetivo

Agrupar las temperaturas registradas por ciudad.

## ✅ Salida esperada (aproximada)

```scala
(Madrid,List(18.5, 21.0, 17.0, 20.5))
(Barcelona,List(20.0, 19.5, 18.0))
(Valencia,List(22.5, 23.0, 21.5))
(Sevilla,List(25.0, 26.5))
```

---

# 🧪 Ejercicio 2: Productos comprados por cliente

## 📥 Datos

```scala
val compras = sc.parallelize(List(
  ("Ana", "Laptop"),
  ("Luis", "Teclado"),
  ("Ana", "Mouse"),
  ("Marta", "Monitor"),
  ("Luis", "Tablet"),
  ("Ana", "Auriculares"),
  ("Carlos", "Impresora"),
  ("Marta", "Webcam"),
  ("Luis", "Ratón"),
  ("Carlos", "SSD"),
  ("Ana", "USB"),
  ("Marta", "Silla")
), 3)

verParticiones(compras)
```

## 🎯 Objetivo

Agrupar los productos comprados por cliente.

## ✅ Salida esperada (aproximada)

```scala
(Ana,List(Laptop, Mouse, Auriculares, USB))
(Luis,List(Teclado, Tablet, Ratón))
(Marta,List(Monitor, Webcam, Silla))
(Carlos,List(Impresora, SSD))
```

---

# 🧪 Ejercicio 3: Notas por estudiante

## 📥 Datos

```scala
val notas = sc.parallelize(List(
  ("Carlos", 8.5),
  ("Marta", 7.0),
  ("Carlos", 9.0),
  ("Ana", 6.5),
  ("Marta", 8.0),
  ("Ana", 7.5),
  ("Luis", 5.5),
  ("Carlos", 10.0),
  ("Luis", 6.0),
  ("Marta", 9.0),
  ("Ana", 8.5),
  ("Luis", 7.0)
), 3)

verParticiones(notas)
```

## 🎯 Objetivo

Agrupar las notas por estudiante.

## ✅ Salida esperada (aproximada)

```scala
(Carlos,List(8.5, 9.0, 10.0))
(Marta,List(7.0, 8.0, 9.0))
(Ana,List(6.5, 7.5, 8.5))
(Luis,List(5.5, 6.0, 7.0))
```

---

# 🧪 Ejercicio 4: Palabras por inicial

## 📥 Datos

```scala
val palabras = sc.parallelize(List(
  ("A", "Azure"),
  ("S", "Spark"),
  ("A", "Almond"),
  ("S", "Scala"),
  ("P", "Python"),
  ("J", "Java"),
  ("A", "Airflow"),
  ("S", "SQL"),
  ("P", "PostgreSQL"),
  ("J", "Jupyter"),
  ("A", "AWS"),
  ("P", "PySpark")
), 3)

verParticiones(palabras)
```

## 🎯 Objetivo

Agrupar palabras según su inicial.

## ✅ Salida esperada (aproximada)

```scala
(A,List(Azure, Almond, Airflow, AWS))
(S,List(Spark, Scala, SQL))
(P,List(Python, PostgreSQL, PySpark))
(J,List(Java, Jupyter))
```

---

# 🧠 Plantilla para resolver todos los ejercicios

```scala
val resultado = rdd.combineByKey(
  (v: TipoValor) => List(v),
  (acc: List[TipoValor], v: TipoValor) => acc :+ v,
  (acc1: List[TipoValor], acc2: List[TipoValor]) => acc1 ++ acc2
)

resultado.collect().foreach(println)
```

---

# 🧪 Caso de Estudio: Análisis de Actividad de Usuarios en una Plataforma Digital

---

## 🏢 Contexto empresarial

Una empresa de e-learning (tipo plataforma de cursos online) quiere analizar el comportamiento de sus usuarios. Cada vez que un usuario realiza una acción en la plataforma (ver un vídeo, completar una lección, hacer un test, etc.), se genera un registro. El equipo de Data Engineering necesita preparar los datos para que el equipo de Data Science pueda analizarlos posteriormente.

---

## 📥 Datos de entrada

Cada registro tiene la siguiente estructura:

```
usuario, tipo_evento, duracion_minutos
```

Ejemplo de dataset:

```scala
val eventos = sc.parallelize(List(
  ("user1", "video", 10.0),
  ("user2", "quiz", 5.0),
  ("user1", "video", 15.0),
  ("user3", "video", 20.0),
  ("user2", "video", 8.0),
  ("user1", "quiz", 7.0),
  ("user3", "quiz", 6.0),
  ("user2", "video", 12.0),
  ("user1", "video", 9.0),
  ("user3", "video", 11.0),
  ("user2", "quiz", 4.0),
  ("user3", "video", 13.0)
), 3)
```

---

## 🔍 Paso 1: Visualizar particiones

```scala
verParticiones(eventos)
```

---

Construir una estructura donde:

```scala
usuario -> List(duraciones)
```

Es decir, agrupar todas las duraciones de actividad por usuario.

## 🧠 ¿Por qué usar `combineByKey`?

Porque queremos:

- Crear una estructura personalizada (List[Double])
- Trabajar eficientemente en entorno distribuido
- Evitar el uso de `groupByKey` (menos eficiente)

---

## 🔄 Transformación necesaria

Primero debemos transformar los datos a:

```scala
(usuario, duracion)
```

---

## 🧪 Tarea propuesta

### 1️⃣ Transformar el dataset

```scala
val eventosPorUsuario = eventos.map { case (user, tipo, duracion) =>
  (user, duracion)
}
```

---

### 2️⃣ Aplicar `combineByKey`

Debes implementar:

```scala
val resultado = eventosPorUsuario.combineByKey(
  ???,
  ???,
  ???
)
```

---

## 📌 Resultado esperado (aproximado)

```scala
(user1,List(10.0, 15.0, 7.0, 9.0))
(user2,List(5.0, 8.0, 12.0, 4.0))
(user3,List(20.0, 6.0, 11.0, 13.0))
```

---

## 🚀 Extensión

Una vez tengas la lista de duraciones por usuario, calcula:

### 🔹 Tiempo total por usuario

```scala
user -> suma_duraciones
```

### 🔹 Tiempo medio por usuario

```scala
user -> promedio_duraciones
```

---

## 💡 Preguntas para reflexión

1. ¿Qué ocurre dentro de cada partición?
2. ¿Cuándo se ejecuta `mergeCombiners`?
3. ¿Por qué `combineByKey` es más flexible que `reduceByKey`?
4. ¿Qué pasaría si usas `groupByKey` en este caso?

---

# Sesión 2   - Spark DataFrames I: Introducción

---

# 🧠 Teoría

## 1. Limitaciones de los RDDs: por qué surgieron los DataFrames

En clases anteriores hemos usado RDDs para procesar datos. Los RDDs son potentes y flexibles, pero tienen un problema importante: **Spark no sabe nada sobre los datos que contienen**. Cuando escribes `rdd.map(linea => linea.split(","))`, Spark solo ve que tiene un RDD de cadenas y que le aplicas una función. No sabe qué hay dentro de esas cadenas, si son números o texto, si algunas columnas son nulas, ni cómo está estructurado el dato. Esto impide cualquier tipo de optimización automática.

| Problema en RDDs | Consecuencia |
| --- | --- |
| Sin conocimiento de la estructura | Spark no puede optimizar el plan de ejecución |
| Sin tipos de columna | No detecta errores hasta el momento de ejecutar |
| Sin nombres de campo | El código usa índices numéricos: `campos(0)`, `campos(2)` |
| Sin estadísticas de columna | No puede elegir el join más eficiente |

Los **DataFrames** nacen en Spark 1.3 para resolver exactamente estos problemas. Son la evolución natural de los RDDs para trabajo con datos estructurados.

> 💡 **Analogía:** un RDD es como una caja de cartón llena de papeles: sabes que hay papeles, pero no qué pone en ellos. Un DataFrame es como una hoja de cálculo con cabecera: cada columna tiene nombre, tipo y Spark puede consultarla, filtrarla y optimizarla sin que tú lo indiques.
> 

---

## 2. ¿Qué es un DataFrame en Spark?

Un **DataFrame** es una colección distribuida de datos organizada en **columnas con nombre y tipo definido**, similar conceptualmente a una tabla de base de datos relacional o a una hoja de cálculo.

```
+------+---------+------+----------+
|  id  | nombre  | edad | ciudad   |
+------+---------+------+----------+
|  1   | Ana     |  28  | Madrid   |
|  2   | Luis    |  34  | Barcelona|
|  3   | Marta   |  22  | Sevilla  |
+------+---------+------+----------+
```

Características clave:

- **Distribuido:** los datos están repartidos entre los nodos del clúster, igual que un RDD.
- **Inmutable:** cada transformación produce un nuevo DataFrame; el original no cambia.
- **Lazy:** las transformaciones no se ejecutan hasta que se llama a una acción.
- **Con schema:** cada columna tiene un nombre y un tipo de dato conocido por Spark.
- **Optimizado:** el motor Catalyst de Spark optimiza automáticamente el plan de ejecución.

### ¿Cómo se relaciona con los RDDs?

Un DataFrame es, internamente, un `RDD[Row]` con un schema adjunto. La diferencia es que Spark sí puede inspeccionar ese schema y generar código optimizado a partir de él.

```
DataFrame  =  RDD[Row]  +  Schema
                              ↓
                    Spark puede optimizar
```

---

## 3. `SparkSession`: el punto de entrada unificado

Para trabajar con DataFrames necesitamos una `SparkSession`. Es el objeto principal desde el que creamos DataFrames, ejecutamos SQL y accedemos a la configuración de Spark. En versiones anteriores de Spark existían `SparkContext`, `SQLContext` y `HiveContext` por separado. A partir de Spark 2.0, `SparkSession` los unifica a todos.

```scala
import org.apache.spark.sql.SparkSession

val spark = SparkSession.builder()
  .appName("DataFrames_Dia14")   // nombre visible en la Spark UI
  .master("local[*]")            // modo local, todos los cores disponibles
  .getOrCreate()                 // crea o reutiliza una sesión existente

// Desde SparkSession podemos acceder al SparkContext si lo necesitamos
val sc = spark.sparkContext

println(s"Spark ${spark.version} listo")
```

> 💡 `.getOrCreate()` es importante: si ya existe una `SparkSession` activa en el proceso (por ejemplo, en la celda anterior del notebook), la reutiliza en lugar de crear una nueva. Esto evita errores por sesiones duplicadas.
> 

### Accesos frecuentes desde `spark`

| Expresión | Qué devuelve |
| --- | --- |
| `spark.sparkContext` | El `SparkContext` subyacente (para RDDs) |
| `spark.version` | Versión de Spark en uso |
| `spark.read` | `DataFrameReader` para cargar datos |
| `spark.sql("SELECT ...")` | Ejecutar SQL directamente |
| `spark.catalog` | Acceso a tablas y vistas registradas |

---

## 4. Crear DataFrames

Hay varias formas de crear un DataFrame según el origen de los datos.

### 4.1 Desde una colección en memoria

La más sencilla para pruebas y aprendizaje. Usamos `createDataFrame` con una secuencia de tuplas y le damos nombres a las columnas con `.toDF(...)`.

```scala
// Forma 1: Seq de tuplas + toDF con nombres de columna
val dfEmpleados = Seq(
  (1, "Ana García",    28, "Ingeniería"),
  (2, "Luis Martínez", 34, "Marketing"),
  (3, "Marta López",   22, "Ingeniería"),
  (4, "Pedro Ruiz",    41, "Dirección")
).toDF("id", "nombre", "edad", "departamento")

dfEmpleados.show()
// +---+-------------+----+------------+
// | id|       nombre|edad|departamento|
// +---+-------------+----+------------+
// |  1|   Ana García|  28|  Ingeniería|
// |  2|Luis Martínez|  34|   Marketing|
// |  3|  Marta López|  22|  Ingeniería|
// |  4|   Pedro Ruiz|  41|   Dirección|
// +---+-------------+----+------------+
```

> ⚠️ **Nota importante para el entorno Almond:** para usar `.toDF(...)` necesitas importar las implicits de Spark. Añade esta línea después de crear la sesión:
> 
> 
> ```scala
> import spark.implicits._
> ```
> 
> Sin este import, el compilador no encontrará el método `.toDF`.
> 

### 4.2 Desde un fichero CSV

```scala
val dfVentas = spark.read
  .option("header", "true")       // primera fila como nombres de columna
  .option("inferSchema", "true")  // detectar tipos automáticamente
  .csv("C:/Curso-Scala/datos/ventas.csv")

dfVentas.show(5)         // muestra las primeras 5 filas
dfVentas.printSchema()   // muestra la estructura con tipos
```

### 4.3 Desde un fichero JSON

```scala
val dfClientes = spark.read
  .option("multiline", "true")    // para JSON con objetos multilínea
  .json("C:/Curso-Scala/datos/clientes.json")

dfClientes.show()
dfClientes.printSchema()
```

### 4.4 Desde un fichero Parquet

```scala
// Parquet lleva el schema integrado en el propio fichero
val dfParquet = spark.read
  .parquet("C:/Curso-Scala/datos/pedidos.parquet")

dfParquet.show()
```

> 💡Empezaremos con CSV y JSON. El formato Parquet lo veremos en profundidad en el Día 17.
> 

---

## 5. Schema: la estructura del DataFrame

El **schema** describe las columnas del DataFrame: sus nombres, tipos de dato y si admiten valores nulos. Es la información que hace que Spark pueda optimizar las operaciones.

### 5.1 Inspeccionar el schema con `printSchema()`

```scala
dfEmpleados.printSchema()
// root
//  |-- id: integer (nullable = true)
//  |-- nombre: string (nullable = true)
//  |-- edad: integer (nullable = true)
//  |-- departamento: string (nullable = true)
```

La salida muestra un árbol con:

- **Nombre** de cada columna
- **Tipo de dato** (`integer`, `string`, `double`, `boolean`, `date`, ...)
- **nullable:** si la columna puede contener valores nulos (`true` = sí puede)

### 5.2 Schema por inferencia vs. schema manual

Cuando usamos `inferSchema = true`, Spark lee una muestra del fichero y adivina los tipos. Es cómodo pero tiene dos inconvenientes: es más lento (tiene que leer los datos dos veces) y puede equivocarse con columnas ambiguas (por ejemplo, un código postal "01234" podría inferirse como `integer` y perder el cero inicial).

La alternativa es definir el schema **manualmente** con `StructType` y `StructField`:

```scala
import org.apache.spark.sql.types._

val schemaPedidos = StructType(List(
  StructField("id_pedido",   IntegerType, nullable = false),
  StructField("id_cliente",  IntegerType, nullable = false),
  StructField("fecha",       StringType,  nullable = true),
  StructField("importe",     DoubleType,  nullable = true),
  StructField("completado",  BooleanType, nullable = true)
))

val dfPedidos = spark.read
  .option("header", "true")
  .schema(schemaPedidos)          // usamos el schema manual
  .csv("C:/Curso-Scala/datos/pedidos.csv")

dfPedidos.printSchema()
// root
//  |-- id_pedido: integer (nullable = false)
//  |-- id_cliente: integer (nullable = false)
//  |-- fecha: string (nullable = true)
//  |-- importe: double (nullable = true)
//  |-- completado: boolean (nullable = true)
```

### Tipos de dato más habituales en Spark

| Tipo Spark | Clase Scala equivalente | Ejemplo de valor |
| --- | --- | --- |
| `IntegerType` | `Int` | `42` |
| `LongType` | `Long` | `1234567890L` |
| `DoubleType` | `Double` | `3.14` |
| `StringType` | `String` | `"hola"` |
| `BooleanType` | `Boolean` | `true` |
| `DateType` | `java.sql.Date` | `"2024-03-15"` |
| `TimestampType` | `java.sql.Timestamp` | `"2024-03-15 10:30:00"` |

---

## 6. Explorar un DataFrame recién cargado

Antes de transformar datos, siempre conviene explorar el DataFrame con estas operaciones básicas:

### `show(n)` — ver filas

```scala
df.show()        // primeras 20 filas (por defecto)
df.show(5)       // primeras 5 filas
df.show(5, truncate = false)  // sin truncar textos largos
```

### `printSchema()` — ver estructura

```scala
df.printSchema()
// root
//  |-- nombre: string (nullable = true)
//  |-- edad: integer (nullable = true)
```

### `count()` — número de filas

```scala
val total = df.count()
println(s"Total de registros: $total")
```

### `columns` — nombres de columnas

```scala
df.columns.foreach(println)
// id
// nombre
// edad
// departamento
```

### `describe()` — estadísticas descriptivas

Calcula automáticamente count, media, desviación estándar, mínimo y máximo para las columnas numéricas:

```scala
df.describe().show()
// +-------+------------------+------+
// |summary|              edad|    id|
// +-------+------------------+------+
// |  count|                 4|     4|
// |   mean|             31.25|   2.5|
// | stddev|  8.26...         |  1.29|
// |    min|                22|     1|
// |    max|                41|     4|
// +-------+------------------+------+

// También puedes limitar a columnas concretas:
df.describe("edad", "id").show()
```

### `dtypes` — tipos de columnas como array

```scala
df.dtypes.foreach { case (nombre, tipo) =>
  println(s"$nombre → $tipo")
}
// id → IntegerType
// nombre → StringType
// edad → IntegerType
// departamento → StringType
```

---

## 📊 Resumen de la sesión

| Concepto | Qué es | Cómo se usa |
| --- | --- | --- |
| `DataFrame` | Tabla distribuida con schema | Principal estructura de Spark para datos estructurados |
| `SparkSession` | Punto de entrada unificado | `SparkSession.builder().appName(...).master(...).getOrCreate()` |
| `spark.read.csv(...)` | Carga desde CSV | Con opciones `header`, `inferSchema`, `schema` |
| `spark.read.json(...)` | Carga desde JSON | Con opción `multiline` si hace falta |
| `StructType` / `StructField` | Schema manual | Más robusto y rápido que `inferSchema` |
| `show()` | Ver filas | `show()`, `show(n)`, `show(n, false)` |
| `printSchema()` | Ver estructura | Árbol con nombres, tipos y nullable |
| `describe()` | Estadísticas | count, mean, stddev, min, max por columna |

---

# 💻 Práctica

---

## 🗂️ Paso previo: crear los ficheros de datos

Antes de empezar con el notebook, crea los ficheros de datos que usaremos. Abre el **Bloc de notas** o cualquier editor de texto como VsCode y guarda los siguientes ficheros en `C:\Curso-Scala\datos\` (crea la carpeta si no existe).

---

### Fichero 1 — `empleados.csv`

```
id,nombre,edad,departamento,salario,activo
1,Ana García,28,Ingeniería,42000,true
2,Luis Martínez,34,Marketing,38000,true
3,Marta López,22,Ingeniería,35000,true
4,Pedro Ruiz,41,Dirección,75000,true
5,Carmen Díaz,29,Marketing,36500,true
6,Jorge Santos,38,Ingeniería,48000,false
7,Elena Vega,31,RRHH,33000,true
8,Tomás Gil,45,Dirección,82000,true
9,Laura Prieto,26,Ingeniería,39000,true
10,Andrés Mora,33,Marketing,41000,false
```

---

### Fichero 2 — `productos.json`

```json
[
  {"id": 101, "nombre": "Laptop Pro", "categoria": "Informatica", "precio": 1299.99, "stock": 45},
  {"id": 102, "nombre": "Teclado Inalámbrico", "categoria": "Perifericos", "precio": 59.90, "stock": 120},
  {"id": 103, "nombre": "Monitor 27\"", "categoria": "Monitores", "precio": 349.00, "stock": 30},
  {"id": 104, "nombre": "Ratón Óptico", "categoria": "Perifericos", "precio": 24.95, "stock": 200},
  {"id": 105, "nombre": "Auriculares USB", "categoria": "Audio", "precio": 89.50, "stock": 75},
  {"id": 106, "nombre": "Webcam HD", "categoria": "Perifericos", "precio": 79.00, "stock": 60},
  {"id": 107, "nombre": "Disco SSD 1TB", "categoria": "Almacenamiento", "precio": 119.99, "stock": 90},
  {"id": 108, "nombre": "Hub USB-C", "categoria": "Perifericos", "precio": 44.90, "stock": 150}
]
```

---

## 🔧 Celda de inicialización

```scala
import $ivy.`org.apache.spark::spark-core:4.1.1`
import $ivy.`org.apache.spark::spark-sql:4.1.1`

import org.apache.spark.sql.SparkSession
import org.apache.spark.sql.types._

val spark = SparkSession.builder()
  .appName("Dia14S1_DataFrames")
  .master("local[*]")
  .config("spark.ui.showConsoleProgress", "false")
  .getOrCreate()

import spark.implicits._   // necesario para .toDF() y otras conversiones

spark.sparkContext.setLogLevel("ERROR")

println(s"✅ Spark ${spark.version} listo")
```

---

## 🔹 Ejercicio 1 — DataFrame desde colección en memoria

Crea tu primer DataFrame desde datos en el propio notebook y practica los métodos de exploración.

```scala
// Crear DataFrame desde una secuencia de tuplas
val dfCursos = Seq(
  (1, "Big Data con Scala", "Avanzado",  140, 4.8),
  (2, "Python para Datos",  "Intermedio", 80, 4.6),
  (3, "SQL Empresarial",    "Básico",     40, 4.9),
  (4, "Machine Learning",   "Avanzado",  120, 4.7),
  (5, "Power BI",           "Básico",     60, 4.5)
).toDF("id", "titulo", "nivel", "horas", "valoracion")

// Ver las filas
println("=== Contenido del DataFrame ===")
dfCursos.show()

// Ver la estructura
println("=== Schema del DataFrame ===")
dfCursos.printSchema()

// Número de filas y columnas
println(s"Filas: ${dfCursos.count()}")
println(s"Columnas: ${dfCursos.columns.length}")
println(s"Nombres de columnas: ${dfCursos.columns.mkString(", ")}")
```

**Salida esperada:**

```
=== Contenido del DataFrame ===
+---+------------------+-----------+-----+----------+
| id|            titulo|      nivel|horas|valoracion|
+---+------------------+-----------+-----+----------+
|  1|Big Data con Scala|   Avanzado|  140|       4.8|
|  2| Python para Datos|Intermedio |   80|       4.6|
|  3|   SQL Empresarial|     Básico|   40|       4.9|
|  4|  Machine Learning|   Avanzado|  120|       4.7|
|  5|          Power BI|     Básico|   60|       4.5|
+---+------------------+-----------+-----+----------+

=== Schema del DataFrame ===
root
 |-- id: integer (nullable = false)
 |-- titulo: string (nullable = true)
 |-- nivel: string (nullable = true)
 |-- horas: integer (nullable = false)
 |-- valoracion: double (nullable = false)

Filas: 5
Columnas: 5
Nombres de columnas: id, titulo, nivel, horas, valoracion
```

**Estadísticas:**

```scala
// Estadísticas descriptivas de columnas numéricas
println("=== Estadísticas descriptivas ===")
dfCursos.describe("horas", "valoracion").show()
```

**Salida esperada:**

```
=== Estadísticas descriptivas ===
+-------+------------------+------------------+
|summary|             horas|        valoracion|
+-------+------------------+------------------+
|  count|                 5|                 5|
|   mean|              88.0|              4.70|
| stddev|  38.47...        |  0.152...        |
|    min|                40|               4.5|
|    max|               140|               4.9|
+-------+------------------+------------------+
```

---

## 🔹 Ejercicio 2 — Cargar un CSV con inferencia de schema

**Carga con inferSchema:**

```scala
val dfEmpleados = spark.read
  .option("header", "true")
  .option("inferSchema", "true")
  .csv("C:/Curso-Scala/datos/empleados.csv")

println("=== Primeras filas ===")
dfEmpleados.show()

println("=== Schema inferido ===")
dfEmpleados.printSchema()
```

**Salida esperada:**

```
=== Primeras filas ===
+---+-------------+----+------------+-------+------+
| id|       nombre|edad|departamento|salario|activo|
+---+-------------+----+------------+-------+------+
|  1|   Ana García|  28|  Ingeniería|  42000|  true|
|  2|Luis Martínez|  34|   Marketing|  38000|  true|
...
+---+-------------+----+------------+-------+------+

=== Schema inferido ===
root
 |-- id: integer (nullable = true)
 |-- nombre: string (nullable = true)
 |-- edad: integer (nullable = true)
 |-- departamento: string (nullable = true)
 |-- salario: integer (nullable = true)
 |-- activo: boolean (nullable = true)
```

**schema manual para comparar:**

```scala
// Ahora definimos el schema manualmente
val schemaEmpleados = StructType(List(
  StructField("id",           IntegerType, nullable = false),
  StructField("nombre",       StringType,  nullable = true),
  StructField("edad",         IntegerType, nullable = true),
  StructField("departamento", StringType,  nullable = true),
  StructField("salario",      DoubleType,  nullable = true),  // Double en vez de Int
  StructField("activo",       BooleanType, nullable = true)
))

val dfEmpleadosTyped = spark.read
  .option("header", "true")
  .schema(schemaEmpleados)
  .csv("C:/Curso-Scala/datos/empleados.csv")

println("=== Schema manual aplicado ===")
dfEmpleadosTyped.printSchema()

// Diferencia: el campo salario ahora es DoubleType
// Útil si luego vamos a calcular medias o porcentajes
```

**Exploración con dtypes:**

```scala
println("=== Tipos de cada columna ===")
dfEmpleadosTyped.dtypes.foreach { case (col, tipo) =>
  println(f"  $col%-15s → $tipo")
}

println(s"\nTotal empleados: ${dfEmpleadosTyped.count()}")

println("\n=== Estadísticas de edad y salario ===")
dfEmpleadosTyped.describe("edad", "salario").show()
```

**Salida esperada:**

```
=== Tipos de cada columna ===
  id              → IntegerType
  nombre          → StringType
  edad            → IntegerType
  departamento    → StringType
  salario         → DoubleType
  activo          → BooleanType

Total empleados: 10

=== Estadísticas de edad y salario ===
+-------+------------------+------------------+
|summary|              edad|           salario|
+-------+------------------+------------------+
|  count|                10|                10|
|   mean|              32.7|           47050.0|
| stddev|   6.94...        |   17156.7...     |
|    min|                22|           33000.0|
|    max|                45|           82000.0|
+-------+------------------+------------------+
```

---

## 🔹 Ejercicio 3 — Cargar y explorar un fichero JSON

```scala
val dfProductos = spark.read
  .option("multiline", "true")
  .json("C:/Curso-Scala/datos/productos.json")

println("=== Productos cargados ===")
dfProductos.show(truncate = false)

println("=== Schema del JSON ===")
dfProductos.printSchema()
```

**Salida esperada:**

```
=== Productos cargados ===
+---+-------------------+--------------+-------+-----+
| id|             nombre|     categoria| precio|stock|
+---+-------------------+--------------+-------+-----+
|101|         Laptop Pro|   Informatica|1299.99|   45|
|102|Teclado Inalámbrico|   Perifericos|  59.90|  120|
|103|        Monitor 27"|     Monitores| 349.00|   30|
...
+---+-------------------+--------------+-------+-----+

=== Schema del JSON ===
root
 |-- id: long (nullable = true)
 |-- nombre: string (nullable = true)
 |-- categoria: string (nullable = true)
 |-- precio: double (nullable = true)
 |-- stock: long (nullable = true)
```

> 💡 Fíjate en que Spark ha inferido `id` y `stock` como `LongType` (entero de 64 bits) en el JSON, mientras que en el CSV los infería como `IntegerType`. Esto es normal: el formato JSON no distingue entre `Int` y `Long`, y Spark elige el tipo más amplio por seguridad. En producción, un schema manual resuelve esta ambigüedad.
> 

**Exploración de columnas y estadísticas:**

```scala
println(s"Total de productos: ${dfProductos.count()}")

println("\n=== Categorías disponibles ===")
dfProductos.select("categoria").distinct().show()

println("=== Rango de precios ===")
dfProductos.describe("precio", "stock").show()
```

**Salida esperada:**

```
Total de productos: 8

=== Categorías disponibles ===
+--------------+
|     categoria|
+--------------+
|   Informatica|
|   Perifericos|
|     Monitores|
|         Audio|
|Almacenamiento|
+--------------+

=== Rango de precios ===
+-------+------------------+------------------+
|summary|            precio|             stock|
+-------+------------------+------------------+
|  count|                 8|                 8|
|   mean|         258.40375|            96.25 |
| stddev|   411.0...       |    55.9...       |
|    min|             24.95|              30.0|
|    max|           1299.99|             200.0|
+-------+------------------+------------------+
```

---

## 🔹 Ejercicio 4 — Comparar inferencia vs. schema manual en JSON

```scala
// Schema manual: controlamos que id y stock sean Int, no Long
val schemaProductos = StructType(List(
  StructField("id",        IntegerType, nullable = false),
  StructField("nombre",    StringType,  nullable = true),
  StructField("categoria", StringType,  nullable = true),
  StructField("precio",    DoubleType,  nullable = true),
  StructField("stock",     IntegerType, nullable = true)
))

val dfProductosTyped = spark.read
  .option("multiline", "true")
  .schema(schemaProductos)
  .json("C:/Curso-Scala/datos/productos.json")

println("=== Schema controlado ===")
dfProductosTyped.printSchema()

println("\n=== Comparación de tipos ===")
println("Con inferSchema:")
dfProductos.dtypes.foreach { case (c, t) => println(s"  $c → $t") }

println("\nCon schema manual:")
dfProductosTyped.dtypes.foreach { case (c, t) => println(s"  $c → $t") }
```

**Salida esperada:**

```
=== Schema controlado ===
root
 |-- id: integer (nullable = true)
 |-- nombre: string (nullable = true)
 |-- categoria: string (nullable = true)
 |-- precio: double (nullable = true)
 |-- stock: integer (nullable = true)

=== Comparación de tipos ===
Con inferSchema:
  id → LongType
  nombre → StringType
  categoria → StringType
  precio → DoubleType
  stock → LongType

Con schema manual:
  id → IntegerType
  nombre → StringType
  categoria → StringType
  precio → DoubleType
  stock → IntegerType
```

---

## 🔹 Ejercicio 5 — `show` con opciones avanzadas

```scala
// show() tiene tres variantes útiles
println("show() por defecto — 20 filas, textos truncados a 20 chars:")
dfEmpleados.show()

println("show(3) — solo las primeras 3 filas:")
dfEmpleados.show(3)

println("show(3, truncate = false) — 3 filas, textos completos:")
dfEmpleados.show(3, truncate = false)

// columns devuelve un Array[String]
println(s"\nColumnas del DataFrame de empleados:")
println(dfEmpleados.columns.mkString(" | "))
// id | nombre | edad | departamento | salario | activo
```

---

# 🏢 Caso de Estudio Propuesto : AgroData Cooperativa

---

## 🌾 Contexto del caso

**AgroData Cooperativa** es una organización agrícola que agrupa a productores de frutas y verduras de cinco provincias españolas. Hasta ahora, cada delegación provincial guardaba sus datos en hojas de cálculo independientes. El nuevo responsable de datos ha decidido centralizar toda la información en un sistema de análisis distribuido con Apache Spark.

El departamento de datos ha exportado la información existente en dos formatos:

- Un fichero **CSV** con el registro de todas las **parcelas** productivas de la cooperativa.
- Un fichero **JSON** con el catálogo de **productos** cultivados y sus características de mercado.

Tu tarea es ser el analista de datos que ingiere estos ficheros en Spark por primera vez, verifica que los datos han cargado correctamente, examina su estructura y extrae un primer informe de situación para la dirección.

---

## 📦 Datos del caso

Crea los siguientes ficheros en `C:\Curso-Scala\datos\agrodata\` antes de abrir el notebook.

---

### Fichero 1 — `parcelas.csv`

```
id_parcela,provincia,municipio,superficie_ha,cultivo,año_alta,en_produccion,rendimiento_kg_ha
P001,Sevilla,Carmona,12.5,Naranja,2015,true,28000
P002,Huelva,Lepe,8.3,Fresa,2018,true,45000
P003,Almería,Níjar,25.0,Tomate,2012,true,85000
P004,Sevilla,Écija,6.7,Aceituna,2009,false,3200
P005,Murcia,Totana,15.2,Limón,2016,true,22000
P006,Almería,El Ejido,30.1,Pimiento,2014,true,62000
P007,Huelva,Moguer,9.8,Fresa,2020,true,41000
P008,Murcia,Lorca,18.4,Melocotón,2011,false,9500
P009,Sevilla,Utrera,22.0,Naranja,2013,true,31000
P010,Almería,Vícar,11.6,Pepino,2019,true,74000
P011,Murcia,Alhama,7.9,Limón,2017,true,20500
P012,Huelva,Cartaya,14.3,Fresa,2015,true,43000
P013,Sevilla,Marchena,5.2,Aceituna,2010,false,2900
P014,Almería,Roquetas,28.7,Tomate,2011,true,88000
P015,Murcia,Mazarrón,16.5,Pimiento,2018,true,58000
```

---

### Fichero 2 — `productos.json`

```json
[
  {
    "codigo": "NAR",
    "nombre": "Naranja",
    "familia": "Citrico",
    "precio_mercado_euro_kg": 0.45,
    "demanda_exportacion": "Alta",
    "certificacion_eco": false
  },
  {
    "codigo": "FRE",
    "nombre": "Fresa",
    "familia": "Baya",
    "precio_mercado_euro_kg": 2.10,
    "demanda_exportacion": "Muy Alta",
    "certificacion_eco": true
  },
  {
    "codigo": "TOM",
    "nombre": "Tomate",
    "familia": "Hortaliza",
    "precio_mercado_euro_kg": 0.85,
    "demanda_exportacion": "Alta",
    "certificacion_eco": false
  },
  {
    "codigo": "ACE",
    "nombre": "Aceituna",
    "familia": "Oleaginosa",
    "precio_mercado_euro_kg": 0.60,
    "demanda_exportacion": "Media",
    "certificacion_eco": true
  },
  {
    "codigo": "LIM",
    "nombre": "Limón",
    "familia": "Citrico",
    "precio_mercado_euro_kg": 0.55,
    "demanda_exportacion": "Alta",
    "certificacion_eco": false
  },
  {
    "codigo": "PIM",
    "nombre": "Pimiento",
    "familia": "Hortaliza",
    "precio_mercado_euro_kg": 1.20,
    "demanda_exportacion": "Muy Alta",
    "certificacion_eco": true
  },
  {
    "codigo": "PEP",
    "nombre": "Pepino",
    "familia": "Hortaliza",
    "precio_mercado_euro_kg": 0.70,
    "demanda_exportacion": "Media",
    "certificacion_eco": false
  },
  {
    "codigo": "MEL",
    "nombre": "Melocotón",
    "familia": "Drupa",
    "precio_mercado_euro_kg": 1.35,
    "demanda_exportacion": "Media",
    "certificacion_eco": false
  }
]
```

---

## ❓ Tareas del analista

### Tarea 1 — Inicialización del entorno

Prepara la sesión de Spark. Es el primer paso obligatorio antes de cualquier otra tarea.

**Lo que debes hacer:**

- Importar las dependencias de Spark 4.1.1 con `$ivy`
- Crear la `SparkSession` con `appName` `"AgroData_Analisis"` en modo `local[*]`
- Importar `spark.implicits._` y `org.apache.spark.sql.types._`
- Configurar el nivel de log a `ERROR`
- Imprimir un mensaje de confirmación con la versión de Spark

**Salida esperada:**

```
✅ AgroData Analytics iniciado — Spark 4.1.1
```

---

### Tarea 2 — Carga del CSV de parcelas con `inferSchema`

El responsable de datos quiere ver rápidamente cómo ha inferido Spark los tipos de cada columna antes de decidir si necesita un schema manual.

**Lo que debes hacer:**

- Cargar `parcelas.csv` con `header = true` e `inferSchema = true`
- Mostrar las primeras 5 filas con `show(5)`
- Imprimir el schema completo con `printSchema()`
- Imprimir el número total de parcelas registradas con `count()`
- Listar los nombres de todas las columnas con `columns`

**Salida esperada (parcial):**

```
=== Primeras 5 parcelas ===
+----------+---------+--------+-------------+-------+--------+-------------+------------------+
|id_parcela|provincia|municipio|superficie_ha|cultivo|año_alta|en_produccion|rendimiento_kg_ha|
+----------+---------+--------+-------------+-------+--------+-------------+------------------+
|      P001|  Sevilla| Carmona|         12.5| Naranja|    2015|         true|             28000|
...

=== Schema inferido ===
root
 |-- id_parcela: string (nullable = true)
 |-- provincia: string (nullable = true)
 |-- municipio: string (nullable = true)
 |-- superficie_ha: double (nullable = true)
 |-- cultivo: string (nullable = true)
 |-- año_alta: integer (nullable = true)
 |-- en_produccion: boolean (nullable = true)
 |-- rendimiento_kg_ha: integer (nullable = true)

Total de parcelas registradas: 15
Columnas: id_parcela | provincia | municipio | superficie_ha | cultivo | año_alta | en_produccion | rendimiento_kg_ha
```

---

### Tarea 3 — Definir el schema manualmente y recargar

El responsable ha revisado el schema inferido y señala dos problemas:

1. `superficie_ha` debería ser `DoubleType` — ✅ correcto tal cual
2. `rendimiento_kg_ha` debería ser `DoubleType` (no `IntegerType`) para poder hacer cálculos de medias con decimales
3. `año_alta` debería ser `IntegerType` — ✅ correcto tal cual
4. `id_parcela` es un código alfanumérico que nunca debería ser nulo: `nullable = false`

**Lo que debes hacer:**

- Definir un `StructType` con los ocho campos corrigiendo los puntos anteriores
- Recargar el CSV usando `.schema(...)` en lugar de `inferSchema`
- Imprimir el nuevo schema con `printSchema()`
- Imprimir los tipos con `dtypes` en formato `columna → tipo` para que la dirección pueda comparar fácilmente

**Salida esperada:**

```
=== Schema manual aplicado ===
root
 |-- id_parcela: string (nullable = false)
 |-- provincia: string (nullable = true)
 |-- municipio: string (nullable = true)
 |-- superficie_ha: double (nullable = true)
 |-- cultivo: string (nullable = true)
 |-- año_alta: integer (nullable = true)
 |-- en_produccion: boolean (nullable = true)
 |-- rendimiento_kg_ha: double (nullable = true)

=== Tipos por columna ===
  id_parcela        → StringType
  provincia         → StringType
  municipio         → StringType
  superficie_ha     → DoubleType
  cultivo           → StringType
  año_alta          → IntegerType
  en_produccion     → BooleanType
  rendimiento_kg_ha → DoubleType
```

---

### Tarea 4 — Primer informe estadístico de las parcelas

La directora de operaciones necesita un resumen numérico rápido de las parcelas para la reunión de mañana.

**Lo que debes hacer:**

- Usar `describe()` sobre las columnas `superficie_ha` y `rendimiento_kg_ha`
- Usar `describe()` también sobre `año_alta`
- Imprimir los resultados con un título descriptivo

**Salida esperada:**

```
=== Estadísticas de superficie y rendimiento ===
+-------+------------------+------------------+
|summary|     superficie_ha| rendimiento_kg_ha|
+-------+------------------+------------------+
|  count|                15|                15|
|   mean|   15.546666...   |        40313.3...|
| stddev|    7.54...       |        26371.4...|
|    min|               5.2|            2900.0|
|    max|              30.1|           88000.0|
+-------+------------------+------------------+

=== Rango de años de alta ===
+-------+--------+
|summary|año_alta|
+-------+--------+
|  count|      15|
|   mean|  2014.6|
| stddev|   3.26..|
|    min|    2009|
|    max|    2020|
+-------+--------+
```

---

### Tarea 5 — Carga del catálogo de productos en JSON

El equipo comercial ha entregado el catálogo de productos en formato JSON. Hay que ingerirlo y verificar que Spark lo interpreta correctamente.

**Lo que debes hacer:**

- Cargar `productos.json` con la opción `multiline = true`
- Mostrar todos los productos con `show(truncate = false)`
- Imprimir el schema con `printSchema()`
- Anotar en un comentario del código qué tipo ha inferido Spark para `precio_mercado_euro_kg` y para `certificacion_eco`, y si te parecen correctos

**Salida esperada:**

```
=== Catálogo de productos ===
+------+---------+----------+-----------------------+-------------------+----------------+
|codigo|  nombre |  familia |precio_mercado_euro_kg |demanda_exportacion|certificacion_eco|
+------+---------+----------+-----------------------+-------------------+----------------+
|   NAR|  Naranja|   Citrico|                   0.45|               Alta|           false|
|   FRE|    Fresa|      Baya|                   2.10|           Muy Alta|            true|
...
+------+---------+----------+-----------------------+-------------------+----------------+

=== Schema del JSON ===
root
 |-- certificacion_eco: boolean (nullable = true)
 |-- codigo: string (nullable = true)
 |-- demanda_exportacion: string (nullable = true)
 |-- familia: string (nullable = true)
 |-- nombre: string (nullable = true)
 |-- precio_mercado_euro_kg: double (nullable = true)
```

> ⚠️ **Atención:** observa que Spark ordena las columnas del JSON alfabéticamente, no en el orden en que aparecen en el fichero. Esto es comportamiento normal del lector JSON de Spark.
> 

---

### Tarea 6 — Schema manual para el catálogo de productos

El equipo comercial indica que `codigo` nunca debería ser nulo (es la clave del producto) y que `precio_mercado_euro_kg` debe ser siempre `DoubleType` con `nullable = false`.

**Lo que debes hacer:**

- Definir un `StructType` para los seis campos del JSON respetando las restricciones anteriores
- Recargar el JSON con el schema manual
- Comparar los tipos resultantes con los de la carga por inferencia usando `dtypes`

---

### Tarea 7 — DataFrame de resumen desde colección en memoria

El director general ha pedido que se incluya en el informe una pequeña tabla resumen creada a mano con los datos más importantes de cada provincia, antes de tener el análisis completo automatizado.

**Lo que debes hacer:**

- Crear el siguiente DataFrame **desde una colección Scala** usando `.toDF(...)`:

| provincia | num_parcelas | superficie_total_ha | parcelas_activas |
| --- | --- | --- | --- |
| Almería | 4 | 95.4 | 4 |
| Huelva | 3 | 32.4 | 3 |
| Murcia | 4 | 58.0 | 3 |
| Sevilla | 4 | 46.4 | 2 |
- Mostrar el DataFrame con `show()`
- Imprimir su schema con `printSchema()`
- Confirmar que Spark ha inferido los tipos correctamente para cada columna
- Imprimir el número de filas con `count()` y los nombres de columnas con `columns`

**Salida esperada del schema:**

```
root
 |-- provincia: string (nullable = true)
 |-- num_parcelas: integer (nullable = false)
 |-- superficie_total_ha: double (nullable = false)
 |-- parcelas_activas: integer (nullable = false)
```

---

## 🗺️ Guía de herramientas → tareas

| Tarea | Herramientas principales |
| --- | --- |
| 1 — Inicialización | `SparkSession.builder`, `$ivy`, `import spark.implicits._` |
| 2 — CSV con inferSchema | `spark.read.option(...).csv(...)`, `show`, `printSchema`, `count`, `columns` |
| 3 — Schema manual CSV | `StructType`, `StructField`, `DoubleType`, `BooleanType`, `.schema(...)`, `dtypes` |
| 4 — Estadísticas | `describe(...)` |
| 5 — JSON con inferSchema | `spark.read.option("multiline","true").json(...)`, `show(truncate=false)`, `printSchema` |
| 6 — Schema manual JSON | `StructType`, `StructField`, `.schema(...)`, `dtypes` |
| 7 — Colección en memoria | `Seq(...).toDF(...)`, `show`, `printSchema`, `count`, `columns` |

---

<aside>

## 💡 Consejo final

En este caso no hay transformaciones ni filtros: el objetivo es **conocer los datos antes de tocarlos**. En la práctica profesional, el primer trabajo de un analista al recibir un fichero nuevo siempre es:

1. Cargarlo
2. Revisar el schema
3. Contar filas
4. Ver estadísticas básicas
5. Anotar anomalías (tipos inesperados, nulos, rangos extraños)

Todo lo que has practicado hoy es exactamente ese flujo. Las transformaciones vienen después. 

</aside>