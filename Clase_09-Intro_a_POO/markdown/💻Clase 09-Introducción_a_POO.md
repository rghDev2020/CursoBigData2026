# 💻Clase 09 - Introducción a POO

---

# Agenda:

<aside>
💡

#### 9:00 - 9:50    → **Prueba Módulo 2**

#### 9:50 - 11:20   → Prueba Módulo 2

#### 11:40 - 12:40  → Programación Orientada a Objetos en Scala I

#### 12:40 - 14:00  → Practicas

</aside>

## Día 6 — Sesión 1 | Programación Orientada a Objetos en Scala I

---

### 1. ¿Qué es la Programación Orientada a Objetos?

La **Programación Orientada a Objetos (POO)** organiza el código en torno a **objetos**: entidades que agrupan datos (atributos) y comportamiento (métodos).

Scala es un lenguaje **híbrido**: combina POO con programación funcional. Esto lo convierte en una herramienta especialmente potente para Big Data, donde ambos paradigmas se usan conjuntamente en Apache Spark.

Los cuatro pilares clásicos de la POO son:

| Pilar | Descripción |
| --- | --- |
| **Encapsulamiento** | Ocultar los detalles internos de un objeto |
| **Herencia** | Una clase puede extender (heredar de) otra |
| **Polimorfismo** | Un mismo método puede comportarse de forma diferente según el tipo |
| **Abstracción** | Modelar conceptos del mundo real como clases |

### 2. Clases en Scala: `class`

Una **clase** es la plantilla o molde para crear objetos. Define qué datos tiene y qué puede hacer.

### Sintaxis básica

```scala
class NombreClase(param1: Tipo1, param2: Tipo2) {
  // atributos y métodos
}
```

### Ejemplo: clase `Persona`

```scala
class Persona(val nombre: String, val edad: Int) {
  def saludar(): String = s"Hola, me llamo $nombre y tengo $edad años."
}
```

Para crear un **objeto** (instancia) de esa clase, usamos `new`:

```scala
val p1 = new Persona("Ana", 30)
println(p1.saludar())
// Hola, me llamo Ana y tengo 30 años.
```

---

### 3. Constructor primario y constructor secundario

### Constructor primario

En Scala, el **constructor primario** está integrado directamente en la firma de la clase. Los parámetros que ponemos en el paréntesis de la clase son a la vez los parámetros del constructor:

```scala
class Producto(val nombre: String, val precio: Double, val stock: Int) {
  def descripcion(): String =
    f"$nombre (${precio}%.2f €) — Stock: $stock ud."
}
```

> 💡 Si el parámetro lleva `val`, se convierte automáticamente en un atributo público de solo lectura. Si lleva `var`, es mutable. Si no lleva ninguno de los dos, es solo un parámetro del constructor, sin acceso desde fuera.
> 

```scala
// Diferencia entre val, var y sin modificador:
class Ejemplo(val lectura: Int, var mutable: Int, sinModif: Int) {
  def mostrar(): Unit = {
    println(lectura)   // accesible
    println(mutable)   // accesible y modificable
    println(sinModif)  // accesible SOLO dentro de la clase
  }
}
```

### Constructor secundario

Un **constructor secundario** se define con `def this(...)` dentro del cuerpo de la clase. Debe llamar obligatoriamente al constructor primario como primera instrucción:

```scala
class Producto(val nombre: String, val precio: Double, val stock: Int) {

  // Constructor secundario: precio por defecto 0.0 y stock 0
  def this(nombre: String) = this(nombre, 0.0, 0)

  def descripcion(): String =
    f"$nombre (${precio}%.2f €) — Stock: $stock ud."
}

val p1 = new Producto("Teclado", 49.99, 10)
val p2 = new Producto("Pendiente de precio")  // usa el constructor secundario

println(p1.descripcion())
// Teclado (49.99 €) — Stock: 10 ud.
println(p2.descripcion())
// Pendiente de precio (0.00 €) — Stock: 0 ud.
```

<aside>

💡 Analogía cuando pides un café en el bar.
El **constructor primario** es el pedido completo, con todos los detalles:

```scala
class Cafe(val tipo: String, val tamano: String, val azucar: Int, val leche: Boolean)
```

Por lo tanto cuando llegas por primera vez a ese bar,  pedirías con todos los detalles como quieres tu café:

```scala
val miCafe = new Cafe("Americano", "grande", 2, false)
// "Un americano grande, con 2 azúcares y sin leche"
// Aqui se está llamando al contructor primario
```

En cambio, cuando ya tienes tiempo visitando el mismo bar dices simplemente **"lo de siempre"**. El barista ya sabe que quieres un cortado mediano con un azúcar. Eso es el **constructor secundario**: una forma abreviada de crear el objeto con valores por defecto.

```scala
class Cafe(val tipo: String, val tamano: String, val azucar: Int, val leche: Boolean) {

  // "Ponme un café negro," — solo dices el tipo, el resto es lo habitual
  def this(tipo: String) = this(tipo, "mediano", 1, true)

  // "Lo de siempre" — ni siquiera especificas el tipo
  def this() = this("cortado", "mediano", 1, true)

  def descripcion(): String = {
    val lecheStr = if (leche) "con leche" else "sin leche"
    s"$tipo $tamano, $azucar azúcar(es), $lecheStr"
  }
}

val pedido1 = new Cafe("Americano", "grande", 2, false)
val pedido2 = new Cafe("Cappuccino")   // tamaño mediano, 1 azúcar, con leche
val pedido3 = new Cafe()               // "Lo de siemper", cortado mediano, 1 azúcar, con leche

println(pedido1.descripcion())  // Americano grande, 2 azúcar(es), sin leche
println(pedido2.descripcion())  // Cappuccino mediano, 1 azúcar(es), con leche
println(pedido3.descripcion())  // cortado mediano, 1 azúcar(es), con leche
```

</aside>

---

### 4. Campos y métodos dentro de una clase

Dentro del cuerpo de la clase podemos definir:

| Elemento | Descripción |
| --- | --- |
| `val` | Atributo inmutable, se calcula una vez |
| `var` | Atributo mutable, puede cambiar |
| `def` | Método (función asociada a la clase) |

```scala
class CuentaBancaria(val titular: String, var saldo: Double) {

  // Atributo derivado (se calcula en el momento de instanciar)
  val banco: String = "BancoScala"

  // Método que modifica el estado
  def ingresar(cantidad: Double): Unit = {
    saldo += cantidad
    println(f"Ingreso de ${cantidad}%.2f €. Nuevo saldo: ${saldo}%.2f €")
  }

  def retirar(cantidad: Double): Unit = {
    if (cantidad <= saldo) {
      saldo -= cantidad
      println(f"Retirada de ${cantidad}%.2f €. Nuevo saldo: ${saldo}%.2f €")
    } else {
      println("Saldo insuficiente.")
    }
  }

  def resumen(): String =
    f"[$banco] Cuenta de $titular — Saldo: ${saldo}%.2f €"
}
```

```scala
val cuenta = new CuentaBancaria("Luis García", 500.0)
cuenta.ingresar(200.0)
cuenta.retirar(100.0)
cuenta.retirar(700.0)
println(cuenta.resumen())
```

**Salida esperada:**

```
Ingreso de 200.00 €. Nuevo saldo: 700.00 €
Retirada de 100.00 €. Nuevo saldo: 600.00 €
Saldo insuficiente.
[BancoScala] Cuenta de Luis García — Saldo: 600.00 €
```

---

### 5. `object`: el singleton de Scala

En Scala, `object` define un **singleton**: una clase de la que solo existe **una única instancia**, creada automáticamente. No se usa `new` para acceder a él.

```scala
object Configuracion {
  val nombreApp: String = "BigDataApp"
  val version: String   = "1.0.0"
  val maxRegistros: Int = 10000

  def info(): String =
    s"$nombreApp v$version — Límite: $maxRegistros registros"
}

println(Configuracion.info())
// BigDataApp v1.0.0 — Límite: 10000 registros
```

<aside>

💡Analogía: La configuración de tu móvil

Tu móvil tiene **una sola configuración**. No creas una nueva configuración cada vez que la abres — siempre accedes a la misma.

```scala
object ConfiguracionMovil {
  var volumen: Int        = 70
  var brillo: Int         = 50
  var modoAvion: Boolean  = false
  val modelo: String      = "ScalaPhone X"

  def activarModoAvion(): Unit = {
    modoAvion = true
    println("✈️  Modo avión activado. WiFi y llamadas desactivados.")
  }

  def subirVolumen(cantidad: Int): Unit = {
    volumen = Math.min(100, volumen + cantidad)
    println(s"🔊 Volumen: $volumen%")
  }

  def estado(): String =
    s"[$modelo] Volumen: $volumen% | Brillo: $brillo% | Avión: $modoAvion"
}

// Desde cualquier parte de la app, siempre es la misma configuración
println(ConfiguracionMovil.estado())

ConfiguracionMovil.subirVolumen(15)
ConfiguracionMovil.activarModoAvion()

println(ConfiguracionMovil.estado())
```

Salida:

```scala
ScalaPhone X] Volumen: 70% | Brillo: 50% | Avión: false
🔊 Volumen: 85%
✈️  Modo avión activado. WiFi y llamadas desactivados.
[ScalaPhone X] Volumen: 85% | Brillo: 50% | Avión: true
```

</aside>

### Diferencia entre `class` y `object`

|  | `class` | `object` |
| --- | --- | --- |
| Instancias | Múltiples (con `new`) | Una sola (sin `new`) |
| Uso típico | Modelar entidades con datos propios | Constantes, utilidades, punto de entrada |
| ¿Necesita `new`? | ✅ Sí | ❌ No |

### `object` como punto de entrada al programa

En Scala 2.13, la forma más habitual de definir el punto de entrada de una aplicación es con `object ... extends App`:

```scala
object MiAplicacion extends App {
  println("¡La aplicación ha arrancado!")
  val p = new Persona("Carlos", 25)
  println(p.saludar())
}
```

> 💡 Con `extends App`, todo el código dentro del cuerpo del `object` se ejecuta automáticamente al lanzar el programa. Es el equivalente al `main` de Java, pero más conciso.
> 

---

### 6. Visibilidad: `private`, `protected` y `public`

Scala controla el acceso a los miembros de una clase mediante modificadores de visibilidad:

| Modificador | Accesible desde |
| --- | --- |
| *(ninguno)* | Público — accesible desde cualquier lugar |
| `private` | Solo desde dentro de la propia clase |
| `protected` | Desde la clase y sus subclases (herencia) |

```scala
class Empleado(val nombre: String, private var salario: Double) {

  // Método público: accesible desde fuera
  def presentarse(): String = s"Soy $nombre"

  // Método público que accede a un campo privado
  def obtenerSalario(): Double = salario

  // Método privado: solo para uso interno
  private def calcularBonus(): Double = salario * 0.10

  def aplicarBonus(): Unit = {
    val bonus = calcularBonus()
    salario += bonus
    println(f"Bonus aplicado: ${bonus}%.2f €. Nuevo salario: ${salario}%.2f €")
  }
}
```

```scala
val emp = new Empleado("Marta", 2000.0)
println(emp.presentarse())
// Soy Marta

println(emp.obtenerSalario())
// 2000.0

emp.aplicarBonus()
// Bonus aplicado: 200.00 €. Nuevo salario: 2200.00 €

```

 La siguiente línea daría error de compilación:

```scala

// println(emp.salario)      // error: salario es private
// emp.calcularBonus()       // error: calcularBonus es private
```

> 💡 **Regla de encapsulamiento:** es buena práctica declarar los datos internos como `private` y exponer solo lo necesario mediante métodos públicos. Esto protege la integridad del objeto.
> 

---

## 💻 Prácticas

---

### 🔹 Ejercicio 1 — Clase `Persona` con atributos y métodos

**Celda 2 — Code (definición de la clase):**

```scala
class Persona(val nombre: String, val apellidos: String, var edad: Int) {

  val nombreCompleto: String = s"$nombre $apellidos"

  def saludar(): String =
    s"Hola, me llamo $nombreCompleto y tengo $edad años."

  def cumplirAnios(): Unit = {
    edad += 1
    println(s"¡Feliz cumpleaños, $nombre! Ahora tienes $edad años.")
  }

  def esMayorDeEdad(): Boolean = edad >= 18

  def descripcion(): String = {
    val estado = if (esMayorDeEdad()) "mayor de edad" else "menor de edad"
    s"$nombreCompleto ($edad años) — $estado"
  }
}
```

**Celda 3 — Code (uso de la clase):**

```scala
val p1 = new Persona("Ana", "García López", 30)
val p2 = new Persona("Carlos", "Martínez Ruiz", 17)

println(p1.saludar())
println(p2.saludar())
```

**Salida esperada:**

```
Hola, me llamo Ana García López y tengo 30 años.
Hola, me llamo Carlos Martínez Ruiz y tengo 17 años.
```

**Celda 4 — Code:**

```scala
println(p1.descripcion())
println(p2.descripcion())
```

**Salida esperada:**

```
Ana García López (30 años) — mayor de edad
Carlos Martínez Ruiz (17 años) — menor de edad
```

**Celda 5 — Code:**

```scala
p2.cumplirAnios()
println(p2.descripcion())
```

**Salida esperada:**

```
¡Feliz cumpleaños, Carlos! Ahora tienes 18 años.
Carlos Martínez Ruiz (18 años) — mayor de edad
```

---

### 🔹 Ejercicio 2 — Clase `Producto` con constructor secundario

**Celda 2 — Code:**

```scala
class Producto(val nombre: String, val precio: Double, var stock: Int) {

  // Constructor secundario: producto sin precio ni stock definidos aún
  def this(nombre: String) = this(nombre, 0.0, 0)

  // Constructor secundario: producto con precio pero sin stock
  def this(nombre: String, precio: Double) = this(nombre, precio, 0)

  def estaDisponible(): Boolean = stock > 0

  def vender(cantidad: Int): Unit = {
    if (cantidad <= stock) {
      stock -= cantidad
      println(f"Vendidas $cantidad ud. de '$nombre'. Stock restante: $stock")
    } else {
      println(s"No hay suficiente stock de '$nombre'. Disponible: $stock ud.")
    }
  }

  def ficha(): String = {
    val disponibilidad = if (estaDisponible()) s"$stock ud." else "Sin stock"
    f"$nombre%-20s | ${precio}%7.2f € | $disponibilidad"
  }
}
```

**Celda 3 — Code:**

```scala
val prod1 = new Producto("Teclado mecánico", 79.99, 15)
val prod2 = new Producto("Ratón inalámbrico", 35.50)   // stock = 0
val prod3 = new Producto("Pendiente de alta")           // precio = 0.0, stock = 0

println("=== Catálogo de productos ===")
println(f"${"Nombre"}%-20s | ${"Precio"}%7s | Stock")
println("-" * 42)
println(prod1.ficha())
println(prod2.ficha())
println(prod3.ficha())
```

**Salida esperada:**

```
=== Catálogo de productos ===
Nombre               |  Precio | Stock
------------------------------------------
Teclado mecánico     |  79.99 € | 15 ud.
Ratón inalámbrico    |  35.50 € | Sin stock
Pendiente de alta    |   0.00 € | Sin stock
```

**Celda 4 — Code:**

```scala
prod1.vender(5)
prod1.vender(12)
```

**Salida esperada:**

```
Vendidas 5 ud. de 'Teclado mecánico'. Stock restante: 10
No hay suficiente stock de 'Teclado mecánico'. Disponible: 10 ud.
```

---

### 🔹 Ejercicio 3 — `object` singleton como módulo de utilidades

**Celda 2 — Code:**

```scala
object ConversorUnidades {

  val VERSION: String = "1.0"

  def kmAMillas(km: Double): Double       = km * 0.621371
  def millasAKm(millas: Double): Double   = millas / 0.621371
  def celsiusAFahrenheit(c: Double): Double = c * 9.0 / 5.0 + 32
  def fahrenheitACelsius(f: Double): Double = (f - 32) * 5.0 / 9.0
  def kgALibras(kg: Double): Double       = kg * 2.20462
  def librasAKg(libras: Double): Double   = libras / 2.20462

  def info(): String = s"ConversorUnidades v$VERSION"
}
```

**Celda 3 — Code:**

```scala
println(ConversorUnidades.info())

val km = 100.0
val millas = ConversorUnidades.kmAMillas(km)
println(f"$km%.1f km = $millas%.2f millas")

val temp = 25.0
val tempF = ConversorUnidades.celsiusAFahrenheit(temp)
println(f"$temp%.1f °C = $tempF%.2f °F")

val peso = 70.0
val libras = ConversorUnidades.kgALibras(peso)
println(f"$peso%.1f kg = $libras%.2f libras")
```

**Salida esperada:**

```
ConversorUnidades v1.0
100.0 km = 62.14 millas
25.0 °C = 77.00 °F
70.0 kg = 154.32 libras
```

---

### 🔹 Ejercicio 4  -  Sistema de empleados

**Celda 2 — Code (clase Empleado):**

```scala
class Empleado(
  val id: Int,
  val nombre: String,
  val departamento: String,
  private var salario: Double
) {

  // Constructor secundario: empleado sin departamento asignado
  def this(id: Int, nombre: String, salario: Double) =
    this(id, nombre, "Sin asignar", salario)

  def obtenerSalario(): Double = salario

  private def calcularBonus(porcentaje: Double): Double =
    salario * (porcentaje / 100.0)

  def aplicarSubida(porcentaje: Double): Unit = {
    val incremento = calcularBonus(porcentaje)
    salario += incremento
    println(f"[$nombre] Subida del ${porcentaje}%.1f%% (+${incremento}%.2f €). Nuevo salario: ${salario}%.2f €")
  }

  def ficha(): String =
    f"[${id}%03d] $nombre%-20s | $departamento%-15s | ${salario}%8.2f €"
}
```

**Celda 3 — Code (creación de empleados):**

```scala
val empleados = List(
  new Empleado(1, "Ana Pérez",    "Tecnología",  2800.0),
  new Empleado(2, "Luis García",  "Ventas",      2200.0),
  new Empleado(3, "María Torres", "Tecnología",  3100.0),
  new Empleado(4, "Pedro Ruiz",   3000.0)          // sin departamento
)

println("=== Plantilla de empleados ===")
println(f"${"ID"}%5s  ${"Nombre"}%-20s  ${"Departamento"}%-15s  ${"Salario"}%10s")
println("-" * 60)
for (e <- empleados) println(e.ficha())
```

**Salida esperada:**

```
=== Plantilla de empleados ===
   ID  Nombre                Departamento     Salario
------------------------------------------------------------
[001] Ana Pérez             | Tecnología      |  2800.00 €
[002] Luis García           | Ventas          |  2200.00 €
[003] María Torres          | Tecnología      |  3100.00 €
[004] Pedro Ruiz            | Sin asignar     |  3000.00 €
```

**Celda 4 — Code (operaciones sobre los empleados):**

```scala
// Subida de sueldo a todos los de Tecnología
println("\n--- Subida del 5% al departamento de Tecnología ---")
for (e <- empleados if e.departamento == "Tecnología") {
  e.aplicarSubida(5.0)
}

// Informe actualizado
println("\n=== Plantilla actualizada ===")
for (e <- empleados) println(e.ficha())

// Calcular salario medio
val salarioTotal = empleados.map(_.obtenerSalario()).sum
val salarioMedio = salarioTotal / empleados.length
println(f"\nSalario medio: ${salarioMedio}%.2f €")
```

**Salida esperada:**

```
--- Subida del 5% al departamento de Tecnología ---
[Ana Pérez] Subida del 5.0% (+140.00 €). Nuevo salario: 2940.00 €
[María Torres] Subida del 5.0% (+155.00 €). Nuevo salario: 3255.00 €

=== Plantilla actualizada ===
[001] Ana Pérez             | Tecnología      |  2940.00 €
[002] Luis García           | Ventas          |  2200.00 €
[003] María Torres          | Tecnología      |  3255.00 €
[004] Pedro Ruiz            | Sin asignar     |  3000.00 €

Salario medio: 2848.75 €
```

---

# Ejercicios Propuestos

---

## Ejercicio P1 — Clase `Libro`

Crea una clase `Libro` con los atributos `titulo` (String), `autor` (String) y `paginas` (Int). Añade un método `descripcion()` que devuelva una cadena con el formato:

```
"El libro 'Don Quijote' de Miguel de Cervantes tiene 1000 páginas."
```

Crea dos instancias distintas e imprime su descripción.

---

## Ejercicio P2 — Clase `Rectangulo`

Crea una clase `Rectangulo` con los atributos `base` (Double) y `altura` (Double). Añade los métodos:

- `area()` → devuelve `base * altura`
- `perimetro()` → devuelve `2 * (base + altura)`
- `esCuadrado()` → devuelve `true` si base y altura son iguales

Crea tres rectángulos distintos y muestra por pantalla sus medidas, área, perímetro y si son cuadrados.

---

## Ejercicio P3 — Clase `Bombilla`

Crea una clase `Bombilla` con un atributo mutable `encendida` (Boolean) que empieza en `false`. Añade los métodos:

- `encender()` → cambia `encendida` a `true` e imprime `"💡 Bombilla encendida."`
- `apagar()` → cambia `encendida` a `false` e imprime `"🌑 Bombilla apagada."`
- `estado()` → devuelve `"encendida"` o `"apagada"` según corresponda

Crea una bombilla, enciéndela, consulta su estado, apágala y vuelve a consultar.

---

## Ejercicio P4 — Clase `Estudiante`

Crea una clase `Estudiante` con los atributos `nombre` (String), `curso` (String) y `nota` (Double). Añade los métodos:

- `aprobado()` → devuelve `true` si la nota es mayor o igual a 5.0
- `calificacion()` → devuelve `"Aprobado"` o `"Suspenso"` según la nota
- `ficha()` → devuelve una línea con nombre, curso, nota y calificación

Crea una lista con 4 estudiantes de distintas notas e imprime la ficha de cada uno.

---

## Ejercicio P5 — `object` Calculadora

Crea un `object` llamado `Calculadora` con los métodos:

- `sumar(a: Double, b: Double)`
- `restar(a: Double, b: Double)`
- `multiplicar(a: Double, b: Double)`
- `dividir(a: Double, b: Double)` → si `b == 0` imprime `"Error: división por cero"` y devuelve `0.0`

Prueba cada operación con al menos dos llamadas, incluyendo una división por cero.

---

## Ejercicio P6 — Clase `Vehiculo` con constructor secundario

Crea una clase `Vehiculo` con los atributos `marca` (String), `modelo` (String), `año` (Int) y `color` (String).

- Añade un constructor secundario que solo reciba `marca` y `modelo`, asignando `año = 2024` y `color = "blanco"` por defecto.
- Añade un método `ficha()` que muestre todos los atributos en una línea.

Crea tres vehículos: uno con todos los parámetros, uno con el constructor secundario, y muestra su ficha.

---

## Ejercicio P7 — Clase `Temporizador`

Crea una clase `Temporizador` con un atributo mutable `segundos` (Int) que recibe el valor inicial en el constructor.

Añade los métodos:

- `avanzar(n: Int)` → suma `n` segundos al contador
- `reiniciar()` → pone `segundos` a 0
- `mostrarTiempo()` → imprime el tiempo en formato `"mm:ss"` (ej: `"02:05"`)

> 💡 Pista: minutos = `segundos / 60`, segundos restantes = `segundos % 60`. Usa `f"${min}%02d:${seg}%02d"` para el formato.
> 

Crea un temporizador, avanza varios intervalos, muestra el tiempo y reinícialo.

---

## Ejercicio P8 — Clase `CajaRegistradora`

Crea una clase `CajaRegistradora` con los atributos `tienda` (String) y un atributo privado mutable `totalVentas` (Double) inicializado a `0.0`.

Añade los métodos:

- `registrarVenta(importe: Double)` → suma el importe a `totalVentas` e imprime el ticket
- `obtenerTotal()` → devuelve `totalVentas`
- `cerrarCaja()` → imprime el total del día y pone `totalVentas` a `0.0`

Simula 4 ventas, consulta el total y cierra la caja.

---

## Ejercicio P9 — Clase `Cancion` y `object` Reproductor

Crea una clase `Cancion` con los atributos `titulo` (String), `artista` (String) y `duracionSegundos` (Int). Añade un método `duracionFormateada()` que devuelva la duración en formato `"mm:ss"`.

Crea un `object` llamado `Reproductor` con:

- Un atributo mutable `volumen` (Int) inicializado a 50
- Un método `reproducir(c: Cancion)` que imprima qué canción suena y a qué volumen
- Un método `subirVolumen(n: Int)` que sume sin superar 100
- Un método `bajarVolumen(n: Int)` que reste sin bajar de 0

Crea tres canciones, reprodúcelas y cambia el volumen entre reproducciones.

---

## Ejercicio P10 — Clase `Factura` con constructor secundario y encapsulamiento

Crea una clase `Factura` con los atributos `numeroFactura` (Int), `cliente` (String), `baseImponible` (Double) y `tipoIVA` (Double). Añade un constructor secundario que solo reciba `numeroFactura`, `cliente` y `baseImponible`, usando un IVA del 21% por defecto.

Añade los métodos (todos con lógica `private` donde corresponda):

- `private def calcularIVA()` → devuelve `baseImponible * tipoIVA / 100`
- `total()` → devuelve `baseImponible + calcularIVA()`
- `imprimir()` → muestra un resumen con número, cliente, base, IVA y total

Crea dos facturas: una con IVA personalizado (10%) y otra usando el constructor secundario.

---

## Ejercicio P11 — Clase `Semaforo`

Crea una clase `Semaforo` con un atributo mutable privado `estado` (String) inicializado a `"rojo"`. Añade los métodos:

- `siguiente()` → avanza al siguiente estado: `rojo → verde → amarillo → rojo`
- `obtenerEstado()` → devuelve el estado actual
- `puedeAvanzar()` → devuelve `true` solo si el estado es `"verde"`

> 💡 Pista: usa `match` para gestionar las transiciones de estado.
> 

Crea un semáforo y simula 6 cambios de estado, imprimiendo en cada paso si se puede avanzar.

---

## Ejercicio P12 — Clase `Pila` (Stack) [Opcional]

Crea una clase `Pila` que modele una pila de elementos de tipo `String`. Usa una `List[String]` privada y mutable como almacenamiento interno.

Añade los métodos:

- `apilar(elemento: String)` → añade el elemento al tope de la pila
- `desapilar()` → elimina y devuelve el elemento del tope; si está vacía imprime `"La pila está vacía"`
- `tope()` → devuelve el elemento del tope sin eliminarlo
- `tamaño()` → devuelve el número de elementos
- `mostrar()` → imprime todos los elementos de arriba a abajo

Simula apilar 4 elementos, mostrar la pila, desapilar dos y volver a mostrar.

---

## Ejercicio P13 — `object` EstadisticasCurso

Crea un `object` llamado `EstadisticasCurso` que reciba una lista de notas (`List[Double]`) en cada operación y calcule:

- `media(notas: List[Double])` → media aritmética
- `notaMaxima(notas: List[Double])` → nota más alta
- `notaMinima(notas: List[Double])` → nota más baja
- `numAprobados(notas: List[Double])` → cuántos tienen nota >= 5.0
- `informe(notas: List[Double])` → imprime un resumen con todos los datos anteriores

Pruébalo con una lista de al menos 8 notas que incluya aprobados y suspensos.

---

## Ejercicio P14 — Clase `CuentaAhorro` con historial

Crea una clase `CuentaAhorro` con los atributos `titular` (String) y un saldo privado inicial (`saldoInicial: Double`). Añade un atributo privado `historial` como `List[String]` (inicialmente vacía) que registre cada operación.

Añade los métodos:

- `ingresar(cantidad: Double)` → suma al saldo y añade la operación al historial
- `retirar(cantidad: Double)` → resta del saldo si hay fondos suficientes; si no, registra el intento fallido
- `obtenerSaldo()` → devuelve el saldo actual
- `imprimirHistorial()` → imprime todas las operaciones realizadas numeradas

Simula al menos 5 operaciones (ingresos, retiradas y un intento fallido) e imprime el historial final.

---

## Ejercicio P15 — Sistema de parking

Crea una clase `Plaza` con los atributos `numero` (Int) y un atributo privado mutable `matricula` (String) inicializado a `""` (vacío = libre).

Añade los métodos:

- `ocupar(matricula: String)` → asigna la matrícula si la plaza está libre; si no, avisa
- `liberar()` → deja la plaza libre si estaba ocupada; si no, avisa
- `estaLibre()` → devuelve `true` si no hay matrícula
- `estado()` → devuelve `"[001] LIBRE"` o `"[001] 1234-ABC"`

Crea un `object` llamado `Parking` con:

- Una lista de 5 plazas (`List[Plaza]`)
- `plazasLibres()` → cuenta cuántas están libres
- `buscarVehiculo(matricula: String)` → devuelve el número de plaza donde está, o avisa si no se encuentra
- `mostrarEstado()` → imprime el estado de todas las plazas

Simula: ocupa 3 plazas, muestra el estado, libera una, busca un vehículo y muestra el estado final.

---