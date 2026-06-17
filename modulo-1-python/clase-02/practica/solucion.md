# Soluciones — Clase 02: Control de Flujo y Estructuras de Datos

> Intenta resolver los ejercicios antes de leer esto. La fricción es parte del aprendizaje.


## Ejercicio 1 — Clasificar una transacción 

```python
monto = 45750

if monto < 1000:
    categoria = "micro transacción"
elif monto < 10000:
    categoria = "transacción estándar"
elif monto < 100000:
    categoria = "transacción importante"
else:
    categoria = "transacción de alto valor"

print(f"Monto: ${monto} — Categoría: {categoria}")
```

**Por qué funciona así:** las condiciones se evalúan en orden y Python ejecuta el primer bloque cuya condición sea `True`. Por eso no necesitás escribir `monto >= 1000 and monto < 10000` en el segundo caso — si llegó al `elif`, ya sabés que el `if` anterior fue `False`, lo que garantiza que el monto es mayor o igual a 1.000.



## Ejercicio 2 — Recorrer una lista de precios 

```python
precios = [8500, 32000, 15900, 7200, 54000, 12300]
impuesto = 0.19

total = 0

for precio in precios:
    precio_final = precio * (1 + impuesto)
    total += precio_final
    print(f"Precio base: ${precio} → Precio final: ${precio_final:,.2f}")

print(f"\nTotal general: ${total:,.2f}")
```

**El patrón acumulador:** `total` se inicializa en `0` antes del bucle y se incrementa en cada iteración. Es uno de los patrones más comunes en programación. Siempre que necesites sumar, contar o construir algo a partir de una colección, vas a usar este patrón.


## Ejercicio 3 — Buscar un registro

```python
inventario = {
    "NB-001": {"nombre": "Notebook Pro",       "precio": 320000, "stock": 5},
    "MS-002": {"nombre": "Mouse Inalámbrico",  "precio": 8500,   "stock": 42},
    "MN-003": {"nombre": 'Monitor 24"',        "precio": 210000, "stock": 0},
    "TC-004": {"nombre": "Teclado Mecánico",   "precio": 45000,  "stock": 12},
}

codigo_busqueda = "MN-003"

producto = inventario.get(codigo_busqueda)

if producto:
    print(f"Producto: {producto['nombre']}")
    print(f"Precio:   ${producto['precio']:,}")
    print(f"Stock:    {producto['stock']} unidades")
    if producto["stock"] == 0:
        print("⚠ Sin stock disponible")
else:
    print(f"Producto no encontrado: {codigo_busqueda}")
```

**Por qué `.get()` y no `[]`:** `inventario["XY-999"]` lanza un `KeyError` que interrumpe el programa. `inventario.get("XY-999")` devuelve `None`, que podés evaluar con un `if`. En data analysis, donde los datos incompletos son la norma, `.get()` es siempre la opción más segura.


## Ejercicio 4 — Filtrar con condiciones 

```python
ventas = [23000, 87000, 45000, 102000, 31500, 68000, 49000, 95000, 12000, 74000]
meta = 50000

sobre_meta = []
bajo_meta = []

for venta in ventas:
    if venta >= meta:
        sobre_meta.append(venta)
    else:
        bajo_meta.append(venta)

total_sobre = sum(sobre_meta)
total_bajo = sum(bajo_meta)

print(f"Sobre la meta:  {len(sobre_meta)} ventas — Total: ${total_sobre:,}")
print(f"Bajo la meta:   {len(bajo_meta)} ventas — Total: ${total_bajo:,}")

# Versión alternativa con list comprehensions (equivalente):
sobre_meta = [v for v in ventas if v >= meta]
bajo_meta  = [v for v in ventas if v < meta]
```

**Dos formas válidas:** el bucle `for` con `append` y la list comprehension hacen exactamente lo mismo. La segunda es más compacta y es el estilo que vas a ver en código Python profesional. Ambas son correctas — lo importante es entender qué está pasando en cada caso.


## Ejercicio 5 — Reporte por categoría

```python
productos = [
    {"nombre": "Notebook",    "categoria": "tecnologia", "precio": 320000, "cantidad": 3},
    {"nombre": "Escritorio",  "categoria": "muebles",    "precio": 85000,  "cantidad": 5},
    {"nombre": "Mouse",       "categoria": "tecnologia", "precio": 8500,   "cantidad": 20},
    {"nombre": "Silla",       "categoria": "muebles",    "precio": 45000,  "cantidad": 8},
    {"nombre": "Monitor",     "categoria": "tecnologia", "precio": 210000, "cantidad": 4},
    {"nombre": "Lámpara",     "categoria": "muebles",    "precio": 12000,  "cantidad": 15},
    {"nombre": "Auriculares", "categoria": "tecnologia", "precio": 32000,  "cantidad": 11},
]

totales_por_categoria = {}

for p in productos:
    cat = p["categoria"]

    if cat not in totales_por_categoria:
        totales_por_categoria[cat] = {"cantidad": 0, "ingreso": 0}

    totales_por_categoria[cat]["cantidad"] += 1
    totales_por_categoria[cat]["ingreso"] += p["precio"] * p["cantidad"]

# Reporte
print("REPORTE POR CATEGORÍA")
print("─" * 36)

categoria_ganadora = None
ingreso_maximo = 0

for categoria, datos in totales_por_categoria.items():
    print(f"\n{categoria.upper()}")
    print(f"  Productos: {datos['cantidad']}")
    print(f"  Ingreso:   ${datos['ingreso']:,}")

    if datos["ingreso"] > ingreso_maximo:
        ingreso_maximo = datos["ingreso"]
        categoria_ganadora = categoria

print(f"\nCategoría con mayor ingreso: {categoria_ganadora} (${ingreso_maximo:,})")
```

**El patrón clave:** `if cat not in totales_por_categoria` inicializa la entrada la primera vez que aparece una categoría. A partir de la segunda, esa condición es `False` y simplemente se acumula. Este patrón — inicializar si no existe, acumular si existe — es exactamente lo que `groupby` hace en pandas por debajo.


## Ejercicio 6 — Validar y limpiar datos

```python
registros = [
    {"nombre": "Notebook",    "precio": 320000, "stock": 5},
    {"nombre": "",            "precio": 8500,   "stock": 10},
    {"nombre": "Monitor",     "precio": -210000,"stock": 2},
    {"nombre": "Teclado",     "precio": 45000,  "stock": 3.5},
    {"nombre": "Mouse",       "precio": 8500,   "stock": 0},
    {"nombre": "Auriculares", "precio": 32000,  "stock": 7},
    {"nombre": "Cámara",      "precio": 0,      "stock": 4},
]

validos = []
invalidos = []

for r in registros:
    motivos = []

    if r["nombre"] == "":
        motivos.append("nombre vacío")

    if r["precio"] <= 0:
        motivos.append("precio inválido")

    if not isinstance(r["stock"], int):
        motivos.append("stock no es entero")

    if motivos:
        registro_invalido = r.copy()
        registro_invalido["motivo"] = ", ".join(motivos)
        invalidos.append(registro_invalido)
    else:
        validos.append(r)

print(f"Registros válidos:   {len(validos)}")
print(f"Registros inválidos: {len(invalidos)}")

print("\nRegistros rechazados:")
for r in invalidos:
    nombre = r["nombre"] if r["nombre"] else "(sin nombre)"
    print(f"  {nombre}: {r['motivo']}")
```

**Dos detalles importantes:**

`isinstance(r["stock"], int)` verifica si un valor es de un tipo específico. `3.5` es `float`, no `int`, así que la condición da `False`. Nótese que `isinstance(True, int)` da `True` porque en Python los booleanos heredan de `int` — un caso especial a tener en cuenta.

Acumular los motivos en una lista antes de decidir permite manejar registros con múltiples problemas a la vez, en lugar de detectar solo el primero.


## Ejercicio 7 — Generador de reportes

```python
transacciones = [
    {"id": "TX001", "pais": "MX", "monto": 45000,  "moneda": "MXN"},
    {"id": "TX002", "pais": "AR", "monto": 128000, "moneda": "ARS"},
    {"id": "TX003", "pais": "CO", "monto": 0,      "moneda": "COP"},
    {"id": "TX004", "pais": "MX", "monto": 87000,  "moneda": "MXN"},
    {"id": "TX005", "pais": "AR", "monto": -5000,  "moneda": "ARS"},
    {"id": "TX006", "pais": "PE", "monto": 32000,  "moneda": "PEN"},
    {"id": "TX007", "pais": "CO", "monto": 215000, "moneda": "COP"},
    {"id": "TX008", "pais": "MX", "monto": 61000,  "moneda": "MXN"},
    {"id": "TX009", "pais": "AR", "monto": 94000,  "moneda": "ARS"},
    {"id": "TX010", "pais": "PE", "monto": 17000,  "moneda": "PEN"},
]

validas = []
rechazadas = []
por_pais = {}

for t in transacciones:
    # Clasificar válidas e inválidas
    if t["monto"] <= 0:
        motivo = "monto cero" if t["monto"] == 0 else "monto negativo"
        rechazadas.append({**t, "motivo": motivo})
        continue

    validas.append(t)

    # Agrupar por país
    pais = t["pais"]
    if pais not in por_pais:
        por_pais[pais] = {"cantidad": 0, "total": 0}

    por_pais[pais]["cantidad"] += 1
    por_pais[pais]["total"] += t["monto"]

# Métricas generales
total_general = sum(t["monto"] for t in validas)
promedio = total_general / len(validas) if validas else 0

# Top 3
top_3 = sorted(validas, key=lambda t: t["monto"], reverse=True)[:3]

# Reporte
sep = "═" * 40
print(sep)
print("REPORTE DE TRANSACCIONES")
print(sep)
print(f"Total procesadas:  {len(transacciones)}")
print(f"Válidas:           {len(validas)}")
print(f"Rechazadas:        {len(rechazadas)}")
print(f"Monto total:       ${total_general:,}")
print(f"Promedio:          ${promedio:,.2f}")

print(f"\n{'DESGLOSE POR PAÍS':}")
print("─" * 40)
for pais, datos in sorted(por_pais.items()):
    print(f"  {pais}: {datos['cantidad']} transacciones — ${datos['total']:,}")

print(f"\nTOP 3 TRANSACCIONES")
print("─" * 40)
for i, t in enumerate(top_3, 1):
    print(f"  {i}. {t['id']} ({t['pais']}): ${t['monto']:,}")

if rechazadas:
    print(f"\nTRANSACCIONES RECHAZADAS")
    print("─" * 40)
    for t in rechazadas:
        print(f"  {t['id']}: {t['motivo']} (${t['monto']})")

print(sep)
```

**Tres cosas nuevas en esta solución:**

`continue` salta al inicio de la siguiente iteración sin ejecutar el resto del bloque. Acá se usa para no procesar las transacciones rechazadas en la lógica de agrupación.

`{**t, "motivo": motivo}` crea un diccionario nuevo copiando todos los pares de `t` y agregando la clave `"motivo"`. El operador `**` desempaqueta el diccionario — es una forma compacta de hacer `.copy()` y agregar un campo al mismo tiempo.

`sorted(validas, key=lambda t: t["monto"], reverse=True)` ordena la lista por el campo `"monto"` de mayor a menor. El parámetro `key` recibe una función que le dice a `sorted()` qué valor usar como criterio. `lambda` es una función anónima de una sola expresión — en este caso, *"dado un elemento `t`, usá `t["monto"]` como criterio de ordenamiento"*.



← [Volver a ejercicios](./notebook.ipynb)