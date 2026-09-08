# ESTUDIO_2.md — Código completo TravelFast (Kotlin)

## Estructura del proyecto
```
src/main/kotlin/
├── Main.kt                  ─ main, menú, procesarReserva
├── model/
│   ├── CategoriaCliente.kt  ─ enum
│   ├── Cliente.kt           ─ data class
│   ├── Pasaje.kt            ─ clase base
│   ├── PasajeAereo.kt
│   ├── PasajeTerrestre.kt
│   └── Reserva.kt           ─ cálculos
└── sealed/
    └── EstadoReserva.kt     ─ sealed class
```

## Mapa rápido de llamadas
| Qué creo | Cómo lo instancio | Qué le llamo |
|---|---|---|
| Cliente | `Cliente("Ana", CategoriaCliente.PREMIUM)` | `.nombre`, `.categoria` |
| Pasaje | `PasajeAereo("Santiago","Buenos Aires",120_000,false)` | `.origen`, `.destino`, `.precioBase`, `.ejecutiva`, `.calcularPrecioFinal()` |
| Reserva | `Reserva(cliente, mutableListOf(pasaje1))` | `.subtotal()`, `.descuento()`, `.cargosAdministrativos()`, `.impuesto()`, `.total()` |

---

## 1. CategoriaCliente
```kotlin
enum class CategoriaCliente(val descuento: Double) {
    REGULAR(0.05), FRECUENTE(0.10), PREMIUM(0.20)   // cuidado: REGULAR con "A"
}
```
**Cómo se usa:**
```kotlin
CategoriaCliente.PREMIUM.descuento          // 0.20  (atributo del enum)
CategoriaCliente.values().forEach { it.name }  // "REGULAR", "FRECUENTE", "PREMIUM"
val c = if (opcion == 1) CategoriaCliente.REGULAR else CategoriaCliente.PREMIUM
```
Truco: guardar el % en el enum evita el `when` en `descuento()`.

## 2. Cliente
```kotlin
data class Cliente(val nombre: String, val categoria: CategoriaCliente)
```
**Cómo se usa:**
```kotlin
val cliente = Cliente("María", CategoriaCliente.FRECUENTE)
cliente.nombre        // "María"
cliente.categoria     // CategoriaCliente.FRECUENTE
cliente.categoria.descuento   // 0.10
```
> Ser `data class` te da `equals`, `hashCode` y `copy()` gratis.

## 3. Pasaje (base, polimorfismo)
```kotlin
open class Pasaje(
    val origen: String,
    val destino: String,
    val precioBase: Int
) {
    open fun calcularPrecioFinal(): Int = precioBase
}
```
**Cómo se usa:**
```kotlin
val p = Pasaje("a", "b", 100)
p.origen        // "a"        (atributo)
p.calcularPrecioFinal()   // 100  (método)
```
> `open` = permite heredar. `open fun` = permite reescribir (override).

## 4. PasajeAereo
```kotlin
class PasajeAereo(origen: String, destino: String, precioBase: Int, val ejecutiva: Boolean)
    : Pasaje(origen, destino, precioBase) {
    override fun calcularPrecioFinal(): Int =
        if (ejecutiva) precioBase + 50_000 else precioBase
}
```
**Cómo se usa:**
```kotlin
val economico = PasajeAereo("Santiago", "Buenos Aires", 120_000, false)
val ejecutivo = PasajeAereo("Santiago", "Miami", 450_000, true)
economico.ejecutiva               // false → precio final = 120.000
ejecutivo.ejecutiva               // true  → precio final = 500.000 (450.000 + 50.000)
ejecutivo.origen                  // hereda: "Santiago"
ejecutivo.calcularPrecioFinal()   // polimorfismo: usa la VERSIÓN override
```
> Nota: `origen`, `destino`, `precioBase` no llevan `val` en el constructor de la subclase porque ya los declara la superclase.

## 5. PasajeTerrestre
```kotlin
class PasajeTerrestre(origen: String, destino: String, precioBase: Int, val vip: Boolean)
    : Pasaje(origen, destino, precioBase) {
    override fun calcularPrecioFinal(): Int =
        if (vip) precioBase + 10_000 else precioBase
}
```
**Cómo se usa:**
```kotlin
val noVip = PasajeTerrestre("Santiago", "Valparaíso", 8_000, false)   // final = 8.000
val vips   = PasajeTerrestre("Santiago", "Mendoza", 25_000, true)     // final = 35.000
```

## 6. Reserva — las 5 fórmulas clave
```kotlin
class Reserva(val cliente: Cliente, val pasajes: MutableList<Pasaje>) {

    fun subtotal() = pasajes.sumOf { it.calcularPrecioFinal() }

    fun descuento() = subtotal() * cliente.categoria.descuento   // 5 / 10 / 20%

    fun cargosAdministrativos() = pasajes.size * 2_000            // $2.000 por pasaje

    fun impuesto() = (subtotal() - descuento() + cargosAdministrativos()) * 0.19

    fun total() = subtotal() - descuento() + cargosAdministrativos() + impuesto()
}
```
**Cómo se llama:**
```kotlin
val pasajes = mutableListOf(economico, vips)          // lista mutable
val reserva = Reserva(cliente, pasajes)               // cliente + lista
reserva.subtotal()               // 120.000 + 35.000 = 155.000
reserva.descuento()              // 155.000 × 0.20    = 31.000 (PREMIUM)
reserva.cargosAdministrativos()  // 2 pasajes × 2.000 = 4.000
reserva.impuesto()               // (155.000−31.000+4.000)×0.19 = 24.320
reserva.total()                  // 155.000−31.000+4.000+24.320 = 152.320
```
- `pasajes.sumOf { it.calcularPrecioFinal() }`: `it` = cada pasaje; **polimorfismo** — cada pasaje usa su propio `calcularPrecioFinal()`.
- `pasajes.add(...)` para agregar al menú; `pasajes.count()` = la lista en Reserva.
- El cliente llega a la categoría por el atributo `cliente.categoria.descuento` (encadenado).
- Orden: impuesto va **después** de descuento + cargos. Nunca cambies ese orden.

## 7. EstadoReserva + procesarReserva
```kotlin
sealed class EstadoReserva {
    object Pendiente : EstadoReserva()
    object Confirmando : EstadoReserva()
    object Emitido : EstadoReserva()
    data class Cancelado(val motivo: String) : EstadoReserva()
}
```
**Cómo se usa:**
```kotlin
EstadoReserva.Pendiente                     // object → acceso directo, sin instanciar
EstadoReserva.Cancelado("Sin pago")         // data class → lleva motivo
val e: EstadoReserva = EstadoReserva.Confirmando   // tipo base guarda subtipo
when (e) {                                   // sealed → el when es EXHAUSTIVO
    EstadoReserva.Pendiente   -> println("Pendiente")
    EstadoReserva.Confirmando -> println("Confirmando")
    EstadoReserva.Emitido     -> println("Emitido")
    is EstadoReserva.Cancelado -> println("Cancelado: ${e.motivo}")
}
```
```kotlin
suspend fun procesarReserva(reserva: Reserva) = runBlocking {   // dentro de Main
    val estados = listOf(EstadoReserva.Pendiente, EstadoReserva.Confirmando, EstadoReserva.Emitido)
    estados.forEach { println(it); delay(3_000) }               // delay(3000) en cada transición
}
```

## 8. Main — flujo
```
1. runBlocking
2. Catálogo (4 pasajes EXACTOS del enunciado)
3. leer nombre
4. elegir categoría con validación (reintento si opción inválida → evita el bug de tipos)
5. bucle menú: 1-4 → agregar pasaje; 0 → terminar
6. mostrar boleta (pasajes, precio final, subtotal, descuento, cargos, impuesto, total)
7. procesarReserva
```
```kotlin
val catalogo = listOf(
    PasajeAereo("Santiago", "Buenos Aires", 120_000, false),   // económica
    PasajeAereo("Santiago", "Miami", 450_000, true),           // ejecutiva
    PasajeTerrestre("Santiago", "Valparaíso", 8_000, false),   // no VIP
    PasajeTerrestre("Santiago", "Mendoza", 25_000, true),      // VIP
)
```
**Agregar al menú (ejemplo de llamada):**
```kotlin
val pasajesReservados = mutableListOf<Pasaje>()
pasajesReservados.add(catalogo[0])                       // index: 0..3
val boleta = Reserva(cliente, pasajesReservados)         // se construye al terminar
catalogo.forEachIndexed { i, p ->                        // "1. Santiago - Buenos Aires"
    println("${i + 1}. ${p.origen} - ${p.destino} - ${p.calcularPrecioFinal()}")
}
```

## 9. Ejemplo numérico de control (para el examen)
Cliente PREMIUM (20%), 2 pasajes: 1 aéreo económica ($120.000) + 1 terrestre VIP ($25.000+$10.000=$35.000):

| concepto | valor |
|---|---|
| subtotal | 120.000 + 35.000 = **155.000** |
| descuento (20%) | 155.000 × 0.20 = **31.000** |
| cargos (2×2.000) | **4.000** |
| base imponible | 155.000 − 31.000 + 4.000 = 128.000 |
| impuesto (19%) | 128.000 × 0.19 = **24.320** |
| total | 155.000 − 31.000 + 4.000 + 24.320 = **152.320** |

## 10. Checklist de errores a evitar
- [ ] `REGULAR` (no `REGLUAR`)
- [ ] REGULAR = 5% → `0.05`, no `0.5`
- [ ] `Confirmando` (no `Confirmador`)
- [ ] métodos con nombres del enunciado: `subtotal()`, `cargosAdministrativos()`
- [ ] `Cliente(nombre, categoria)` — parámetro `categoria`
- [ ] sin `import java.lang.IO.readln`
- [ ] catálogo = los 4 pasajes del enunciado (Buenos Aires, Miami, Valparaíso, Mendoza)
- [ ] opción inválida de menú → no romper con `String` en un `CategoriaCliente`
