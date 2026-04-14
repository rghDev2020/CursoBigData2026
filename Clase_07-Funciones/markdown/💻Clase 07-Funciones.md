# 💻Clase 07 - Funciones

---

# Agenda:

<aside>
💡

#### 9:00 - 9:50    →  **Sesión 1: Funciones en Scala**

#### 9:50 - 11:20   → Practicas

#### 11:40 - 12:40  → Practicas

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
// Con def — tiene nombre
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

**Mas ejemplos:**

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