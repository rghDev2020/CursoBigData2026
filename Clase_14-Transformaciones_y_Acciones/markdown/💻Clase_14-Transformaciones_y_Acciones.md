# 💻Clase 14 - Transformaciones y Acciones

---

# Agenda:

<aside>
💡

#### 9:00 - 9:50    →  Sesión 1  - Transformaciones en RDDs

#### 9:50 - 11:20   → Ejercicios y caso de uso

#### 11:40 - 12:40  → Sesión 2 - Acciones

#### 12:40 - 14:00  → Ejercicios y caso de uso

</aside>

# Sesión 1 : Transformaciones en RDDs

---

## 1. ¿Qué es una transformación?

En la sesión anterior vimos que los RDDs son **perezosos** (lazy): no hacen nada hasta que se llama a una acción. Las **transformaciones** son operaciones que **producen un nuevo RDD** a partir de uno existente, sin ejecutar nada todavía.

```
RDD original  ──transform──▶  RDD nuevo  ──transform──▶  RDD final  ──acción──▶  resultado
```

Piénsalo como una cadena de instrucciones que Spark anota pero no ejecuta. Solo cuando dices "dame el resultado" (acción: `collect`, `count`, `take`…) Spark ejecuta toda la cadena de una vez.

---

## 2. Transformaciones elemento a elemento

### 2.1 `map` — transformar cada elemento

Aplica una función a **cada elemento** del RDD y devuelve un RDD con los resultados. Relación 1:1 (un elemento entra, un elemento sale).

```scala
// Ejemplo: elevar al cuadrado una lista de números
val numeros = sc.parallelize(List(1, 2, 3, 4, 5))
val cuadrados = numeros.map(n => n * n)
cuadrados.collect()
// Array(1, 4, 9, 16, 25)
```

```scala
// Ejemplo: convertir a mayúsculas
val palabras = sc.parallelize(List("scala", "spark", "big data"))
val mayusculas = palabras.map(p => p.toUpperCase)
mayusculas.collect()
// Array("SCALA", "SPARK", "BIG DATA")
```

---

### 2.2 `flatMap` — transformar y aplanar

Como `map`, pero cada elemento puede producir **cero, uno o varios** elementos de salida. El resultado se "aplana" en un único RDD.

```
Entrada:  ["hola mundo", "spark es rápido"]
map:      [["hola","mundo"], ["spark","es","rápido"]]   ← lista de listas
flatMap:  ["hola", "mundo", "spark", "es", "rápido"]    ← lista plana
```

```scala
val frases = sc.parallelize(List("hola mundo", "spark es rápido"))

// Con map obtenemos arrays dentro del RDD:
frases.map(f => f.split(" ")).collect()
// Array(Array("hola","mundo"), Array("spark","es","rápido"))

// Con flatMap obtenemos palabras individuales:
val palabras = frases.flatMap(f => f.split(" "))
palabras.collect()
// Array("hola", "mundo", "spark", "es", "rápido")
```

> 💡 `flatMap` es la transformación clave del **WordCount**, el "Hola Mundo" del Big Data.
> 

---

### 2.3 `filter` — quedarse con los que cumplen una condición

Devuelve un RDD con solo los elementos para los que la función devuelve `true`.

```scala
val numeros = sc.parallelize(1 to 10)
val pares = numeros.filter(n => n % 2 == 0)
pares.collect()
// Array(2, 4, 6, 8, 10)
```

```scala
val nombres = sc.parallelize(List("Ana", "Borja", "Carlos", "Beatriz", "Alberto"))
val conB = nombres.filter(n => n.startsWith("B"))
conB.collect()
// Array("Borja", "Beatriz")
```

---

## 3. Transformaciones de conjunto

### 3.1 `union` — combinar dos RDDs

Une dos RDDs del mismo tipo. **No elimina duplicados.**

```scala
val rdd1 = sc.parallelize(List(1, 2, 3))
val rdd2 = sc.parallelize(List(3, 4, 5))
rdd1.union(rdd2).collect()
// Array(1, 2, 3, 3, 4, 5)   ← el 3 aparece dos veces
```

---

### 3.2 `intersection` — elementos comunes

Devuelve solo los elementos que están en **ambos** RDDs. Elimina duplicados.

```scala
val rdd1 = sc.parallelize(List(1, 2, 3, 4))
val rdd2 = sc.parallelize(List(3, 4, 5, 6))
rdd1.intersection(rdd2).collect()
// Array(3, 4)
```

> ⚠️ `intersection` es costoso porque requiere un **shuffle** (redistribución de datos entre particiones). Úsalo con moderación.
> 

---

### 3.3 `distinct` — eliminar duplicados

```scala
val conRepetidos = sc.parallelize(List(1, 2, 2, 3, 3, 3, 4))
conRepetidos.distinct().collect()
// Array(1, 2, 3, 4)   ← orden no garantizado
```

---

## 4. Transformaciones clave-valor (`Pair RDDs`)

Cuando los elementos del RDD son **tuplas `(clave, valor)`**, se desbloquean operaciones especiales muy útiles para análisis de datos.

### 4.1 `groupByKey` — agrupar valores por clave

Agrupa todos los valores que comparten la misma clave en una lista.

```scala
val ventas = sc.parallelize(List(
  ("norte", 100), ("sur", 200), ("norte", 150), ("sur", 50), ("este", 300)
))
val agrupado = ventas.groupByKey()
agrupado.collect()
// Array(("norte", [100,150]), ("sur", [200,50]), ("este", [300]))
```

> ⚠️ **Problema:** `groupByKey` mueve **todos** los valores por la red antes de agruparlos. Para grandes volúmenes, esto puede ser muy lento. Si solo necesitas un agregado (suma, media…), usa `reduceByKey`.
> 

---

### 4.2 `reduceByKey` — reducir valores por clave

Combina los valores de la misma clave aplicando una función de dos parámetros. Es mucho más eficiente que `groupByKey` porque **precombina localmente** antes de enviar por la red.

```scala
val ventas = sc.parallelize(List(
  ("norte", 100), ("sur", 200), ("norte", 150), ("sur", 50), ("este", 300)
))
val totalPorZona = ventas.reduceByKey((a, b) => a + b)
totalPorZona.collect()
// Array(("norte", 250), ("sur", 250), ("este", 300))
```

|  | `groupByKey` | `reduceByKey` |
| --- | --- | --- |
| ¿Qué mueve por la red? | Todos los valores | Valores parcialmente combinados |
| Uso ideal | Cuando necesitas la lista completa | Cuando solo necesitas un agregado |
| Rendimiento | 🐢 Lento con muchos datos | 🐇 Mucho más eficiente |

---

### 4.3 `sortBy` y `sortByKey` — ordenar

```scala
val numeros = sc.parallelize(List(5, 3, 1, 4, 2))
numeros.sortBy(n => n).collect()
// Array(1, 2, 3, 4, 5)

numeros.sortBy(n => n, ascending = false).collect()
// Array(5, 4, 3, 2, 1)
```

Para Pair RDDs:

```scala
val pares = sc.parallelize(List(("c", 3), ("a", 1), ("b", 2)))
pares.sortByKey().collect()
// Array(("a",1), ("b",2), ("c",3))

pares.sortByKey(ascending = false).collect()
// Array(("c",3), ("b",2), ("a",1))
```

---

### 4.4 `join` — combinar dos Pair RDDs por clave

Equivalente al `JOIN` de SQL. Une dos Pair RDDs por la clave común.

```scala
val empleados = sc.parallelize(List(
  (1, "Ana"), (2, "Borja"), (3, "Carlos")
))
val salarios = sc.parallelize(List(
  (1, 2500.0), (2, 3100.0), (4, 2800.0)  // clave 3 no existe, clave 4 no existe en empleados
))

val resultado = empleados.join(salarios)
resultado.collect()
// Array((1,("Ana",2500.0)), (2,("Borja",3100.0)))
// ← solo las claves que están en AMBOS RDDs (inner join)
```

Para incluir también los que no tienen pareja:

```scala
// leftOuterJoin: todos los de la izquierda, con o sin pareja
empleados.leftOuterJoin(salarios).collect()
// Array((1,("Ana",Some(2500.0))), (2,("Borja",Some(3100.0))), (3,("Carlos",None)))
```

---

## 5. Resumen

```
RDD de entrada
    │
    ├─ map(f)           → RDD del mismo tamaño, cada elem transformado
    ├─ flatMap(f)       → RDD potencialmente más grande, aplanado
    ├─ filter(f)        → RDD más pequeño, solo los que cumplen condición
    ├─ distinct()       → RDD sin duplicados
    ├─ union(rdd2)      → RDD más grande (todos los elem de ambos)
    ├─ intersection(r2) → RDD más pequeño (solo los comunes)
    │
    └─ [Pair RDD (clave,valor)]
         ├─ groupByKey()    → (clave, Iterable[valor])
         ├─ reduceByKey(f)  → (clave, valor_reducido)
         ├─ sortByKey()     → Pair RDD ordenado por clave
         └─ join(rdd2)      → (clave, (v1, v2))
```

---

# 💻 Practica — Transformaciones en RDDs

---

---

<aside>

## ⚙️ Preparación del entorno: Solo para los que están usando spark-shell

### 1. Procedimiento recomendado (no es obligatorio):

1. Escribir el código en un archivo `.scala`.
2. Ejecutar ese archivo desde `spark-shell` usando `:load`.
3. Guardar la salida de la sesión en un archivo `.txt`.
4. Entregar ambos archivos.

Este procedimiento se parece más a una forma ordenada y profesional de trabajar, porque separa claramente:

- el **código fuente**
- la **evidencia de ejecución**
- los **resultados obtenidos**

---

### **2. Regla importante para varias clases y sesiones**

Como habrá **varias clases** y **varias sesiones de trabajo**, **no se debe reutilizar siempre el mismo archivo `.scala`**.

### **Regla de organización**

Para **cada clase** o para **cada sesión**, el estudiante debe crear un archivo `.scala` diferente.

Por ejemplo:

- `clase01.scala`
- `clase02.scala`
- `clase03.scala`
- `clase04.scala`

O también:

- `sesion01.scala`
- `sesion02.scala`
- `sesion03.scala`

Del mismo modo, cada sesión debe generar también su propio archivo de salida:

- `clase01.txt`
- `clase02.txt`
- `clase03.txt`

La mejor práctica es que **cada clase tenga su propio `.scala` y su propio `.txt`**.

Por ejemplo:

- `clase14.scala` y `clase14.txt`
- `clase15.scala` y `clase15.txt`
- `clase16.scala` y `clase16.txt`

---

## **3. Estructura recomendada de carpetas**

Se recomienda crear una carpeta principal de trabajo y una subcarpeta para las salidas.

Ejemplo:

```
C:\practicas-spark\
│
├── clase14.scala
├── clase15.scala
├── clase16.scala
│
└── entregas\
    ├── clase14.txt
    ├── clase15.txt
    └── clase16.txt
```

De este modo:

- los archivos `.scala` quedan en la carpeta principal
- los archivos `.txt` quedan agrupados dentro de `entregas`

---

## **4. Paso 1: crear la carpeta de trabajo**

La primera vez, el estudiante debe crear su carpeta de trabajo en PowerShell.

```powershell
New-Item -ItemType Directory -Force -Path "C:\practicas-spark"
New-Item -ItemType Directory -Force -Path "C:\practicas-spark\entregas"
```

Si la carpeta ya existe, PowerShell no dará problemas porque se usa `-Force`.

---

## **5. Paso 2: crear el archivo `.scala` de la clase correspondiente**

Para cada clase o sesión, el estudiante debe crear un archivo nuevo.

### **Ejemplos**

Para la clase 14:

```
C:\practicas-spark\clase14.scala
```

Para la clase 15:

```
C:\practicas-spark\clase15.scala
```

Para la clase 16:

```
C:\practicas-spark\clase16.scala
```

Ese archivo puede crearse con Visual Studio Code

---

## **6. Paso 3: escribir el código en el archivo `.scala`**

El estudiante debe escribir todos los ejercicios de esa clase dentro del archivo correspondiente.

### **Ejemplo de `clase14.scala`**

```scala
// ==========================================
// CLASE 14 - SPARK SHELL
// Alumno: Nombre Apellido
// Fecha: 22/04/2026
// ==========================================

println("===== EJERCICIO 1 =====")
val numeros = sc.parallelize(1 to 100)
println(s"Cantidad total:${numeros.count()}")

println("===== EJERCICIO 2 =====")
val pares = numeros.filter(_ % 2 == 0)
println(s"Números pares:${pares.collect().mkString(", ")}")

println("===== EJERCICIO 3 =====")
val cuadrados = numeros.map(x => x * x)
println(s"Primeros 10 cuadrados:${cuadrados.take(10).mkString(", ")}")
```

---

## **7. Paso 4: abrir PowerShell en la carpeta de trabajo**

Antes de ejecutar nada, el estudiante debe situarse en la carpeta donde está el archivo `.scala`.

Ejemplo:

```powershell
cd C:\practicas-spark
```

Para comprobar que el archivo existe:

```powershell
Get-ChildItem
```

Debe aparecer el archivo correspondiente, por ejemplo:

- `clase14.scala`

---

## **8. Paso 5: iniciar `spark-shell` guardando la salida en un `.txt`**

Ahora el estudiante debe arrancar `spark-shell` y al mismo tiempo guardar toda la salida de la sesión en un archivo `.txt`.

### **Ejemplo para la clase 14**

```powershell
spark-shell --master local[*] 2>&1 | Tee-Object -FilePath "C:\practicas-spark\entregas\clase14.txt"
```

### **Ejemplo para la clase 15**

```powershell
spark-shell --master local[*] 2>&1 | Tee-Object -FilePath "C:\practicas-spark\entregas\clase15.txt"
```

### **¿Qué hace este comando?**

- abre `spark-shell`
- muestra la ejecución en pantalla
- guarda la salida en un archivo `.txt`

### **Muy importante**

Cada clase debe generar su propio archivo `.txt`. No conviene usar siempre el mismo archivo para todas las clases, porque mezclaría todas las ejecuciones.

---

## **9. Paso 6: reducir el nivel de logs dentro de `spark-shell`**

Una vez que `spark-shell` se haya abierto, el estudiante debe escribir:

```scala
sc.setLogLevel("ERROR")
```

Esto sirve para que aparezcan menos mensajes internos de Spark y la salida quede más limpia y más fácil de revisar.

---

## **10. Paso 7: cargar el archivo `.scala` de la sesión**

Dentro de `spark-shell`, el estudiante debe ejecutar el archivo correspondiente a la clase.

### **Ejemplo para la clase 14**

```scala
:load C:\practicas-spark\clase14.scala
```

Si ya se encuentra situado en la carpeta correcta, también puede usar:

```scala
:load clase14.scala
```

### **Ejemplo para la clase 15**

```scala
:load clase15.scala
```

### **¿Qué hace `:load`?**

El comando `:load` hace que `spark-shell`:

- lea el archivo `.scala`
- ejecute automáticamente todas sus líneas
- muestre los resultados en pantalla
- deje registrada la salida en el `.txt`

---

## **11. Paso 8: esperar a que termine la ejecución**

Cuando se carga el archivo, el estudiante debe esperar a que terminen todos los ejercicios. Es importante no cerrar la shell antes de tiempo. Si el archivo contiene varios ejercicios, conviene que cada uno empiece con un encabezado como este:

```scala
println("===== EJERCICIO 4 =====")
```

Así, la salida quedará más clara en el archivo `.txt`.

---

## **12. Paso 9: cerrar `spark-shell`**

Cuando toda la ejecución haya terminado, el estudiante debe salir con:

```scala
:quit
```

Esto finaliza la sesión.

---

## **13. Paso 10: comprobar los archivos finales**

Al terminar, el estudiante debe verificar que existen los dos archivos que debe entregar.

### **Para la clase 14**

- `C:\practicas-spark\clase14.scala`
- `C:\practicas-spark\entregas\clase14.txt`

### **Para la clase 15**

- `C:\practicas-spark\clase15.scala`
- `C:\practicas-spark\entregas\clase15.txt`

Puede comprobarlo con:

```powershell
Get-ChildItem C:\practicas-spark
Get-ChildItem C:\practicas-spark\entregas
```

---

## **14. Paso 11: revisar antes de entregar**

Antes de entregar, el estudiante debe abrir ambos archivos.

### **En el archivo `.scala`**

Debe revisar:

- que esté todo el código
- que no falte ningún ejercicio
- que el archivo corresponda a la clase correcta
- que el código esté ordenado

### **En el archivo `.txt`**

Debe revisar:

- que aparezcan los resultados
- que no haya errores inesperados
- que se vea claramente cada ejercicio
- que la salida corresponda a la clase correcta

---

</aside>

---

## 📝 Ejercicios

---

### 🟢 Ejercicio 1 — `map`: transformar temperaturas

Tienes una lista de temperaturas en Celsius. Conviértelas a Fahrenheit con la fórmula `F = C * 9/5 + 32`.

```scala
// CELDA — Ejercicio 1
val celsius = sc.parallelize(List(0.0, 20.0, 37.0, 100.0, -10.0))

val fahrenheit = celsius.map(c => c * 9.0 / 5.0 + 32.0)

fahrenheit.collect().foreach(f => println(f"$f%.1f °F"))
```

**Salida esperada:**

```
32.0 °F
68.0 °F
98.6 °F
212.0 °F
14.0 °F
```

📌 **Vía A:** guarda el notebook (Ctrl+S). **Vía B:** la salida ya quedó registrada en el `.txt`.

---

### 🟢 Ejercicio 2 — `filter`: filtrar productos

Tienes una lista de productos con precio. Filtra los que cuestan más de 50€.

```scala
// CELDA — Ejercicio 2
val productos = sc.parallelize(List(
  ("Teclado", 35.99),
  ("Monitor", 199.99),
  ("Ratón", 22.50),
  ("Auriculares", 89.00),
  ("Alfombrilla", 12.00),
  ("Webcam", 65.00)
))

val caros = productos
  .filter { case (nombre, precio) => precio > 50.0 }
  .sortBy { case (nombre, precio) => -precio }  // ordenar de mayor a menor precio

caros.collect().foreach { case (n, p) => println(s"$n → $p€") }
```

> 💡 Spark no garantiza el orden de los elementos en un RDD. Si queremos una salida reproducible, ordenamos explícitamente con `sortBy`.
> 

**Salida esperada:**

```
Monitor → 199.99€
Auriculares → 89.0€
Webcam → 65.0€
```

. **Vía B:** la salida ya quedó registrada en el `.txt`.

---

### 🟢 Ejercicio 3 — `flatMap`: contar palabras

Dada una lista de frases, cuenta cuántas palabras tiene en total. Usa `flatMap` para obtener todas las palabras en un único RDD plano y luego `count()` para contar.

```scala
// CELDA — Ejercicio 3
val frases = sc.parallelize(List(
  "Apache Spark es un motor de procesamiento rápido",
  "Scala es el lenguaje principal de Spark",
  "Los RDDs son la base de Spark"
))

// flatMap divide cada frase por espacios y aplana el resultado
val palabras = frases.flatMap(f => f.split(" "))

println(s"Total de palabras: ${palabras.count()}")
```

**Salida esperada:**

```
Total de palabras: 22
```

> 💡 ¿Por qué 22? La primera frase tiene 8 palabras, la segunda 7 y la tercera 7. En total: 8 + 7 + 7 = 22.
> 

---

### 🟡 Ejercicio 4 — `distinct` + `union`: operaciones de conjunto

Tienes los asistentes de dos eventos. Encuentra quién asistió a alguno de los dos (unión sin repetidos).

```scala
// CELDA — Ejercicio 4
val eventoA = sc.parallelize(List("Ana", "Borja", "Carmen", "David"))
val eventoB = sc.parallelize(List("Carmen", "David", "Elena", "Fran"))

val todosAsistentes = eventoA.union(eventoB).distinct()

println("Asistentes a algún evento (sin repetidos):")
todosAsistentes.collect().sorted.foreach(println)
```

**Salida esperada:**

```
Asistentes a algún evento (sin repetidos):
Ana
Borja
Carmen
David
Elena
Fran
```

---

### 🟡 Ejercicio 5 — `intersection`: asistentes a ambos eventos

Usando los mismos datos de los dos eventos, encuentra quién asistió a los **dos** eventos.

> ⚠️ **Importante:** define los RDDs dentro de esta misma celda. Aunque los hayas creado en el ejercicio anterior, volver a definirlos aquí garantiza que el código funcione correctamente aunque ejecutes las celdas en distinto orden.
> 

```scala
// CELDA — Ejercicio 5
// Redefinimos los RDDs para no depender de celdas anteriores
val eventoA2 = sc.parallelize(List("Ana", "Borja", "Carmen", "David"))
val eventoB2 = sc.parallelize(List("Carmen", "David", "Elena", "Fran"))

val ambosEventos = eventoA2.intersection(eventoB2)

println("Asistieron a AMBOS eventos:")
ambosEventos.collect().sorted.foreach(println)
```

**Salida esperada:**

```
Asistieron a AMBOS eventos:
Carmen
David
```

---

### 🟡 Ejercicio 6 — `reduceByKey`: ventas por departamento

Suma el total de ventas por departamento.

```scala
// CELDA — Ejercicio 6
val ventas = sc.parallelize(List(
  ("Electrónica", 1200),
  ("Ropa", 340),
  ("Electrónica", 890),
  ("Hogar", 560),
  ("Ropa", 120),
  ("Electrónica", 450),
  ("Hogar", 230)
))

val totalPorDepto = ventas.reduceByKey((a, b) => a + b)

println("Ventas totales por departamento:")
totalPorDepto.collect().sortBy(_._1).foreach {
  case (depto, total) => println(s"  $depto: ${total}€")
}
```

**Salida esperada:**

```
Ventas totales por departamento:
  Electrónica: 2540€
  Hogar: 790€
  Ropa: 460€
```

---

### 🟡 Ejercicio 7 — `sortBy`: ranking de ventas

Usando el resultado del ejercicio anterior, muestra los departamentos ordenados de mayor a menor venta con su posición en el ranking.

> ⚠️ **Importante:** este ejercicio necesita que hayas ejecutado la celda del Ejercicio 6 primero, ya que usa el RDD `totalPorDepto`.
> 

```scala
// CELDA — Ejercicio 7
// sortBy(_._2, ascending = false) ordena por el segundo campo (ventas) de mayor a menor
val ranking = totalPorDepto.sortBy(_._2, ascending = false).collect()

println("Ranking de departamentos (mayor a menor venta):")
ranking.zipWithIndex.foreach { case ((depto, total), i) =>
  println(s"  ${i + 1}. $depto — ${total}€")
}
```

> 💡 `zipWithIndex` combina cada elemento con su posición (0, 1, 2…). Sumamos 1 para que el ranking empiece en 1.
> 

**Salida esperada:**

```
Ranking de departamentos (mayor a menor venta):
  1. Electrónica — 2540€
  2. Hogar — 790€
  3. Ropa — 460€
```

---

### 🔴 Ejercicio 8 — `join`: cruzar clientes con pedidos

Tienes dos RDDs: clientes y sus pedidos. Cruza ambos para mostrar el nombre del cliente junto con el importe de cada pedido.

```scala
// CELDA — Ejercicio 8
val clientes = sc.parallelize(List(
  (101, "Laura Sánchez"),
  (102, "Miguel Torres"),
  (103, "Patricia Ruiz"),
  (104, "Andrés López")
))

val pedidos = sc.parallelize(List(
  (101, 250.0),
  (102, 89.5),
  (101, 430.0),  // Laura tiene dos pedidos
  (103, 175.0),
  (105, 310.0)   // cliente 105 no existe en clientes
))

val resultado = clientes.join(pedidos)

println("Cliente → Pedido:")
resultado.collect().sortBy(_._1).foreach {
  case (id, (nombre, importe)) =>
    println(f"  [ID $id] $nombre → $importe%.2f€")
}
```

**Salida esperada:**

```
Cliente → Pedido:
  [ID 101] Laura Sánchez → 250.00€
  [ID 101] Laura Sánchez → 430.00€
  [ID 102] Miguel Torres → 89.50€
  [ID 103] Patricia Ruiz → 175.00€
```

> ℹ️ El cliente 104 (Andrés López) no aparece porque no tiene pedidos. El pedido del cliente 105 no aparece porque ese ID no existe en el RDD de clientes. Esto es un **inner join**.
> 

---

### 🔴 Ejercicio 9 — Pipeline completo: WordCount con ranking

El ejercicio clásico del Big Data: contar las palabras de un texto y mostrar las 5 más frecuentes. Fíjate cómo se encadenan 4 transformaciones en un solo pipeline.

```scala
// CELDA — Ejercicio 9
val texto = sc.parallelize(List(
  "spark es rápido spark es potente",
  "scala es el lenguaje de spark",
  "spark procesa datos a gran velocidad",
  "los datos son el petróleo de scala",
  "spark scala big data de datos procesamiento"
))

val conteo = texto
  .flatMap(linea => linea.split(" "))  // paso 1: separar en palabras
  .map(palabra => (palabra, 1))         // paso 2: cada palabra vale 1
  .reduceByKey((a, b) => a + b)         // paso 3: sumar por palabra
  .sortBy(_._2, ascending = false)      // paso 4: ordenar de mayor a menor

println("Top 5 palabras más frecuentes:")
conteo.take(5).foreach { case (palabra, n) =>
  println(s"  '$palabra' → $n veces")
}
```

> 💡 **Traza del pipeline paso a paso:**
> 
> - Después de `flatMap`: `["spark", "es", "rápido", "spark", "es", ...]`
> - Después de `map`: `[("spark",1), ("es",1), ("rápido",1), ("spark",1), ...]`
> - Después de `reduceByKey`: `[("spark",5), ("es",3), ("datos",3), ...]`
> - Después de `sortBy`: ordenado de mayor a menor frecuencia

**Salida esperada:**

```
Top 5 palabras más frecuentes:
  'spark' → 5 veces
  'es' → 3 veces
  'datos' → 3 veces
  'scala' → 3 veces
  'de' → 3 veces
```

---

### 🔴 Ejercicio 10 — Análisis de logs de acceso

Tienes un RDD que simula líneas de un log web. Calcula el número de peticiones por código de estado HTTP.

```scala
// CELDA — Ejercicio 10
val logs = sc.parallelize(List(
  "192.168.1.1 GET /index.html 200",
  "10.0.0.2 POST /login 200",
  "192.168.1.3 GET /datos 404",
  "10.0.0.5 GET /index.html 200",
  "192.168.1.1 GET /admin 403",
  "10.0.0.8 GET /datos 404",
  "192.168.1.9 POST /login 500",
  "10.0.0.2 GET /index.html 200",
  "192.168.1.3 GET /admin 403",
  "10.0.0.5 GET /login 200"
))

// Extraer el código de estado (último campo) y contar
val peticionesPorCodigo = logs
  .map(linea => linea.split(" ").last)     // extraer el código
  .map(codigo => (codigo, 1))
  .reduceByKey((a, b) => a + b)
  .sortByKey()

println("Peticiones por código HTTP:")
peticionesPorCodigo.collect().foreach { case (codigo, total) =>
  val texto = if (total == 1) "petición" else "peticiones"
  println(s"  HTTP $codigo → $total $texto")
}
```

**Salida esperada:**

```
Peticiones por código HTTP:
  HTTP 200 → 5 peticiones
  HTTP 403 → 2 peticiones
  HTTP 404 → 2 peticiones
  HTTP 500 → 1 petición
```

---

---

## 🔧 Resolución de problemas comunes

### Los resultados de `collect()` no están en el orden esperado

**Normal en Spark.** Los RDDs son distribuidos y el orden no está garantizado salvo que uses `sortBy` o `sortByKey`. Si el orden importa, añade un `sorted` al resultado o usa `sortBy`.

---

### En spark-shell: el prompt desaparece tras ejecutar código largo

**Normal.** Spark está procesando. Espera a que vuelva el prompt `scala>`. No cierres la ventana.

---

### `groupByKey` muy lento con muchos datos

Sustituye por `reduceByKey` siempre que sea posible. Si necesitas la lista completa de valores (no un agregado), entonces `groupByKey` es la única opción, pero úsalo con cautela.

## 📚 Referencia rápida de transformaciones RDD

| Transformación | Tipo | Descripción |
| --- | --- | --- |
| `map(f)` | Elemento | 1 entrada → 1 salida |
| `flatMap(f)` | Elemento | 1 entrada → N salidas (aplana) |
| `filter(f)` | Elemento | Conserva solo los que cumplen `f` |
| `distinct()` | Conjunto | Elimina duplicados |
| `union(rdd2)` | Conjunto | Todos los elementos de ambos |
| `intersection(rdd2)` | Conjunto | Solo los comunes |
| `groupByKey()` | Pair RDD | (k, Iterable[v]) |
| `reduceByKey(f)` | Pair RDD | (k, v_reducido) — eficiente |
| `sortBy(f)` | Orden | Ordena por una función |
| `sortByKey()` | Pair RDD | Ordena por clave |
| `join(rdd2)` | Pair RDD | Inner join por clave |
| `leftOuterJoin(r2)` | Pair RDD | Left join, preserva todos de la izq. |

---

# 🏢 Caso de Estudio Propuesto 1: MercaData S.L.

---

## 🏢 Contexto de la empresa

**MercaData S.L.** es una cadena de supermercados con tiendas en cuatro provincias españolas: Madrid, Barcelona, Valencia y Sevilla. Su departamento de tecnología ha decidido migrar el análisis de ventas a Apache Spark para poder procesar el volumen creciente de datos de forma distribuida.

Te han contratado como analista de datos para realizar el primer análisis real sobre los datos de la última semana. Los datos llegan en bruto como colecciones de cadenas de texto, tal y como los exporta su sistema de caja antiguo.

---

## 📂 Los datos

El sistema de caja exporta cada transacción como una línea de texto con este formato:

```
"ID_TIENDA|PROVINCIA|PRODUCTO|CATEGORIA|IMPORTE|EMPLEADO_ID"
```

Aquí están los datos de la semana:

```scala
val transacciones = sc.parallelize(List(
  "T01|Madrid|Leche Entera|Lácteos|1.20|E03",
  "T02|Barcelona|Pan de Molde|Panadería|1.85|E07",
  "T03|Valencia|Leche Entera|Lácteos|1.20|E11",
  "T04|Madrid|Zumo de Naranja|Bebidas|2.40|E03",
  "T05|Sevilla|Yogur Natural|Lácteos|0.75|E15",
  "T06|Barcelona|Agua Mineral|Bebidas|0.60|E07",
  "T07|Madrid|Cerveza Rubia|Bebidas|1.10|E04",
  "T08|Valencia|Pan de Molde|Panadería|1.85|E12",
  "T09|Sevilla|Leche Entera|Lácteos|1.20|E15",
  "T10|Madrid|Yogur Natural|Lácteos|0.75|E03",
  "T11|Barcelona|Zumo de Naranja|Bebidas|2.40|E08",
  "T12|Valencia|Cerveza Rubia|Bebidas|1.10|E11",
  "T13|Sevilla|Agua Mineral|Bebidas|0.60|E16",
  "T14|Madrid|Leche Entera|Lácteos|1.20|E04",
  "T15|Barcelona|Pan de Molde|Panadería|1.85|E07",
  "T16|Valencia|Yogur Natural|Lácteos|0.75|E12",
  "T17|Sevilla|Zumo de Naranja|Bebidas|2.40|E15",
  "T18|Madrid|Agua Mineral|Bebidas|0.60|E03",
  "T19|Barcelona|Leche Entera|Lácteos|1.20|E08",
  "T20|Valencia|Pan de Molde|Panadería|1.85|E11",
  "T21|Sevilla|Cerveza Rubia|Bebidas|1.10|E16",
  "T22|Madrid|Zumo de Naranja|Bebidas|2.40|E04",
  "T23|Barcelona|Yogur Natural|Lácteos|0.75|E07",
  "T24|Valencia|Leche Entera|Lácteos|1.20|E12",
  "T25|Sevilla|Pan de Molde|Panadería|1.85|E15"
))
```

Además, el departamento de RRHH te proporciona el nombre de cada empleado:

```scala
val empleados = sc.parallelize(List(
  ("E03", "Carmen Vidal"),
  ("E04", "Luis Herrero"),
  ("E07", "Marta Soler"),
  ("E08", "Diego Fuentes"),
  ("E11", "Ana Romero"),
  ("E12", "Pablo Leal"),
  ("E15", "Rosa Cano"),
  ("E16", "Javier Mora")
))
```

---

## 🎯 Misión

El director de operaciones necesita respuestas a **6 preguntas de negocio** antes de la reunión del lunes. Tu trabajo es responderlas todas usando únicamente transformaciones RDD. Cada pregunta es independiente. Puedes resolverlas en el orden que prefieras, pero **todas parten del RDD `transacciones`** definido arriba.

---

## ❓ Preguntas de negocio

---

### Pregunta 1 — ¿Cuánto ha facturado cada provincia?

El director quiere saber el importe total vendido en cada provincia, ordenado de mayor a menor.

**Pista:** cada línea tiene la provincia en la posición 1 y el importe en la posición 4 (recuerda que `split("\\|")` divide por el carácter `|` y que `toDouble` convierte texto a número).

**Salida esperada:**

```
Facturación por provincia (mayor a menor):
  1. Madrid     → 9.65€
  2. Barcelona  → 8.65€
  3. Valencia   → 7.95€
  4. Sevilla    → 7.90€
```

> ⚠️ Los importes exactos dependen de los datos. Lo importante es que el resultado esté ordenado de mayor a menor y muestre todas las provincias.
> 

---

### Pregunta 2 — ¿Qué categorías de producto se venden en Madrid?

El jefe de compras quiere saber qué categorías están presentes en las tiendas de Madrid, **sin repetidos**.

**Pista:** filtra primero por provincia, luego extrae la categoría y elimina duplicados.

**Salida esperada:**

```
Categorías en Madrid:
  Bebidas
  Lácteos
```

> ℹ️ Madrid tiene transacciones de Bebidas y Lácteos, pero no de Panadería esta semana. El resultado puede variar si los datos cambian.
> 

---

### Pregunta 3 — ¿Cuántas transacciones ha gestionado cada empleado?

RRHH necesita saber cuántas ventas ha procesado cada empleado esta semana, mostrando su **nombre completo** (no solo el ID).

**Pista:** necesitas dos pasos. Primero cuenta las transacciones por ID de empleado con `reduceByKey`. Luego cruza ese resultado con el RDD `empleados` usando `join` para sustituir el ID por el nombre.

**Salida esperada:**

```
Transacciones por empleado:
  Ana Romero      → 3 transacciones
  Carmen Vidal    → 4 transacciones
  Diego Fuentes   → 2 transacciones
  Javier Mora     → 2 transacciones
  Luis Herrero    → 3 transacciones
  Marta Soler     → 4 transacciones
  Pablo Leal      → 3 transacciones
  Rosa Cano       → 4 transacciones
```

---

### Pregunta 4 — ¿Qué productos se venden tanto en Madrid como en Barcelona?

El equipo de marketing quiere lanzar una promoción conjunta en ambas ciudades y necesita saber qué productos tienen presencia en las **dos**.

**Pista:** crea un RDD de productos de Madrid, otro de Barcelona y usa `intersection`.

**Salida esperada:**

```
Productos en Madrid Y Barcelona:
  Agua Mineral
  Leche Entera
  Yogur Natural
  Zumo de Naranja
```

---

### Pregunta 5 — ¿Cuál es la facturación total por categoría?

Finanzas necesita el desglose de ingresos por categoría de producto para el informe mensual.

**Salida esperada:**

```
Facturación por categoría:
  Bebidas    → 14.70€
  Lácteos    → 10.20€
  Panadería  → 9.25€
```

---

### Pregunta 6 — Lista completa de productos únicos disponibles en la cadena

El equipo de logística necesita el catálogo completo de productos que se han vendido esta semana en cualquier provincia, sin repetidos y en orden alfabético.

**Pista:** extrae el producto de cada línea, aplica `distinct()` y ordena.

**Salida esperada:**

```
Catálogo de productos:
  Agua Mineral
  Cerveza Rubia
  Leche Entera
  Pan de Molde
  Yogur Natural
  Zumo de Naranja
```

---

## 📋 Plantilla de trabajo (opcional)

Puedes usar esta estructura de celdas para organizar tu solución:

```scala
// ── CELDA 0 — Datos ──────────────────────────────────────
// (pega aquí los RDDs de transacciones y empleados)

// ── CELDA 1 — Pregunta 1 ─────────────────────────────────
// Facturación por provincia

// ── CELDA 2 — Pregunta 2 ─────────────────────────────────
// Categorías en Madrid

// ── CELDA 3 — Pregunta 3 ─────────────────────────────────
// Transacciones por empleado (con nombre)

// ── CELDA 4 — Pregunta 4 ─────────────────────────────────
// Productos en Madrid Y Barcelona

// ── CELDA 5 — Pregunta 5 ─────────────────────────────────
// Facturación por categoría

// ── CELDA 6 — Pregunta 6 ─────────────────────────────────
// Catálogo completo de productos
```

---

## 🔑 Guía de transformaciones a usar

Esta tabla te recuerda qué transformaciones son útiles para cada tipo de tarea. No todas se usan en todas las preguntas.

| Necesito… | Transformación |
| --- | --- |
| Extraer un campo de cada línea | `map` |
| Dividir una línea en campos | `map` + `split` |
| Quedarme solo con algunas filas | `filter` |
| Eliminar duplicados | `distinct` |
| Elementos comunes entre dos RDDs | `intersection` |
| Todos los elementos de dos RDDs | `union` |
| Sumar valores por clave | `reduceByKey` |
| Cruzar dos RDDs por una clave común | `join` |
| Ordenar de mayor a menor | `sortBy(_._2, ascending = false)` |
| Ordenar alfabéticamente | `sortBy(x => x)` o `sortByKey()` |

---

# Sesión 2 - Acciones en RDDs y Persistencia

---

## 1. Transformaciones vs. acciones: recordatorio clave

En la sesión anterior trabajamos con **transformaciones**: operaciones que producen un nuevo RDD pero no ejecutan nada hasta que se les pide un resultado. Hoy nos centramos en las **acciones**: operaciones que **sí ejecutan** el pipeline completo y devuelven un resultado al driver o lo guardan en disco.

```
RDD  ──map──▶  RDD  ──filter──▶  RDD  ──reduceByKey──▶  RDD
                                                           │
                                              acción: collect()
                                                           │
                                                     Array[...]
                                                  (resultado en el driver)
```

> 💡 Cada vez que llamas a una acción, Spark recorre **todo el linaje** desde el principio y ejecuta todas las transformaciones acumuladas. Si llamas a dos acciones sobre el mismo RDD, Spark ejecuta el pipeline **dos veces**. La solución a esto es la **persistencia**, que veremos al final de esta sesión.
> 

---

## 2. Acciones de recuperación de datos

Estas acciones devuelven datos al programa driver (tu notebook o tu shell).

### 2.1 `collect()` — traer todos los elementos

Recoge **todos** los elementos del RDD y los devuelve como un `Array` en el driver.

```scala
val numeros = sc.parallelize(List(10, 20, 30, 40, 50))
val resultado = numeros.collect()
// resultado: Array[Int] = Array(10, 20, 30, 40, 50)
```

> ⚠️ Úsalo solo con RDDs pequeños. Si el RDD tiene millones de filas, `collect()` intentará cargarlos todos en la memoria del driver y puede provocar un `OutOfMemoryError`.
> 

---

### 2.2 `count()` — contar elementos

Devuelve el número total de elementos del RDD como un `Long`.

```scala
val palabras = sc.parallelize(List("spark", "scala", "big", "data", "rdd"))
palabras.count()
// Long = 5
```

---

### 2.3 `take(n)` — primeros n elementos

Devuelve un `Array` con los primeros `n` elementos. Mucho más seguro que `collect()` cuando solo necesitas echar un vistazo a los datos.

```scala
val numeros = sc.parallelize(1 to 100)
numeros.take(5)
// Array[Int] = Array(1, 2, 3, 4, 5)
```

> 💡 `take(n)` no garantiza orden a menos que el RDD esté ordenado. Úsalo para inspección rápida, no para resultados finales.
> 

---

### 2.4 `first()` — el primer elemento

Equivalente a `take(1)(0)`. Devuelve el primer elemento del RDD.

```scala
val letras = sc.parallelize(List("a", "b", "c", "d"))
letras.first()
// String = "a"
```

---

### 2.5 `takeSample(withReplacement, n)` — muestra aleatoria

Devuelve `n` elementos elegidos aleatoriamente. Útil para exploración de datos.

```scala
val datos = sc.parallelize(1 to 1000)
datos.takeSample(false, 5)
// Array[Int] = Array(423, 17, 891, 234, 66)  ← varía en cada ejecución
```

El primer parámetro indica si el muestreo es **con reemplazamiento** (`true`) o **sin reemplazamiento** (`false`).

---

## 3. Acciones de agregación

Estas acciones combinan todos los elementos del RDD en un único valor de resultado.

### 3.1 `reduce` — combinar todos los elementos con una función

Aplica una función de dos parámetros de forma repetida hasta obtener un único valor. La función debe ser **asociativa y conmutativa** (el orden de aplicación no debe importar), porque Spark puede aplicarla en paralelo en distintas particiones.

```scala
val numeros = sc.parallelize(List(1, 2, 3, 4, 5))

// Sumar todos
val suma = numeros.reduce((a, b) => a + b)
// suma: Int = 15

// Encontrar el máximo
val maximo = numeros.reduce((a, b) => if (a > b) a else b)
// maximo: Int = 5
```

Piénsalo como doblar una lista por la mitad repetidamente y combinar cada par:

```
[1, 2, 3, 4, 5]
 └──3──┘  └──9──┘
    └────12────┘ + 3
         └───15───┘
```

> ⚠️ `reduce` falla si el RDD está vacío. Si puede haber RDDs vacíos, usa `fold` en su lugar.
> 

---

### 3.2 `fold` — como `reduce`, pero con valor inicial

`fold` recibe un **valor neutro** (zero value) que se usa como punto de partida en cada partición y también para combinar los resultados parciales de las particiones.

```scala
val numeros = sc.parallelize(List(1, 2, 3, 4, 5))

// fold con neutro 0 para suma: igual que reduce
val suma = numeros.fold(0)((a, b) => a + b)
// suma: Int = 15

// fold con neutro 10: el 10 se aplica una vez por partición
// En modo local con 1 partición: 10 + 1 + 2 + 3 + 4 + 5 = 25
val conNeutro = numeros.fold(10)((a, b) => a + b)
// conNeutro: Int = 25  (en local con 1 partición)
```

> 💡 **¿Cuándo usar `fold` vs `reduce`?**
> 
> - Usa `fold` cuando el RDD puede estar vacío o cuando necesitas un valor inicial definido.
> - El valor neutro debe ser el **elemento neutro** de la operación: `0` para suma, `1` para producto, `""` para concatenación de strings.

---

### 3.3 `aggregate` — reducir cambiando el tipo del resultado

Es la acción de agregación más flexible. Permite que el **tipo del resultado sea diferente** al tipo de los elementos del RDD. Necesita tres argumentos:

1. **`zeroValue`**: valor inicial del acumulador
2. **`seqOp`**: cómo combinar un acumulador con un elemento del RDD
3. **`combOp`**: cómo combinar dos acumuladores de particiones distintas

**Ejemplo: calcular media con una sola pasada**

En lugar de hacer `sum()` y `count()` por separado (dos acciones = dos pasadas), lo hacemos en una sola pasada acumulando `(suma, conteo)`:

```scala
val numeros = sc.parallelize(List(1, 2, 3, 4, 5))

val (suma, conteo) = numeros.aggregate((0, 0))(
  (acc, elem) => (acc._1 + elem, acc._2 + 1),  // seqOp: acumula suma y conteo
  (a,   b)    => (a._1 + b._1,  a._2 + b._2)   // combOp: combina dos acumuladores
)

val media = suma.toDouble / conteo
// suma: 15, conteo: 5, media: 3.0
println(s"Suma: $suma, Conteo: $conteo, Media: $media")
// Suma: 15, Conteo: 5, Media: 3.0
```

Otra aplicación: **calcular mínimo y máximo en una sola pasada**

```scala
val datos = sc.parallelize(List(4, 7, 2, 9, 1, 5, 8, 3, 6))

val (minVal, maxVal) = datos.aggregate((Int.MaxValue, Int.MinValue))(
  (acc, x) => (math.min(acc._1, x), math.max(acc._2, x)),
  (a,   b) => (math.min(a._1, b._1), math.max(a._2, b._2))
)

println(s"Mínimo: $minVal, Máximo: $maxVal")
// Mínimo: 1, Máximo: 9
```

---

## 4. Acciones de efecto lateral

Estas acciones **no devuelven un valor útil** al driver. En cambio, ejecutan una función por su efecto (imprimir, escribir en base de datos, enviar a un sistema externo).

### 4.1 `foreach` — ejecutar una función en cada elemento

```scala
val ventas = sc.parallelize(List(120, 85, 200, 45, 310))

ventas.foreach(v => println(s"Venta: $v€"))
// Venta: 120€
// Venta: 85€
// Venta: 200€
// Venta: 45€
// Venta: 310€
```

> ⚠️ **Trampa habitual:** si dentro de `foreach` intentas acumular resultados en una variable del driver (como `var total = 0; rdd.foreach(x => total += x)`), ese código se ejecuta en los executors y la variable del driver **no se modifica**. Para acumuladores distribuidos existen los `Accumulator` (tema avanzado). Si solo necesitas sumar, usa `reduce` o `fold`.
> 

---

### 4.2 `foreachPartition` — ejecutar una función por partición

Como `foreach`, pero la función recibe un `Iterator` con todos los elementos de una partición en lugar de elemento a elemento. Útil para operaciones costosas de inicializar (conexiones a base de datos, clientes HTTP) donde prefieres abrir la conexión una vez por partición y no una vez por elemento.

```scala
val datos = sc.parallelize(List("registro1", "registro2", "registro3", "registro4"), 2)

datos.foreachPartition { particion =>
  println(s"--- Nueva partición ---")
  particion.foreach(elem => println(s"  Procesando: $elem"))
}
// --- Nueva partición ---
//   Procesando: registro1
//   Procesando: registro2
// --- Nueva partición ---
//   Procesando: registro3
//   Procesando: registro4
```

---

## 5. Acciones de escritura en disco

### 5.1 `saveAsTextFile` — guardar como ficheros de texto

Guarda cada elemento del RDD como una línea de texto. Spark crea una **carpeta** (no un fichero único) con tantos ficheros `part-00000`, `part-00001`… como particiones tenga el RDD.

```scala
val resultado = sc.parallelize(List("línea 1", "línea 2", "línea 3"))
resultado.saveAsTextFile("C:/Curso-Scala/salida/mi_resultado")
// Crea la carpeta mi_resultado/ con part-00000, part-00001...
```

> 💡 Para leer después lo que guardaste: `sc.textFile("C:/Curso-Scala/salida/mi_resultado")`
> 

> ⚠️ Si la carpeta ya existe, Spark lanza un error. Borra la carpeta antes de volver a ejecutar.
> 

---

### 5.2 `saveAsObjectFile` — guardar objetos serializados

Guarda el RDD en formato binario serializado de Java. Útil para guardar RDDs intermedios y recuperarlos en otro job de Spark sin perder los tipos.

```scala
val numeros = sc.parallelize(List(1, 2, 3, 4, 5))
numeros.saveAsObjectFile("C:/Curso-Scala/salida/numeros_binario")

// Para leerlo de nuevo:
val recuperado = sc.objectFile[Int]("C:/Curso-Scala/salida/numeros_binario")
recuperado.collect()
// Array(1, 2, 3, 4, 5)
```

---

## 6. Persistencia: no recalcular lo que ya calculaste

### 6.1 El problema: cada acción recalcula todo

```
val rdd = sc.parallelize(datos)
  .filter(condicion)
  .map(transformacion)

rdd.count()    // ← Spark ejecuta filter + map desde cero
rdd.collect()  // ← Spark ejecuta filter + map OTRA VEZ desde cero
```

Si el RDD resulta de un pipeline largo o de leer un fichero grande, este doble cálculo es muy costoso.

### 6.2 `cache()` — guardar en memoria

La forma más sencilla: guarda el RDD en memoria la primera vez que se materializa. Las siguientes acciones lo reutilizan sin recalcular.

```scala
val rdd = sc.parallelize(1 to 1000000)
  .filter(n => n % 2 == 0)
  .map(n => n * n)

rdd.cache()  // marca el RDD para ser persistido en memoria

rdd.count()    // ← primera ejecución: calcula Y guarda en caché
rdd.collect()  // ← segunda ejecución: lee de la caché, no recalcula
```

> 💡 `cache()` es un alias de `persist(StorageLevel.MEMORY_ONLY)`.
> 

---

### 6.3 `persist(nivel)` — control fino del almacenamiento

Permite elegir **dónde** guardar el RDD:

| Nivel | Dónde se guarda | RAM usada | CPU extra | Cuándo usarlo |
| --- | --- | --- | --- | --- |
| `MEMORY_ONLY` | Solo RAM | Alta | Baja | RDD que cabe en memoria, se reutiliza mucho |
| `MEMORY_AND_DISK` | RAM; si no cabe, disco | Media | Media | RDD grande que puede no caber en memoria |
| `DISK_ONLY` | Solo disco | Baja | Alta | RDD enorme, memoria escasa |

```scala
import org.apache.spark.storage.StorageLevel

val rdd = sc.parallelize(1 to 1000000).map(n => n * n)

// Solo en memoria (por defecto con cache())
rdd.persist(StorageLevel.MEMORY_ONLY)

// En memoria y disco si no cabe
rdd.persist(StorageLevel.MEMORY_AND_DISK)

// Solo en disco
rdd.persist(StorageLevel.DISK_ONLY)
```

---

### 6.4 `unpersist()` — liberar la caché

Cuando ya no necesites el RDD en caché, libera la memoria explícitamente:

```scala
rdd.unpersist()
```

> 💡 Spark también gestiona la caché automáticamente con una política LRU (Least Recently Used): si la memoria se llena, expulsa los RDDs menos usados recientemente. Pero en sesiones largas con muchos RDDs, llamar a `unpersist()` manualmente es una buena práctica.
> 

---

## 7. Resumen: ¿qué acción uso en cada caso?

| Necesito… | Acción |
| --- | --- |
| Ver todos los datos (RDD pequeño) | `collect()` |
| Saber cuántos elementos hay | `count()` |
| Ver los primeros N elementos | `take(n)` |
| Ver solo el primero | `first()` |
| Una muestra aleatoria | `takeSample(false, n)` |
| Combinar todos en un valor del mismo tipo | `reduce` |
| Combinar con valor inicial o RDD posiblemente vacío | `fold` |
| Combinar cambiando el tipo (ej: media, min+max) | `aggregate` |
| Ejecutar algo por cada elemento (sin devolver) | `foreach` |
| Ejecutar algo por partición (ej: conexión BD) | `foreachPartition` |
| Guardar como texto legible | `saveAsTextFile` |
| Guardar para reusar en Spark | `saveAsObjectFile` |

---

---

# 💻 Practica— Acciones en RDDs y Persistencia

---

### 🟢 Ejercicio 1 — `count`, `take`, `first`: explorar un RDD

Crea un RDD con los nombres de 10 ciudades españolas y usa las acciones básicas para explorarlo.

```scala
// CELDA — Ejercicio 1
val ciudades = sc.parallelize(List(
  "Madrid", "Barcelona", "Valencia", "Sevilla", "Zaragoza",
  "Málaga", "Murcia", "Palma", "Bilbao", "Alicante"
))

println(s"Total de ciudades: ${ciudades.count()}")
println(s"Primera ciudad: ${ciudades.first()}")
println(s"Primeras 3: ${ciudades.take(3).mkString(", ")}")
println(s"Muestra aleatoria de 4: ${ciudades.takeSample(false, 4).mkString(", ")}")
```

**Salida esperada:**

```
Total de ciudades: 10
Primera ciudad: Madrid
Primeras 3: Madrid, Barcelona, Valencia
Muestra aleatoria de 4: (varía en cada ejecución)
```

---

### 🟢 Ejercicio 2 — `reduce`: encontrar el máximo y la suma

Tienes una lista de temperaturas registradas durante una semana. Usa `reduce` para encontrar la temperatura máxima y para sumarlas todas.

```scala
// CELDA — Ejercicio 2
val temperaturas = sc.parallelize(List(18.5, 22.0, 15.3, 27.8, 24.1, 19.6, 21.4))

val tempMaxima  = temperaturas.reduce((a, b) => if (a > b) a else b)
val tempMinima  = temperaturas.reduce((a, b) => if (a < b) a else b)
val sumaTotal   = temperaturas.reduce((a, b) => a + b)

println(f"Temperatura máxima: $tempMaxima%.1f °C")
println(f"Temperatura mínima: $tempMinima%.1f °C")
println(f"Suma total:         $sumaTotal%.1f °C")
```

**Salida esperada:**

```
Temperatura máxima: 27.8 °C
Temperatura mínima: 15.3 °C
Suma total:         148.7 °C
```

---

### 🟢 Ejercicio 3 — `fold`: concatenar palabras con separador

Usa `fold` para unir una lista de palabras en una sola frase, añadiendo un espacio entre cada una. Fíjate en el papel del valor neutro.

```scala
// CELDA — Ejercicio 3
val palabras = sc.parallelize(List("Apache", "Spark", "con", "Scala"), 1)
// Nota: usamos 1 partición para que el resultado sea predecible con fold

val frase = palabras.fold("")((acc, palabra) =>
  if (acc.isEmpty) palabra else acc + " " + palabra
)

println(s"Frase: '$frase'")
println(s"¿Por qué el valor neutro es cadena vacía? Porque '' + palabra = palabra (elemento neutro de la concatenación)")
```

**Salida esperada:**

```
Frase: 'Apache Spark con Scala'
¿Por qué el valor neutro es cadena vacía? Porque '' + palabra = palabra (elemento neutro de la concatenación)
```

> 💡 Fijamos `numSlices = 1` al crear el RDD porque `fold` aplica el `zeroValue` una vez **por partición**. Con más de una partición, el resultado dependería del número de particiones, lo cual puede sorprender a quien no lo sabe.
> 

---

### 🟡 Ejercicio 4 — `aggregate`: media, mínimo y máximo en una sola pasada

Tienes las puntuaciones de un examen. Calcula la media, el mínimo y el máximo en una única acción `aggregate`, sin recorrer el RDD más de una vez.

```scala
// CELDA — Ejercicio 4
val notas = sc.parallelize(List(6.5, 8.0, 4.5, 9.5, 7.0, 5.5, 8.5, 6.0, 7.5, 10.0))

// El acumulador es una tupla de tres: (suma, conteo, max)
// zeroValue: suma=0.0, conteo=0, max=mínimo posible
val (suma, conteo, maximo) = notas.aggregate((0.0, 0, Double.MinValue))(
  (acc, nota) => (acc._1 + nota, acc._2 + 1, math.max(acc._3, nota)),   // seqOp
  (a, b)      => (a._1 + b._1,  a._2 + b._2, math.max(a._3, b._3))     // combOp
)

val media = suma / conteo

println(f"Notas analizadas: $conteo")
println(f"Media:   $media%.2f")
println(f"Máxima:  $maximo%.1f")
```

**Salida esperada:**

```
Notas analizadas: 10
Media:   7.30
Máxima:  10.0
```

> 💡 Fíjate en que el acumulador tiene **tres campos** `(suma, conteo, max)` pero los elementos del RDD son simples `Double`. Ese es el poder de `aggregate`: el tipo del acumulador puede ser completamente distinto al tipo de los elementos.
> 

---

### 🟡 Ejercicio 5 — `foreach`: imprimir con formato

Tienes un Pair RDD de productos con su precio. Usa `foreach` para imprimir cada producto con un formato de etiqueta de precio.

```scala
// CELDA — Ejercicio 5
val catalogo = sc.parallelize(List(
  ("Café con leche",   1.80),
  ("Tostada con tomate", 2.50),
  ("Zumo de naranja",  2.20),
  ("Croissant",        1.60),
  ("Agua mineral",     1.00)
))

println("=== CARTA DE PRECIOS ===")
catalogo
  .sortBy(_._2)  // ordenar por precio antes de imprimir
  .foreach { case (producto, precio) =>
    println(f"  %-25s $precio%.2f €".format(producto))
  }
```

**Salida esperada:**

```
=== CARTA DE PRECIOS ===
  Agua mineral              1.00 €
  Croissant                 1.60 €
  Café con leche            1.80 €
  Zumo de naranja           2.20 €
  Tostada con tomate        2.50 €
```

> ⚠️ Recuerda: si intentaras acumular la suma dentro del `foreach` con `var total = 0.0; catalogo.foreach(x => total += x._2)`, el resultado en el driver siempre sería `0.0` porque el código se ejecuta en los executors. Para sumar, usa `reduce` o `map` + `reduce`.
> 

---

### 🟡 Ejercicio 6 — `saveAsTextFile` y lectura: guardar y recuperar

Guarda los resultados de un análisis en disco y vuélvelos a leer para verificar que se guardaron correctamente.

```scala
// CELDA — Ejercicio 6a: guardar
val ventas = sc.parallelize(List(
  "Enero,12500",
  "Febrero,9800",
  "Marzo,15200",
  "Abril,11300",
  "Mayo,18700"
))

val rutaSalida = "C:/Curso-Scala/salida/ventas_mensuales"

// Si la carpeta ya existe de una ejecución anterior, la borramos primero
import org.apache.hadoop.fs.{FileSystem, Path}
val fs = FileSystem.get(sc.hadoopConfiguration)
if (fs.exists(new Path(rutaSalida))) fs.delete(new Path(rutaSalida), true)

ventas.saveAsTextFile(rutaSalida)
println(s"✅ RDD guardado en $rutaSalida")
```

```scala
// CELDA — Ejercicio 6b: leer y verificar
val recuperado = sc.textFile(rutaSalida)

println(s"Líneas recuperadas: ${recuperado.count()}")
println("Contenido:")
recuperado.collect().foreach(println)
```

**Salida esperada:**

```
✅ RDD guardado en C:/Curso-Scala/salida/ventas_mensuales

Líneas recuperadas: 5
Contenido:
Enero,12500
Febrero,9800
Marzo,15200
Abril,11300
Mayo,18700
```

> 💡 La carpeta `ventas_mensuales/` contendrá un fichero `part-00000` (y posiblemente más, según las particiones). Spark siempre escribe en una carpeta, nunca en un fichero único, para poder paralelizar la escritura.
> 

---

### 🔴 Ejercicio 7 — Persistencia: medir el impacto del `cache()`

Construye un pipeline con varias transformaciones y mide la diferencia de tiempo entre ejecutarlo sin caché (dos pasadas) y con caché (una sola pasada real).

```scala
// CELDA — Ejercicio 7
val datos = sc.parallelize(1 to 500000)

// Pipeline con varias transformaciones
val procesado = datos
  .filter(n => n % 3 == 0)
  .map(n => n * n)
  .filter(n => n % 2 != 0)

// --- Sin caché: cada acción recalcula el pipeline ---
val t0 = System.currentTimeMillis()
val cuenta1 = procesado.count()
val t1 = System.currentTimeMillis()
val suma1 = procesado.reduce((a, b) => a + b)
val t2 = System.currentTimeMillis()

println("=== SIN CACHÉ ===")
println(f"  count()  → $cuenta1 elementos  (${t1 - t0} ms)")
println(f"  reduce() → suma = $suma1  (${t2 - t1} ms)")
println(f"  Total: ${t2 - t0} ms (el pipeline se ejecutó DOS veces)")

// --- Con caché: el pipeline se ejecuta una sola vez ---
procesado.cache()

val t3 = System.currentTimeMillis()
val cuenta2 = procesado.count()   // ← primera acción: calcula Y guarda en caché
val t4 = System.currentTimeMillis()
val suma2 = procesado.reduce((a, b) => a + b)  // ← segunda acción: lee de la caché
val t5 = System.currentTimeMillis()

println("\n=== CON CACHÉ ===")
println(f"  count()  → $cuenta2 elementos  (${t4 - t3} ms, incluye cálculo + caché)")
println(f"  reduce() → suma = $suma2  (${t5 - t4} ms, solo lectura de caché)")
println(f"  Total: ${t5 - t3} ms")

procesado.unpersist()
println("\n✅ Caché liberada con unpersist()")
```

**Salida esperada (los tiempos varían según el equipo):**

```
=== SIN CACHÉ ===
  count()  → 83333 elementos  (320 ms)
  reduce() → suma = ...       (310 ms)
  Total: 630 ms (el pipeline se ejecutó DOS veces)

=== CON CACHÉ ===
  count()  → 83333 elementos  (340 ms, incluye cálculo + caché)
  reduce() → suma = ...       (45 ms, solo lectura de caché)
  Total: 385 ms

✅ Caché liberada con unpersist()
```

> 💡 La primera acción con caché tarda un poco más porque guarda los datos mientras calcula. Pero la segunda acción es drásticamente más rápida. Cuantas más acciones hagas sobre el mismo RDD, más rentable es el caché.
> 

---

### 🔴 Ejercicio 8 — `persist(MEMORY_AND_DISK)`: elegir el nivel adecuado

Usa `persist` con nivel `MEMORY_AND_DISK` y observa la diferencia respecto a `cache()`. Este ejercicio también repasa las transformaciones de la sesión anterior.

```scala
// CELDA — Ejercicio 8
import org.apache.spark.storage.StorageLevel

val textos = sc.parallelize(List(
  "spark es rápido y escalable",
  "scala es el lenguaje de spark",
  "los rdds son la base de spark",
  "la persistencia mejora el rendimiento",
  "cache y persist son herramientas clave",
  "memory and disk es un nivel intermedio",
  "unpersist libera la memoria del cluster"
))

// Pipeline: wordcount completo
val conteo = textos
  .flatMap(_.split(" "))
  .map(palabra => (palabra, 1))
  .reduceByKey(_ + _)
  .filter(_._2 > 1)  // solo palabras que aparecen más de una vez

// Persistir en memoria y disco (más seguro si los datos son grandes)
conteo.persist(StorageLevel.MEMORY_AND_DISK)

// Primera acción: materializa y persiste
println("Palabras que aparecen más de una vez:")
conteo.collect().sortBy(-_._2).foreach { case (w, n) =>
  println(s"  '$w' → $n veces")
}

// Segunda acción: usa la caché
println(s"\nTotal de palabras repetidas: ${conteo.count()}")

conteo.unpersist()
```

**Salida esperada:**

```
Palabras que aparecen más de una vez:
  'spark' → 3 veces
  'es' → 3 veces
  'la' → 3 veces
  'y' → 2 veces
  'el' → 2 veces
  'de' → 2 veces
  'son' → 2 veces

Total de palabras repetidas: 7
```

---

## 🔧 Resolución de problemas comunes

### Error: `org.apache.hadoop.mapred.FileAlreadyExistsException`

La carpeta de salida del `saveAsTextFile` ya existe de una ejecución anterior. El código del Ejercicio 6 ya incluye la limpieza automática con `fs.delete(...)`. Si lo ves en otro ejercicio, borra la carpeta manualmente desde el Explorador de Windows o usa el mismo patrón.

---

### `fold` con resultado extraño (mayor de lo esperado)

Si el `fold` devuelve un número más alto de lo esperado, es porque el `zeroValue` se aplica una vez por partición. Con `parallelize(lista, 4)` y `fold(10)(suma)`, el 10 se suma 4 veces (una por partición) más una vez al combinar los resultados. Para evitar sorpresas en ejemplos de clase, crea el RDD con `numSlices = 1` o usa `reduce` cuando no necesites el valor neutro.

---

### Los tiempos del Ejercicio 7 no muestran diferencia

En modo local con pocos datos los tiempos pueden ser muy similares porque todo cabe en memoria de todas formas y el overhead de Spark domina. Con 500.000 elementos suele verse diferencia. Si no se aprecia, aumenta a `1 to 2000000`.

---

---

## 📚 Referencia rápida de acciones RDD

| Acción | Devuelve | Notas |
| --- | --- | --- |
| `collect()` | `Array[T]` | ⚠️ Solo con RDDs pequeños |
| `count()` | `Long` | Número de elementos |
| `take(n)` | `Array[T]` | Primeros n (sin orden garantizado) |
| `first()` | `T` | El primero |
| `takeSample(wr, n)` | `Array[T]` | Muestra aleatoria |
| `reduce(f)` | `T` | Requiere RDD no vacío |
| `fold(z)(f)` | `T` | Con valor neutro; z se aplica por partición |
| `aggregate(z)(seq, comb)` | `U` | Resultado puede ser de otro tipo |
| `foreach(f)` | `Unit` | Efecto lateral en executors |
| `foreachPartition(f)` | `Unit` | Una llamada por partición |
| `saveAsTextFile(path)` | `Unit` | Crea carpeta con ficheros part-* |
| `saveAsObjectFile(path)` | `Unit` | Binario, releer con `objectFile[T]` |

---

# 🏢 Caso de Estudio Propuesto 2 : 📞 Atención360 S.L.

---

## 🏢 Contexto de la empresa

**Atención360 S.L.** es una empresa de atención al cliente que gestiona llamadas de soporte técnico para varias compañías de telecomunicaciones. Su director de operaciones necesita un informe semanal con métricas de rendimiento: duración de las llamadas, satisfacción del cliente y tasa de resolución por agente.

El equipo de datos ha volcado los registros de la semana en un fichero plano. Te encargan procesar esos datos con Spark y responder a **seis preguntas de negocio** usando exclusivamente acciones RDD.

---

## 📂 Los datos

Cada registro representa una llamada atendida. El formato es:

```
"AGENTE_ID|NOMBRE_AGENTE|DURACION_SEG|SATISFACCION|ESTADO"
```

- `AGENTE_ID`: código del agente (A01, A02, A03)
- `NOMBRE_AGENTE`: nombre completo
- `DURACION_SEG`: duración de la llamada en segundos
- `SATISFACCION`: puntuación del cliente del 1 (muy malo) al 5 (excelente)
- `ESTADO`: `RESUELTA` o `NO_RESUELTA`

```scala
val registros = sc.parallelize(List(
  "A01|Laura Méndez|245|4|RESUELTA",
  "A02|Carlos Reyes|180|5|RESUELTA",
  "A01|Laura Méndez|320|3|NO_RESUELTA",
  "A03|Sofía Ibáñez|95|5|RESUELTA",
  "A02|Carlos Reyes|410|2|NO_RESUELTA",
  "A01|Laura Méndez|150|5|RESUELTA",
  "A03|Sofía Ibáñez|280|4|RESUELTA",
  "A02|Carlos Reyes|190|4|RESUELTA",
  "A01|Laura Méndez|530|1|NO_RESUELTA",
  "A03|Sofía Ibáñez|210|5|RESUELTA",
  "A02|Carlos Reyes|175|5|RESUELTA",
  "A01|Laura Méndez|90|4|RESUELTA",
  "A03|Sofía Ibáñez|340|3|NO_RESUELTA",
  "A02|Carlos Reyes|265|4|RESUELTA",
  "A03|Sofía Ibáñez|120|5|RESUELTA",
  "A01|Laura Méndez|480|2|NO_RESUELTA",
  "A02|Carlos Reyes|155|5|RESUELTA",
  "A03|Sofía Ibáñez|390|3|NO_RESUELTA",
  "A01|Laura Méndez|220|4|RESUELTA",
  "A02|Carlos Reyes|310|3|RESUELTA"
))
```

---

## 🎯 Misión

Responde las seis preguntas usando las acciones vistas en clase: `count`, `take`, `first`, `reduce`, `fold`, `aggregate`, `foreach` y `saveAsTextFile`. Cada pregunta indica qué acción se espera que uses.

Todas las preguntas parten del mismo RDD `registros`. Como vas a usarlo varias veces, **persístelo en memoria antes de empezar a responder**:

```scala
registros.cache()
```

---

## ❓ Preguntas de negocio

---

### Pregunta 1 — Inspección inicial

**Acción a usar:** `count`, `first`, `take`

Antes de analizar nada, el director quiere saber cuántas llamadas hay en total y ver un par de registros para confirmar que el formato es correcto.

Muestra: el total de llamadas, el primer registro, y los tres primeros registros.

**Salida esperada:**

```
Total de llamadas: 20
Primer registro: A01|Laura Méndez|245|4|RESUELTA
Primeras 3:
  A01|Laura Méndez|245|4|RESUELTA
  A02|Carlos Reyes|180|5|RESUELTA
  A01|Laura Méndez|320|3|NO_RESUELTA
```

---

### Pregunta 2 — Estadísticas de duración

**Acción a usar:** `reduce`

El responsable de calidad quiere saber la duración **total**, la **máxima** y la **mínima** de todas las llamadas de la semana.

**Pista:** para obtener cada estadística necesitarás extraer la duración de cada línea con `map` y luego aplicar `reduce` tres veces. La duración está en la posición 2 del registro (`split("\\|")(2).toInt`).

**Salida esperada:**

```
Duración total:   5155 segundos
Duración máxima:  530 segundos
Duración mínima:  90 segundos
```

---

### Pregunta 3 — Tasa de resolución global

**Acción a usar:** `filter` + `count`

Recursos Humanos necesita saber cuántas llamadas se resolvieron y cuántas no, y calcular el porcentaje de resolución sobre el total.

**Salida esperada:**

```
Llamadas resueltas:     14
Llamadas no resueltas:  6
Tasa de resolución:     70.00%
```

---

### Pregunta 4 — Media de satisfacción y duración en una sola pasada

**Acción a usar:** `aggregate`

El director de operaciones quiere la **media de satisfacción** y la **media de duración** del conjunto completo. Como el RDD está en caché y queremos ser eficientes, calcula ambas medias en una sola acción `aggregate`.

El acumulador tendrá la forma `(sumaDuracion, sumaSatisfaccion, conteo)`.

**Pista:** el zeroValue es `(0, 0, 0)`. En `seqOp` extrae la duración y la satisfacción de cada línea y acumúlalas. En `combOp` suma los dos acumuladores campo a campo.

**Salida esperada:**

```
Media de duración:      257.8 segundos
Media de satisfacción:  3.80 / 5
```

---

### Pregunta 5 — La llamada más larga

**Acción a usar:** `reduce`

El equipo de formación quiere revisar la llamada de mayor duración para usarla como ejemplo de caso complejo.

Muestra todos los datos de esa llamada.

**Salida esperada:**

```
Llamada de mayor duración:
  Agente:        Laura Méndez (A01)
  Duración:      530 segundos
  Satisfacción:  1 / 5
  Estado:        NO_RESUELTA
```

---

### Pregunta 6 — Informe por agente y guardado en disco

**Acciones a usar:** `filter` + `aggregate` por agente, `foreach` para imprimir, `saveAsTextFile` para guardar

Esta es la pregunta central del informe semanal. Para cada agente calcula:

- Número de llamadas atendidas
- Duración media de sus llamadas
- Satisfacción media de sus clientes
- Número de llamadas resueltas

Primero imprime el informe en pantalla con `foreach`. Luego guarda las líneas del informe en disco con `saveAsTextFile` en `C:/Curso-Scala/salida/informe_agentes`.

**Pista:** trabaja agente a agente: filtra el RDD por cada agente, aplica `aggregate` para obtener sus métricas y construye una línea de texto con el resumen. Guarda esas líneas en un nuevo RDD.

**Salida esperada en pantalla:**

```
========= INFORME SEMANAL DE AGENTES =========
  Laura Méndez  (A01): 7 llamadas | dur. media: 291s | sat. media: 3.29 | resueltas: 4/7
  Carlos Reyes  (A02): 7 llamadas | dur. media: 241s | sat. media: 4.00 | resueltas: 6/7
  Sofía Ibáñez  (A03): 6 llamadas | dur. media: 239s | sat. media: 4.17 | resueltas: 4/6
==============================================
```

---

## 📋 Plantilla de trabajo (opcional)

```scala
// ── CELDA 0 — Datos y caché ──────────────────────────────
// (define registros y llama a registros.cache())

// ── CELDA 1 — Pregunta 1: inspección inicial ─────────────

// ── CELDA 2 — Pregunta 2: estadísticas de duración ───────

// ── CELDA 3 — Pregunta 3: tasa de resolución ─────────────

// ── CELDA 4 — Pregunta 4: medias con aggregate ───────────

// ── CELDA 5 — Pregunta 5: llamada más larga ──────────────

// ── CELDA 6 — Pregunta 6: informe por agente ─────────────
//   6a: calcular métricas por agente
//   6b: imprimir con foreach
//   6c: guardar con saveAsTextFile

// ── CELDA 7 — Liberar caché ──────────────────────────────
// registros.unpersist()
```

---

## 🔑 Guía de acciones a usar

| Necesito… | Acción |
| --- | --- |
| Saber cuántos registros hay | `count()` |
| Ver un registro de muestra | `first()` |
| Ver los primeros N registros | `take(n)` |
| Obtener un único valor (suma, max, min) | `reduce` |
| Obtener varios valores a la vez (media, conteo…) | `aggregate` |
| Ejecutar código por cada elemento sin devolver valor | `foreach` |
| Guardar resultados en disco | `saveAsTextFile` |
| Reducir el número de pasadas sobre el RDD | `cache()` antes de la primera acción |