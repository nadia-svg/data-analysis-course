# Solución — Clase 04: List Comprehensions y Limpieza de Strings

> ⚠️ **Intentá resolver los ejercicios antes de mirar esto.**

### Ejercicio 1 — Tu primera comprehension
 
```python
precios = [12000, 8500, 34000, 5200, 19900]
 
# Versión 1: con bucle for
precios_con_iva = []
for p in precios:
    precios_con_iva.append(p * 1.19)
 
print(precios_con_iva)
 
# Versión 2: con list comprehension
precios_con_iva = [p * 1.19 for p in precios]
 
print(precios_con_iva)
# Ambas versiones dan: [14280.0, 10115.0, 40460.0, 6188.0, 23681.0]
```
 
**Lo que cambia entre las dos versiones:** el resultado es idéntico. Lo que cambia es cuánto código necesitás escribir y leer para llegar a él. La comprehension reemplaza tres líneas (lista vacía, bucle, `.append()`) por una sola expresión que dice exactamente lo mismo.
 
 
### Ejercicio 2 — Filtrar sin transformar 
 
```python
edades = [22, 35, 28, 41, 19, 33, 27, 45, 24, 38]
 
mayores_30 = [e for e in edades if e > 30]
 
print(mayores_30)
# [35, 41, 33, 45, 38]
```
 
**La diferencia con el ejercicio anterior:** acá la expresión antes del `for` es simplemente `e` — el mismo valor, sin transformar. La comprehension funciona igual sin importar si transformás o no; el `if` decide qué entra a la lista nueva, independientemente de si hay una transformación antes.
 
 
### Ejercicio 3 — Limpiar espacios y símbolos
 
```python
codigos_postales = ["  1642", "1900 ", " 5000 ", "1000", "  3400  "]
 
# Versión con bucle
codigos_limpios = []
for c in codigos_postales:
    codigos_limpios.append(c.strip())
 
# Versión con comprehension
codigos_limpios = [c.strip() for c in codigos_postales]
 
print(codigos_limpios)
# ['1642', '1900', '5000', '1000', '3400']
```
 
**Nota:** `.strip()` no afecta a `"1000"`, que ya estaba limpio — simplemente no encuentra espacios para quitar y devuelve el string igual. Por eso podés aplicarlo a toda la lista sin preocuparte por cuáles ya estaban bien.
 
 
### Ejercicio 4 — Combinar transformación y condición
 
```python
notas = [8.3, 5.5, 6.0, 4.2, 9.1, 5.9, 7.8, 6.5, 3.0, 8.0]
 
aprobados = [f"Aprobado: {n}" for n in notas if n >= 6]
 
for a in aprobados:
    print(a)
 
# Aprobado: 8.3
# Aprobado: 6.0
# Aprobado: 9.1
# Aprobado: 7.8
# Aprobado: 6.5
# Aprobado: 8.0
```
 
**La respuesta a la pregunta:** la transformación siempre va primero, inmediatamente después del corchete de apertura. La condición va al final, después del `for`. El orden es fijo: `[transformación for elemento in lista if condición]`. No podés invertirlo — Python espera esa estructura exacta.
 
 
### Ejercicio 5 — Normalizar categorías
 
```python
categorias_crudas = ["  tecnologia", "MUEBLES", "Tecnologia ", "muebles", " ROPA", "ropa  "]
 
def normalizar_categoria(categoria):
    return categoria.strip().capitalize()
 
categorias_normalizadas = [normalizar_categoria(c) for c in categorias_crudas]
print(categorias_normalizadas)
# ['Tecnologia', 'Muebles', 'Tecnologia', 'Muebles', 'Ropa', 'Ropa']
 
categorias_unicas = list(set(categorias_normalizadas))
print(categorias_unicas)
# ['Tecnologia', 'Muebles', 'Ropa']  (el orden puede variar)
```
 
**Por qué `set()`:** un `set` es una colección que no permite elementos duplicados. Al convertir la lista normalizada en un `set`, Python automáticamente descarta las repeticiones. Como necesitás el resultado como lista (para mantener consistencia con el resto del código), lo volvés a convertir con `list()`. El orden de un set no está garantizado — si necesitás un orden específico, tendrías que ordenarlo después con `sorted()`.
 
 
## Ejercicio 6 — Limpiar precios con formato mixto
 
```python
precios_usa = ["$1,200.50", "$45.00", "N/A", "$3,890.25", "", "$120.00"]
 
def limpiar_precio_usa(valor):
    try:
        limpio = valor.strip().replace("$", "").replace(",", "")
        return float(limpio)
    except ValueError:
        return None
 
precios_procesados = [limpiar_precio_usa(p) for p in precios_usa]
print(precios_procesados)
# [1200.5, 45.0, None, 3890.25, None, 120.0]
 
precios_validos = [p for p in precios_procesados if p is not None]
print(precios_validos)
# [1200.5, 45.0, 3890.25, 120.0]
```
 
**La respuesta a la pregunta de reflexión:** en el artículo, el punto era separador de miles (`"45.000"` significaba 45.000) y había que eliminarlo. Acá la coma es separador de miles y el punto es decimal (`"$1,200.50"` significa 1.200,50). Por eso la limpieza solo elimina `$` y `,`, dejando el punto intacto — porque en este formato, el punto sí es información que `float()` necesita para interpretar los decimales correctamente. La lógica de limpieza no es universal: depende del formato de origen de los datos.
 
 
### Ejercicio 7 — Reporte de clientes
 
```python
emails_crudos = [
    "  Ana.Torres@mail.com",
    "LUIS@MAIL.COM  ",
    "   ",
    "clara_diaz@mail.com",
    "sin-arroba.com",
    "",
    "  Pedro@Mail.Com",
]
 
emails_validos = [
    e.strip().lower()
    for e in emails_crudos
    if e.strip() != "" and "@" in e
]
 
print(emails_validos)
# ['ana.torres@mail.com', 'luis@mail.com', 'clara_diaz@mail.com', 'pedro@mail.com']
```
 
**Dos condiciones combinadas con `and`:** la comprehension filtra con dos chequeos a la vez — que el string no quede vacío después de `.strip()`, y que contenga `"@"`. Ambas tienen que cumplirse para que el email entre a la lista final. Fijate que la transformación (`.strip().lower()`) se aplica solo a los que pasan el filtro, no a todos — eso es exactamente lo que hace el `if` al final de la comprehension.
 
**Sobre `.lower()`:** este método no apareció antes en el artículo, pero es el mismo tipo de herramienta que `.capitalize()` o `.title()` — convierte todo el string a minúsculas. Para emails es la normalización más común, porque las direcciones no distinguen mayúsculas de minúsculas, pero dos strings con distinto casing se tratarían como diferentes si no los normalizás antes de comparar o guardar.
 
---
 
← [Volver a ejercicios](./notebook.ipynb)