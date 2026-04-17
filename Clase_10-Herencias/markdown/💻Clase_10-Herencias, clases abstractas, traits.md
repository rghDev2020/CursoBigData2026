# 💻Clase 10 - Herencias, clases abstractas, traits y mixins

---

# Agenda:

<aside>
💡

#### 9:00 - 9:50    →  herencias, clases abstractas, traits y mixins

#### 9:50 - 11:20   → Ejercicios

#### 11:40 - 12:40  → Ejercicios

#### 12:40 - 14:00  → Ejercicios

</aside>

---

### 1. Herencia: `extends` y `override`

> La **herencia** permite que una clase hija reutilice y amplíe el comportamiento de una clase padre. En Scala se usa la palabra clave `extends`.
> 

<aside>

`override` es la palabra clave que **obliga** a la clase hija a declarar explícitamente que está reemplazando un método ya definido en la clase padre.

En Scala, a diferencia de Java, **`override` es obligatorio**. Si intentas sobreescribir un método sin indicarlo, el compilador lanza un error. Esto es una decisión de diseño deliberada: evita sobreescrituras accidentales. Sin `override` el compilador rechaza el código.

</aside>

```scala
class Animal(val nombre: String) {
  def sonido(): String = "..."
  def describir(): String = s"$nombre dice: ${sonido()}"
}

class Perro(nombre: String) extends Animal(nombre) {
  override def sonido(): String = "¡Guau!"
}

class Gato(nombre: String) extends Animal(nombre) {
  override def sonido(): String = "¡Miau!"
}

val animales: List[Animal] = List(
  new Perro("Rex"),
  new Gato("Whiskers"),
  new Animal("Criatura desconocida")
)

for (a <- animales) println(a.describir())
```

**Salida esperada:**

```
Rex dice: ¡Guau!
Whiskers dice: ¡Miau!
Criatura desconocida dice: ...
```

> Fíjate en algo importante: la lista es de tipo `List[Animal]`, pero cada elemento ejecuta **su propio** `sonido()`. Esto es el **polimorfismo**: el mismo método se comporta de forma diferente según el tipo real del objeto. Es uno de los pilares más poderosos de la POO.
> 

### Reglas clave de la herencia en Scala

| Regla | Descripción |
| --- | --- |
| `extends` | La clase hija hereda todos los miembros públicos y protegidos |
| `override` | **Obligatorio** en Scala para sobreescribir un método de la clase padre |
| `super` | Permite llamar al método del padre desde la clase hija |
| Solo una clase padre | Scala no permite herencia múltiple de clases (sí de traits) |

### Uso de `super` :

<aside>

`super` es la palabra clave que permite llamar al método de la **clase padre** desde dentro de la clase hija, en lugar de reemplazarlo completamente.

</aside>

> Imagina una receta familiar. La abuela tiene su receta de tortilla: *huevos, patata y sal*. Tú haces tu propia versión, pero en lugar de escribir la receta entera desde cero, dices: *"hago la receta de la abuela y además le añado cebolla"*.
> 
> 
> ```scala
> class Abuela {
>   def receta(): String = "huevos + patata + sal"
> }
> 
> class Nieto extends Abuela {
>   override def receta(): String =
>     super.receta() + " + cebolla"
> }
> 
> val yo = new Nieto()
> println(yo.receta())
> // huevos + patata + sal + cebolla
> ```
> 
> `super.receta()` no repite el código de la abuela, sino que lo **reutiliza** y le añade algo encima. Si la abuela cambia su receta, el nieto hereda el cambio automáticamente.
> 

**Ejemplo**

```scala
class Vehiculo(val marca: String) {
  def describir(): String = s"Vehículo de marca $marca"
}

class Coche(marca: String, val puertas: Int) extends Vehiculo(marca) {
  override def describir(): String =
    super.describir() + s" con $puertas puertas"
}

val c = new Coche("Toyota", 4)
println(c.describir())
// Vehículo de marca Toyota con 4 puertas
```

---

### 2. Clases abstractas: `abstract class`

> Un **contrato de trabajo** define que todo empleado debe `fichar()` al entrar y `realizarTarea()`. Pero el contrato en sí no especifica las tareas, ya que es una plantilla de contrato . Cada empleado concreto (programador, diseñador) lo firma y define cómo realiza su tarea específica.
> 

<aside>

> Una **clase abstracta** es una clase que **no se puede instanciar directamente**. Define una plantilla con algunos métodos sin implementar (abstractos) que las clases hijas deben completar obligatoriamente.
> 
</aside>

```scala
abstract class Figura {
  val nombre: String                  // atributo abstracto
  def area(): Double                  // método abstracto (sin cuerpo)
  def perimetro(): Double             // método abstracto (sin cuerpo)

  // Método concreto: tiene implementación, lo heredan todas las figuras
  def descripcion(): String =
    f"$nombre — Área: ${area()}%.2f | Perímetro: ${perimetro()}%.2f"
}
```

Las clases hijas deben implementar **todos** los miembros abstractos:

```scala
abstract class Figura {
  val nombre: String
  def area(): Double
  def perimetro(): Double
  def descripcion(): String =
    f"$nombre — Área: ${area()}%.2f — Perímetro: ${perimetro()}%.2f"
}

class Circulo(val radio: Double) extends Figura {
  val nombre: String      = "Círculo"
  def area(): Double      = scala.math.Pi * radio * radio
  def perimetro(): Double = 2 * scala.math.Pi * radio
}

class RectanguloFig(val base: Double, val altura: Double) extends Figura {
  val nombre: String      = "Rectángulo"
  def area(): Double      = base * altura
  def perimetro(): Double = 2 * (base + altura)
}

class TrianguloEquilatero(val lado: Double) extends Figura {
  val nombre: String      = "Triángulo equilátero"
  def area(): Double      = (scala.math.sqrt(3) / 4) * lado * lado
  def perimetro(): Double = 3 * lado
}

val figuras: List[Figura] = List(
  new Circulo(5.0),
  new RectanguloFig(4.0, 6.0),
  new TrianguloEquilatero(3.0)
)

for (f <- figuras) println(f.descripcion())
```

**Salida esperada:**

```
Círculo — Área: 78.54 | Perímetro: 31.42
Rectángulo — Área: 24.00 | Perímetro: 20.00
Triángulo equilátero — Área: 3.90 | Perímetro: 9.00
```

> 💡 Si intentas hacer `new Figura()` obtendrás un error de compilación: `class Figura is abstract; cannot be instantiated`. Esto es exactamente lo que queremos — nadie puede crear una "figura genérica" sin forma concreta.
> 

---

### 3. `trait`: la interfaz con superpoderes

<aside>

Un **`trait`** es una unidad de abstracción que encapsula definiciones de métodos y valores, tanto abstractos como concretos, y que puede ser incorporada a cualquier clase mediante `extends` o `with`. A diferencia de una clase, un `trait` no admite parámetros de constructor ni puede instanciarse directamente. Su propósito es definir un contrato de comportamiento — total o parcial — que las clases adoptantes deben completar en los miembros abstractos, y del que heredan automáticamente los miembros ya implementados.

</aside>

> 
> 
> 
> Un **`trait`** es como un **carné de habilidades** que puedes añadir a cualquier persona, independientemente de quién sea. Imagina el carné de **Primeros Auxilios**. Cualquier persona — un profesor, un bombero, un cocinero — puede obtenerlo. El carné ya sabe cómo presentarse (*"Titular del carné de Primeros Auxilios"*), pero la forma en que cada persona aplica esos conocimientos depende de ella: el profesor los aplica en el aula, el bombero en un accidente, el cocinero en su restaurante. Lo más poderoso del `trait` es que una misma persona puede tener **varios carnés a la vez**: Primeros Auxilios, Conducción, Idiomas... y combinarlos todos. Eso es lo que en Scala se llama **mixin** y no es posible con `abstract class`, que solo permite heredar de una única clase padre.
> 

```scala
trait Saludable {
  def saludar(): String              // abstracto: cada clase lo implementa

  // Método con implementación por defecto
  def presentarse(): String = s"Me presento: ${saludar()}"
}
```

Una clase puede **implementar** (también se dice "mezclar") un trait con `extends`:

```scala
class Medico(val nombre: String) extends Saludable {
  def saludar(): String = s"Buenos días, soy el Dr. $nombre"
}

class Paciente(val nombre: String) extends Saludable {
  def saludar(): String = s"Hola, me llamo $nombre"
}

val m = new Medico("Rodríguez")
val p = new Paciente("Laura")

println(m.presentarse())
// Me presento: Buenos días, soy el Dr. Rodríguez
println(p.presentarse())
// Me presento: Hola, me llamo Laura
```

---

### 4. Diferencia entre `trait` y `abstract class`

Esta es una de las preguntas más frecuentes al aprender Scala. La tabla siguiente resume cuándo usar cada uno:

|  | `abstract class` | `trait` |
| --- | --- | --- |
| ¿Puede tener constructor con parámetros? | ✅ Sí | ❌ No |
| ¿Puede tener métodos abstractos? | ✅ Sí | ✅ Sí |
| ¿Puede tener métodos con implementación? | ✅ Sí | ✅ Sí |
|  |  |  |

<aside>

La diferencia más importante en la práctica:

**`abstract class`** modela **lo que algo es**. Un `Perro` *es* un `Animal`. Hay una relación jerárquica clara entre padre e hijo.

**`trait`** modela **lo que algo puede hacer**. Un `Perro` *puede* `Nadar`, *puede* `Correr`. Son capacidades independientes que se suman.

</aside>

**Regla práctica:**

- Usa `abstract class` cuando la jerarquía tiene un **constructor con parámetros** o representa una relación "es un tipo de" muy clara.
- Usa `trait` cuando quieras añadir **capacidades o comportamientos** que pueden compartir clases de distinta naturaleza.

---

### 5. Mixins: combinar múltiples traits

La verdadera potencia de los traits está en los **mixins**: una clase puede mezclar varios traits a la vez, obteniendo las capacidades de todos ellos.

```scala
trait Volador {
  def volar(): String = "¡Estoy volando! 🦅"
}

trait Nadador {
  def nadar(): String = "¡Estoy nadando! 🏊"
}

trait Corredor {
  def correr(): String = "¡Estoy corriendo! 🏃"
}

// Un pato puede hacer las tres cosas
class Pato(val nombre: String) extends Volador with Nadador with Corredor {
  def presentarse(): String = s"Soy $nombre y puedo:"
}

// Un pez solo nada
class Pez(val nombre: String) extends Nadador {
  def presentarse(): String = s"Soy $nombre y puedo:"
}

val pato = new Pato("Donald")
println(pato.presentarse())
println("  " + pato.volar())
println("  " + pato.nadar())
println("  " + pato.correr())

val pez = new Pez("Nemo")
println(pez.presentarse())
println("  " + pez.nadar())
```

**Salida esperada:**

```
Soy Donald y puedo:
  ¡Estoy volando! 🦅
  ¡Estoy nadando! 🏊
  ¡Estoy corriendo! 🏃
Soy Nemo y puedo:
  ¡Estoy nadando! 🏊
```

> 💡 En Spark, los traits se usan extensamente. Por ejemplo, los `DataFrame` y los `Dataset` comparten comportamientos comunes a través de traits internos. Entender este mecanismo es clave para leer y comprender el código de Spark.
> 

### Sintaxis para combinar clase base y traits

```scala
class MiClase extends ClaseBase with Trait1 with Trait2 with Trait3
```

Si no hay clase base (solo traits):

```
class MiClase extends Trait1 with Trait2 with Trait3
```

---

### 6. Ejemplo : sistema de medios de pago

Veamos cómo encajan todos los conceptos en un ejemplo realista:

```scala
// Trait: capacidad de pagar
trait Pagable {
  def pagar(importe: Double): String
  def saldo(): Double
  def resumen(): String = f"Saldo disponible: ${saldo()}%.2f €"
}

// Trait: capacidad de retirar efectivo
trait RetiradaEfectivo {
  def retirarEfectivo(cantidad: Double): String
}

// Clase abstracta base para todos los medios de pago
abstract class MedioPago(val titular: String) extends Pagable {
  val tipo: String   // abstracto
}

// Tarjeta de débito: paga y puede retirar efectivo
class TarjetaDebito(titular: String, private var _saldo: Double)
    extends MedioPago(titular) with RetiradaEfectivo {

  val tipo: String = "Tarjeta Débito"

  def saldo(): Double = _saldo

  def pagar(importe: Double): String =
    if (importe <= _saldo) {
      _saldo -= importe
      f"[$tipo] Pago de ${importe}%.2f € aceptado. ${resumen()}"
    } else s"[$tipo] Fondos insuficientes."

  def retirarEfectivo(cantidad: Double): String =
    if (cantidad <= _saldo) {
      _saldo -= cantidad
      f"[$tipo] Retirada de ${cantidad}%.2f €. ${resumen()}"
    } else s"[$tipo] Saldo insuficiente para retirar efectivo."
}

// Tarjeta de crédito: paga con límite de crédito
class TarjetaCredito(titular: String, val limiteCredito: Double)
    extends MedioPago(titular) {

  val tipo: String = "Tarjeta Crédito"
  private var _deuda: Double = 0.0

  def saldo(): Double = limiteCredito - _deuda

  def pagar(importe: Double): String =
    if (importe <= saldo()) {
      _deuda += importe
      f"[$tipo] Cargo de ${importe}%.2f €. ${resumen()}"
    } else s"[$tipo] Límite de crédito superado."
}
```

```scala
val debito  = new TarjetaDebito("Ana López", 800.0)
val credito = new TarjetaCredito("Ana López", 1500.0)

println(debito.pagar(150.0))
println(debito.retirarEfectivo(50.0))
println(debito.pagar(700.0))

println(credito.pagar(400.0))
println(credito.pagar(1200.0))
```

**Salida esperada:**

```
[Tarjeta Débito] Pago de 150.00 € aceptado. Saldo disponible: 650.00 €
[Tarjeta Débito] Retirada de 50.00 €. Saldo disponible: 600.00 €
[Tarjeta Débito] Fondos insuficientes.
[Tarjeta Crédito] Cargo de 400.00 €. Saldo disponible: 1100.00 €
[Tarjeta Crédito] Límite de crédito superado.
```

---

### 🔑 Resumen de conceptos clave

| Concepto | Sintaxis | Para qué sirve |
| --- | --- | --- |
| Herencia | `class Hija extends Padre` | Reutilizar y ampliar una clase |
| Sobreescritura | `override def metodo()` | Cambiar el comportamiento heredado |
| Llamada al padre | `super.metodo()` | Reutilizar la lógica del padre |
| Clase abstracta | `abstract class` | Plantilla no instanciable con miembros abstractos |
| Trait | `trait` | Comportamiento reutilizable y combinable |
| Mixin | `extends T1 with T2` | Combinar varios traits en una clase |
| Polimorfismo | `List[Animal]` con `Perro`, `Gato`... | Un tipo común, comportamientos distintos |

---

## 💻 Práctica

---

### 🔹 Ejercicio 1 — Jerarquía de animales con herencia

**Celda 2 — Code (clase base y subclases):**

```scala
class Animal(val nombre: String, val especie: String) {
  def sonido(): String  = "..."
  def moverse(): String = "moviéndose"

  def ficha(): String =
    s"[$especie] $nombre — Sonido: ${sonido()} — Se desplaza: ${moverse()}"
}

class Perro(nombre: String) extends Animal(nombre, "Perro") {
  override def sonido(): String  = "¡Guau!"
  override def moverse(): String = "corriendo"
  def buscarPelota(): String = s"$nombre va a buscar la pelota 🎾"
}

class Pajaro(nombre: String) extends Animal(nombre, "Pájaro") {
  override def sonido(): String  = "¡Pío!"
  override def moverse(): String = "volando"
}

class Pez(nombre: String) extends Animal(nombre, "Pez") {
  override def sonido(): String  = "..."
  override def moverse(): String = "nadando"
}
```

**Celda 3 — Code (uso polimórfico):**

```scala
val animales: List[Animal] = List(
  new Perro("Rex"),
  new Pajaro("Tweety"),
  new Pez("Nemo"),
  new Perro("Lassie")
)

println("=== Registro de animales ===")
for (a <- animales) println(a.ficha())
```

**Salida esperada:**

```
=== Registro de animales ===
[Perro] Rex — Sonido: ¡Guau! — Se desplaza: corriendo
[Pájaro] Tweety — Sonido: ¡Pío! — Se desplaza: volando
[Pez] Nemo — Sonido: ... — Se desplaza: nadando
[Perro] Lassie — Sonido: ¡Guau! — Se desplaza: corriendo
```

**Celda 4 — Code (método propio de subclase):**

```scala
// Los métodos específicos de la subclase requieren el tipo concreto
val miPerro = new Perro("Rex")
println(miPerro.buscarPelota())
// Rex va a buscar la pelota 🎾
```

---

### 🔹 Ejercicio 2 — Formas geométricas con `abstract class`

**Celda 2 — Code:**

```scala
abstract class FormaGeometrica {
  val nombre: String
  val color: String

  def area(): Double
  def perimetro(): Double

  // Método concreto compartido por todas las formas
  def describir(): String =
    f"$nombre ($color) — Área: ${area()}%.2f cm² | Perímetro: ${perimetro()}%.2f cm"

  def esMasGrandeQue(otra: FormaGeometrica): Boolean =
    this.area() > otra.area()
}

class Cuadrado(val lado: Double, val color: String) extends FormaGeometrica {
  val nombre: String      = "Cuadrado"
  def area(): Double      = lado * lado
  def perimetro(): Double = 4 * lado
}

class CirculoGeom(val radio: Double, val color: String) extends FormaGeometrica {
  val nombre: String      = "Círculo"
  def area(): Double      = Math.PI * radio * radio
  def perimetro(): Double = 2 * Math.PI * radio
}

class TrianguloRect(val cateto1: Double, val cateto2: Double, val color: String)
    extends FormaGeometrica {
  val nombre: String      = "Triángulo rectángulo"
  def area(): Double      = (cateto1 * cateto2) / 2
  def perimetro(): Double = cateto1 + cateto2 + Math.sqrt(cateto1*cateto1 + cateto2*cateto2)
}
```

**Celda 3 — Code:**

```scala
val formas: List[FormaGeometrica] = List(
  new Cuadrado(5.0, "rojo"),
  new CirculoGeom(3.0, "azul"),
  new TrianguloRect(3.0, 4.0, "verde")
)

println("=== Catálogo de formas ===")
for (f <- formas) println(f.describir())

// Encontrar la forma con mayor área
val mayor = formas.reduce((a, b) => if (a.esMasGrandeQue(b)) a else b)
println(s"\nForma con mayor área: ${mayor.nombre}")
```

**Salida esperada:**

```
=== Catálogo de formas ===
Cuadrado (rojo) — Área: 25.00 cm² | Perímetro: 20.00 cm
Círculo (azul) — Área: 28.27 cm² | Perímetro: 18.85 cm
Triángulo rectángulo (verde) — Área: 6.00 cm² | Perímetro: 12.00 cm

Forma con mayor área: Círculo
```

---

### 🔹 Ejercicio 3 — Traits y mixins: sistema de empleados con roles (20 min)

**Celda 2 — Code:**

```scala
trait Gestionable {
  def gestionarEquipo(): String = "Gestiona un equipo de personas"
}

trait Programable {
  def programar(lenguaje: String): String = s"Programa en $lenguaje"
}

trait Vendible {
  def vender(producto: String): String = s"Vende $producto a clientes"
}

trait Formable {
  def impartirFormacion(tema: String): String = s"Imparte formación sobre $tema"
}

// Cada empleado mezcla los traits según su rol
class DirectorTecnico(val nombre: String)
    extends Gestionable with Programable with Formable {
  def presentarse(): String = s"Soy $nombre, Director Técnico"
}

class Desarrollador(val nombre: String)
    extends Programable with Formable {
  def presentarse(): String = s"Soy $nombre, Desarrollador"
}

class ComercialTecnico(val nombre: String)
    extends Vendible with Programable {
  def presentarse(): String = s"Soy $nombre, Comercial Técnico"
}
```

**Celda 3 — Code:**

```scala
val director = new DirectorTecnico("Laura Martínez")
val dev      = new Desarrollador("Carlos Ruiz")
val comercial = new ComercialTecnico("Ana Torres")

println(director.presentarse())
println("  → " + director.gestionarEquipo())
println("  → " + director.programar("Scala"))
println("  → " + director.impartirFormacion("Big Data"))

println(dev.presentarse())
println("  → " + dev.programar("Python"))
println("  → " + dev.impartirFormacion("Spark"))

println(comercial.presentarse())
println("  → " + comercial.vender("licencia de software"))
println("  → " + comercial.programar("SQL"))
```

**Salida esperada:**

```
Soy Laura Martínez, Director Técnico
  → Gestiona un equipo de personas
  → Programa en Scala
  → Imparte formación sobre Big Data
Soy Carlos Ruiz, Desarrollador
  → Programa en Python
  → Imparte formación sobre Spark
Soy Ana Torres, Comercial Técnico
  → Vende licencia de software a cliente
  → Programa en SQL
```

---

### 🔹 Ejercicio 4 — Sistema de ventas con herencia y traits

**Celda 2 — Code (traits y clase abstracta):**

```scala
trait Descuentable {
  def aplicarDescuento(porcentaje: Double, precio: Double): Double =
    precio * (1 - porcentaje / 100)
}

trait Imprimible {
  def imprimir(): Unit
}

abstract class ProductoTienda(val codigo: String, val nombre: String, val precioBase: Double)
    extends Descuentable with Imprimible {

  val categoria: String
  def precioFinal(): Double   // abstracto: cada tipo calcula su precio final

  def imprimir(): Unit =
    println(f"[$codigo] $nombre%-25s | $categoria%-12s | ${precioFinal()}%8.2f €")
}
```

**Celda 3 — Code (subclases):**

```scala
class ProductoFisico(codigo: String, nombre: String, precioBase: Double, val pesoKg: Double)
    extends ProductoTienda(codigo, nombre, precioBase) {

  val categoria: String = "Físico"
  val gastosEnvio: Double = if (pesoKg <= 1.0) 2.99 else 4.99

  // El precio final incluye gastos de envío
  def precioFinal(): Double = precioBase + gastosEnvio
}

class ProductoDigital(codigo: String, nombre: String, precioBase: Double)
    extends ProductoTienda(codigo, nombre, precioBase) {

  val categoria: String = "Digital"

  // Descuento automático del 10% por ser digital
  def precioFinal(): Double = aplicarDescuento(10.0, precioBase)
}

class ProductoSuscripcion(codigo: String, nombre: String, precioBase: Double, val meses: Int)
    extends ProductoTienda(codigo, nombre, precioBase) {

  val categoria: String = "Suscripción"

  // Descuento por volumen: 5% si >= 6 meses, 15% si >= 12
  def precioFinal(): Double = {
    val descuento = if (meses >= 12) 15.0 else if (meses >= 6) 5.0 else 0.0
    aplicarDescuento(descuento, precioBase * meses)
  }
}
```

**Celda 4 — Code (catálogo y operaciones):**

```scala
val catalogo: List[ProductoTienda] = List(
  new ProductoFisico("F001", "Teclado mecánico", 79.99, 0.8),
  new ProductoFisico("F002", "Monitor 27\"", 299.99, 4.5),
  new ProductoDigital("D001", "Curso de Scala", 49.99),
  new ProductoDigital("D002", "Antivirus Pro", 39.99),
  new ProductoSuscripcion("S001", "Cloud Storage 100GB", 2.99, 12),
  new ProductoSuscripcion("S002", "Streaming Music", 9.99, 3)
)

println("=== CATÁLOGO DE PRODUCTOS ===")
println(f"${"Código"}%6s  ${"Nombre"}%-25s  ${"Categoría"}%-12s  ${"Precio"}%10s")
println("-" * 62)
for (p <- catalogo) p.imprimir()

val totalCatalogo = catalogo.map(_.precioFinal()).sum
println(f"\nValor total del catálogo: ${totalCatalogo}%.2f €")

val masBarato = catalogo.minBy(_.precioFinal())
val masCaroBR = catalogo.maxBy(_.precioFinal())
println(s"Más barato:  ${masBarato.nombre} (${f"${masBarato.precioFinal()}%.2f"} €)")
println(s"Más caro:    ${masCaroBR.nombre} (${f"${masCaroBR.precioFinal()}%.2f"} €)")
```

**Salida esperada:**

```
=== CATÁLOGO DE PRODUCTOS ===
Código  Nombre                     Categoría     Precio
--------------------------------------------------------------
[F001]  Teclado mecánico           | Físico       |    82.98 €
[F002]  Monitor 27"                | Físico       |   304.98 €
[D001]  Curso de Scala             | Digital      |    44.99 €
[D002]  Antivirus Pro              | Digital      |    35.99 €
[S001]  Cloud Storage 100GB        | Suscripción  |    30.49 €
[S002]  Streaming Music            | Suscripción  |    29.97 €

Valor total del catálogo: 529.40 €
Más barato:  Streaming Music (29.97 €)
Más caro:    Monitor 27" (304.98 €)
```

---

---

# Mas ejercicios resueltos

---

<aside>

Añade comentarios al código explicando que hace cada cose. Corrige los errores que encuentres, tambien es posible que la salida esperada no coincida totalmente con la salida real.

</aside>

## 🔹 Ejercicio 1 — Mi primera clase hija

---

**Enunciado:**

Crea una clase `Persona` con un atributo `nombre: String` y un método `presentarse(): String` que devuelva `"Hola, soy [nombre]"`. Luego crea una clase `Estudiante` que herede de `Persona` y sobreescriba `presentarse()` para devolver `"Hola, soy [nombre] y soy estudiante"`.

Instancia un `Estudiante` con nombre `"Ana"` e imprime el resultado de `presentarse()`.

```scala
class Persona(val nombre: String) {
  def presentarse(): String = s"Hola, soy $nombre"
}

class Estudiante(nombre: String) extends Persona(nombre) {
  override def presentarse(): String = s"Hola, soy $nombre y soy estudiante"
}

val e = new Estudiante("Ana")
println(e.presentarse())
```

**Salida esperada:**

```
Hola, soy Ana y soy estudiante
```

---

## 🔹 Ejercicio 2 — Herencia con `super`

**Enunciado:**

Crea una clase `Empleado` con atributos `nombre: String` y `empresa: String`, y un método `info(): String` que devuelva `"[nombre] trabaja en [empresa]"`. Crea una subclase `Gerente` que añada el atributo `departamento: String` y sobreescriba `info()` usando `super` para reutilizar la información del padre, añadiendo `", dirige el departamento de [departamento]"`.

```scala
class Empleado(val nombre: String, val empresa: String) {
  def info(): String = s"$nombre trabaja en $empresa"
}

class Gerente(nombre: String, empresa: String, val departamento: String)
    extends Empleado(nombre, empresa) {
  override def info(): String =
    super.info() + s", dirige el departamento de $departamento"
}

val g = new Gerente("Luis", "DataCorp", "Ingeniería")
println(g.info())
```

**Salida esperada:**

```
Luis trabaja en DataCorp, dirige el departamento de Ingeniería
```

---

## 🔹 Ejercicio 3 — Polimorfismo con lista de animales

**Enunciado:**

Crea una clase `Animal` con atributo `nombre: String` y método `sonido(): String` que devuelva `"..."`. Crea las subclases `Vaca` y `Oveja` que sobreescriban `sonido()` con `"¡Muuu!"` y `"¡Beee!"` respectivamente. Crea una `List[Animal]` con un animal de cada tipo y usa un `for` para imprimir `"[nombre]: [sonido]"` de cada uno.

```scala
class Animal(val nombre: String) {
  def sonido(): String = "..."
}

class Vaca(nombre: String) extends Animal(nombre) {
  override def sonido(): String = "¡Muuu!"
}

class Oveja(nombre: String) extends Animal(nombre) {
  override def sonido(): String = "¡Beee!"
}

val animales: List[Animal] = List(
  new Vaca("Lola"),
  new Oveja("Pepa"),
  new Animal("Criatura misteriosa")
)

for (a <- animales) println(s"${a.nombre}: ${a.sonido()}")
```

**Salida esperada:**

```
Lola: ¡Muuu!
Pepa: ¡Beee!
Criatura misteriosa: ...
```

---

## 🔹 Ejercicio 4 — Mi primer trait

**Enunciado:**

Define un `trait` llamado `Describible` con un método abstracto `descripcion(): String` y un método concreto `mostrar(): Unit` que imprima el resultado de `descripcion()` precedido de `"→ "`. Crea una clase `Ciudad` con atributos `nombre: String` y `pais: String` que implemente el trait. El método `descripcion()` debe devolver `"[nombre] está en [pais]"`.

```scala
trait Describible {
  def descripcion(): String
  def mostrar(): Unit = println(s"→ ${descripcion()}")
}

class Ciudad(val nombre: String, val pais: String) extends Describible {
  def descripcion(): String = s"$nombre está en $pais"
}

val c = new Ciudad("Sevilla", "España")
c.mostrar()
```

**Salida esperada:**

```
→ Sevilla está en España
```

---

## 🔹 Ejercicio 5 — Clase abstracta: figuras simples

**Enunciado:**

Crea una `abstract class Forma` con atributo abstracto `nombre: String` y método abstracto `area(): Double`. Añade un método concreto `resumen(): String` que devuelva `"[nombre] tiene un área de [area] unidades²"` con dos decimales. Implementa dos subclases: `Cuadrado(lado: Double)` y `Circulo(radio: Double)`.

```scala
abstract class Forma {
  val nombre: String
  def area(): Double
  def resumen(): String = f"$nombre tiene un área de ${area()}%.2f unidades²"
}

class Cuadrado(val lado: Double) extends Forma {
  val nombre: String = "Cuadrado"
  def area(): Double = lado * lado
}

class Circulo(val radio: Double) extends Forma {
  val nombre: String = "Círculo"
  def area(): Double = Math.PI * radio * radio
}

val formas: List[Forma] = List(new Cuadrado(4.0), new Circulo(3.0))
for (f <- formas) println(f.resumen())
```

**Salida esperada:**

```
Cuadrado tiene un área de 16.00 unidades²
Círculo tiene un área de 28.27 unidades²
```

---

## 🔹 Ejercicio 6 — Mixins: robot multiusos

**Enunciado:**

Define tres traits: `Hablador` con método concreto `hablar(): String` que devuelva `"Bip bop, hablo."`, `Calculador` con método concreto `calcular(a: Int, b: Int): Int` que devuelva `a + b`, y `Sensor` con método concreto `medir(): String` que devuelva `"Temperatura: 22°C"`. Crea una clase `RobotAvanzado(val id: String)` que mezcle los tres traits. Instancia el robot e invoca los tres métodos imprimiendo sus resultados.

```scala
trait Hablador {
  def hablar(): String = "Bip bop, hablo."
}

trait Calculador {
  def calcular(a: Int, b: Int): Int = a + b
}

trait Sensor {
  def medir(): String = "Temperatura: 22°C"
}

class RobotAvanzado(val id: String) extends Hablador with Calculador with Sensor

val robot = new RobotAvanzado("R2D2")
println(s"Robot ${robot.id}:")
println(robot.hablar())
println(s"3 + 5 = ${robot.calcular(3, 5)}")
println(robot.medir())
```

**Salida esperada:**

```
Robot R2D2:
Bip bop, hablo.
3 + 5 = 8
Temperatura: 22°C
```

---

## 🔹 Ejercicio 7 — Clase abstracta con cálculo de nómina

**Enunciado:**

Crea una `abstract class Trabajador` con atributos `nombre: String` y `salarioBase: Double`, y un método abstracto `salarioNeto(): Double`. Añade un método concreto `ficha(): String` que devuelva `"[nombre] — Salario neto: [salarioNeto] €"` con dos decimales.

Implementa tres subclases:

- `TrabajadorFijo(nombre, salarioBase)` → salario neto = salarioBase
- `TrabajadorPorHoras(nombre, precioHora: Double, horas: Int)` → salario neto = precioHora * horas
- `TrabajadorComisiones(nombre, salarioBase, ventas: Double)` → salario neto = salarioBase + ventas * 0.05

Crea una lista con un trabajador de cada tipo y muestra la ficha de cada uno.

```scala
abstract class Trabajador(val nombre: String, val salarioBase: Double) {
  def salarioNeto(): Double
  def ficha(): String = f"$nombre — Salario neto: ${salarioNeto()}%.2f €"
}

class TrabajadorFijo(nombre: String, salarioBase: Double)
    extends Trabajador(nombre, salarioBase) {
  def salarioNeto(): Double = salarioBase
}

class TrabajadorPorHoras(nombre: String, val precioHora: Double, val horas: Int)
    extends Trabajador(nombre, 0.0) {
  def salarioNeto(): Double = precioHora * horas
}

class TrabajadorComisiones(nombre: String, salarioBase: Double, val ventas: Double)
    extends Trabajador(nombre, salarioBase) {
  def salarioNeto(): Double = salarioBase + ventas * 0.05
}

val trabajadores: List[Trabajador] = List(
  new TrabajadorFijo("Marta García", 2100.0),
  new TrabajadorPorHoras("Pedro Sanz", 15.0, 120),
  new TrabajadorComisiones("Elena Vidal", 1500.0, 8000.0)
)

for (t <- trabajadores) println(t.ficha())
```

**Salida esperada:**

```
Marta García — Salario neto: 2100.00 €
Pedro Sanz — Salario neto: 1800.00 €
Elena Vidal — Salario neto: 1900.00 €
```

---

## 🔹 Ejercicio 8 — Trait con estado: registros de acceso

**Enunciado:**

Define un trait `Auditable` con un método abstracto `identificador(): String` y un método concreto `registrarAcceso(): Unit` que imprima `"[timestamp] Acceso registrado para: [identificador]"` donde el timestamp es el resultado de `java.time.LocalTime.now().withNano(0)`.

Crea las clases `UsuarioAdmin(val login: String)` y `UsuarioInvitado(val login: String)` que implementen el trait. El `identificador()` de admin devuelve `"ADMIN:[login]"` y el de invitado `"GUEST:[login]"`.

```scala
trait Auditable {
  def identificador(): String
  def registrarAcceso(): Unit = {
    val ts = java.time.LocalTime.now().withNano(0)
    println(s"$ts Acceso registrado para: ${identificador()}")
  }
}

class UsuarioAdmin(val login: String) extends Auditable {
  def identificador(): String = s"ADMIN:$login"
}

class UsuarioInvitado(val login: String) extends Auditable {
  def identificador(): String = s"GUEST:$login"
}

val admin    = new UsuarioAdmin("jlopez")
val invitado = new UsuarioInvitado("anonimo01")

admin.registrarAcceso()
invitado.registrarAcceso()
```

**Salida esperada** *(la hora varía, el formato es fijo)*:

```
10:45:32 Acceso registrado para: ADMIN:jlopez
10:45:32 Acceso registrado para: GUEST:anonimo01
```

---

## 🔹 Ejercicio 9 — Herencia en cadena

**Enunciado:**

Crea tres clases en cadena de herencia:

1. `Ser` con método `tipo(): String` → `"Ser vivo"`
2. `Mamifero` extiende `Ser`, sobreescribe `tipo()` → `"Mamífero"` y añade `temperatura(): String` → `"Temperatura corporal: 37°C"`
3. `Humano(val nombre: String)` extiende `Mamifero`, sobreescribe `tipo()` usando `super` → `"[resultado padre] — Humano"` y añade `saludar(): String` → `"Hola, me llamo [nombre]"`

Instancia un `Humano("Raquel")` y llama a los tres métodos: `tipo()`, `temperatura()` y `saludar()`.

```scala
class Ser {
  def tipo(): String = "Ser vivo"
}

class Mamifero extends Ser {
  override def tipo(): String = "Mamífero"
  def temperatura(): String   = "Temperatura corporal: 37°C"
}

class Humano(val nombre: String) extends Mamifero {
  override def tipo(): String = super.tipo() + " — Humano"
  def saludar(): String       = s"Hola, me llamo $nombre"
}

val h = new Humano("Raquel")
println(h.tipo())
println(h.temperatura())
println(h.saludar())
```

**Salida esperada:**

```
Mamífero — Humano
Temperatura corporal: 37°C
Hola, me llamo Raquel
```

---

## 🔹 Ejercicio 10 — Trait `Comparable`: ordenar por precio

**Enunciado:**

Define un trait `Comparable[A]` con un método abstracto `esMayorQue(otro: A): Boolean`. Crea una clase `Producto(val nombre: String, val precio: Double)` que implemente `Comparable[Producto]`; el método `esMayorQue` devuelve `true` si el precio del producto actual supera al del otro.

Crea una lista de tres productos y usa `filter` para obtener los que cuestan más de 50 €, y `reduce` para encontrar el más caro.

```scala
trait Comparable[A] {
  def esMayorQue(otro: A): Boolean
}

class Producto(val nombre: String, val precio: Double) extends Comparable[Producto] {
  def esMayorQue(otro: Producto): Boolean = this.precio > otro.precio
  override def toString: String = f"$nombre (${precio}%.2f €)"
}

val productos = List(
  new Producto("Auriculares", 39.99),
  new Producto("Tablet", 249.99),
  new Producto("Ratón inalámbrico", 29.99)
)

val caros = productos.filter(_.esMayorQue(new Producto("_", 50.0)))
println("Productos por encima de 50 €:")
for (p <- caros) println(s"  $p")

val masCaro = productos.reduce((a, b) => if (a.esMayorQue(b)) a else b)
println(s"Más caro: $masCaro")
```

**Salida esperada:**

```
Productos por encima de 50 €:
  Tablet (249.99 €)
Más caro: Tablet (249.99 €)
```

---

## 🔹 Ejercicio 11 — Abstract class y herencia: vehículos eléctricos

**Enunciado:**

Crea una `abstract class VehiculoElectrico` con atributos `marca: String` y `bateria: Int` (kWh), un método abstracto `autonomia(): Int` (kilómetros) y un método concreto `estado(): String` con el formato:

```
[marca] | Batería: [bateria] kWh | Autonomía: [autonomia] km
```

Implementa:

- `Turismo(marca, bateria)` → autonomia = bateria * 6
- `Motocicleta(marca, bateria)` → autonomia = bateria * 10
- `Furgoneta(marca, bateria, val carga: Int)` → autonomia = bateria * 4 (la carga reduce eficiencia)

Crea una lista con un vehículo de cada tipo y muestra el estado de cada uno. Al final imprime la autonomía total sumada.

```scala
abstract class VehiculoElectrico(val marca: String, val bateria: Int) {
  def autonomia(): Int
  def estado(): String =
    s"$marca | Batería: $bateria kWh | Autonomía: ${autonomia()} km"
}

class Turismo(marca: String, bateria: Int) extends VehiculoElectrico(marca, bateria) {
  def autonomia(): Int = bateria * 6
}

class Motocicleta(marca: String, bateria: Int) extends VehiculoElectrico(marca, bateria) {
  def autonomia(): Int = bateria * 10
}

class Furgoneta(marca: String, bateria: Int, val carga: Int)
    extends VehiculoElectrico(marca, bateria) {
  def autonomia(): Int = bateria * 4
}

val flota: List[VehiculoElectrico] = List(
  new Turismo("Tesla", 75),
  new Motocicleta("Zero", 14),
  new Furgoneta("Renault", 52, 800)
)

println("=== Flota eléctrica ===")
for (v <- flota) println(v.estado())

val autonomiaTotal = flota.map(_.autonomia()).sum
println(s"\nAutonomía total combinada: $autonomiaTotal km")
```

**Salida esperada:**

```
=== Flota eléctrica ===
Tesla | Batería: 75 kWh | Autonomía: 450 km
Zero | Batería: 14 kWh | Autonomía: 140 km
Renault | Batería: 52 kWh | Autonomía: 208 km

Autonomía total combinada: 798 km
```

---

## 🔹 Ejercicio 12 — Traits compuestos: sistema de pagos

**Enunciado:**

Define dos traits:

- `Pagable` con método abstracto `calcularTotal(): Double` y método concreto `mostrarTotal(): Unit` que imprima `"Total a pagar: [total] €"` con dos decimales.
- `Reembolsable` con método abstracto `calcularReembolso(): Double` y método concreto `mostrarReembolso(): Unit` que imprima `"Reembolso aplicable: [reembolso] €"` con dos decimales.

Crea las clases:

- `PedidoEstandar(val importe: Double)` → implementa solo `Pagable`; total = importe
- `PedidoConDevolucion(val importe: Double, val porcentajeDevolucion: Double)` → implementa `Pagable` y `Reembolsable`; total = importe, reembolso = importe * porcentajeDevolucion / 100

```scala
trait Pagable {
  def calcularTotal(): Double
  def mostrarTotal(): Unit = println(f"Total a pagar: ${calcularTotal()}%.2f €")
}

trait Reembolsable {
  def calcularReembolso(): Double
  def mostrarReembolso(): Unit = println(f"Reembolso aplicable: ${calcularReembolso()}%.2f €")
}

class PedidoEstandar(val importe: Double) extends Pagable {
  def calcularTotal(): Double = importe
}

class PedidoConDevolucion(val importe: Double, val porcentajeDevolucion: Double)
    extends Pagable with Reembolsable {
  def calcularTotal(): Double     = importe
  def calcularReembolso(): Double = importe * porcentajeDevolucion / 100
}

val p1 = new PedidoEstandar(120.0)
val p2 = new PedidoConDevolucion(200.0, 15.0)

println("--- Pedido estándar ---")
p1.mostrarTotal()

println("--- Pedido con devolución ---")
p2.mostrarTotal()
p2.mostrarReembolso()
```

**Salida esperada:**

```
--- Pedido estándar ---
Total a pagar: 120.00 €
--- Pedido con devolución ---
Total a pagar: 200.00 €
Reembolso aplicable: 30.00 €
```

---

## 🔹 Ejercicio 13 — Polimorfismo avanzado: catálogo de medios

**Enunciado:**

Crea una `abstract class Medio` con atributo abstracto `tipo: String`, atributo `titulo: String` y método abstracto `duracion(): String`. Añade el método concreto `ficha(): String` → `"[tipo] | [titulo] | Duración: [duracion]"`.

Implementa:

- `Libro(titulo: String, val paginas: Int)` → tipo = `"Libro"`, duracion = `"~[paginas/25] horas de lectura"`
- `Pelicula(titulo: String, val minutos: Int)` → tipo = `"Película"`, duracion = `"[minutos] min"`
- `Podcast(titulo: String, val episodios: Int, val minsPorEpisodio: Int)` → tipo = `"Podcast"`, duracion = `"[episodios] episodios × [minsPorEpisodio] min"`

Crea una lista heterogénea con al menos 5 medios y muestra la ficha de cada uno.

```scala
abstract class Medio(val titulo: String) {
  val tipo: String
  def duracion(): String
  def ficha(): String = s"$tipo | $titulo | Duración: ${duracion()}"
}

class Libro(titulo: String, val paginas: Int) extends Medio(titulo) {
  val tipo: String = "Libro"
  def duracion(): String = s"~${paginas / 25} horas de lectura"
}

class Pelicula(titulo: String, val minutos: Int) extends Medio(titulo) {
  val tipo: String = "Película"
  def duracion(): String = s"$minutos min"
}

class Podcast(titulo: String, val episodios: Int, val minsPorEpisodio: Int)
    extends Medio(titulo) {
  val tipo: String = "Podcast"
  def duracion(): String = s"$episodios episodios × $minsPorEpisodio min"
}

val biblioteca: List[Medio] = List(
  new Libro("Scala para principiantes", 350),
  new Libro("Clean Code", 450),
  new Pelicula("Blade Runner 2049", 164),
  new Pelicula("Interstellar", 169),
  new Podcast("Data Engineering Weekly", 38, 45)
)

println("=== Catálogo de medios ===")
for (m <- biblioteca) println(m.ficha())
```

**Salida esperada:**

```
=== Catálogo de medios ===
Libro | Scala para principiantes | Duración: ~14 horas de lectura
Libro | Clean Code | Duración: ~18 horas de lectura
Película | Blade Runner 2049 | Duración: 164 min
Película | Interstellar | Duración: 169 min
Podcast | Data Engineering Weekly | Duración: 38 episodios × 45 min
```

---

## 🔹 Ejercicio 14 — Mixins con lógica: sistema de notificaciones

**Enunciado:**

Define tres traits de canal de notificación, cada uno con un método concreto:

- `CanalEmail` → `notificarEmail(msg: String): Unit` imprime `"[EMAIL] [msg]"`
- `CanalSMS` → `notificarSMS(msg: String): Unit` imprime `"[SMS] [msg]"`
- `CanalPush` → `notificarPush(msg: String): Unit` imprime `"[PUSH] [msg]"`

Crea una `abstract class Notificador(val destinatario: String)` con un método abstracto `enviar(msg: String): Unit`.

Implementa:

- `NotificadorBasico` → extiende `Notificador` y mezcla solo `CanalEmail`; `enviar` llama a `notificarEmail`
- `NotificadorCompleto` → extiende `Notificador` y mezcla los tres canales; `enviar` llama a los tres

```scala
trait CanalEmail {
  def notificarEmail(msg: String): Unit = println(s"[EMAIL] $msg")
}

trait CanalSMS {
  def notificarSMS(msg: String): Unit = println(s"[SMS] $msg")
}

trait CanalPush {
  def notificarPush(msg: String): Unit = println(s"[PUSH] $msg")
}

abstract class Notificador(val destinatario: String) {
  def enviar(msg: String): Unit
}

class NotificadorBasico(destinatario: String)
    extends Notificador(destinatario) with CanalEmail {
  def enviar(msg: String): Unit = notificarEmail(s"Para $destinatario: $msg")
}

class NotificadorCompleto(destinatario: String)
    extends Notificador(destinatario) with CanalEmail with CanalSMS with CanalPush {
  def enviar(msg: String): Unit = {
    val texto = s"Para $destinatario: $msg"
    notificarEmail(texto)
    notificarSMS(texto)
    notificarPush(texto)
  }
}

val n1 = new NotificadorBasico("usuario@correo.com")
val n2 = new NotificadorCompleto("Carlos López")

println("=== Notificador básico ===")
n1.enviar("Tu pedido ha sido confirmado")

println("\n=== Notificador completo ===")
n2.enviar("Alerta de seguridad en tu cuenta")
```

**Salida esperada:**

```
=== Notificador básico ===
[EMAIL] Para usuario@correo.com: Tu pedido ha sido confirmado

=== Notificador completo ===
[EMAIL] Para Carlos López: Alerta de seguridad en tu cuenta
[SMS] Para Carlos López: Alerta de seguridad en tu cuenta
[PUSH] Para Carlos López: Alerta de seguridad en tu cuenta
```

---

## 🔹 Ejercicio 15 —  gestor de sensores IoT

**Enunciado:**

Construye un pequeño sistema de sensores para una planta industrial.

**Paso 1 — Define los traits:**

```scala
trait Medible {
  def leer(): Double           // valor actual del sensor
  def unidad(): String         // "°C", "%", "ppm", etc.
}

trait Alertable {
  def umbralMaximo(): Double
  def estaEnAlerta(): Boolean = leer() > umbralMaximo()
  def leer(): Double           // compartido con Medible
}
```

**Paso 2 — Clase abstracta base:**

```scala
abstract class Sensor(val id: String, val ubicacion: String)
    extends Medible with Alertable {

  def informe(): String = {
    val alerta = if (estaEnAlerta()) " ⚠️ ALERTA" else " ✅ OK"
    f"[$id] $ubicacion%-20s | ${leer()}%6.1f ${unidad()}%-5s | Máx: ${umbralMaximo()}%.1f$alerta"
  }
}
```

**Paso 3 — Sensores concretos:**

```scala
class SensorTemperatura(id: String, ubicacion: String, val valorActual: Double)
    extends Sensor(id, ubicacion) {
  def leer(): Double       = valorActual
  def unidad(): String     = "°C"
  def umbralMaximo(): Double = 80.0
}

class SensorHumedad(id: String, ubicacion: String, val valorActual: Double)
    extends Sensor(id, ubicacion) {
  def leer(): Double       = valorActual
  def unidad(): String     = "%"
  def umbralMaximo(): Double = 70.0
}

class SensorCO2(id: String, ubicacion: String, val valorActual: Double)
    extends Sensor(id, ubicacion) {
  def leer(): Double       = valorActual
  def unidad(): String     = "ppm"
  def umbralMaximo(): Double = 1000.0
}
```

**Paso 4 — Panel de control:**

```scala
val sensores: List[Sensor] = List(
  new SensorTemperatura("T-01", "Sala de servidores", 75.3),
  new SensorTemperatura("T-02", "Almacén norte",      92.1),
  new SensorHumedad("H-01",     "Sala limpia",         45.0),
  new SensorHumedad("H-02",     "Laboratorio",         73.5),
  new SensorCO2("C-01",         "Oficina planta 1",   850.0),
  new SensorCO2("C-02",         "Sala de reuniones", 1250.0)
)

println("=== PANEL DE CONTROL — SENSORES IoT ===")
println(f"${"ID"}%6s  ${"Ubicación"}%-20s  ${"Lectura"}%12s  ${"Estado"}%10s")
println("-" * 70)
for (s <- sensores) println(s.informe())

val enAlerta = sensores.filter(_.estaEnAlerta())
println(s"\nSensores en alerta: ${enAlerta.size} de ${sensores.size}")
for (s <- enAlerta) println(s"  ⚠️  ${s.id} — ${s.ubicacion}")
```

**Salida esperada:**

```
=== PANEL DE CONTROL — SENSORES IoT ===
ID      Ubicación              Lectura       Estado
----------------------------------------------------------------------
[T-01] Sala de servidores     |   75.3 °C    | Máx: 80.0 ✅ OK
[T-02] Almacén norte          |   92.1 °C    | Máx: 80.0 ⚠️ ALERTA
[H-01] Sala limpia            |   45.0 %     | Máx: 70.0 ✅ OK
[H-02] Laboratorio            |   73.5 %     | Máx: 70.0 ⚠️ ALERTA
[C-01] Oficina planta 1       |  850.0 ppm   | Máx: 1000.0 ✅ OK
[C-02] Sala de reuniones      | 1250.0 ppm   | Máx: 1000.0 ⚠️ ALERTA

Sensores en alerta: 3 de 6
  ⚠️  T-02 — Almacén norte
  ⚠️  H-02 — Laboratorio
  ⚠️  C-02 — Sala de reuniones
```

---

---