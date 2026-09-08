# ERRORES.md — TravelFast

Revisión del proyecto `TravelFast` contra el enunciado `EjeEva1Enunciado.pdf`.

---

## Críticos (no compila / incorrecto)

### 1. `Main.kt:39` — Error de tipo en la selección de categoría
```kotlin
else -> "Opcion no valida"
```
Se asigna un `String`, pero `categoria` espera un `CategoriaCliente`. Es el error de compilación:
`Argument type mismatch: actual type is 'Comparable<CategoriaCliente & String>', but 'CategoriaCliente' was expected.`

**Solución:** manejar la opción inválida con reintento, no con una cadena de texto.

### 2. `CategoriaCliente.kt` — Categoría mal escrita
```kotlin
REGLUAR("Cliente regluar")
```
Falta la "a": debería ser `REGULAR` (y la descripción `"Cliente regular"`). Este error se propaga a todo el código (`CategoriaCliente.REGLUAR` en `Main.kt:36` y `Reserva.kt:16`).

---

## Requisitos del enunciado no cumplidos

### 3. `EstadoReserva.kt` — Estados de la clase sellada
El enunciado exige: `Pendiente`, `Confirmando`, `Emitido`, `Cancelado(motivo)`.
- El código usa `Confirmador` (debería ser **`Confirmando`**).
- `Cancelado` está definido pero nunca se utiliza en el flujo.

### 4. `Main.kt` — `procesarReserva` con delays
El enunciado pide `delay(3000)` en **cada transición** de estado. Solo hay 2 delays para 3 estados, y no se usa `Cancelado`.

### 5. `Main.kt` — Catálogo de pasajes distinto al enunciado
El enunciado pide:
- Aéreo "Santiago → Buenos Aires", $120.000, económica.
- Aéreo "Santiago → Miami", $450.000, ejecutiva.
- Terrestre "Santiago → Valparaíso", $8.000, VIP = false.
- Terrestre "Santiago → Mendoza", $25.000, VIP = true.

El código usa **Santiago→Tokio** y **Santiago→Puerto Varas** con otros precios.

---

## Lógica incorrecta

### 6. `Reserva.kt:17` — Descuento REGULAR
```kotlin
descuento = subTotal() * 0.5
```
El enunciado dice **5%** (0.05). Actualmente aplica un 50%.

---

## Prácticas / menores

### 7. `Cliente.kt` — Nombre del parámetro
El enunciado define `Cliente(nombre, categoria)`. El código usa `categoriaCliente`. No es un error, pero se aleja de la firma del enunciado.

### 8. Nombres de métodos
El enunciado usa `subtotal()`, `cargosAdministrativos()`. El código usa `subTotal()` y `cargoAdministrativos()` (singular). Difieren del enunciado.

### 9. `EstadoReserva.kt` — Método de manejo de estado
`manejoEstadoPasaje(estado)` recibe el estado como parámetro de una función definida en la propia clase sellada. Es redundante (el estado ya es el `this`).

### 10. `Main.kt:8` — Import innecesario
`import java.lang.IO.readln` no es necesario; `readln` ya existe en la stdlib de Kotlin.
