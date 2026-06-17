# Clase 03 — Funciones y Manejo de Errores

> Escribir código que dure: modular, legible y que no explote


Observa este código. Hace algo simple: calcula el precio final de tres productos aplicando impuesto.

```python
precio_notebook = 320000 * 1.19
precio_mouse = 8500 * 1.19
precio_teclado = 45000 * 1.19
```

Funciona. Pero ahora imagina que la tasa cambia de 19% a 21%. Tienes que modificar tres líneas. Si el programa tiene cincuenta productos, modificas cincuenta líneas. Si te olvidas una, el resultado es silenciosamente incorrecto — no hay error, solo un número mal calculado que nadie detecta. Ese es el problema que las funciones resuelven.

### Funciones: escribir una vez, usar muchas veces

Una función es un bloque de código con nombre que puedes ejecutar cuando quieras, pasándole los datos que necesita y recibiendo un resultado de vuelta. Ya usaste funciones todo el tiempo — `print()`, `len()`, `sum()`, `sorted()` son funciones que alguien más escribió. Ahora vas a escribir las tuyas.

```python
def calcular_precio_final(precio, tasa_impuesto):
    return precio * (1 + tasa_impuesto)
```

`def` indica que estás definiendo una función. Lo que va entre paréntesis son los parámetros, los datos que la función necesita para trabajar. `return` devuelve el resultado al código que la llamó.

Eso define la función pero no la ejecuta. Para ejecutarla, la llamas por su nombre:

```python
precio_notebook = calcular_precio_final(320000, 0.19)
precio_mouse    = calcular_precio_final(8500, 0.19)
precio_teclado  = calcular_precio_final(45000, 0.19)
```

Ahora si la tasa cambia, modificas un solo lugar: la definición de la función. Todo lo demás se actualiza solo.

#### Parámetros con valor por defecto

Puedes definir valores por defecto para los parámetros. Si al llamar la función no pasas ese argumento, Python usa el valor por defecto:

```python
def calcular_precio_final(precio, tasa_impuesto=0.19):
    return precio * (1 + tasa_impuesto)

calcular_precio_final(320000)          # usa 0.19
calcular_precio_final(320000, 0.21)    # usa 0.21
```

Los parámetros con valor por defecto siempre van al final. Y puedes pasar argumentos por nombre, lo que hace el código más legible cuando hay varios:

```python
calcular_precio_final(precio=320000, tasa_impuesto=0.21)
calcular_precio_final(tasa_impuesto=0.21, precio=320000)  # el orden no importa
```

#### Devolver múltiples valores

Una función puede devolver más de un valor separando con comas:

```python
def analizar_ventas(ventas):
    total    = sum(ventas)
    promedio = total / len(ventas)
    maximo   = max(ventas)
    minimo   = min(ventas)
    return total, promedio, maximo, minimo

ventas = [45000, 32000, 67500, 28000, 91000]
total, promedio, maximo, minimo = analizar_ventas(ventas)

print(f"Total:    ${total:,}")
print(f"Promedio: ${promedio:,.2f}")
print(f"Máximo:   ${maximo:,}")
print(f"Mínimo:   ${minimo:,}")
```

#### Funciones que llaman a otras funciones

El mayor beneficio de las funciones aparece cuando empiezas a componerlas. Mira este código sin funciones:

```python
registros = [
    {"nombre": "Notebook", "precio": 320000, "stock": 5},
    {"nombre": "",         "precio": 8500,   "stock": 10},
    {"nombre": "Monitor",  "precio": -1000,  "stock": 2},
]

valor_total = 0
validos = []

for r in registros:
    if r["nombre"] == "" or r["precio"] <= 0 or not isinstance(r["stock"], int):
        continue
    validos.append(r)
    valor_total += r["precio"] * r["stock"]
```

Funciona, pero la lógica de validación y la lógica de cálculo están mezcladas. Si quieres cambiar las reglas de validación, tienes que leer y modificar el bucle completo. Ahora con funciones:

```python
def es_valido(registro):
    if registro["nombre"] == "":
        return False
    if registro["precio"] <= 0:
        return False
    if not isinstance(registro["stock"], int):
        return False
    return True

def calcular_valor(registro):
    return registro["precio"] * registro["stock"]

def procesar_registros(registros):
    validos = []
    valor_total = 0
    for r in registros:
        if es_valido(r):
            validos.append(r)
            valor_total += calcular_valor(r)
    return validos, valor_total
```

Cada función hace una sola cosa. `es_valido` no calcula nada, solo valida. `calcular_valor` no valida nada, solo calcula. `procesar_registros` coordina a las otras dos. Si las reglas de validación cambian, modificas solo `es_valido`. Si la fórmula de valor cambia, modificas solo `calcular_valor`. El resto no se toca.

### Scope: dónde vive cada variable

Prueba ejecutar esto:

```python
def calcular_descuento(precio):
    resultado = precio * 0.85
    return resultado

calcular_descuento(10000)
print(resultado)
```

El programa lanza un `NameError: name 'resultado' is not defined`. Pero acabas de asignarla ¿por qué no existe?Porque `resultado` es una variable local, existe solo dentro de la función donde fue creada. Cuando la función termina, esa variable desaparece. Desde afuera no puedes verla ni usarla.

Esto no es un bug, es una característica. Si todas las variables de todas las funciones vivieran en el mismo espacio, dos funciones que usen el nombre `resultado` se pisarían entre sí. El scope local es lo que permite que cada función sea independiente. Las variables definidas fuera de todas las funciones tienen scope global y son visibles desde cualquier parte del programa. En general conviene evitar depender de ellas dentro de las funciones. Una función predecible recibe todo lo que necesita por parámetros y devuelve todo lo que produce por `return` — sin depender de nada externo.

### Manejo de errores con try/except

Mira qué pasa cuando procesas datos del mundo real:

```python
def calcular_ingreso(registro):
    return float(registro["monto"]) * registro["cantidad"]

registros = [
    {"monto": "45000", "cantidad": 3},
    {"monto": "N/A",   "cantidad": 2},
    {"monto": "87000", "cantidad": 1},
]

for r in registros:
    print(calcular_ingreso(r))
```

El programa procesa el primer registro correctamente. En el segundo, `float("N/A")` lanza un `ValueError` y el programa se detiene. El tercer registro nunca se procesa. En un pipeline real que analiza diez mil registros, un solo valor inesperado detiene todo, eso no es aceptable.

Los errores en Python se llaman excepciones. La estructura `try` / `except` te permite decidir qué hacer cuando ocurre una, en lugar de dejar que el programa termine:

```python
def calcular_ingreso(registro):
    try:
        return float(registro["monto"]) * registro["cantidad"]
    except ValueError:
        return None
```

Python intenta ejecutar el bloque `try`. Si ocurre un `ValueError`, ejecuta el bloque `except` en lugar de terminar. Si no ocurre ninguna excepción, el bloque `except` se ignora.

```python
for r in registros:
    resultado = calcular_ingreso(r)
    if resultado is not None:
        print(f"Ingreso: ${resultado:,}")
    else:
        print(f"Registro inválido: {r}")
```

Ahora el pipeline completo:

```
Ingreso: $135,000
Registro inválido: {'monto': 'N/A', 'cantidad': 2}
Ingreso: $87,000
```

Los tres registros se procesaron. El segundo generó un error que se capturó y manejó sin interrumpir nada.

#### Capturar excepciones específicas

Especificar el tipo de excepción en el `except` es importante. Si atrapas todo sin distinguir, puedes ocultar errores que sí indican un problema real.

```python
def procesar_venta(venta):
    try:
        monto = float(venta["monto"])
        impuesto = monto * 0.19
        return monto + impuesto
    except KeyError:
        print("Error: falta el campo 'monto'")
        return None
    except ValueError:
        print(f"Error: '{venta['monto']}' no es un número válido")
        return None
    except ZeroDivisionError:
        print("Error: división por cero")
        return None
```

Las excepciones más comunes en data analysis son:

- `ValueError`: el valor tiene el tipo correcto pero no el formato esperado — `float("N/A")`
- `KeyError`: la clave no existe en el diccionario — `registro["monto"]` cuando no hay campo `"monto"`
- `TypeError`: los tipos son incompatibles para la operación — `"precio" + 1000`
- `ZeroDivisionError`: división por cero — frecuente al calcular promedios de listas vacías
- `IndexError`: el índice no existe en la lista

#### Lanzar excepciones propias con raise

A veces el dato es técnicamente convertible pero inválido para el negocio. Un monto de `-5000` es un número válido, pero no tiene sentido como valor de venta. Para ese caso existe `raise`: lanzar una excepción manualmente con un mensaje descriptivo.

```python
def procesar_venta(venta):
    try:
        monto = float(venta["monto"])
        if monto <= 0:
            raise ValueError(f"El monto debe ser positivo, se recibió: {monto}")
        return monto * 1.19
    except (KeyError, ValueError) as e:
        return {"error": str(e)}
```

`raise` interrumpe la ejecución y lanza la excepción que le indiques. En este caso, el `except ValueError` más abajo la captura igual que capturaría un `float("N/A")` — los dos son `ValueError`, sin importar de dónde vinieron.

#### Las cláusulas else y finally

`try` / `except` tiene dos cláusulas opcionales:

```python
def leer_precio(valor):
    try:
        precio = float(valor)
    except ValueError:
        print(f"No se pudo convertir '{valor}'")
        return None
    else:
        # Se ejecuta solo si NO hubo excepción
        return precio
    finally:
        # Se ejecuta siempre, haya o no excepción
        print("Procesamiento finalizado")
```

`else` se ejecuta solo cuando el bloque `try` terminó sin errores. `finally` se ejecuta siempre — es útil para cerrar archivos, liberar recursos o registrar que la operación terminó independientemente del resultado.

### Todo junto

Combinando funciones y manejo de errores, el mismo pipeline que antes explotaba con datos imperfectos ahora los procesa de forma robusta:

```python
def limpiar_monto(valor):
    try:
        return float(str(valor).replace(",", "").replace("$", "").strip())
    except ValueError:
        return None

def clasificar_venta(monto):
    if monto < 10000:
        return "pequeña"
    elif monto < 100000:
        return "mediana"
    else:
        return "grande"

def procesar_dataset(registros):
    procesados = []
    rechazados = 0

    for r in registros:
        monto = limpiar_monto(r.get("monto"))

        if monto is None or monto <= 0:
            rechazados += 1
            continue

        procesados.append({
            "id":        r.get("id", "sin_id"),
            "monto":     monto,
            "categoria": clasificar_venta(monto),
            "total":     monto * 1.19,
        })

    return procesados, rechazados

registros = [
    {"id": "V001", "monto": "45000"},
    {"id": "V002", "monto": "$1,250.50"},
    {"id": "V003", "monto": "N/A"},
    {"id": "V004", "monto": "  87000  "},
    {"id": "V005", "monto": "-300"},
    {"id": "V006", "monto": "210000"},
]

procesados, rechazados = procesar_dataset(registros)

print(f"Procesados: {len(procesados)} | Rechazados: {rechazados}")
print()

for p in procesados:
    print(f"{p['id']} | ${p['monto']:>10,.2f} | {p['categoria']:<8} | Total: ${p['total']:,.2f}")
```

Cada función hace una sola cosa. Los errores se manejan cerca de donde ocurren. El pipeline principal se lee de arriba a abajo sin necesidad de comentarios. Ese es el código que puede crecer.

---

## Resumen

| Concepto | Para qué sirve |
|----------|----------------|
| `def` | Definir una función con nombre y parámetros |
| `return` | Devolver un valor desde una función |
| Parámetros por defecto | Hacer que ciertos argumentos sean opcionales |
| Scope local | Las variables de una función no existen fuera de ella |
| `try` / `except` | Manejar errores sin que el programa se detenga |
| `else` (en try) | Ejecutar código solo si no hubo excepción |
| `finally` | Ejecutar código siempre, haya o no excepción |
| `raise` | Lanzar una excepción manualmente con un mensaje propio |
| `ValueError` | El valor no tiene el formato esperado |
| `KeyError` | La clave no existe en el diccionario |
| `TypeError` | Los tipos son incompatibles para la operación |
| `ZeroDivisionError` | División por cero |

---

## Recursos adicionales

- [Python Docs — Defining Functions](https://docs.python.org/3/tutorial/controlflow.html#defining-functions)
- [Python Docs — Errors and Exceptions](https://docs.python.org/3/tutorial/errors.html)
- [Real Python — Python Functions](https://realpython.com/defining-your-own-python-function/)
- [Real Python — Python Exceptions](https://realpython.com/python-exceptions/)

---

## Práctica

→ [Ver ejercicios](./practica/ejercicios.md)

---

*← [Clase 02 — Control de Flujo y Estructuras de Datos](../clase-02/README.md) · [Módulo 1](../README.md) · Clase 04 — Introducción a NumPy →*