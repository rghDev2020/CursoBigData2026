# 💻Clase 13 -

---

# Agenda:

<aside>
💡

#### 9:00 - 9:50    →  Sesión 1  - Arquitectura de Spark

#### 9:50 - 11:20   → Ejercicios y caso de uso

#### 11:40 - 12:40  → Sesión 2 - RDD

#### 12:40 - 14:00  → Ejercicios y caso de uso

</aside>

# Sesión 1 : Arquitectura de Spark

---

<aside>

## 🖥️ Antes de empezar: la Spark Shell desde PowerShell

> Esta sección es para los estudiantes que ejecutan Spark directamente desde PowerShell en Windows, sin notebook. Si trabajas con Jupyter en VSCode, puedes saltar al bloque de teoría.
> 

### ¿Qué es la Spark Shell?

La **Spark Shell** (`spark-shell`) es una consola interactiva de Scala que arranca con Spark ya integrado. No necesitas crear ni configurar nada: en cuanto entra, tienes disponibles automáticamente dos objetos listos para usar:

| Variable | Tipo | Para qué sirve |
| --- | --- | --- |
| `spark` | `SparkSession` | Trabajar con DataFrames, SQL |
| `sc` | `SparkContext` | Trabajar con RDDs |

Es un entorno **REPL** (Read-Eval-Print Loop): escribe código → pulsa Enter → Scala lo evalúa y muestra el resultado inmediatamente. Es ideal para explorar datos y probar código rápidamente.

### 1. Arrancar la Spark Shell

Abre **PowerShell** y escribe:

```powershell
spark-shell
```

Si el comando no se reconoce, usa la ruta completa donde está instalado Spark:

```powershell
C:\spark\bin\spark-shell
```

Para arrancar con todos los cores de la máquina disponibles (recomendado en clase):

```powershell
spark-shell --master local[*]
```

Verás una salida larga con logs de inicio. Es normal. Al final aparece el **prompt de Scala**:

```bash
Welcome to
      ____              __
     / __/__  ___ _____/ /__
    _\ \/ _ \/ _ `/ __/  '_/
   /___/ .__/\_,_/_/ /_/\_\   version 4.1.1
      /_/

Using Scala version 2.13.x
...
scala>
```

Cuando ves `scala>` significa que Spark está activo y esperando código.

### 2. Verificar que todo funciona

Escribe directamente en el prompt (sin crear nada, `spark` y `sc` ya existen):

```scala
spark.version
```

Respuesta esperada:

```
res0: String = 4.1.1
```

```scala
sc.master
```

Respuesta esperada:

```
res1: String = local[*]
```

```scala
sc.defaultParallelism
```

Respuesta esperada (depende de los cores de tu máquina):

```
res2: Int = 8
```

> ⚠️ **Error frecuente:** si intentas escribir `val spark = SparkSession.builder()...` en la shell, obtendrás un error porque `spark` ya está definido. En la Spark Shell **nunca** hay que crear la sesión manualmente.
> 

### 3. Escribir código: línea a línea vs. bloques multilínea

**Una sola línea** — escríbela y pulsa Enter:

```scala
val x = 10 * 5
```

Respuesta inmediata:

```
x: Int = 50
```

**Un bloque de varias líneas** — usa el modo `:paste`:

1. Escribe `:paste` y pulsa Enter.
2. Pega o escribe el bloque completo.
3. Pulsa `Ctrl+D` para ejecutarlo.

```
scala> :paste
// Entering paste mode (ctrl-D to finish)

val numeros = List(1, 2, 3, 4, 5)
val dobles  = numeros.map(_ * 2)
println(dobles)

// Ctrl+D aquí
```

Resultado:

```
List(2, 4, 6, 8, 10)
```

> 💡 **Consejo:** para los ejercicios de clase, siempre usa `:paste` cuando el código tiene más de una línea. Si intentas pegar varias líneas sin `:paste`, Scala puede interpretar cada línea por separado y dar errores.
> 

### 4. Comandos esenciales de la Spark Shell

| Comando | Qué hace |
| --- | --- |
| `:paste` | Entra en modo multilínea. Termina con `Ctrl+D` |
| `:help` | Muestra todos los comandos disponibles |
| `:history` | Muestra el historial de comandos ejecutados |
| `:type <expr>` | Muestra el tipo de una expresión sin ejecutarla |
| `:reset` | Limpia todas las variables y definiciones de la sesión |
| `:quit` | Sale de la Spark Shell correctamente |
| `Ctrl+C` | Interrumpe una operación que está en curso (no sale de la shell) |
| `Ctrl+D` | Sale de la shell (equivalente a `:quit`) |

### 5. Los logs: qué son y cómo reducirlos

Al ejecutar operaciones verás mensajes como este mezclados con tus resultados:

```
26/04/21 10:23:44 INFO DAGScheduler: Job 0 finished: count at <console>:1, took 0.452 s
```

Esto **no es un error**. Son logs informativos de Spark que muestran lo que está haciendo internamente. Para reducirlos y que no ensucien la salida, ejecuta esto al inicio de la sesión (con `:paste`):

```scala
import org.apache.log4j.{Level, Logger}
Logger.getLogger("org").setLevel(Level.WARN)
Logger.getLogger("akka").setLevel(Level.WARN)
```

A partir de ese momento solo verás mensajes de tipo `WARN` o `ERROR`, que sí son relevantes.

### 6. Salir de la Spark Shell

Cuando termines la sesión, sal siempre de forma limpia para que Spark libere los recursos correctamente:

```scala
:quit
```

O simplemente pulsa `Ctrl+D`. Verás el mensaje:

```
Spark session stopped.
```

### 7. Diferencias a tener en cuenta respecto al notebook

Si un compañero trabaja con Jupyter y tú con la Spark Shell, el **código Spark es idéntico**. La única diferencia práctica es:

| Aspecto | Jupyter + Almond (VSCode) | Spark Shell (PowerShell) |
| --- | --- | --- |
| Crear `SparkSession` | Sí, manualmente con `$ivy` | Ya existe como `spark` |
| `SparkContext` | `val sc = spark.sparkContext` | Ya existe como `sc` |
| Bloques multilínea | Una celda completa | `:paste` + `Ctrl+D` |
| Guardar el trabajo | Archivo `.ipynb` | Copiar manualmente a un `.scala` |
| Logs | Filtrables con `Logger` | Mezclados en consola (filtrar con `Logger`) |
</aside>

---

## 🧠 Teoría — Arquitectura de Spark

![image.png](image.png)

![image.png](image%201.png)

![image.png](image%202.png)

![image.png](image%203.png)

![image.png](image%204.png)

![image.png](image%205.png)

![image.png](image%206.png)

![image.png](image%207.png)

![image.png](image%208.png)

![image.png](image%209.png)

![image.png](image%2010.png)

![image.png](image%2011.png)

![image.png](image%2012.png)

---

### Resumen de la arquitectura

```
Tu código Scala
       │
       ▼
  SparkSession (Driver)
       │
       ├──► DAG Scheduler → Jobs → Stages → Tasks
       │
       ├──► Cluster Manager (pide recursos)
       │
       └──► Executors (ejecutan las Tasks sobre las particiones)
                │
                └──► Datos (CSV, Parquet, memoria...)
```

---

## 💻 Práctica

---

### 🔹 Ejercicio 1 — Arrancar Spark y verificar la configuración

**Objetivo:** confirmar que Spark arranca correctamente y explorar la configuración del entorno.

---

> 📓 **Jupyter (VSCode):** ejecuta las celdas de inicialización habituales (`$ivy` + `SparkSession.builder`). Una vez arrancado, `spark` y `sc` están disponibles igual que en la shell.
> 
> 
> 💻 **Spark Shell (PowerShell):** arranca con `spark-shell --master local[*]`. Cuando aparezca `scala>`, `spark` y `sc` ya existen. Usa `:paste` para bloques multilínea.
> 

---

**Reduce los logs antes de continuar** (aplica en ambos entornos, con `:paste` en la shell):

```scala
import org.apache.log4j.{Level, Logger}
Logger.getLogger("org").setLevel(Level.WARN)
Logger.getLogger("akka").setLevel(Level.WARN)
```

**Verifica la configuración:**

```scala
println(s"✅ Spark versión:       ${spark.version}")
println(s"✅ Scala versión:       ${scala.util.Properties.versionString}")
println(s"✅ Cores disponibles:   ${sc.defaultParallelism}")
println(s"✅ App Name:            ${sc.appName}")
println(s"✅ Master:              ${sc.master}")
println(s"🌐 Spark UI:            http://localhost:4040")
```

---

**Salida esperada (en ambos entornos):**

```
✅ Spark versión: 4.1.1
✅ Scala versión: Scala library version 2.13.x
✅ Cores disponibles: 8   ← (depende de tu máquina)
✅ App Name: Dia11-Arquitectura
✅ Master: local[*]
🌐 Spark UI: http://localhost:4040
```

> 🎯 **Comprueba:** abre `http://localhost:4040` en el navegador. Deberías ver la Spark UI con la pestaña **Jobs** vacía por ahora. No cierres esa pestaña: la usaremos durante los ejercicios.
> 

---

### 🔹 Ejercicio 2 — Generar Jobs y verlos en la Spark UI

---

**Paso 1 — Crear datos:**

```scala
// Creamos un RDD con números del 1 al 1.000.000
val numeros = sc.parallelize(1 to 1000000)
println(s"Particiones creadas: ${numeros.getNumPartitions}")
```

Salida esperada:

```
Particiones creadas: 8   ← (igual al número de cores)
```

---

**Paso 2 — Primera acción: count()**

```scala
// Esta acción dispara el primer Job
val total = numeros.count()
println(s"Total elementos: $total")
```

> 🔍 **Ve a la Spark UI ahora.** En la pestaña **Jobs** verás que ha aparecido un job nuevo llamado `count`. Haz clic en él para ver los **Stages** y las **Tasks**. Anota: ¿cuántos stages tiene? ¿Cuántas tasks?
> 

---

**Paso 3 — Segunda acción: filter + count (dos transformaciones, un job)**

```scala
// Transformación 1: filtrar pares (lazy, no ejecuta)
val pares = numeros.filter(_ % 2 == 0)

// Transformación 2: multiplicar por 3 (lazy, no ejecuta)
val paresPorTres = pares.map(_ * 3)

// ACCIÓN: ahora sí ejecuta todo el plan
val resultado = paresPorTres.count()
println(s"Cantidad de (pares × 3): $resultado")
```

> 🔍 **Vuelve a la Spark UI.** Ha aparecido un segundo job. Compara su estructura con el anterior: ¿tiene más stages? ¿Por qué?
> 

---

**Paso 4 — Tercera acción: reducir a un único valor**

```scala
// Suma de todos los números pares multiplicados por 3
val suma = paresPorTres.reduce(_ + _)
println(s"Suma total: $suma")
```

> 🔍 **En la Spark UI** este job tendrá 2 stages. ¿Por qué? Porque `reduce` necesita un **shuffle** parcial: primero cada executor suma su parte (Stage 1) y luego el Driver combina los resultados parciales (Stage 2).
> 

---

### 🔹 Ejercicio 3 — Visualizar el DAG

---

**Paso 1 — Crear una cadena de transformaciones:**

```scala
val ventas = sc.parallelize(List(
  ("Madrid", 1200.0),
  ("Barcelona", 800.0),
  ("Madrid", 950.0),
  ("Valencia", 600.0),
  ("Barcelona", 1100.0),
  ("Madrid", 700.0),
  ("Valencia", 850.0),
  ("Barcelona", 300.0)
))

// Transformaciones (ninguna ejecuta todavía)
val ventasMadrid    = ventas.filter(_._1 == "Madrid")
val importesMadrid  = ventasMadrid.map(_._2)
val totalMadrid     = importesMadrid.reduce(_ + _)

println(f"Total ventas Madrid: $totalMadrid%.2f €")
```

---

**Paso 2 — Ver el linaje del RDD:**

```scala
// toDebugString muestra el DAG en texto
println(importesMadrid.toDebugString)
```

Salida esperada (aproximada):

```
(8) MappedRDD[3] at map at <console>:1 []
 |  FilteredRDD[2] at filter at <console>:1 []
 |  ParallelCollectionRDD[1] at parallelize at <console>:1 []
```

> Léelo de abajo hacia arriba: primero se paralleliza, luego se filtra, luego se mapea.
> 

---

**Paso 3 — Un ejemplo con shuffle (operación por clave):**

```scala
// groupByKey fuerza un shuffle → límite entre stages
val ventasPorCiudad = ventas.groupByKey()

// toDebugString antes de la acción
println(ventasPorCiudad.toDebugString)
```

Busca en la salida la línea que contiene `ShuffledRDD`. Eso marca el punto donde Spark tiene que reorganizar los datos entre particiones.

```scala
// Ahora la acción
val resumen = ventasPorCiudad.mapValues(importes => importes.sum)
resumen.collect().foreach { case (ciudad, total) =>
  println(f"  $ciudad%-15s → $total%.2f €")
}
```

Salida esperada:

```
  Madrid          → 2850.00 €
  Barcelona       → 2200.00 €
  Valencia        → 1450.00 €
```

> 🔍 **En la Spark UI**, este último job tiene **2 stages** porque `groupByKey` introduce un shuffle. Puedes ver en la pestaña **Stages** cuántos datos se escribieron y leyeron durante el shuffle (columna "Shuffle Write" y "Shuffle Read").
> 

---

### 🔹 Ejercicio 4 — Identificar stages y tasks

Ejecuta el siguiente código y luego responde las preguntas usando la Spark UI:

```scala
val palabras = sc.parallelize(List(
  "spark", "scala", "big", "data", "spark", "es", "rapido",
  "scala", "es", "elegante", "spark", "escala", "bien",
  "big", "data", "necesita", "spark"
))

// Contar frecuencia de cada palabra
val frecuencias = palabras
  .map(palabra => (palabra, 1))
  .reduceByKey(_ + _)
  .sortBy(_._2, ascending = false)

frecuencias.collect().foreach { case (palabra, count) =>
  println(f"  $palabra%-15s: $count veces")
}
```

**Preguntas :**

1. ¿Cuántos **Jobs** se crearon al ejecutar `.collect()`?
2. ¿Cuántos **Stages** tiene el job principal?
3. ¿Cuántas **Tasks** hay en total? ¿Coincide con el número de particiones × stages?
4. ¿En qué stage se produce el **shuffle**? ¿Cómo lo identificas en la UI?
5. En la pestaña **Executors**, ¿cuánta memoria está usando Spark?

---

### 🔧 Sección de troubleshooting

| Problema | Causa probable | Solución |
| --- | --- | --- |
| `http://localhost:4040` no carga | La sesión Spark no está activa | Confirma que el notebook tiene el kernel correcto y que la celda de SparkSession se ejecutó sin errores |
| Puerto 4040 en uso | Otra sesión Spark está corriendo | Spark usará 4041, 4042… Busca el puerto correcto en el log de arranque |
| En Jupyter: `ivy` no descarga las dependencias | Sin conexión o Maven lento | Espera o conecta a internet. La primera descarga puede tardar 2–3 minutos |
| En la shell: logs INFO llenan la pantalla | Nivel de log por defecto | Usa `:paste` y ejecuta: `import org.apache.log4j._; Logger.getRootLogger.setLevel(Level.WARN)` |
| En la shell: `spark-shell` no se reconoce | Spark no está en el PATH | Usa la ruta completa: `C:\spark\bin\spark-shell` |
| En la shell: error al pegar código multilínea | Pegado sin `:paste` | Escribe `:paste`, pega el bloque completo y pulsa `Ctrl+D` |
| Error `already defined` al redefinir `spark` en la shell | `spark` ya existe en la sesión | No redefinas `spark`: ya está disponible automáticamente |
| En Jupyter: `spark` ya definido al reejecutar la celda | La sesión sigue activa | Usa `.getOrCreate()`: si ya existe, la reutiliza sin error |

## 📬 Cómo entregar tu tarea

<aside>

### Si trabajas con Jupyter en VSCode

Guarda el notebook (`.ipynb`) y entrégalo directamente. El notebook ya contiene el código y la salida de cada celda juntos en el mismo fichero.

</aside>

---

<aside>

### Si trabajas con la Spark Shell

La shell no guarda nada automáticamente, así que el flujo de trabajo recomendado es el siguiente:

### Paso 1 — Escribe tu código en un fichero `.scala`

Abre **VSCode**, crea un fichero nuevo y guárdalo con extensión `.scala`, por ejemplo:

```
C:\Curso-Scala\tareas\tarea_dia11.scala
```

Escribe tu código ahí, como si fueran celdas del notebook pero en un fichero de texto plano. Puedes usar comentarios para separar los ejercicios:

```scala
// ─────────────────────────────────────────
// Tarea Día 11 — Arquitectura de Spark
// Nombre: Tu Nombre Aquí
// ─────────────────────────────────────────

// Ejercicio 1
val numeros = sc.parallelize(1 to 1000000)
println(s"Particiones: ${numeros.getNumPartitions}")

// Ejercicio 2
val pares = numeros.filter(_ % 2 == 0)
val resultado = pares.count()
println(s"Pares: $resultado")
```

> 💡 VSCode resalta la sintaxis de Scala si tienes instalada la extensión **Scala (Metals)** o **Scala Syntax**. No es obligatorio, pero ayuda a escribir sin errores.
> 

### Paso 2 — Ejecuta el fichero desde la Spark Shell

Arranca la Spark Shell normalmente:

```powershell
spark-shell --master local[*]
```

Una vez dentro, carga y ejecuta tu fichero con `:load`:

```scala
:load C:/Curso-Scala/tareas/tarea_dia11.scala
```

> ⚠️ **Importante:** usa barras `/` o `\\` en la ruta, no `\` sola. PowerShell y la JVM las interpretan mejor así.
> 

Verás cómo la shell ejecuta el fichero línea a línea y muestra los resultados en pantalla.

### Paso 3 — Guarda la salida en un fichero de log

Para capturar todo lo que aparece en pantalla (tu código + los resultados), arranca la shell así desde PowerShell:

```powershell
spark-shell --master local[*] 2>&1 | Tee-Object -FilePath C:\Curso-Scala\tareas\salida_dia11.txt
```

`Tee-Object` muestra la salida en pantalla **y** la guarda simultáneamente en el fichero `.txt`. Al terminar la sesión con `:quit`, el fichero estará completo.

### Qué entregar

Para cada tarea entrega **dos ficheros**:

| Fichero | Contenido |
| --- | --- |
| `tarea_dia11.scala` | Tu código limpio y comentado |
| `salida_dia11.txt` | La salida de la ejecución capturada con `Tee-Object` |

### Resumen del flujo completo

```
1. Escribe el código en VSCode  →  tarea_dia11.scala
         │
         ▼
2. Arranca la shell con Tee-Object  →  todo queda grabado en salida_dia11.txt
         │
         ▼
3. Ejecuta :load tarea_dia11.scala dentro de la shell
         │
         ▼
4. Comprueba que la salida es correcta  →  :quit
         │
         ▼
5. Entrega los dos ficheros
```

</aside>

> 📌 **Nota de entorno:** recuerda ejecutar `spark.stop()` (Jupyter) o `:quit` (Spark Shell) al terminar la sesión para liberar recursos.
> 

# 🏢 Caso de Estudio Propuesto 1

## LogiTrack S.A.: Primer análisis con Apache Spark

---

## 🏭 La empresa

**LogiTrack S.A.** es una empresa de logística y transporte de mercancías que opera en cinco ciudades españolas: Madrid, Barcelona, Valencia, Sevilla y Bilbao. Cada día sus camiones realizan cientos de entregas y el sistema de gestión registra cada operación en ficheros de log.

El departamento de operaciones lleva meses quejándose del mismo problema: los informes de fin de día tardan **más de dos horas** en generarse porque el sistema actual los calcula en un único ordenador, procesando los registros uno a uno.

El director de tecnología ha tomado una decisión: migrar el procesamiento de datos a **Apache Spark**. Tú eres el técnico que acaba de incorporarse al proyecto y hoy es tu primer día trabajando con el entorno Spark de la empresa.

Tu misión en esta sesión es **verificar que el entorno funciona correctamente** y **explorar cómo Spark organiza el trabajo** antes de empezar a procesar datos reales.

---

## 📋 Datos del escenario

El sistema de LogiTrack genera registros con el siguiente formato:

```
ID_entrega,ciudad_origen,ciudad_destino,estado,duracion_minutos
```

Ejemplos reales del fichero de hoy:

```
E001,Madrid,Barcelona,completada,245
E002,Valencia,Sevilla,completada,312
E003,Barcelona,Bilbao,en_ruta,180
E004,Madrid,Valencia,completada,198
E005,Sevilla,Madrid,completada,275
E006,Bilbao,Barcelona,retrasada,420
E007,Valencia,Madrid,completada,210
E008,Madrid,Sevilla,completada,290
E009,Barcelona,Valencia,en_ruta,155
E010,Madrid,Bilbao,retrasada,480
E011,Sevilla,Barcelona,completada,330
E012,Valencia,Bilbao,completada,265
E013,Madrid,Barcelona,completada,252
E014,Bilbao,Madrid,completada,300
E015,Barcelona,Sevilla,retrasada,395
E016,Madrid,Valencia,completada,185
E017,Sevilla,Bilbao,en_ruta,210
E018,Madrid,Barcelona,completada,238
E019,Valencia,Barcelona,completada,175
E020,Madrid,Sevilla,completada,310
```

---

## 🎯 Tareas a realizar

El caso de estudio está dividido en **cuatro tareas**. Cada una corresponde a un concepto clave dado en clase. Léelas todas antes de empezar.

---

### Tarea 1 — Arrancar el entorno y verificar la arquitectura local

El director técnico quiere confirmar que el entorno Spark está correctamente configurado antes de autorizar el inicio del proyecto.

**Paso 1 — Prepara el fichero de datos.**

Crea el directorio y el fichero de registros de LogiTrack:

```powershell
# En PowerShell
New-Item -ItemType Directory -Force -Path "C:\Curso-Scala\logitrack"

$registros = @"
E001,Madrid,Barcelona,completada,245
E002,Valencia,Sevilla,completada,312
E003,Barcelona,Bilbao,en_ruta,180
E004,Madrid,Valencia,completada,198
E005,Sevilla,Madrid,completada,275
E006,Bilbao,Barcelona,retrasada,420
E007,Valencia,Madrid,completada,210
E008,Madrid,Sevilla,completada,290
E009,Barcelona,Valencia,en_ruta,155
E010,Madrid,Bilbao,retrasada,480
E011,Sevilla,Barcelona,completada,330
E012,Valencia,Bilbao,completada,265
E013,Madrid,Barcelona,completada,252
E014,Bilbao,Madrid,completada,300
E015,Barcelona,Sevilla,retrasada,395
E016,Madrid,Valencia,completada,185
E017,Sevilla,Bilbao,en_ruta,210
E018,Madrid,Barcelona,completada,238
E019,Valencia,Barcelona,completada,175
E020,Madrid,Sevilla,completada,310
"@
$registros | Out-File -FilePath "C:\Curso-Scala\logitrack\entregas_hoy.txt" -Encoding UTF8
```

**Paso 2 — Arranca Spark y genera el informe de configuración.**

Arranca tu entorno habitual (Jupyter o Spark Shell) y ejecuta el siguiente bloque. Producirá un **informe de configuración** que entregarás al director técnico:

```scala
println("=" * 55)
println("  INFORME DE CONFIGURACIÓN — LogiTrack S.A.")
println("=" * 55)
println(s"  Versión Spark:        ${spark.version}")
println(s"  Versión Scala:        ${scala.util.Properties.versionString}")
println(s"  Modo de ejecución:    ${sc.master}")
println(s"  Cores disponibles:    ${sc.defaultParallelism}")
println(s"  Nombre de la app:     ${sc.appName}")
println(s"  Spark UI disponible:  http://localhost:4040")
println("=" * 55)
```

**Salida esperada:**

```
=======================================================
  INFORME DE CONFIGURACIÓN — LogiTrack S.A.
=======================================================
  Versión Spark:        4.1.1
  Versión Scala:        Scala library version 2.13.x
  Modo de ejecución:    local[*]
  Cores disponibles:    8
  Nombre de la app:     Spark shell
  Spark UI disponible:  http://localhost:4040
=======================================================
```

**Preguntas — responde por escrito:**

1. Spark está arrancado en modo `local[*]`. En la arquitectura de Spark, ¿qué componente hace el papel de **Driver** en este momento? ¿Y de **Executor**? ¿Y de **Cluster Manager**?
2. El campo "Cores disponibles" muestra el número de hilos que Spark usará en paralelo. ¿Con qué concepto de la arquitectura se relaciona directamente este número?
3. ¿Qué diferencia habría en la arquitectura si Spark estuviera configurado como `yarn` en lugar de `local[*]`?

---

### Tarea 2 — El DAG en acción: observar cómo Spark planifica el trabajo

El jefe de operaciones quiere saber cuántos registros de entrega hay hoy en el sistema y cuántas están en cada estado. Antes de darte los resultados, necesitas entender **cómo Spark va a ejecutar ese trabajo**.

**Paso 1 — Carga los datos y define las transformaciones (sin ejecutar nada aún).**

```scala
// Carga el fichero de entregas
val entregas = sc.textFile("C:/Curso-Scala/logitrack/entregas_hoy.txt")

// Transformación 1: quedarse solo con las líneas que no estén vacías
val lineasValidas = entregas.filter(_.trim.nonEmpty)

// Transformación 2: extraer el estado de cada entrega (campo índice 3)
val estados = lineasValidas.map(linea => linea.split(",")(3))

// Transformación 3: preparar para contar por estado (pares clave-valor)
val paresEstado = estados.map(estado => (estado, 1))
```

> ⏸️ **Para aquí un momento.** Hasta esta línea, Spark **no ha hecho absolutamente nada**. Solo ha construido el plan. ¿Puedes describir ese plan de memoria antes de continuar?
> 

**Paso 2 — Dispara la primera acción y observa la Spark UI.**

```scala
// ACCIÓN: ahora sí Spark ejecuta todo el plan
val totalEntregas = lineasValidas.count()
println(s"Total de entregas registradas hoy: $totalEntregas")
```

Abre inmediatamente `http://localhost:4040` en el navegador.

**Paso 3 — Dispara una segunda acción sobre el mismo plan.**

```scala
// Contar por estado (requiere agrupar: introduce un shuffle)
val conteoEstados = paresEstado.reduceByKey(_ + _)
conteoEstados.collect().foreach { case (estado, cantidad) =>
  println(f"  $estado%-15s: $cantidad entregas")
}
```

**Salida esperada:**

```
  completada     : 13 entregas
  en_ruta        : 4 entregas
  retrasada      : 3 entregas
```

**Preguntas — responde por escrito:**

1. Cuando ejecutaste `lineasValidas.count()`, ¿cuántos **Jobs** aparecieron en la Spark UI? ¿Por qué ese número y no otro?
2. Cuando ejecutaste `conteoEstados.collect()`, ¿cuántos **Stages** tiene ese job? Explica por qué hay más de uno. ¿En qué punto del plan se produce el **shuffle**?
3. Explica con tus propias palabras qué es la **evaluación perezosa** usando el ejemplo de LogiTrack: ¿qué pasó entre el momento en que definiste `paresEstado` y el momento en que llamaste a `.collect()`?

---

### Tarea 3 — Explorar la Spark UI como herramienta de diagnóstico

El director técnico quiere que documentes lo que ves en la Spark UI después de ejecutar las operaciones anteriores. Esto servirá como referencia para el equipo cuando Spark esté en producción.

**Paso 1 — Ejecuta este bloque adicional para generar más actividad en la UI.**

```scala
// Análisis de duración de entregas
val duraciones = entregas
  .filter(_.trim.nonEmpty)
  .map(_.split(","))
  .filter(_.length == 5)
  .map(campos => campos(4).toInt)

val totalEntregas  = duraciones.count()
val duracionMedia  = duraciones.sum() / totalEntregas
val duracionMaxima = duraciones.max()
val duracionMinima = duraciones.min()

println(s"Entregas analizadas:  $totalEntregas")
println(f"Duración media:       $duracionMedia minutos")
println(s"Duración máxima:      $duracionMaxima minutos")
println(s"Duración mínima:      $duracionMinima minutos")
```

**Salida esperada:**

```
Entregas analizadas:  20
Duración media:       272 minutos
Duración máxima:      480 minutos
Duración mínima:      155 minutos
```

**Paso 2 — Documenta lo que observas en la Spark UI.**

Abre `http://localhost:4040` y rellena la siguiente tabla con lo que ves. Es tu entregable para el director técnico:

| Pestaña de la UI | ¿Qué información encuentras? | ¿Cuántos elementos hay? |
| --- | --- | --- |
| **Jobs** |  |  |
| **Stages** |  |  |
| **Executors** |  |  |
| **Environment** |  |  |

**Preguntas — responde por escrito:**

1. En la pestaña **Jobs**, localiza el job generado por `duraciones.max()`. ¿Cuántos stages tiene? Haz clic en él y describe brevemente lo que ves en el **DAG visual**.
2. En la pestaña **Stages**, busca algún stage que tenga datos en las columnas "Shuffle Write" o "Shuffle Read". ¿Cuál de tus operaciones lo provocó? ¿Por qué esa operación en concreto necesita un shuffle?
3. En la pestaña **Executors**, ¿cuántos executors hay activos? ¿Cuánta memoria está usando el executor? ¿Qué te dice esto sobre cómo Spark está usando los recursos de tu máquina en modo `local[*]`?

---

### Tarea 4 — Razonamiento sobre la arquitectura

Esta tarea es teórica. No requiere ejecutar código, solo razonar sobre lo que has visto en las tareas anteriores. El director técnico de LogiTrack está evaluando si Spark es la herramienta adecuada. Te ha enviado el siguiente correo:

---

> *"He visto que en vuestras pruebas Spark está configurado en modo `local[*]` y que todo corre en un solo ordenador. Eso no tiene ninguna ventaja sobre nuestro sistema actual, ¿no? ¿Para qué queremos Spark si solo lo vamos a usar en una máquina?"*
> 
> 
> *"Además, no entiendo por qué Spark no ejecuta el código inmediatamente cuando se escribe. Si escribes una instrucción, ¿no debería ejecutarse ya? Eso de esperar me parece que solo añade complejidad."*
> 
> — Carlos Mendoza, Director de Tecnología, LogiTrack S.A.
> 

**Redacta una respuesta profesional al correo de Carlos** abordando sus dos preguntas. Tu respuesta debe:

- Explicar el papel del **modo local** en el desarrollo y por qué es diferente al modo en clúster, usando los conceptos de Driver, Executor y Cluster Manager.
- Defender la **evaluación perezosa** con un argumento concreto basado en lo que has observado hoy en la Spark UI (usa el ejemplo de las entregas de LogiTrack).
- Tener un tono profesional pero comprensible para alguien no técnico.
- Tener una extensión de entre 150 y 250 palabras.

---

---

> 📌 **Recuerda:** al terminar, cierra la sesión correctamente con `spark.stop()` (Jupyter) o `:quit` (Spark Shell).
> 

# Sesión 2 - RDDs: Resilient Distributed Datasets

---

![image.png](image%2013.png)

![image.png](image%2014.png)

![image.png](image%2015.png)

![image.png](image%2016.png)

![image.png](image%2017.png)

![image.png](image%2018.png)

![image.png](image%2019.png)

![image.png](image%2020.png)

![image.png](image%2021.png)

![image.png](image%2022.png)

![image.png](image%2023.png)

![image.png](image%2024.png)

![image.png](image%2025.png)

---

### Resumen visual

```
FUENTE DE DATOS
(colección / fichero)
        │
        ▼
   sc.parallelize()
   sc.textFile()
        │
        ▼
┌───────────────┐
│     RDD       │  ← inmutable, distribuido en particiones, con linaje
└───────┬───────┘
        │
        │  Transformación (lazy)  →  nuevo RDD
        │  .map()  .filter()  .flatMap()  .distinct() ...
        │
        ▼
┌───────────────┐
│   RDD nuevo   │  ← todavía no ha ejecutado nada
└───────┬───────┘
        │
        │  Acción (eager)  →  resultado
        │  .count()  .collect()  .take()  .reduce() ...
        │
        ▼
   RESULTADO
(valor / Array / fichero)
```

---

## 💻 Práctica

> 💻 **Spark Shell:** arranca con `spark-shell --master local[*]`. Usa `:paste` para bloques multilínea. Recuerda que `spark` y `sc` ya existen.
> 
> 
> 📓 **Jupyter:** ejecuta primero las celdas de inicialización (`$ivy` + `SparkSession`).
> 

---

### 🔹 Ejercicio 1 — Crear RDDs y explorar sus propiedades

**Paso 1 — RDD desde una colección:**

```scala
// Lista de temperaturas registradas en una semana (°C)
val temperaturas = sc.parallelize(
  List(22.5, 19.0, 25.3, 18.7, 30.1, 27.8, 21.4)
)

println(s"Número de elementos:   ${temperaturas.count()}")
println(s"Número de particiones: ${temperaturas.getNumPartitions}")
println(s"Primer elemento:       ${temperaturas.first()}")
println(s"Máxima temperatura:    ${temperaturas.max()}")
println(s"Mínima temperatura:    ${temperaturas.min()}")
```

Salida esperada:

```
Número de elementos:   7
Número de particiones: 8   ← (depende de tus cores)
Primer elemento:       22.5
Máxima temperatura:    30.1
Mínima temperatura:    18.7
```

---

**Paso 2 — Controlar el número de particiones:**

```scala
// Forzar exactamente 3 particiones
val tempParticionado = sc.parallelize(
  List(22.5, 19.0, 25.3, 18.7, 30.1, 27.8, 21.4),
  numSlices = 3
)

println(s"Particiones: ${tempParticionado.getNumPartitions}")   // 3

// Ver qué hay en cada partición (útil para depurar)
tempParticionado.glom().collect().zipWithIndex.foreach { case (particion, idx) =>
  println(s"  Partición $idx: ${particion.mkString(", ")}")
}
```

Salida esperada (aproximada):

```
Particiones: 3
  Partición 0: 22.5, 19.0
  Partición 1: 25.3, 18.7, 30.1
  Partición 2: 27.8, 21.4
```

> 💡 `glom()` agrupa los elementos de cada partición en un Array. Es una forma muy útil de visualizar cómo Spark ha repartido los datos.
> 

---

**Paso 3 — Crear un fichero de texto y leerlo con `textFile`:**

Primero crea el fichero. En PowerShell o en una celda de terminal:

```powershell
# PowerShell: crear el fichero de datos
$contenido = @"
Madrid,Enero,12500.50
Barcelona,Enero,9800.00
Valencia,Enero,5400.75
Madrid,Febrero,13200.00
Barcelona,Febrero,10500.50
Valencia,Febrero,4900.25
Madrid,Marzo,14800.00
"@
$contenido | Out-File -FilePath "C:\Curso-Scala\datos\ventas_trimestre.txt" -Encoding UTF8
```

Ahora léelo con Spark:

```scala
val ventas = sc.textFile("C:/Curso-Scala/datos/ventas_trimestre.txt")

println(s"Líneas leídas:     ${ventas.count()}")
println(s"Primera línea:     ${ventas.first()}")
println(s"Particiones:       ${ventas.getNumPartitions}")

// Ver las primeras 3 líneas
ventas.take(3).foreach(println)
```

Salida esperada:

```
Líneas leídas:     7
Primera línea:     Madrid,Enero,12500.50
Particiones:       2
Madrid,Enero,12500.50
Barcelona,Enero,9800.00
Valencia,Enero,5400.75
```

---

### 🔹 Ejercicio 2 — Transformaciones básicas

```scala
val ventas = sc.textFile("C:/Curso-Scala/datos/ventas_trimestre.txt")
```

**Transformación 1 — `map`: extraer solo los importes**

```scala
// Cada línea es "Ciudad,Mes,Importe" → extraemos solo el importe
val importes = ventas.map { linea =>
  val campos = linea.split(",")
  campos(2).toDouble
}

println(s"Importes extraídos: ${importes.count()}")
println(s"Total ventas:       ${importes.sum()}")
println(s"Media de ventas:    ${importes.mean()}")
```

Salida esperada:

```
Importes extraídos: 7
Total ventas:       71101.0
Media de ventas:    10157.28...
```

---

**Transformación 2 — `filter`: solo ventas superiores a 10.000 €**

```scala
val ventasAltas = ventas.filter { linea =>
  val importe = linea.split(",")(2).toDouble
  importe > 10000.0
}

println(s"Ventas > 10.000 €: ${ventasAltas.count()}")
ventasAltas.collect().foreach(println)
```

Salida esperada:

```
Ventas > 10.000 €: 4
Madrid,Enero,12500.50
Madrid,Febrero,13200.00
Barcelona,Febrero,10500.50
Madrid,Marzo,14800.00
```

---

**Transformación 3 — `map` sobre el resultado del `filter`**

```scala
// Encadenar: filtramos y luego extraemos la ciudad
val ciudadesConVentasAltas = ventas
  .filter(_.split(",")(2).toDouble > 10000.0)
  .map(_.split(",")(0))

ciudadesConVentasAltas.collect().foreach(println)
```

Salida esperada:

```
Madrid
Madrid
Madrid
Barcelona
```

---

**Transformación 4 — `distinct`: ciudades únicas con ventas altas**

```scala
val ciudadesUnicas = ciudadesConVentasAltas.distinct()
ciudadesUnicas.collect().foreach(println)
```

Salida esperada:

```
Madrid
Barcelona
```

---

**Transformación 5 — `flatMap`: separar cada línea en campos individuales**

```scala
// flatMap: cada línea produce varios elementos
val todosLosCampos = ventas.flatMap(linea => linea.split(","))
println(s"Total campos: ${todosLosCampos.count()}")   // 7 líneas × 3 campos = 21
todosLosCampos.take(6).foreach(println)
```

Salida esperada:

```
Total campos: 21
Madrid
Enero
12500.50
Barcelona
Enero
9800.00
```

---

### 🔹 Ejercicio 3 — Ver el linaje con `toDebugString`

```scala
val rddBase     = sc.textFile("C:/Curso-Scala/datos/ventas_trimestre.txt")
val rddFiltrado = rddBase.filter(_.contains("Madrid"))
val rddMapeado  = rddFiltrado.map(_.split(",")(2).toDouble)

// Muestra el DAG en texto: léelo de abajo hacia arriba
println(rddMapeado.toDebugString)
```

Salida esperada (aproximada):

```
(2) MapPartitionsRDD[3] at map at <console>:1 []
 |  FilteredRDD[2] at filter at <console>:1 []
 |  MapPartitionsRDD[1] at textFile at <console>:1 []
 |  HadoopRDD[0] at textFile at <console>:1 []
```

Léelo de abajo hacia arriba:

1. `HadoopRDD` — lectura del fichero
2. `MapPartitionsRDD` — conversión de líneas (interno de `textFile`)
3. `FilteredRDD` — tu `filter`
4. `MapPartitionsRDD` — tu `map`

> 💡 Cada nivel tiene un número entre corchetes `(2)` que indica el número de particiones en ese punto del linaje.
> 

---

### 🔹 Ejercicio 4 — Persistencia: midiendo el impacto de `cache()`

**Objetivo:** comprobar empíricamente que `cache()` acelera las acciones repetidas.

```scala
import org.apache.spark.storage.StorageLevel

// RDD con procesamiento no trivial
val rddPesado = sc.parallelize(1 to 500000)
  .map(x => x * x)
  .filter(x => x % 3 == 0)
  .map(x => x.toString + "_procesado")

// --- SIN cache ---
val t0 = System.currentTimeMillis()
rddPesado.count()
val t1 = System.currentTimeMillis()
rddPesado.count()
val t2 = System.currentTimeMillis()

println(s"Sin cache — 1ª ejecución: ${t1 - t0} ms")
println(s"Sin cache — 2ª ejecución: ${t2 - t1} ms")

// --- CON cache ---
rddPesado.cache()

val t3 = System.currentTimeMillis()
rddPesado.count()   // esta primera ejecuta y guarda en memoria
val t4 = System.currentTimeMillis()
rddPesado.count()   // esta segunda usa la memoria directamente
val t5 = System.currentTimeMillis()

println(s"Con cache  — 1ª ejecución: ${t4 - t3} ms  ← carga en memoria")
println(s"Con cache  — 2ª ejecución: ${t5 - t4} ms  ← desde memoria ✓")

// Liberar la memoria al terminar
rddPesado.unpersist()
println("Caché liberada.")
```

Salida esperada (los tiempos varían según la máquina):

```
Sin cache — 1ª ejecución: 380 ms
Sin cache — 2ª ejecución: 340 ms
Con cache  — 1ª ejecución: 420 ms  ← carga en memoria
Con cache  — 2ª ejecución:  45 ms  ← desde memoria ✓
Caché liberada.
```

> 🔍 **Comprueba en la Spark UI** (`http://localhost:4040`) la pestaña **Storage**: durante la ejecución con caché verás el RDD listado con su tamaño en memoria. Después de `unpersist()`, desaparece.
> 

---

### 🔧 Troubleshooting

| Problema | Causa probable | Solución |
| --- | --- | --- |
| `textFile` lanza error de ruta | Barras invertidas en Windows | Usa `/` o `\\` en lugar de `\`: `"C:/Curso-Scala/datos/archivo.txt"` |
| `collect()` tarda mucho o falla | RDD demasiado grande para traer al Driver | Usa `take(10)` en vez de `collect()` para previsualizar |
| La 2ª ejecución con `cache()` no es más rápida | El RDD aún no estaba en memoria | La primera acción después de `cache()` es la que carga; la segunda es la rápida |
| `getNumPartitions` devuelve 1 | Fichero muy pequeño o un solo bloque | Normal con datos de clase; en producción los ficheros grandes generan más particiones |
| Error `toDouble` en el `map` | Línea vacía al final del fichero | Añade `.filter(_.trim.nonEmpty)` antes de parsear |

# 🏢 Caso de Estudio Propuesto 2

## MediRed: Análisis de registros hospitalarios con RDDs

---

## 🏥 La empresa

**MediRed** es una red de cinco hospitales públicos distribuidos por la Comunidad de Madrid. Cada hospital registra diariamente sus ingresos de pacientes en ficheros de texto plano que se consolidan por la noche en un servidor central.

El departamento de análisis sanitario necesita responder cada mañana a un conjunto fijo de preguntas operativas: ¿cuántos pacientes ingresaron ayer?, ¿cuál fue la especialidad con más ingresos?, ¿qué hospitales superaron su capacidad media?, ¿cuántos pacientes fueron dados de alta el mismo día?.

Hasta ahora estos informes se generaban con scripts Python que tardaban entre 40 y 90 minutos dependiendo del volumen. El equipo de datos ha decidido migrar el procesamiento a **Apache Spark con Scala** para reducir ese tiempo a menos de cinco minutos.

Tú eres el técnico de datos encargado de desarrollar el primer prototipo. Hoy trabajarás directamente con **RDDs** para procesar el fichero de ingresos de ayer y generar los indicadores que el departamento necesita.

---

## 📋 Datos del escenario

El fichero de ingresos tiene el siguiente formato:

```
ID_paciente,hospital,especialidad,tipo_ingreso,horas_estancia
```

Valores posibles:

- **hospital:** HospitalNorte, HospitalSur, HospitalEste, HospitalOeste, HospitalCentral
- **especialidad:** Cardiologia, Traumatologia, Pediatria, Urgencias, Neurologia, Oncologia
- **tipo_ingreso:** programado, urgente, traslado
- **horas_estancia:** número entero (0 significa alta el mismo día del ingreso)

---

**Paso previo — Crea el fichero de datos.**

Ejecuta esto en PowerShell antes de arrancar Spark:

```powershell
New-Item -ItemType Directory -Force -Path "C:\Curso-Scala\medired"

$ingresos = @"
P001,HospitalNorte,Cardiologia,programado,48
P002,HospitalSur,Urgencias,urgente,0
P003,HospitalCentral,Pediatria,programado,24
P004,HospitalEste,Traumatologia,urgente,72
P005,HospitalNorte,Neurologia,programado,96
P006,HospitalOeste,Urgencias,urgente,0
P007,HospitalSur,Cardiologia,traslado,120
P008,HospitalCentral,Oncologia,programado,168
P009,HospitalEste,Urgencias,urgente,0
P010,HospitalNorte,Traumatologia,urgente,36
P011,HospitalSur,Pediatria,programado,48
P012,HospitalOeste,Neurologia,programado,72
P013,HospitalCentral,Urgencias,urgente,0
P014,HospitalNorte,Cardiologia,programado,60
P015,HospitalEste,Oncologia,traslado,144
P016,HospitalSur,Urgencias,urgente,6
P017,HospitalOeste,Traumatologia,urgente,48
P018,HospitalCentral,Cardiologia,programado,84
P019,HospitalNorte,Pediatria,programado,24
P020,HospitalEste,Neurologia,programado,96
P021,HospitalSur,Traumatologia,urgente,12
P022,HospitalOeste,Urgencias,urgente,0
P023,HospitalCentral,Neurologia,programado,120
P024,HospitalNorte,Urgencias,urgente,3
P025,HospitalEste,Cardiologia,programado,72
P026,HospitalSur,Oncologia,traslado,200
P027,HospitalOeste,Pediatria,programado,36
P028,HospitalCentral,Traumatologia,urgente,24
P029,HospitalNorte,Oncologia,programado,150
P030,HospitalEste,Urgencias,urgente,0
"@
$ingresos | Out-File -FilePath "C:\Curso-Scala\medired\ingresos_ayer.txt" -Encoding UTF8
```

---

## 🎯 Tareas a realizar

El caso de estudio tiene **cuatro tareas**. Cada una corresponde a un bloque conceptual de la sesión. Lee el enunciado completo de cada tarea antes de escribir código.

---

### Tarea 1 — Crear el RDD base y explorar sus propiedades

El primer paso de cualquier análisis con Spark es cargar los datos y entender con qué estructura estás trabajando. Antes de calcular nada, el equipo necesita saber que los datos se han cargado correctamente y cómo los ha distribuido Spark internamente.

**Paso 1 — Carga el fichero y explora el RDD.**

```scala
// Cargar el fichero de ingresos
val ingresos = sc.textFile("C:/Curso-Scala/medired/ingresos_ayer.txt")

// Exploración básica
println(s"Total de registros cargados: ${ingresos.count()}")
println(s"Número de particiones:       ${ingresos.getNumPartitions}")
println(s"Primer registro:             ${ingresos.first()}")
println("--- Primeros 5 registros ---")
ingresos.take(5).foreach(println)
```

Salida esperada:

```
Total de registros cargados: 30
Número de particiones:       2
Primer registro:             P001,HospitalNorte,Cardiologia,programado,48
--- Primeros 5 registros ---
P001,HospitalNorte,Cardiologia,programado,48
P002,HospitalSur,Urgencias,urgente,0
P003,HospitalCentral,Pediatria,programado,24
P004,HospitalEste,Traumatologia,urgente,72
P005,HospitalNorte,Neurologia,programado,96
```

---

**Paso 2 — Visualiza cómo Spark ha repartido los datos entre particiones.**

```scala
// Ver el contenido de cada partición
ingresos.glom().collect().zipWithIndex.foreach { case (particion, idx) =>
  println(s"Partición $idx → ${particion.length} registros")
  println(s"  Primero: ${particion.head}")
  println(s"  Último:  ${particion.last}")
}
```

Salida esperada (aproximada):

```
Partición 0 → 15 registros
  Primero: P001,HospitalNorte,Cardiologia,programado,48
  Último:  P015,HospitalEste,Oncologia,traslado,144
Partición 1 → 15 registros
  Primero: P016,HospitalSur,Urgencias,urgente,6
  Último:  P030,HospitalEste,Urgencias,urgente,0
```

---

**Paso 3 — Fuerza un número diferente de particiones y compara.**

```scala
// Mismo fichero, pero con 4 particiones explícitas
val ingresosConParticiones = sc.textFile(
  "C:/Curso-Scala/medired/ingresos_ayer.txt",
  minPartitions = 4
)
println(s"Particiones con minPartitions=4: ${ingresosConParticiones.getNumPartitions}")
```

**Preguntas — responde por escrito:**

1. El RDD `ingresos` cargó los datos en 2 particiones. ¿Qué significa eso en términos de cómo Spark distribuiría el trabajo si estuvieras en un clúster real con varios nodos?
2. Explica con tus palabras qué es el **linaje** de `ingresos`. Si una partición se perdiera durante el procesamiento, ¿qué haría Spark para recuperarla?
3. ¿Por qué `ingresos.count()` es una **acción** y `ingresos.filter(...)` sería una **transformación**? ¿Qué devuelve cada una?

---

### Tarea 2 — Transformaciones encadenadas: construir los indicadores del informe

El departamento de análisis necesita exactamente cuatro indicadores para su informe matutino. Construye cada uno aplicando transformaciones sobre el RDD base. Recuerda: **ninguna transformación ejecuta hasta que llames a una acción**.

---

**Indicador 1 — Total de ingresos urgentes.**

```scala
// Transformación: filtrar solo los urgentes
val ingresosUrgentes = ingresos.filter { linea =>
  linea.split(",")(3) == "urgente"
}

// Acción: contar
val totalUrgentes = ingresosUrgentes.count()
println(s"Ingresos urgentes: $totalUrgentes")
```

Salida esperada:

```
Ingresos urgentes: 14
```

---

**Indicador 2 — Pacientes dados de alta el mismo día (horas_estancia == 0).**

```scala
// Transformación: filtrar estancias de 0 horas
val altaMismoDia = ingresos
  .filter(_.trim.nonEmpty)
  .filter { linea =>
    val campos = linea.split(",")
    campos(4).toInt == 0
  }

// Acción: contar
println(s"Altas el mismo día: ${altaMismoDia.count()}")

// Acción adicional: ver los IDs de esos pacientes
val idsPacientesAlta = altaMismoDia.map(_.split(",")(0))
println("IDs de pacientes con alta el mismo día:")
idsPacientesAlta.collect().foreach(id => print(s"$id "))
println()
```

Salida esperada:

```
Altas el mismo día: 7
IDs de pacientes con alta el mismo día:
P002 P006 P009 P013 P022 P024 P030
```

---

**Indicador 3 — Especialidades presentes en el fichero (sin duplicados).**

```scala
// map extrae la especialidad de cada línea
// distinct elimina las repetidas
val especialidades = ingresos
  .map(_.split(",")(2))
  .distinct()

println(s"Número de especialidades: ${especialidades.count()}")
println("Especialidades:")
especialidades.collect().sorted.foreach(e => println(s"  - $e"))
```

Salida esperada:

```
Número de especialidades: 6
Especialidades:
  - Cardiologia
  - Neurologia
  - Oncologia
  - Pediatria
  - Traumatologia
  - Urgencias
```

---

**Indicador 4 — Lista de hospitales con ingresos de traslado.**

```scala
val hospitalesConTraslado = ingresos
  .filter(_.split(",")(3) == "traslado")
  .map(_.split(",")(1))
  .distinct()

println("Hospitales con ingresos por traslado:")
hospitalesConTraslado.collect().sorted.foreach(h => println(s"  - $h"))
```

Salida esperada:

```
Hospitales con ingresos por traslado:
  - HospitalEste
  - HospitalNorte
  - HospitalSur
```

---

**Preguntas — responde por escrito:**

1. En el Indicador 2 se llama a `.count()` y luego a `.collect()` sobre `altaMismoDia`. Sin `cache()`, ¿cuántas veces recorre Spark el fichero original para completar esas dos acciones? Explica por qué.
2. `distinct()` introduce un **shuffle** interno. ¿Qué significa eso en términos de movimiento de datos entre particiones? ¿Lo verías reflejado como un stage adicional en la Spark UI?
3. En el Indicador 3, la transformación `.map(_.split(",")(2))` produce un **RDD nuevo**. El RDD `ingresos` original, ¿qué le ha ocurrido? ¿Ha cambiado de alguna forma?

---

### Tarea 3 — Verificar el linaje con `toDebugString`

Antes de presentar el prototipo al departamento, quieres demostrar que entiendes exactamente qué plan de ejecución ha construido Spark para cada indicador. Usa `toDebugString` para inspeccionarlo.

**Paso 1 — Linaje simple (sin shuffle).**

```scala
// RDD con dos transformaciones encadenadas, sin shuffle
val ingresosUrgentesHospital = ingresos
  .filter(_.split(",")(3) == "urgente")
  .map(_.split(",")(1))

println("=== Linaje de ingresosUrgentesHospital ===")
println(ingresosUrgentesHospital.toDebugString)
```

Salida esperada (aproximada):

```
=== Linaje de ingresosUrgentesHospital ===
(2) MapPartitionsRDD[X] at map at <console>:1 []
 |  FilteredRDD[X] at filter at <console>:1 []
 |  MapPartitionsRDD[X] at textFile at <console>:1 []
 |  HadoopRDD[X] at textFile at <console>:1 []
```

---

**Paso 2 — Linaje con shuffle (distinct).**

```scala
val especialidadesUnicas = ingresos
  .map(_.split(",")(2))
  .distinct()

println("=== Linaje de especialidadesUnicas ===")
println(especialidadesUnicas.toDebugString)
```

Busca en la salida la línea que contiene `ShuffledRDD`. Eso marca el punto donde Spark necesita reorganizar los datos entre particiones para eliminar duplicados.

---

**Paso 3 — Construye tú el linaje a mano.**

Sin ejecutar código, escribe en papel o en comentarios cómo sería el `toDebugString` del siguiente RDD:

```scala
val resultado = ingresos
  .filter(_.trim.nonEmpty)
  .filter(_.split(",")(3) == "programado")
  .map(_.split(",")(1))
  .distinct()
```

Luego ejecútalo y comprueba si tu predicción era correcta:

```scala
println(resultado.toDebugString)
```

**Preguntas — responde por escrito:**

1. Cuando lees `toDebugString` de abajo hacia arriba, ¿qué representa cada nivel? ¿Qué indica el número entre paréntesis al principio de cada línea (por ejemplo, `(2)`)?
2. ¿En qué se diferencia el linaje del Paso 1 (sin shuffle) del Paso 2 (con shuffle)? ¿Qué línea marca esa diferencia y qué implica para la ejecución en términos de stages?

---

### Tarea 4 — Persistencia: el mismo RDD, varias consultas

El departamento de análisis ha confirmado que cada mañana ejecutará **al menos cinco consultas diferentes** sobre los datos de ingresos del día anterior. Esto significa que Spark recalcularía el linaje completo —desde la lectura del fichero— cinco veces si no se aplica ninguna estrategia de persistencia.

Tu tarea es demostrar empíricamente que `cache()` mejora el rendimiento en este escenario, y justificar ante el equipo cuándo tiene sentido usarlo.

**Paso 1 — Prepara un RDD de base procesada que se reutilizará.**

```scala
// Este RDD representa los datos ya parseados y limpios,
// listos para cualquier consulta posterior
val ingresosLimpios = ingresos
  .filter(_.trim.nonEmpty)
  .filter(_.split(",").length == 5)
  .map { linea =>
    val c = linea.split(",")
    (c(0), c(1), c(2), c(3), c(4).toInt)
  }

println(s"Registros limpios: ${ingresosLimpios.count()}")
```

Salida esperada:

```
Registros limpios: 30
```

---

**Paso 2 — Mide el tiempo SIN cache (tres consultas consecutivas).**

```scala
val t0 = System.currentTimeMillis()
val q1 = ingresosLimpios.filter(_._4 == "urgente").count()
val t1 = System.currentTimeMillis()

val q2 = ingresosLimpios.filter(_._5 == 0).count()
val t2 = System.currentTimeMillis()

val q3 = ingresosLimpios.map(_._2).distinct().count()
val t3 = System.currentTimeMillis()

println("--- Sin cache ---")
println(s"  Consulta 1 (urgentes):        $q1 → ${t1 - t0} ms")
println(s"  Consulta 2 (alta mismo día):  $q2 → ${t2 - t1} ms")
println(s"  Consulta 3 (hospitales únicos): $q3 → ${t3 - t2} ms")
println(s"  Tiempo total: ${t3 - t0} ms")
```

---

**Paso 3 — Mide el tiempo CON cache (mismas tres consultas).**

```scala
// Activar la persistencia en memoria
ingresosLimpios.cache()

val t4 = System.currentTimeMillis()
val r1 = ingresosLimpios.filter(_._4 == "urgente").count()   // ← carga en memoria
val t5 = System.currentTimeMillis()

val r2 = ingresosLimpios.filter(_._5 == 0).count()           // ← desde memoria
val t6 = System.currentTimeMillis()

val r3 = ingresosLimpios.map(_._2).distinct().count()        // ← desde memoria
val t7 = System.currentTimeMillis()

println("--- Con cache ---")
println(s"  Consulta 1 (urgentes):          $r1 → ${t5 - t4} ms  ← carga en memoria")
println(s"  Consulta 2 (alta mismo día):    $r2 → ${t6 - t5} ms  ← desde memoria")
println(s"  Consulta 3 (hospitales únicos): $r3 → ${t7 - t6} ms  ← desde memoria")
println(s"  Tiempo total: ${t7 - t4} ms")

// Liberar la memoria al terminar
ingresosLimpios.unpersist()
println("Caché liberada.")
```

Salida esperada (los tiempos varían según la máquina):

```
--- Sin cache ---
  Consulta 1 (urgentes):          14 → 320 ms
  Consulta 2 (alta mismo día):    7  → 295 ms
  Consulta 3 (hospitales únicos): 5  → 340 ms
  Tiempo total: 955 ms

--- Con cache ---
  Consulta 1 (urgentes):          14 → 410 ms  ← carga en memoria
  Consulta 2 (alta mismo día):    7  →  35 ms  ← desde memoria
  Consulta 3 (hospitales únicos): 5  →  62 ms  ← desde memoria
  Tiempo total: 507 ms
```

> 🔍 **Comprueba en la Spark UI** (`http://localhost:4040`), pestaña **Storage**: durante la ejecución con caché verás `ingresosLimpios` listado con su tamaño en memoria. Tras `unpersist()`, desaparece.
> 

---

**Preguntas — responde por escrito:**

1. La primera consulta con `cache()` no es más rápida que sin él, sino igual o incluso algo más lenta. ¿Por qué? ¿Qué está haciendo Spark en esa primera ejecución que no hacía antes?
2. El equipo propone aplicar `cache()` a **todos** los RDDs del pipeline, no solo a `ingresosLimpios`. ¿Es eso una buena idea? Argumenta cuándo tiene sentido cachear y cuándo no, usando el concepto de **linaje** para justificarlo.
3. Si los datos de ingresos de MediRed fueran tan grandes que no cupieran en la RAM del servidor, ¿qué nivel de persistencia usarías en lugar de `cache()`? ¿Por qué?

---

> 📌 **Recuerda:** al terminar, cierra la sesión correctamente con `spark.stop()` (Jupyter) o `:quit` (Spark Shell).
>