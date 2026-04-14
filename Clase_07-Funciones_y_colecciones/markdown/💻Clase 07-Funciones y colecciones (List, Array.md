# 💻Clase 07 - Funciones y colecciones (List, Array, Vector)

---

# Agenda:

<aside>
💡

#### 9:00 - 9:50    →  **Sesión 1: Funciones en Scala**

#### 9:50 - 11:20   → Practicas

#### 11:40 - 12:40  → Sesión 2: Colecciones en Scala I: `List`, `Array` y `Vector`

#### 12:40 - 14:00  → Practicas

</aside>

# Sesión 1. Funciones en Scala

### 1. Declaración de funciones con `def`

> La sintaxis básica de una función en Scala es:
> 

```
def nombreFuncion(param1: Tipo1, param2: Tipo2): TipoRetorno = cuerpo
```

```scala
def saludar(nombre: String): String = s"Hola, $nombre!"

saludar("Scala")   

// el kernel muestra: res0: String = "Hola, Scala!"
```

> 💡 **Ventaja del notebook:** si la última expresión de la celda devuelve un valor, Almond lo muestra directamente sin `println`. Esto resulta muy cómodo para probar funciones.
> 

Para funciones con varias instrucciones se usa un bloque `{}`. La última expresión del bloque es el valor de retorno (no se necesita `return`):

```scala
def describir(nombre: String, edad: Int): String = {
  val categoria = if (edad >= 18) "adulto" else "menor"
  s"$nombre tiene $edad años y es $categoria"   // ← valor de retorno
}

describir("Laura", 25)
// res1: String = "Laura tiene 25 años y es adulto"
```

> 💡 En Scala **no se usa `return`** en el 99% de los casos. La última expresión del bloque es automáticamente el valor devuelto.
> 

---

### 2. Parámetros y tipos de retorno

### 2.1 Inferencia del tipo de retorno

Con tipo de retorno explícito (recomendado para funciones reutilizables).

```scala
def sumar(a: Int, b: Int): Int = a + b

sumar(2,3)
```

Sin tipo de retorno (Scala lo infiere)

```scala
def restar(a: Int, b: Int) = a - b
restar(2,5)
```

### 2.2 Parámetros con valor por defecto

> Los **parámetros con valor por defecto** (también llamados *default parameters*) son parámetros de una función que tienen un valor predefinido que se usa automáticamente si el llamador no proporciona ese argumento.
> 

**Ejemplo 1:**

```scala
def conectar(host: String, puerto: Int = 5432): String =
  s"Conectando a $host:$puerto"
```

- `host` es un parámetro obligatorio — siempre hay que pasarlo.
- `puerto` es un parámetro por defecto .Tiene el valor por defecto `5432` — si no se pasa, Scala lo usa automáticamente.

```scala
conectar("localhost")        // usa puerto 5432 por defecto
```

```scala
conectar("servidor", 3306)  // especifica puerto
```

**Ejemplo  2:**

```scala
def saludar(nombre: String, saludo: String = "Hola", puntos: Int = 3): String =
  s"$saludo, $nombre" + "!" * puntos
```

Puedes llamarla de varias formas:

```scala
saludar("Ana")                        // "Hola, Ana!!!"
saludar("Ana", "Buenos días")         // "Buenos días, Ana!!!"
saludar("Ana", "Hey", 1)             // "Hey, Ana!"
```

<aside>
💡

**Tip de Scala:** puedes usar **argumentos con nombre** para saltar parámetros intermedios:

```scala
saludar("Ana", puntos = 5)           // "Hola, Ana!!!!!"
```

</aside>

### 2.3 Parámetros nombrados

> Los **parámetros nombrados** (*named parameters*) permiten pasar los argumentos a una función especificando explícitamente el nombre del parámetro, en lugar de depender del orden en que están definidos.
> 

```scala
def crearUsuario(nombre: String, rol: String, activo: Boolean): String =
  s"[$rol] $nombre — activo: $activo"
```

```scala
// El orden no importa cuando se usan parámetros nombrados
crearUsuario(rol = "admin", activo = true, nombre = "Ana")
```

### 2.4 Funciones `Unit` — sin valor de retorno

Cuando una función solo produce efectos (como imprimir), su tipo de retorno es `Unit`:

**Ejemplo 1:**

```scala
def despedirse(nombre: String): Unit =
  s"Adiós, $nombre"

despedirse("Ana")    // 
despedirse("Luis")   // 
```

Salida:

```
defined function despedirse
```

**Ejemplo 2:**

```scala
def despedirse(nombre: String): String =
  s"Adiós, $nombre"

despedirse("Ana")    // imprime: Adiós, Ana
despedirse("Luis")   // imprime: Adiós, Luis
```

Salida:

```
defined function despedirse
res30_1: String = "Adiós, Ana"
res30_2: String = "Adiós, Luis"
```

<aside>
💡

#### ¿Para que se usa `Unit`  en Big Data ?

1. **Escribir resultados al disco o a S3**
    
    ```scala
    def guardarResultados(df: DataFrame, ruta: String): Unit =
      df.write.parquet(ruta)
    ```
    
    *No devuelve nada — solo escribe el archivo.*
    
2. **Registrar logs y métricas**
    
    ```scala
    def logearMetrica(nombre: String, valor: Long): Unit =
      logger.info(s"Métrica $nombre = $valor registros procesados")
    ```
    
3. **Inicializar Spark**
    
    ```scala
    def iniciarSpark(): Unit = {
      val spark = SparkSession.builder()
        .appName("MiJob")
        .getOrCreate()
    }
    ```
    
4. **Enviar alertas o notificaciones**
    
    ```scala
    def alertarError(mensaje: String): Unit =
      slack.enviar(s"🚨 Error en pipeline: $mensaje")
    ```
    
</aside>

---

### 3. Funciones anónimas (lambdas)

> Una **función anónima** es una función sin nombre. Se define con la sintaxis `(params) => cuerpo`:
> 

**Ejemplo 1:** Comparación entre una función habitual (`def`) y una función anónima:

- **Función con** `def` **:**

```scala
Con def — tiene nombre
def saludar(nombre: String): String = s"Hola, $nombre!"
saludar("Ana")
```

Salida:

```
defined function saludar
res32_1: String = "Hola, Ana!"
```

- **Función anónima (pura):**

```scala
((nombre: String) => s"Hola, $nombre!")("Ana")
```

Salida:

```
res35: String = "Hola, Ana!"
```

- **Anónima guardada en `val` - el nombre lo da el `val`**

```scala
val saludar = (nombre: String) => s"Hola, $nombre!"
saludar("Ana")
```

Salida:

```
saludar: String => String = ammonite.$sess.cmd36$Helper$$Lambda$3092/0x00000250d3a771a8@2dd66673
res36_1: String = "Hola, Ana!"
```

Ejemplo 2: **Calcular IVA**

```scala
// 1. Con def
def calcularIVAMétodo(precio: Double): Double = {
  precio * 0.21
}

// Uso
val total = calcularIVAMétodo(100.0)
println(total) // 21.0
```

```scala
// 2. Anonima pura
((precio: Double) => (precio * 0.21))(100.0)
```

```scala
// 3. Guardada en val
val calcularIVAAnónima = (precio: Double) => precio * 0.21

// Uso
val total = calcularIVAAnónima(100.0)
println(total) // 21.0
```

**Mas ejemplo:**

```scala
val doblar = (x: Int) => x * 2

doblar(5)    // res: Int = 10
doblar(12)   // res: Int = 24
```

```scala
val sumar = (a: Int, b: Int) => a + b
sumar(3, 7)   // res: Int = 10
```

```scala
// Forma abreviada con _ (cuando el parámetro se usa exactamente una vez)
val incrementar: Int => Int = _ + 1
incrementar(9)   // res: Int = 10
```

> Las lambdas son especialmente útiles como argumentos de otras funciones:
> 

```scala
def aplicar(f: Int => Int, numero: Int): Int = f(numero)

// Le pasamos una lambda
aplicar(x => x * 2, 5)  // 10
aplicar(x => x + 1, 5)  // 6
aplicar(x => x * x, 5)  // 25
```

Salida:

```
defined function aplicar
res43_1: Int = 10
res43_2: Int = 6
res43_3: Int = 25
```

---

### 4. Funciones de orden superior: `map`, `filter`, `reduce`

> Una **función de orden superior** es aquella que recibe o devuelve otra función. Son el corazón de la programación funcional en Scala y la base de las transformaciones en Spark.
> 

### 4.1 `map` — transformar cada elemento

```scala
val precios = List(10.0, 25.0, 50.0, 8.0)

val preciosConIVA = precios.map(p => p * 1.21)
// List(12.1, 30.25, 60.5, 9.68)
```

```scala
// Forma abreviada con _
val dobles = precios.map(_ * 2)
// List(20.0, 50.0, 100.0, 16.0)
```

### 4.2 `filter` — seleccionar elementos

```scala
val numeros = List(1, 2, 3, 4, 5, 6, 7, 8, 9, 10)

val pares   = numeros.filter(_ % 2 == 0)
// List(2, 4, 6, 8, 10)
```

```scala
val mayores = numeros.filter(_ > 5)
// List(6, 7, 8, 9, 10)
```

### 4.3 `reduce` — agregar en un único valor

```scala
val numeros = List(1, 2, 3, 4, 5)

val suma = numeros.reduce((acc, x) => acc + x)   // 15
```

- `acc` es la abreviatura de **acumulador**.
- `reduce` recorre la lista y va acumulando el resultado paso a paso:

```
List(1, 2, 3, 4, 5)

Paso 1: acc=1,  x=2  → 1+2  = 3
Paso 2: acc=3,  x=3  → 3+3  = 6
Paso 3: acc=6,  x=4  → 6+4  = 10
Paso 4: acc=10, x=5  → 10+5 = 15
```

`acc` es solo un nombre — puedes llamarlo como quieras:

```scala
numeros.reduce((total, x) => total + x)   // igual
numeros.reduce((a, b) => a + b)            // igual
numeros.reduce(_ + _)                      // igual, forma más corta
```

**Mas ejemplos:**

```scala
val producto = numeros.reduce(_ * _)                  // 120
```

```scala
val maximo   = numeros.reduce((a, b) => if (a > b) a else b)  // maximo: Int = 5
```

Paso a paso:

```
List(1, 2, 3, 4, 5)

Paso 1: a=1, b=2  → ¿1 > 2? No  → devuelve b=2
Paso 2: a=2, b=3  → ¿2 > 3? No  → devuelve b=3
Paso 3: a=3, b=4  → ¿3 > 4? No  → devuelve b=4
Paso 4: a=4, b=5  → ¿4 > 5? No  → devuelve b=5

resultado: 5
```

### Encadenando operaciones

```scala
val ventas = List(120.0, 45.0, 200.0, 30.0, 180.0, 60.0)

val totalVentasGrandes = ventas
  .filter(_ >= 100.0)   // filtra ventas >= 100
  .map(_ * 1.21)        // aplica IVA
  .reduce(_ + _)        // suma todo

println(f"Total ventas grandes con IVA: $totalVentasGrandes%.2f €")
// Total ventas grandes con IVA: 605.00 €
```

> 💡 Este patrón `filter → map → reduce` es exactamente la misma filosofía que usan los DataFrames de Apache Spark. Lo que aprendes aquí sobre colecciones de Scala se aplica directamente en el Módulo 3.
> 

---

## 💻 Práctica

---

### 🔹 Ejercicio 1 — Funciones puras para cálculos matemáticos

**Celda  — Code (definición de funciones):**

```scala
def areaCirculo(radio: Double): Double =
 scala.math.Pi * radio * radio

def areaRectangulo(base: Double, altura: Double): Double =
  base * altura

def areaTrapecio(baseMayor: Double, baseMenor: Double, altura: Double): Double =
  ((baseMayor + baseMenor) / 2) * altura

def celsiusAFahrenheit(celsius: Double): Double =
  (celsius * 9.0 / 5.0) + 32

def esDivisible(numero: Int, divisor: Int): Boolean =
  numero % divisor == 0
```

Salida:

```
defined function areaCirculo
defined function areaRectangulo
defined function areaTrapecio
defined function celsiusAFahrenheit
defined function esDivisible
```

> 💡 Las definiciones de función en una celda quedan disponibles para todas las celdas siguientes del notebook durante la sesión. Ejecuta siempre esta celda antes de las siguientes.
> 

**Celda 3 — Code:**

```scala
println("=== Áreas ===")
println(f"Círculo r=5:          ${areaCirculo(5)}%.2f")
println(f"Rectángulo 8×3:       ${areaRectangulo(8, 3)}%.2f")
println(f"Trapecio (10,6) h=4:  ${areaTrapecio(10, 6, 4)}%.2f")
```

**Salida esperada:**

```
=== Áreas ===
Círculo r=5:          78.54
Rectángulo 8×3:       24.00
Trapecio (10,6) h=4:  32.00
```

**Celda 4 — Code:**

```scala
println("=== Conversión de temperatura ===")
for (temp <- List(0.0, 20.0, 37.0, 100.0)) {
  println(f"  $temp%.1f°C → ${celsiusAFahrenheit(temp)}%.1f°F")
}
```

**Salida esperada:**

```
=== Conversión de temperatura ===
  0.0°C → 32.0°F
  20.0°C → 68.0°F
  37.0°C → 98.6°F
  100.0°C → 212.0°F
```

**Celda 5 — Code:**

```scala
val numero = 42
println(s"=== Divisibilidad de $numero ===")
for (d <- List(2, 3, 5, 7)) {
  val resultado = if (esDivisible(numero, d)) "sí" else "no"
  println(s"  ¿$numero es divisible entre $d? $resultado")
}
```

**Salida esperada:**

```
=== Divisibilidad de 42 ===
  ¿42 es divisible entre 2? sí
  ¿42 es divisible entre 3? sí
  ¿42 es divisible entre 5? no
  ¿42 es divisible entre 7? sí
```

---

### 🔹 Ejercicio 2 — `map`, `filter` y `reduce` sobre listas

**Celda 2 — Code:**

```scala
val alumnos = List(
  ("Ana",    8.5),
  ("Luis",   4.2),
  ("Marta",  9.1),
  ("Carlos", 5.8),
  ("Elena",  3.9),
  ("Pedro",  7.4)
)

// Extraer solo los nombres
val nombres = alumnos.map(_._1)
println(s"Alumnos: ${nombres.mkString(", ")}")

// Extraer solo las notas (la última expresión la muestra Almond automáticamente)
val notas = alumnos.map(_._2)
notas
```

**Salida esperada:**

```
Alumnos: Ana, Luis, Marta, Carlos, Elena, Pedro
res0: List[Double] = List(8.5, 4.2, 9.1, 5.8, 3.9, 7.4)
```

<aside>
💡

Este ejercicio trabaja con una lista de **tuplas** — cada elemento tiene dos valores juntos: nombre y nota.

**La lista:**

```scala
val alumnos = List(
  ("Ana",    8.5),
  ("Luis",   4.2),
  ("Marta",  9.1),
  ("Carlos", 5.8),
  ("Elena",  3.9),
  ("Pedro",  7.4)
)
```

Cada elemento es una tupla `(String, Double)` — posición `_1` es el nombre, posición `_2` es la nota.

Extraer solo los nombres — `_1`:

```scala
val nombres = alumnos.map(_._1)
// List("Ana", "Luis", "Marta", "Carlos", "Elena", "Pedro")
```

`mkString(", ")`:

```scala
println(s"Alumnos: ${nombres.mkString(", ")}")
// Alumnos: Ana, Luis, Marta, Carlos, Elena, Pedro
```

- `mkString` convierte la lista en un String poniendo `", "` entre cada elemento.
- Sin `mkString` imprimiría `List(Ana, Luis, ...)`.

Extraer solo las notas — `_2`:

```scala
val notas = alumnos.map(_._2)
// List(8.5, 4.2, 9.1, 5.8, 3.9, 7.4)
```

- `_._2` significa: de cada tupla `_`, coge el segundo elemento `_2`.

Resumen visual:

```
("Ana", 8.5)
   ↑      ↑
  _1     _2
```

</aside>

**Celda 3 — Code:**

<aside>
💡

Explica linea por linea el código

</aside>

```scala
val aprobados = alumnos.filter(_._2 >= 5.0)
println(s"Aprobados (${aprobados.length}):")
aprobados.foreach { case (nombre, nota) =>
  println(f"  $nombre%-8s → $nota%.1f")
}

val suspensos = alumnos.filter(_._2 < 5.0)
println(s"\nSuspensos (${suspensos.length}):")
suspensos.foreach { case (nombre, nota) =>
  println(f"  $nombre%-8s → $nota%.1f")
}
```

**Salida esperada:**

```
Aprobados (4):
  Ana      → 8.5
  Marta    → 9.1
  Carlos   → 5.8
  Pedro    → 7.4

Suspensos (2):
  Luis     → 4.2
  Elena    → 3.9
```

**Celda 4 — Code:**

<aside>
💡

Explica línea por línea el código

</aside>

```scala
val notaMaxima = notas.reduce((a, b) => if (a > b) a else b)
val notaMinima = notas.reduce((a, b) => if (a < b) a else b)
val notaMedia  = notas.reduce(_ + _) / notas.length

println("=== Estadísticas ===")
println(f"  Nota máxima: $notaMaxima%.1f")
println(f"  Nota mínima: $notaMinima%.1f")
println(f"  Nota media:  $notaMedia%.2f")
```

**Salida esperada:**

```
=== Estadísticas ===
  Nota máxima: 9.1
  Nota mínima: 3.9
  Nota media:  6.48
```

---

### 🔹 Ejercicio  — Transformador de texto con funciones

<aside>
💡

Explica de que trata el código

</aside>

**Celda 2 — Code (definición de funciones):**

```scala
def limpiar(texto: String): String =
  texto.trim

def capitalizar(texto: String): String =
  texto.split(" ").map(_.capitalize).mkString(" ")

def contarPalabras(texto: String): Int =
  texto.trim.split("\\s+").length
```

**Celda 3 — Code:**

```scala
val frases = List(
  "  scala es un lenguaje poderoso  ",
  "big data transforma las empresas",
  "apache spark usa scala",
  "  la programación funcional es elegante  ",
  "java y scala comparten la jvm"
)

println("=== Frases originales ===")
frases.foreach(f => println(s"  '$f'"))
```

**Salida esperada:**

```
=== Frases originales ===
  '  scala es un lenguaje poderoso  '
  'big data transforma las empresas'
  'apache spark usa scala'
  '  la programación funcional es elegante  '
  'java y scala comparten la jvm'
```

**Celda 4 — Code:**

```scala
val frasesProcesadas = frases
  .map(limpiar)
  .map(capitalizar)

println("=== Frases procesadas ===")
frasesProcesadas.foreach(f => println(s"  $f"))
```

**Salida esperada:**

```
=== Frases procesadas ===
  Scala Es Un Lenguaje Poderoso
  Big Data Transforma Las Empresas
  Apache Spark Usa Scala
  La Programación Funcional Es Elegante
  Java Y Scala Comparten La Jvm
```

**Celda 5 — Code:**

```scala
val frasesLargas = frasesProcesadas.filter(f => contarPalabras(f) > 4)

println("=== Frases con más de 4 palabras ===")
frasesLargas.foreach(f => println(s"  $f"))
```

**Salida esperada:**

```
=== Frases con más de 4 palabras ===
  Scala Es Un Lenguaje Poderoso
  Big Data Transforma Las Empresas
  La Programación Funcional Es Elegante
  Java Y Scala Comparten La Jvm
```

**Celda 6 — Code:**

```scala
val totalPalabras = frasesProcesadas.map(contarPalabras).reduce(_ + _)

println("=== Resumen ===")
println(s"  Frases totales:           ${frases.length}")
println(s"  Frases largas (>4 pal.):  ${frasesLargas.length}")
println(s"  Total de palabras:        $totalPalabras")
println(f"  Media palabras/frase:     ${totalPalabras.toDouble / frases.length}%.1f")
```

**Salida esperada:**

```
=== Resumen ===
  Frases totales:           5
  Frases largas (>4 pal.):  4
  Total de palabras:        22
  Media palabras/frase:     4.4
```

---

# Ejercicios propuestos

## 🔹 Ejercicio P1

Define una función `calcularPerimetro` que reciba la base y la altura de un rectángulo y devuelva su perímetro.

Pruébala con los siguientes valores:

```
base=6, altura=4   → 20
base=10, altura=3  → 26
```

---

## 🔹 Ejercicio P2

Define una función `esMayorDeEdad` que reciba una edad (`Int`) y devuelva un `Boolean` indicando si la persona es mayor de edad (18 o más).

Pruébala con:

```
esMayorDeEdad(17)  → false
esMayorDeEdad(18)  → true
esMayorDeEdad(25)  → true
```

---

## 🔹 Ejercicio P3

Define una función `euroADolar` que convierta una cantidad en euros a dólares usando el tipo de cambio `1.08`.

Pruébala con:

```
euroADolar(100.0)  → 108.0
euroADolar(250.0)  → 270.0
```

---

## 🔹 Ejercicio P4

Define una función `saludo` con dos parámetros:

- `nombre: String` — obligatorio
- `idioma: String` — con valor por defecto `"es"`

Si `idioma` es `"es"` devuelve `"Hola, <nombre>!"`, si es `"en"` devuelve `"Hello, <nombre>!"`.

Pruébala con:

```
saludo("Ana")           → "Hola, Ana!"
saludo("Ana", "en")     → "Hello, Ana!"
```

---

## 🔹 Ejercicio P5

Dada la siguiente lista:

```scala
val temperaturas = List(18.0, 22.5, 30.1, 15.3, 27.8)
```

Usa `map` para convertir todas las temperaturas de Celsius a Fahrenheit con la fórmula `(c * 9.0 / 5.0) + 32`.

Salida esperada:

```
List(64.4, 72.5, 86.18, 59.54, 82.04)
```

---

## 🔹 Ejercicio P6

Dada la siguiente lista:

```scala
val palabras = List("scala", "spark", "hadoop", "kafka", "flink")
```

Usa `filter` para quedarte solo con las palabras que tengan más de 5 letras.

Salida esperada:

```
List(hadoop, kafka)
```

---

## 🔹 Ejercicio P7

Dada la siguiente lista:

```scala
val numeros = List(3, 7, 2, 9, 4, 6)
```

Usa `reduce` para encontrar el número **mínimo** de la lista.

Salida esperada:

```
2
```

---

## 🔹 Ejercicio P8

Implementa la misma función de tres formas distintas.

La función recibe un número `Int` y devuelve ese número multiplicado por sí mismo (al cuadrado).

```scala
// 1. Con def
???

// 2. Anónima pura — se ejecuta directamente con el valor 7
???

// 3. Guardada en val
???
```

Salida esperada en los tres casos para el valor `7`:

```
49
```

---

## 🔹 Ejercicio P9

Implementa la misma función de tres formas distintas.

La función recibe dos `String` y los concatena separados por un espacio.

```scala
// 1. Con def
???

// 2. Anónima pura — pruébala con ("Big", "Data")
???

// 3. Guardada en val
???
```

Salida esperada para `("Big", "Data")`:

```
"Big Data"
```

---

## 🔹 Ejercicio P10

Dada la siguiente lista de tuplas `(producto, precio)`:

```scala
val productos = List(
  ("Teclado",  45.0),
  ("Ratón",    20.0),
  ("Monitor", 180.0),
  ("Webcam",   35.0),
  ("Auriculares", 60.0)
)
```

Extrae solo los nombres de los productos con un precio superior a `40.0`.

Salida esperada:

```
List(Teclado, Monitor, Auriculares)
```

---

## 🔹 Ejercicio P11

Usando la misma lista de `productos` del ejercicio anterior:

1. Filtra los productos con precio mayor de `30.0`
2. Aplica un descuento del 10% a cada precio (`precio * 0.90`)
3. Suma todos los precios resultantes con `reduce`

Salida esperada:

```
270.0
```

---

## 🔹 Ejercicio P12

Define una función `contarVocales` que reciba un `String` y devuelva cuántas vocales contiene (considera solo minúsculas: `a, e, i, o, u`).

Pista: puedes usar `.filter` sobre los caracteres del String con `.toList`.

Pruébala con:

```
contarVocales("scala")     → 2
contarVocales("big data")  → 3
```

---

## 🔹 Ejercicio P13

Dada la siguiente lista de nombres:

```scala
val nombres = List("ana", "LUIS", "Marta", "cARLOS", "elena")
```

Encadena operaciones para:

1. Convertir todos los nombres a minúsculas con `.map(_.toLowerCase)`
2. Capitalizar la primera letra de cada uno con `.map(_.capitalize)`
3. Ordenarlos alfabéticamente con `.sorted`

Salida esperada:

```
List(Ana, Carlos, Elena, Luis, Marta)
```

---

## 🔹 Ejercicio P14

Define una función de orden superior `aplicarDosVeces` que reciba:

- Una función `f: Int => Int`
- Un número `n: Int`

Y aplique `f` dos veces sobre `n`.

Pruébala con:

```scala
aplicarDosVeces(x => x * 3, 2)   // → 18  (2*3=6, 6*3=18)
aplicarDosVeces(x => x + 10, 5)  // → 25  (5+10=15, 15+10=25)
```

---

## 🔹 Ejercicio P15

Dada la siguiente lista de frases:

```scala
val frases = List(
  "Scala es funcional",
  "Big Data con Spark",
  "Programación en la JVM",
  "Datos y más datos",
  "Apache Kafka y Scala"
)
```

1. Filtra las frases que contengan la palabra `"Scala"`
2. Convierte cada frase filtrada a mayúsculas con `.toUpperCase`
3. Cuenta cuántas palabras hay en total en las frases resultantes usando `map(_.split(" ").length)` y `reduce(_ + _)`

Salida esperada:

```
Frases con Scala: List(SCALA ES FUNCIONAL, APACHE KAFKA Y SCALA)
Total de palabras: 7
```

# Sesión 2. Colecciones en Scala I: `List`, `Array` y `Vector`

---

## 1. ¿Qué es una colección?

Una **colección** es una estructura que agrupa varios valores bajo un mismo nombre. En lugar de declarar `val nota1 = 8.5`, `val nota2 = 6.0`, `val nota3 = 9.1`… puedes agruparlos en una sola colección y operarlos todos a la vez.

Scala tiene un sistema de colecciones muy rico. En esta sesión trabajamos con las tres más fundamentales:

| Colección | Mutable | Ordenada | Acceso por índice | Uso típico |
| --- | --- | --- | --- | --- |
| `List[T]` | ❌ No | ✅ Sí | Lento (O(n)) | Procesamiento funcional, Spark |
| `Array[T]` | ✅ Sí | ✅ Sí | Rápido (O(1)) | Datos fijos en memoria, interop Java |
| `Vector[T]` | ❌ No | ✅ Sí | Rápido (~O(1)) | Colección inmutable con acceso eficiente |

> 💡 **Regla general:** en Scala funcional (y en Spark) preferimos **colecciones inmutables**. `List` y `Vector` son inmutables por defecto. `Array` es mutable y se usa cuando necesitas acceso por índice o interoperabilidad con código Java.
> 

---

## 2. Colecciones mutables vs. inmutables

```scala
// INMUTABLE: no puedes cambiar su contenido una vez creada
val lista = List(1, 2, 3)
// lista(0) = 99  // ← ERROR de compilación

// MUTABLE: puedes modificar elementos en su posición
val array = Array(1, 2, 3)
array(0) = 99    // ← correcto: array ahora es Array(99, 2, 3)
```

> La **inmutabilidad** es una de las propiedades más importantes de la programación funcional. Cuando los datos no pueden cambiar de forma inesperada, el código es más predecible, más fácil de depurar y más seguro en entornos distribuidos como Spark.
> 

---

## 3. `List[T]` — La colección estrella de Scala

> `List` es la colección funcional por excelencia. Internamente es una **lista enlazada** (linked list):
> 

> ❓ Una **linked list** (lista enlazada) es una estructura de datos donde cada elemento (nodo) apunta al siguiente.
> 
> 
> ![image.png](image.png)
> 
> A diferencia de un array, **los elementos no están en posiciones contiguas en memoria**, sino conectados mediante referencias.
> 

<aside>
💡

Una linked list es eficiente para insertar/eliminar, pero lenta para acceso directo.

</aside>

```scala
List(10, 20, 30)  →  10 :: 20 :: 30 :: Nil
```

- `Nil` es la lista vacía (el final de la cadena). Aquí `Nil` indica: *“ya no hay más elementos”.*
- `::` (pronunciado "cons") añade un elemento al frente. es un operador para añadir un elemento al inicio de una lista.

`::` SOLO añade por delante:

```scala
val lista = List(2, 3)
val nueva = 1 :: lista  // OK → List(1,2,3)
```

### 3.1 Crear una lista

```scala
// Forma habitual
val notas: List[Double] = List(8.5, 6.0, 9.1, 4.8)

// Lista de Strings
val paises: List[String] = List("España", "Francia", "Italia")

// Lista vacía (tipada)
val vacia: List[Int] = List.empty[Int]
// o equivalente:
val vacia2: List[Int] = Nil
```

### 3.2 Operaciones básicas

```scala
val nums = List(10, 20, 30, 40, 50)

// Acceso a elementos
println(nums.head)        // 10  (primer elemento)
println(nums.tail)        // List(20, 30, 40, 50)  (todo menos el primero)
println(nums(2))          // 30  (índice 0-based, pero lento en List)
println(nums.last)        // 50  (último elemento)

// Información sobre la lista
println(nums.length)      // 5
println(nums.isEmpty)     // false
println(Nil.isEmpty)      // true
println(nums.contains(30))// true
println(nums.indexOf(40)) // 3
```

### 3.3 Añadir elementos

En Scala, "añadir" a una lista inmutable **crea una nueva lista**; la original no cambia:

```scala
val original = List(2, 3, 4)

// :: añade al FRENTE (eficiente — O(1))
val conPrimero = 1 :: original       // List(1, 2, 3, 4)

// :+ añade al FINAL (menos eficiente — O(n))
val conUltimo = original :+ 5        // List(2, 3, 4, 5)

// ::: concatena dos listas
val listA = List(1, 2)
val listB = List(3, 4, 5)
val concatenada = listA ::: listB    // List(1, 2, 3, 4, 5)

println(original)     // List(2, 3, 4)  ← no ha cambiado
println(conPrimero)   // List(1, 2, 3, 4)
println(concatenada)  // List(1, 2, 3, 4, 5)
```

> 💡 **Regla de rendimiento:** en `List`, añadir al frente con `::` es la operación más barata. Añadir al final con `:+` recorre toda la lista. Si necesitas añadir mucho al final, considera usar `Vector` en su lugar.
> 

### 3.4 Métodos funcionales esenciales

```scala
val nums = List(1, 2, 3, 4, 5, 6)

// map: transforma cada elemento → nueva lista del mismo tamaño
val dobles = nums.map(n => n * 2)          // List(2, 4, 6, 8, 10, 12)

// filter: conserva solo los elementos que cumplen la condición
val pares = nums.filter(n => n % 2 == 0)   // List(2, 4, 6)

// foreach: ejecuta una acción por elemento (no devuelve lista)
nums.foreach(n => println(n))

// mkString: une la lista en un String
println(nums.mkString(", "))               // "1, 2, 3, 4, 5, 6"
println(nums.mkString("[", ", ", "]"))     // "[1, 2, 3, 4, 5, 6]"

// sum, min, max (para listas numéricas)
println(nums.sum)   // 21
println(nums.min)   // 1
println(nums.max)   // 6
```

---

## 4. `Array[T]` — Mutable, con acceso por índice

> `Array` es equivalente al array de Java. Los elementos están en posiciones contiguas de memoria, lo que hace el **acceso por índice muy rápido** (tiempo constante O(1)).
> 

```scala
// Crear un Array
val temperaturas: Array[Double] = Array(18.5, 22.0, 15.3, 25.8, 19.1)

// Acceso por índice (empieza en 0)
println(temperaturas(0))   // 18.5
println(temperaturas(4))   // 19.1

// Modificar un elemento (¡es mutable!)
temperaturas(2) = 16.0
println(temperaturas(2))   // 16.0

// Información
println(temperaturas.length)       // 5
println(temperaturas.contains(22.0)) // true

// Recorrer con for
for (t <- temperaturas) {
  println(f"  Temperatura: $t%.1f°C")
}
```

### Convertir entre Array y List

```scala
val array = Array(1, 2, 3, 4, 5)
val lista  = array.toList    // Array → List (inmutable)

val lista2 = List(10, 20, 30)
val array2 = lista2.toArray  // List → Array (mutable)
```

### ¿Cuándo usar `Array` en lugar de `List`?

| Situación | Recomendado |
| --- | --- |
| Acceso frecuente por índice | `Array` |
| Modificar elementos en su posición | `Array` |
| Interoperabilidad con código Java | `Array` |
| Procesamiento funcional / Spark | `List` o `Vector` |
| Añadir al frente con `::` | `List` |

---

## 5. `Vector[T]` — Lo mejor de ambos mundos

`Vector` es una colección **inmutable** (como `List`) pero con **acceso por índice eficiente** (como `Array`). Internamente usa un árbol de fanout 32, lo que da acceso y actualización en tiempo prácticamente constante.

```scala
val ciudades = Vector("Madrid", "Barcelona", "Valencia", "Sevilla")

// Acceso por índice (eficiente)
println(ciudades(0))     // "Madrid"
println(ciudades(3))     // "Sevilla"

// Información
println(ciudades.length) // 4
println(ciudades.head)   // "Madrid"
println(ciudades.last)   // "Sevilla"

// "Añadir" crea un nuevo Vector (inmutable)
val conBilbao = ciudades :+ "Bilbao"
println(ciudades.length)    // 4  ← original sin cambios
println(conBilbao.length)   // 5

// Actualizar un elemento (crea nuevo Vector)
val actualizado = ciudades.updated(1, "Málaga")
println(actualizado)   // Vector(Madrid, Málaga, Valencia, Sevilla)
println(ciudades)      // Vector(Madrid, Barcelona, Valencia, Sevilla)  ← sin cambios
```

### Métodos comunes compartidos por `List`, `Array` y `Vector`

| Método | Descripción | Ejemplo |
| --- | --- | --- |
| `.length` / `.size` | Número de elementos | `lista.length` → `5` |
| `.isEmpty` | ¿Está vacía? | `Nil.isEmpty` → `true` |
| `.contains(x)` | ¿Contiene el elemento? | `lista.contains(3)` → `true` |
| `.indexOf(x)` | Posición del elemento (-1 si no existe) | `lista.indexOf(10)` → `2` |
| `.head` | Primer elemento | `lista.head` → `1` |
| `.last` | Último elemento | `lista.last` → `5` |
| `.tail` | Todos menos el primero | `lista.tail` → `List(2,3,4,5)` |
| `.take(n)` | Primeros n elementos | `lista.take(3)` → `List(1,2,3)` |
| `.drop(n)` | Todos excepto los primeros n | `lista.drop(3)` → `List(4,5)` |
| `.reverse` | Lista invertida | `lista.reverse` → `List(5,4,3,2,1)` |
| `.sorted` | Lista ordenada (requiere tipo ordenable) | `lista.sorted` |
| `.distinct` | Elimina duplicados | `List(1,2,2,3).distinct` → `List(1,2,3)` |
| `.sum` | Suma (numéricas) | `lista.sum` → `15` |
| `.min` / `.max` | Mínimo / máximo | `lista.max` → `5` |
| `.mkString(sep)` | Une en String | `lista.mkString(", ")` → `"1, 2, 3"` |

---

## 6. Conexión con Spark

En Spark usarás colecciones Scala constantemente:

- Para crear **RDDs** de prueba: `sc.parallelize(List(1, 2, 3, 4, 5))`
- Para recoger resultados con `.collect()`, que devuelve un `Array[T]`
- Para filtrar, transformar y reducir datos con `map`, `filter`, `reduce` — exactamente los mismos métodos que acabas de aprender

Todo lo que practicas hoy con `List` lo aplicarás directamente sobre millones de registros en Spark.

---

# 💻 Práctica -

---

## 🔹 Ejercicio 1 — Primeros pasos con `List`

```scala
// Creación de listas
val frutas: List[String] = List("manzana", "naranja", "plátano", "uva", "kiwi")
val numeros: List[Int]   = List(15, 3, 42, 8, 27, 1, 56)
val vacia: List[Double]  = List.empty[Double]

println(s"Frutas:  ${frutas.mkString(", ")}")
println(s"Números: ${numeros.mkString(", ")}")
println(s"Vacía:   ${vacia.isEmpty}")
```

**Salida esperada:**

```
Frutas:  manzana, naranja, plátano, uva, kiwi
Números: 15, 3, 42, 8, 27, 1, 56
Vacía:   true
```

**Celda 3 — Code:**

```scala
// Acceso y metadatos
println(s"Primera fruta:   ${frutas.head}")
println(s"Última fruta:    ${frutas.last}")
println(s"Sin la primera:  ${frutas.tail}")
println(s"Total frutas:    ${frutas.length}")
println(s"¿Tiene 'uva'?    ${frutas.contains("uva")}")
println(s"Posición 'kiwi': ${frutas.indexOf("kiwi")}")
```

**Salida esperada:**

```
Primera fruta:   manzana
Última fruta:    kiwi
Sin la primera:  List(naranja, plátano, uva, kiwi)
Total frutas:    5
¿Tiene 'uva'?    true
Posición 'kiwi': 4
```

**Celda 4 — Code:**

```scala
// Operaciones de construcción (inmutabilidad en acción)
val original = List(2, 3, 4)

val conUno   = 1 :: original          // añadir al frente
val conCinco = original :+ 5          // añadir al final
val doble    = original ::: original  // concatenar consigo misma

println(s"original:  $original")
println(s"conUno:    $conUno")
println(s"conCinco:  $conCinco")
println(s"doble:     $doble")
```

**Salida esperada:**

```
original:  List(2, 3, 4)
conUno:    List(1, 2, 3, 4)
conCinco:  List(2, 3, 4, 5)
doble:     List(2, 3, 4, 2, 3, 4)
```

**Celda 5 — Code:**

```scala
// Operaciones estadísticas y ordenación
val nums = List(15, 3, 42, 8, 27, 1, 56)

println(s"Suma:     ${nums.sum}")
println(s"Mínimo:   ${nums.min}")
println(s"Máximo:   ${nums.max}")
println(s"Ordenada: ${nums.sorted}")
println(s"Inversa:  ${nums.reverse}")
println(s"Primeros 3: ${nums.take(3)}")
println(s"Sin primeros 3: ${nums.drop(3)}")
```

**Salida esperada:**

```
Suma:     152
Mínimo:   1
Máximo:   56
Ordenada: List(1, 3, 8, 15, 27, 42, 56)
Inversa:  List(56, 1, 27, 8, 42, 3, 15)
Primeros 3: List(15, 3, 42)
Sin primeros 3: List(8, 27, 1, 56)
```

---

## 🔹 Ejercicio 2 — Transformar y filtrar listas

**Celda 2 — Code:**

```scala
val temperaturas = List(18.5, 22.0, 15.3, 25.8, 19.1, 30.2, 11.4, 28.7)

// map: convertir de Celsius a Fahrenheit
val fahrenheit = temperaturas.map(c => c * 9.0 / 5.0 + 32.0)

println("Temperaturas en °C y °F:")
temperaturas.zip(fahrenheit).foreach { case (c, f) =>
  println(f"  $c%.1f°C  →  $f%.1f°F")
}
```

**Salida esperada:**

```
Temperaturas en °C y °F:
  18.5°C  →  65.3°F
  22.0°C  →  71.6°F
  15.3°C  →  59.5°F
  25.8°C  →  78.4°F
  19.1°C  →  66.4°F
  30.2°C  →  86.4°F
  11.4°C  →  52.5°F
  28.7°C  →  83.7°F
```

**Celda 3 — Code:**

```scala
// filter: separar días calurosos y frescos
val calurosos = temperaturas.filter(t => t >= 25.0)
val frescos   = temperaturas.filter(t => t < 15.0)
val templados = temperaturas.filter(t => t >= 15.0 && t < 25.0)

println(s"Días calurosos (≥25°C): ${calurosos.mkString(", ")}")
println(s"Días frescos   (<15°C): ${frescos.mkString(", ")}")
println(s"Días templados:         ${templados.mkString(", ")}")
println(f"\nMedia general: ${temperaturas.sum / temperaturas.length}%.2f°C")
```

**Salida esperada:**

```
Días calurosos (≥25°C): 25.8, 30.2, 28.7
Días frescos   (<15°C): 11.4
Días templados:         18.5, 22.0, 15.3, 19.1

Media general: 21.38°C
```

**Celda 4 — Code:**

```scala
// Combinar map + filter + mkString en una pipeline
val palabras = List("scala", "big", "data", "spark", "hadoop", "kafka", "flink")

val resultado = palabras
  .filter(p => p.length > 4)       // solo palabras de más de 4 letras
  .map(p => p.capitalize)          // primera letra en mayúscula
  .sorted                          // orden alfabético
  .mkString(", ")

println(s"Tecnologías (>4 letras, ordenadas): $resultado")
```

**Salida esperada:**

```
Tecnologías (>4 letras, ordenadas): Hadoop, Kafka, Scala, Spark
```

---

## 🔹 Ejercicio 3 — Comparativa `List` vs `Array`

**Celda 2 — Code:**

```scala
// Array: acceso y modificación por índice
val calificaciones: Array[Double] = Array(7.5, 8.0, 6.5, 9.2, 5.8)

println("Array original:")
calificaciones.zipWithIndex.foreach { case (nota, i) =>
  println(f"  [$i] $nota%.1f")
}

// Modificar la nota en la posición 2 (corrección de examen)
calificaciones(2) = 7.0
println(s"\nTras corrección en posición 2:")
println(calificaciones.mkString(", "))
```

**Salida esperada:**

```
Array original:
  [0] 7.5
  [1] 8.0
  [2] 6.5
  [3] 9.2
  [4] 5.8

Tras corrección en posición 2:
7.5, 8.0, 7.0, 9.2, 5.8
```

**Celda 3 — Code:**

```scala
// List: intentar modificar un elemento → no es posible
val listaNotas: List[Double] = List(7.5, 8.0, 6.5, 9.2, 5.8)

// Para "actualizar" una List hay que crear una nueva:
val listaNueva = listaNotas.zipWithIndex.map {
  case (nota, 2) => 7.0   // sustituir el elemento en posición 2
  case (nota, _) => nota  // mantener el resto
}

println(s"Lista original: $listaNotas")
println(s"Lista nueva:    $listaNueva")
```

**Salida esperada:**

```
Lista original: List(7.5, 8.0, 6.5, 9.2, 5.8)
Lista nueva:    List(7.5, 8.0, 7.0, 9.2, 5.8)
```

**Celda 4 — Code:**

```scala
// Conversión entre tipos
val arrayBase  = Array(10, 20, 30, 40, 50)
val listaDesde = arrayBase.toList
val vectorDesde = arrayBase.toVector

println(s"Array:  ${arrayBase.mkString(", ")}")
println(s"List:   $listaDesde")
println(s"Vector: $vectorDesde")
println(s"\nTipos:")
println(s"  ${arrayBase.getClass.getSimpleName}")
println(s"  ${listaDesde.getClass.getSimpleName}")
println(s"  ${vectorDesde.getClass.getSimpleName}")
```

**Salida esperada:**

```
Array:  10, 20, 30, 40, 50
List:   List(10, 20, 30, 40, 50)
Vector: Vector(10, 20, 30, 40, 50)

Tipos:
  int[]
  $colon$colon
  VectorImpl
```

---

## 🔹 Ejercicio 4 — Programa: gestión de una lista de estudiantes

> Construiremos un pequeño sistema de gestión usando `List` de tuplas `(nombre, nota)`, aplicando todo lo aprendido.
> 

**Celda 2 — Code:**

```scala
// Definición de datos
val estudiantes: List[(String, Double)] = List(
  ("Ana García",    8.5),
  ("Luis Martín",   4.2),
  ("Marta López",   9.1),
  ("Carlos Ruiz",   5.8),
  ("Elena Sanz",    3.9),
  ("Pedro Jiménez", 7.4),
  ("Laura Torres",  6.3),
  ("Diego Navarro", 8.8)
)

println(s"Total de estudiantes: ${estudiantes.length}")
println("\nLista completa:")
estudiantes.foreach { case (nombre, nota) =>
  println(f"  $nombre%-20s $nota%.1f")
}
```

**Salida esperada:**

```
Total de estudiantes: 8

Lista completa:
  Ana García           8.5
  Luis Martín          4.2
  Marta López          9.1
  Carlos Ruiz          5.8
  Elena Sanz           3.9
  Pedro Jiménez        7.4
  Laura Torres         6.3
  Diego Navarro        8.8
```

**Celda 3 — Code:**

```scala
// Separar aprobados y suspensos
val aprobados  = estudiantes.filter { case (_, nota) => nota >= 5.0 }
val suspensos  = estudiantes.filter { case (_, nota) => nota < 5.0 }

println(s"✅ Aprobados (${aprobados.length}):")
aprobados
  .sortBy { case (_, nota) => -nota }  // orden descendente por nota
  .foreach { case (nombre, nota) => println(f"   $nombre%-20s $nota%.1f") }

println(s"\n❌ Suspensos (${suspensos.length}):")
suspensos.foreach { case (nombre, nota) =>
  println(f"   $nombre%-20s $nota%.1f")
}
```

**Salida esperada:**

```
✅ Aprobados (6):
   Marta López          9.1
   Diego Navarro        8.8
   Ana García           8.5
   Pedro Jiménez        7.4
   Laura Torres         6.3
   Carlos Ruiz          5.8

❌ Suspensos (2):
   Luis Martín          4.2
   Elena Sanz           3.9
```

**Celda 4 — Code:**

```scala
// Estadísticas del grupo
val todasLasNotas = estudiantes.map { case (_, nota) => nota }
val media         = todasLasNotas.sum / todasLasNotas.length
val notaMax       = todasLasNotas.max
val notaMin       = todasLasNotas.min

// Mejor y peor estudiante
val mejorEstudiante  = estudiantes.maxBy { case (_, nota) => nota }
val peorEstudiante   = estudiantes.minBy { case (_, nota) => nota }

println("=== Estadísticas del Grupo ===")
println(f"  Media:            $media%.2f")
println(f"  Nota más alta:    $notaMax%.1f  (${mejorEstudiante._1})")
println(f"  Nota más baja:    $notaMin%.1f  (${peorEstudiante._1})")
println(f"  Tasa de aprobados: ${aprobados.length * 100.0 / estudiantes.length}%.0f%%")
```

**Salida esperada:**

```
=== Estadísticas del Grupo ===
  Media:            6.75
  Nota más alta:    9.1  (Marta López)
  Nota más baja:    3.9  (Elena Sanz)
  Tasa de aprobados: 75%
```

**Celda 5 — Code:**

```scala
// Clasificar por tramos de nota
def tramo(nota: Double): String = nota match {
  case n if n >= 9.0              => "Sobresaliente"
  case n if n >= 7.0 && n < 9.0  => "Notable"
  case n if n >= 5.0 && n < 7.0  => "Aprobado"
  case _                          => "Suspenso"
}

println("=== Calificaciones por tramo ===")
estudiantes
  .sortBy { case (nombre, _) => nombre }
  .foreach { case (nombre, nota) =>
    println(f"  $nombre%-20s $nota%.1f  → ${tramo(nota)}")
  }
```

**Salida esperada:**

```
=== Calificaciones por tramo ===
  Ana García           8.5  → Notable
  Carlos Ruiz          5.8  → Aprobado
  Diego Navarro        8.8  → Notable
  Elena Sanz           3.9  → Suspenso
  Laura Torres         6.3  → Aprobado
  Luis Martín          4.2  → Suspenso
  Marta López          9.1  → Sobresaliente
  Pedro Jiménez        7.4  → Notable
```

---

# Ejercicios propuestos

## Ejercicio P1 — Mi lista de películas favoritas

Crea una `List[String]` con al menos seis títulos de películas. A continuación:

- Imprime cuántas películas hay en la lista.
- Imprime la primera y la última película.
- Imprime la lista ordenada alfabéticamente.
- Comprueba si una película concreta (tú decides el título) está en la lista.

**Salida de referencia** *(los títulos son un ejemplo)*:

```
Total de películas: 6
Primera: Alien
Última:  Top Gun
Ordenada: List(Alien, Blade Runner, Dune, Interstellar, Matrix, Top Gun)
¿Está 'Dune'? true
```

---

## Ejercicio P2 — Termómetro de invierno

Tienes las temperaturas mínimas registradas durante una semana (en °C):

```
-3.0, 1.5, -0.5, 4.2, -2.1, 0.8, 3.3
```

Con esa `List[Double]`:

- Imprime cuántos días se registraron temperaturas **bajo cero**.
- Imprime la temperatura mínima y la máxima de la semana.
- Imprime la media semanal con dos decimales.

**Salida de referencia:**

```
Días bajo cero: 3
Temperatura mínima: -3.0°C
Temperatura máxima: 4.2°C
Media semanal: 0.46°C
```

---

## Ejercicio P3 — Ampliar y combinar listas

Parte de estas dos listas:

```scala
val grupoPar   = List(2, 4, 6, 8, 10)
val grupoImpar = List(1, 3, 5, 7, 9)
```

- Añade el número `0` al frente de `grupoPar` con `::`.
- Añade el número `11` al final de `grupoImpar` con `:+`.
- Concatena las dos listas resultantes en una sola con `:::`.
- Imprime la lista final ordenada de menor a mayor.

**Salida de referencia:**

```
grupoPar ampliada:   List(0, 2, 4, 6, 8, 10)
grupoImpar ampliada: List(1, 3, 5, 7, 9, 11)
Combinada ordenada:  List(0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11)
```

---

## Ejercicio P4 — Inventario de una tienda

Crea un `Array[String]` con estos cinco productos:

```
"Teclado", "Ratón", "Monitor", "Auriculares", "Webcam"
```

- Imprime el producto en la posición 2 (índice 0-based).
- El producto en la posición 3 ha cambiado de nombre a `"Cascos"`. Modifícalo directamente.
- Imprime el array completo tras la modificación.
- Convierte el array a `List` e imprime el tipo de la colección resultante con `.getClass.getSimpleName`.

**Salida de referencia:**

```
Producto en posición 2: Monitor
Tras actualización: Array(Teclado, Ratón, Monitor, Cascos, Webcam)
Tipo tras conversión: $colon$colon
```

---

## Ejercicio P5 — Vector de capitales

Crea un `Vector[String]` con las capitales de cinco países europeos a tu elección.

- Imprime la capital en la posición 1.
- "Actualiza" la capital en la posición 0 con otro nombre usando `.updated(0, "NuevoNombre")` y guarda el resultado en un nuevo `val`.
- Demuestra que el `Vector` original **no ha cambiado** imprimiendo ambos.
- Imprime la longitud del vector actualizado.

**Salida de referencia** *(capitales de ejemplo)*:

```
Capital en posición 1: París
Vector original:    Vector(Madrid, París, Roma, Berlín, Lisboa)
Vector actualizado: Vector(Atenas, París, Roma, Berlín, Lisboa)
Longitud: 5
```

---

## Ejercicio P6 — Filtro de edades

Tienes una lista de edades de participantes en un concurso:

```scala
val edades = List(17, 34, 22, 15, 45, 19, 28, 16, 31, 42)
```

- Filtra los participantes **mayores de edad** (≥ 18 años) y guárdalos en `mayores`.
- Filtra los **menores de edad** (< 18 años) y guárdalos en `menores`.
- Calcula el porcentaje de mayores de edad sobre el total con dos decimales.
- Imprime la edad más alta entre los menores y la edad más baja entre los mayores.

**Salida de referencia:**

```
Mayores de edad (7): List(34, 22, 45, 19, 28, 31, 42)
Menores de edad (3): List(17, 15, 16)
Porcentaje mayores: 70,00%
Edad más alta entre menores: 17
Edad más baja entre mayores: 19
```

---

## Ejercicio P7 — Transformación de precios

Una tienda online tiene esta lista de precios en euros:

```scala
val precios = List(12.99, 45.00, 8.50, 120.00, 33.75, 5.20, 89.99)
```

- Aplica un **descuento del 15%** a todos los precios con `map` y guarda el resultado en `conDescuento`.
- Filtra los precios originales que sean **superiores a 30 €**.
- Calcula el **ahorro total** restando la suma de `conDescuento` a la suma de `precios`.
- Imprime los tres resultados con dos decimales.

**Salida de referencia:**

```
Precios con 15% dto: List(11.04, 38.25, 7.23, 102.00, 28.69, 4.42, 76.49)
Precios > 30€:       List(45.00, 120.00, 33.75, 89.99)
Ahorro total:        47.19€
```

---

## Ejercicio P8 — Longitud de palabras

Parte de esta lista de palabras:

```scala
val palabras = List("procesamiento", "datos", "scala", "distribuido",
                    "nodo", "clúster", "pipeline", "streaming")
```

- Usa `map` para crear una nueva lista de tuplas `(palabra, longitud)`.
- Filtra las palabras con **más de 7 letras**.
- Ordena ese subconjunto de mayor a menor longitud usando `.sortBy(- _._2)`.
- Imprime cada par `(palabra → n letras)` en una línea separada.

**Salida de referencia:**

```
Palabras con más de 7 letras (ordenadas por longitud desc):
  procesamiento → 13 letras
  distribuido   → 11 letras
  streaming     → 9 letras
  pipeline      → 8 letras
  clúster       → 7 letras  ← no aparece (≤7)
```

> ⚠️ Nota: "clúster" tiene 7 letras, así que **no** pasa el filtro `> 7`.
> 

---

## Ejercicio P9 — Marcador de un partido

Tienes el registro de goles de un partido de baloncesto como `Array[Int]`, donde cada elemento es la puntuación anotada en cada cuarto:

```scala
val localArray    = Array(21, 18, 25, 19)
val visitanteArray = Array(17, 22, 20, 24)
```

- Calcula la puntuación total de cada equipo con `.sum`.
- Determina el ganador comparando los totales e imprímelo.
- Convierte ambos arrays a `List` y usa `zip` para crear una lista de tuplas `(puntosLocal, puntosVisitante)` cuarto a cuarto.
- Imprime el marcador parcial de cada cuarto.

**Salida de referencia:**

```
Local:     83 puntos
Visitante: 83 puntos
Resultado: Empate

Marcador por cuartos:
  Q1: 21 - 17
  Q2: 18 - 22
  Q3: 25 - 20
  Q4: 19 - 24
```

---

## Ejercicio P10 — Catálogo de libros con filtros encadenados

Define esta lista de tuplas `(título, autor, año)`:

```scala
val libros: List[(String, String, Int)] = List(
  ("Clean Code",            "Robert C. Martin", 2008),
  ("The Pragmatic Programmer", "Dave Thomas",   1999),
  ("Scala for the Impatient",  "Cay Horstmann", 2012),
  ("Programming in Scala",     "Odersky",       2016),
  ("Designing Data-Intensive", "Martin Kleppmann", 2017),
  ("Structure and Interpretation", "Abelson",   1996),
  ("Domain Driven Design",     "Eric Evans",    2003)
)
```

- Filtra los libros publicados **a partir del año 2000**.
- De ese subconjunto, extrae solo los **títulos** con `map`.
- Ordénalos alfabéticamente e imprímelos numerados (1., 2., 3.…).
- Imprime además cuántos libros del catálogo completo son **anteriores al año 2000**.

**Salida de referencia:**

```
Libros desde el año 2000 (ordenados):
  1. Clean Code
  2. Designing Data-Intensive
  3. Domain Driven Design
  4. Programming in Scala
  5. Scala for the Impatient

Libros anteriores a 2000: 2
```

---

## Ejercicio P11 — Análisis de ventas mensuales

Una empresa registra sus ventas mensuales (en miles de €) durante un año:

```scala
val ventas = List(45.2, 38.7, 52.1, 61.0, 58.4, 70.3,
                  66.9, 72.5, 55.8, 49.3, 41.6, 80.2)
val meses  = List("Ene","Feb","Mar","Abr","May","Jun",
                  "Jul","Ago","Sep","Oct","Nov","Dic")
```

- Usa `zip` para emparejar cada mes con su venta.
- Imprime los **tres meses con mayor venta** (usa `.sortBy` y `.take`).
- Calcula e imprime la venta **total anual** y la **media mensual**.
- Imprime cuántos meses superaron la media.

**Salida de referencia:**

```
Top 3 meses:
  Dic → 80.2k€
  Ago → 72.5k€
  Jun → 70.3k€

Total anual: 691.00k€
Media mensual: 57.58k€
Meses por encima de la media: 6
```

---

## Ejercicio P12 — Deduplicación y comparación de conjuntos

Tienes dos listas de IDs de usuarios que visitaron una web en dos días distintos:

```scala
val diaUno = List(101, 205, 307, 101, 412, 205, 519, 307, 624)
val diaDos = List(205, 412, 731, 101, 519, 888, 412, 731)
```

- Elimina los duplicados de cada lista con `.distinct`.
- Calcula cuántos **usuarios únicos** visitaron cada día.
- Encuentra los usuarios que visitaron **los dos días** usando `.filter` y `.contains`.
- Encuentra los usuarios que visitaron **solo el día uno** (estaban en `diaUno` pero no en `diaDos`).

**Salida de referencia:**

```
Día 1 — usuarios únicos: 6  → List(101, 205, 307, 412, 519, 624)
Día 2 — usuarios únicos: 5  → List(205, 412, 731, 101, 519, 888)

Visitaron ambos días:    List(101, 205, 412, 519)
Solo visitaron el día 1: List(307, 624)
```

---

## Ejercicio P13 — Clasificador de números con `match`

Define una función `clasificar(n: Int): String` que use `match` para devolver:

- `"negativo"` si `n < 0`
- `"cero"` si `n == 0`
- `"par positivo"` si `n > 0` y es par
- `"impar positivo"` si `n > 0` y es impar

Luego aplica esa función sobre esta lista con `map`:

```scala
val numeros = List(-7, 0, 4, 15, -2, 8, 33, 0, -1, 100)
```

Imprime cada número junto a su clasificación, alineando las columnas. Al final, imprime cuántos elementos hay de cada categoría.

**Salida de referencia:**

```
  -7  → negativo
   0  → cero
   4  → par positivo
  15  → impar positivo
  -2  → negativo
   8  → par positivo
  33  → impar positivo
   0  → cero
  -1  → negativo
 100  → par positivo

negativos:       3
ceros:           2
pares positivos: 3
impares positivos: 2
```

---

## Ejercicio P14 — Historial de accesos con `Vector`

Simula un historial de accesos a un sistema usando `Vector[String]`. Parte de un vector vacío y añade entradas una a una con `:+`:

```
"usuario_01 login"
"usuario_02 login"
"usuario_01 logout"
"usuario_03 login"
"usuario_02 logout"
"usuario_03 logout"
```

- Imprime el historial completo con su número de línea (1, 2, 3…).
- Filtra e imprime solo los eventos de tipo `"login"` (usa `.contains("login")`).
- Cuenta cuántos usuarios distintos aparecen en el historial extrayendo el nombre (parte antes del espacio) con `.map(_.split(" ")(0))` y luego `.distinct`.
- Demuestra que el vector original **no ha cambiado** imprimiendo su longitud antes y después de cada operación.

**Salida de referencia:**

```
Historial (6 entradas):
  1. usuario_01 login
  2. usuario_02 login
  3. usuario_01 logout
  4. usuario_03 login
  5. usuario_02 logout
  6. usuario_03 logout

Eventos login (3):
  usuario_01 login
  usuario_02 login
  usuario_03 login

Usuarios únicos: 3 → List(usuario_01, usuario_02, usuario_03)
Longitud del vector original: 6  ← sin cambios
```

---

## Ejercicio P15 — Mini pipeline de procesamiento de texto

Tienes este fragmento de texto ya dividido en palabras:

```scala
val texto = List(
  "scala", "es", "un", "lenguaje", "scala", "funcional",
  "y", "orientado", "a", "objetos", "scala", "es", "potente",
  "para", "big", "data", "y", "spark", "usa", "scala"
)
```

Construye un **pipeline en una sola expresión encadenada** (sin variables intermedias) que:

1. Elimine las palabras con **menos de 3 letras** (`filter`).
2. Elimine los **duplicados** (`distinct`).
3. Ordene el resultado **alfabéticamente** (`sorted`).
4. Convierta cada palabra a **mayúsculas** (`map(_.toUpperCase)`).
5. Una el resultado en un único `String` separado por `" | "` (`mkString`).

Imprime el resultado final. Luego, por separado, imprime cuántas palabras únicas de 3 o más letras hay.

**Salida de referencia:**

```
Pipeline resultado:
BIG | DATA | FUNCIONAL | LENGUAJE | OBJETOS | ORIENTADO | POTENTE | SCALA | SPARK | USA

Palabras únicas (≥3 letras): 10
```

---