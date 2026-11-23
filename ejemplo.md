# Consolidado Técnico de Arquitectura y Código Fuente

> **Proyecto:** Sistema de Gestión y Reserva de Aerolínea
> **Tipo de Documento:** Expediente Técnico Consolidado
> **Alcance:** Estructura relativa de directorios y código fuente íntegro del sistema.

---

## 1. Guía de Estructura de Directorios

Jerarquía relativa de directorios y componentes del proyecto (se excluyen entornos virtuales y archivos de persistencia de datos):

```text
.
├── app.py
├── data/
│   └── [Persistencia JSON - Archivos de datos omitidos]
├── lista_pasajeros.py
├── pasajero.py
├── persistencia.py
├── prueba_consola.py
├── requirements.txt
├── revisar.py
├── static/
│   ├── css/
│   │   └── estilos.css
│   └── js/
│       ├── api.js
│       ├── avion.js
│       ├── buscar.js
│       ├── clic.js
│       ├── datos-demo.js
│       ├── eliminar.js
│       ├── listar.js
│       ├── main.js
│       ├── mensajes.js
│       ├── modificar.js
│       ├── pestanas.js
│       ├── progreso.js
│       ├── reservar.js
│       ├── semaforo.js
│       ├── tablero.js
│       ├── tooltip.js
│       ├── tren.js
│       ├── utilidades.js
│       └── vista3d.js
├── templates/
│   └── index.html
├── verificador.py
└── vuelo.py
```

---

## 2. Índice General de Módulos

### Configuración y Dependencias
- [requirements.txt](#archivo-requirements-txt)

### Backend y Lógica de Negocio (Python / Flask)
- [app.py](#archivo-app-py)
- [vuelo.py](#archivo-vuelo-py)
- [pasajero.py](#archivo-pasajero-py)
- [lista_pasajeros.py](#archivo-lista-pasajeros-py)
- [persistencia.py](#archivo-persistencia-py)
- [verificador.py](#archivo-verificador-py)
- [revisar.py](#archivo-revisar-py)
- [prueba_consola.py](#archivo-prueba-consola-py)

### Vistas e Interfaz (HTML)
- [templates/index.html](#archivo-templates-index-html)

### Hojas de Estilo (CSS)
- [static/css/estilos.css](#archivo-static-css-estilos-css)

### Frontend y Módulos de Cliente (JavaScript)
- [static/js/api.js](#archivo-static-js-api-js)
- [static/js/main.js](#archivo-static-js-main-js)
- [static/js/avion.js](#archivo-static-js-avion-js)
- [static/js/reservar.js](#archivo-static-js-reservar-js)
- [static/js/modificar.js](#archivo-static-js-modificar-js)
- [static/js/eliminar.js](#archivo-static-js-eliminar-js)
- [static/js/buscar.js](#archivo-static-js-buscar-js)
- [static/js/listar.js](#archivo-static-js-listar-js)
- [static/js/tablero.js](#archivo-static-js-tablero-js)
- [static/js/progreso.js](#archivo-static-js-progreso-js)
- [static/js/semaforo.js](#archivo-static-js-semaforo-js)
- [static/js/pestanas.js](#archivo-static-js-pestanas-js)
- [static/js/clic.js](#archivo-static-js-clic-js)
- [static/js/tooltip.js](#archivo-static-js-tooltip-js)
- [static/js/tren.js](#archivo-static-js-tren-js)
- [static/js/vista3d.js](#archivo-static-js-vista3d-js)
- [static/js/utilidades.js](#archivo-static-js-utilidades-js)
- [static/js/mensajes.js](#archivo-static-js-mensajes-js)
- [static/js/datos-demo.js](#archivo-static-js-datos-demo-js)

---

## 3. Código Fuente Consolidado

### Categoría: Configuración y Dependencias

<a id="archivo-requirements-txt"></a>
#### Archivo: `requirements.txt`
- **Ruta relativa:** `requirements.txt`
- **Lenguaje:** `text`

```text
flask>=3.0
```

---

### Categoría: Backend y Lógica de Negocio (Python / Flask)

<a id="archivo-app-py"></a>
#### Archivo: `app.py`
- **Ruta relativa:** `app.py`
- **Lenguaje:** `python`

```python
# ==============================================================
#  app.py  ·  El servidor de AeroLista
#  Desde ahora, cada paso agrega sus rutas nuevas ENCIMA de la
#  línea que dice ENCENDER EL SERVIDOR (casi al final del archivo).
# ==============================================================

import traceback                       # 🆕 (paso 28) muestra errores completos en la terminal

# Flask:           crea la aplicación
# jsonify:         🆕 convierte un dict de Python en una respuesta JSON
# render_template: busca un archivo dentro de templates/ y lo entrega
# request:         🆕 (paso 29) permite leer lo que nos manda el navegador
from flask import Flask, jsonify, render_template, request
from werkzeug.exceptions import HTTPException   # 🆕 (paso 28) errores web como "404 no existe"

import verificador                               # 🆕 pruebas automáticas (panel de progreso)
import revisar                                   # 🆕 revisa lo que retornan tus métodos
import persistencia                              # 🆕 guarda y lee el archivo JSON
from lista_pasajeros import ListaPasajeros       # 🆕 la lista enlazada
from pasajero import Pasajero                    # 🆕 la clase Pasajero (la usaremos en el paso 29)
from vuelo import VUELO, asiento_existe, total_asientos   # 🆕 más funciones de vuelo.py

# 🆕 La ÚNICA lista enlazada del programa. Vive en la memoria mientras Flask esté encendido.
lista = ListaPasajeros()
persistencia.cargar(lista)        # lee data/reservas.json y hace ingresar() con cada reserva
print(f"  Reservas cargadas en la lista enlazada: {lista.cantidad}")

app = Flask(__name__)


@app.get("/")
def inicio():
    # 🆕 render_template hace 3 cosas:
    #    1. busca templates/index.html
    #    2. reemplaza cada {{ vuelo.algo }} con el dato de este diccionario
    #       (por ejemplo {{ vuelo.origen }} → "Guayaquil")
    #    3. envía el HTML ya completo al navegador
    return render_template("index.html", vuelo=VUELO)


# ==============================================================
#  RESPUESTAS DE LA API: todas tienen la misma forma
#     { "ok": true/false, "datos": ..., "mensaje": "...", "avisos": [...] }
# ==============================================================
def respuesta_ok(datos=None, mensaje="", avisos=None, codigo=200):
    """Respuesta de ÉXITO. codigo 200 = 'todo bien' (201 = 'creado')."""
    return jsonify({
        "ok": True,                 # salió bien
        "datos": datos,             # lo que pidió el navegador
        "mensaje": mensaje,         # un texto para mostrar al usuario
        "avisos": avisos or [],     # advertencias opcionales (si no hay, lista vacía)
    }), codigo


def respuesta_error(mensaje, codigo=400):
    """Respuesta de ERROR. 400 = datos incorrectos · 404 = no existe."""
    return jsonify({"ok": False, "mensaje": mensaje}), codigo


# ==============================================================
#  API: rutas que responden JSON (las llama el JavaScript)
# ==============================================================
@app.get("/api/vuelo")
def ver_vuelo():
    """Datos del avión: el JavaScript los usa para DIBUJAR filas y asientos."""
    return respuesta_ok(VUELO)      # el diccionario VUELO viaja como JSON

def a_diccionarios(pasajeros):
    """[Pasajero, Pasajero, ...]  →  [dict, dict, ...]  (para poder enviarlos como JSON)."""
    resultado = []
    for pasajero in pasajeros:
        resultado.append(pasajero.a_diccionario())
    return resultado


@app.get("/api/resumen")
def ver_resumen():
    """Tablero: cuántas reservas hay y si la lista está vacía."""
    cantidad = revisar.entero(lista.longitud(), "longitud")            # ✍ TU método
    vacia = revisar.booleano(lista.esta_vacia(), "esta_vacia")         # ✍ TU método
    return respuesta_ok({"cantidad": cantidad, "vacia": vacia, "total_asientos": total_asientos()})


@app.get("/api/reservas")
def listar_reservas():
    """LISTAR todas las reservas, o solo una clase si llega ?clase=Ejecutiva"""
    # request.args son los datos que vienen en la dirección después del ?
    clase = request.args.get("clase", "")          # si no viene, vale ""
    if clase == "":
        pasajeros = revisar.lista_de_pasajeros(lista.listar(), "listar")                    # ✍ TU método
    else:
        pasajeros = revisar.lista_de_pasajeros(lista.filtrar_por_clase(clase), "filtrar_por_clase")   # ✍
    return respuesta_ok(a_diccionarios(pasajeros))


@app.get("/api/reservas/<int:numero>")
def buscar_reserva(numero):
    """BUSCAR por número. <int:numero> toma ese pedazo de la dirección y lo vuelve int:
    /api/reservas/1009  →  numero = 1009"""
    pasajero = revisar.pasajero_o_none(lista.buscar(numero), "buscar")                       # ✍
    if pasajero is None:
        return respuesta_error(f"No existe la reserva #{numero}", 404)    # 404 = no existe
    return respuesta_ok(pasajero.a_diccionario(), f"Reserva #{numero} encontrada")


@app.get("/api/asientos/<asiento>")
def buscar_asiento(asiento):
    """¿Quién está sentado en este asiento? (clic en un asiento rojo)"""
    asiento = asiento.upper()                       # "7d" → "7D"
    pasajero = revisar.pasajero_o_none(lista.buscar_por_asiento(asiento), "buscar_por_asiento")  # ✍
    if pasajero is None:
        return respuesta_ok(None, f"El asiento {asiento} está libre")      # libre NO es un error
    return respuesta_ok(pasajero.a_diccionario(), f"En el {asiento} viaja {pasajero.nombre}")


@app.get("/api/cedulas/<cedula>")
def buscar_cedula(cedula):
    """BUSCAR por cédula."""
    pasajero = revisar.pasajero_o_none(lista.buscar_por_cedula(cedula), "buscar_por_cedula")    # ✍
    if pasajero is None:
        return respuesta_error(f"Ningún pasajero tiene la cédula {cedula}", 404)
    return respuesta_ok(pasajero.a_diccionario(), "Pasajero encontrado")


# ==============================================================
#  ERRORES: en vez de romperse, Flask responde un JSON que explica
#  qué pasó, y la página lo muestra en un mensaje.
# ==============================================================
@app.errorhandler(NotImplementedError)
def metodo_pendiente(error):
    """Un método todavía tiene su línea  raise NotImplementedError(...)"""
    metodo = str(error)                             # "listar"
    return jsonify({"ok": False, "pendiente": True, "metodo": metodo,
                    "mensaje": f"Falta implementar {metodo}() en lista_pasajeros.py"}), 501


@app.errorhandler(revisar.ErrorEnTuMetodo)
def metodo_con_problema(error):
    """Un método retornó algo que no corresponde (lo detectó revisar.py)"""
    return jsonify({"ok": False, "error_codigo": True, "mensaje": str(error)}), 500


@app.errorhandler(Exception)
def error_inesperado(error):
    """Cualquier otro error de Python (por ejemplo AttributeError en tu código)"""
    if isinstance(error, HTTPException):            # 404, 405...: rutas que no existen
        return jsonify({"ok": False, "mensaje": f"{error.code} {error.name}"}), error.code
    traceback.print_exc()                           # el detalle completo sale en la terminal
    return jsonify({"ok": False, "error_codigo": True,
                    "mensaje": f"Error en el código Python → {type(error).__name__}: {error}"}), 500

def leer_formulario():
    """Saca los datos que mandó el JavaScript (en formato JSON)."""
    datos = request.get_json(silent=True) or {}     # el cuerpo del pedido como dict (o {} si viene vacío)
    nombre = str(datos.get("nombre", "")).strip()   # .get(clave, valor_si_no_existe)
    cedula = str(datos.get("cedula", "")).strip()
    edad = str(datos.get("edad", "")).strip()
    asiento = str(datos.get("asiento", "")).strip().upper()
    return nombre, cedula, edad, asiento            # retorna 4 valores a la vez (una tupla)


def validar_campos(nombre, cedula, edad, asiento):
    """Revisa los datos. Retorna el problema (texto) o "" si todo está bien."""
    if len(nombre) < 2:
        return "Escribe el nombre del pasajero"
    if not (cedula.isdigit() and len(cedula) == 10):
        return "La cédula debe tener 10 dígitos"
    if not str(edad).isdigit() or int(edad) > 120:
        return "La edad debe ser un número entre 0 y 120"
    if not asiento_existe(asiento):                 # (vuelo.py)
        return f"El asiento '{asiento}' no existe en este avión"
    return ""


@app.post("/api/reservas")
def reservar():
    """AÑADIR: validar → crear Pasajero → ingresar → guardar JSON."""
    nombre, cedula, edad, asiento = leer_formulario()
    problema = validar_campos(nombre, cedula, edad, asiento)
    if problema != "":
        return respuesta_error(problema)            # 400 = datos incorrectos

    # ¿El asiento está libre? ¿La cédula es nueva? Esto usa TUS métodos de búsqueda.
    # Mientras no los programes, la reserva se hace igual pero SIN esta revisión (con un aviso).
    avisos = []
    try:                                            # try: intenta...
        if lista.buscar_por_asiento(asiento) is not None:                                       # ✍
            return respuesta_error(f"El asiento {asiento} ya está reservado")
    except NotImplementedError:                     # ...y si el método aún no existe:
        avisos.append("No se revisó si el asiento estaba libre: falta buscar_por_asiento()")
    try:
        if lista.buscar_por_cedula(cedula) is not None:                                         # ✍
            return respuesta_error(f"La cédula {cedula} ya tiene una reserva en este vuelo")
    except NotImplementedError:
        avisos.append("No se revisó si la cédula se repetía: falta buscar_por_cedula()")

    pasajero = Pasajero(persistencia.nuevo_numero(), nombre, cedula, int(edad), asiento)
    lista.ingresar(pasajero)                        # ✅ el método resuelto: engancha al final
    persistencia.guardar(lista)                     # 💾 escribe data/reservas.json
    mensaje = f"Reserva #{pasajero.numero} creada: {nombre} en el asiento {asiento}"
    return respuesta_ok(pasajero.a_diccionario(), mensaje, avisos, 201)   # 201 = creado


@app.put("/api/reservas/<int:numero>")
def modificar_reserva(numero):
    """MODIFICAR nombre, edad y/o asiento de una reserva."""
    nombre, cedula, edad, asiento = leer_formulario()
    pasajero = revisar.pasajero_o_none(lista.buscar(numero), "buscar")                        # ✍
    if pasajero is None:
        return respuesta_error(f"No existe la reserva #{numero}", 404)
    problema = validar_campos(nombre, pasajero.cedula, edad, asiento)   # la cédula no cambia
    if problema != "":
        return respuesta_error(problema)

    avisos = []
    try:
        ocupante = lista.buscar_por_asiento(asiento)                                            # ✍
        # el asiento nuevo debe estar libre... o ser el suyo (si no lo cambió)
        if ocupante is not None and ocupante.numero != numero:
            return respuesta_error(f"El asiento {asiento} ya es de {ocupante.nombre}")
    except NotImplementedError:
        avisos.append("No se revisó si el asiento nuevo estaba libre: falta buscar_por_asiento()")

    cambiado = revisar.booleano(lista.modificar(numero, nombre, int(edad), asiento), "modificar")  # ✍
    if not cambiado:
        return respuesta_error(f"No se pudo modificar la reserva #{numero}", 404)
    persistencia.guardar(lista)
    actualizado = lista.buscar(numero)
    return respuesta_ok(actualizado.a_diccionario(), f"Reserva #{numero} actualizada", avisos)


@app.delete("/api/reservas/<int:numero>")
def eliminar_reserva(numero):
    """ELIMINAR (cancelar) una reserva: el asiento vuelve a quedar libre."""
    eliminado = revisar.pasajero_o_none(lista.eliminar(numero), "eliminar")                   # ✍
    if eliminado is None:
        return respuesta_error(f"No existe la reserva #{numero}", 404)
    persistencia.guardar(lista)
    mensaje = f"Reserva #{numero} cancelada: el asiento {eliminado.asiento} quedó libre"
    return respuesta_ok(eliminado.a_diccionario(), mensaje)


@app.post("/api/restaurar")
def restaurar_datos():
    """Botón 'Datos de ejemplo' (paso 30): por si tu código dañó las reservas."""
    persistencia.restaurar(lista)
    return respuesta_ok(None, "Se restauraron las 12 reservas de ejemplo")

@app.get("/api/progreso")
def ver_progreso():
    """Panel 'Progreso del backend': prueba tus métodos con una lista de práctica."""
    return respuesta_ok(verificador.revisar_todo())

# ===== ENCENDER EL SERVIDOR =====
if __name__ == "__main__":
    app.run(debug=True, port=5050)
```

---

<a id="archivo-vuelo-py"></a>
#### Archivo: `vuelo.py`
- **Ruta relativa:** `vuelo.py`
- **Lenguaje:** `python`

```python
# ==============================================================
#  vuelo.py  ·  Datos fijos del vuelo y del avión
# --------------------------------------------------------------
#  La app maneja UN solo vuelo. El avión tiene 12 filas:
#    - Filas 1 y 2  → clase Ejecutiva  (asientos A C | D F     → 4 por fila)
#    - Filas 3 a 12 → clase Económica  (asientos A B C | D E F → 6 por fila)
#    Total: 2 x 4 + 10 x 6 = 68 asientos.
#  Un asiento se escribe como fila + letra: "1A", "7D", "12F".
# ==============================================================

# Un diccionario con todos los datos. Las claves están entre comillas.
VUELO = {
    "codigo": "AL-204",                 # código del vuelo
    "aerolinea": "AeroLista",
    "origen": "Guayaquil",              # ciudad de salida
    "codigo_origen": "GYE",             # código del aeropuerto de salida
    "destino": "Quito",
    "codigo_destino": "UIO",
    "fecha": "2026-12-20",
    "hora": "07:30",
    "filas": 12,                                         # cuántas filas tiene el avión
    "filas_ejecutiva": 2,                                # las primeras 2 filas son Ejecutiva
    "letras_ejecutiva": ["A", "C", "D", "F"],            # asientos de una fila Ejecutiva
    "letras_economica": ["A", "B", "C", "D", "E", "F"],  # asientos de una fila Económica
    "filas_salida": [6, 7],             # filas junto a las salidas de emergencia (sobre las alas)
}


def total_asientos():
    """Calcula cuántos asientos tiene el avión (68)."""
    # filas de Ejecutiva x asientos por fila Ejecutiva
    ejecutiva = VUELO["filas_ejecutiva"] * len(VUELO["letras_ejecutiva"])
    # filas de Económica x asientos por fila Económica
    economica = (VUELO["filas"] - VUELO["filas_ejecutiva"]) * len(VUELO["letras_economica"])
    return ejecutiva + economica


def separar_asiento(asiento):
    """Separa '12A' en (12, 'A'). Si el texto no tiene forma de asiento, retorna (0, '')."""
    asiento = str(asiento).strip().upper()   # quitamos espacios y pasamos a mayúsculas
    if len(asiento) < 2:                     # "A" o "" no son asientos válidos
        return 0, ""
    fila_texto = asiento[:-1]                # todo menos la última letra  → "12"
    letra = asiento[-1]                      # solo la última letra        → "A"
    if not fila_texto.isdigit():             # "XA" no sirve: la fila debe ser un número
        return 0, ""
    return int(fila_texto), letra            # convertimos "12" en el número 12


def clase_de_asiento(asiento):
    """'1A' → 'Ejecutiva'   ·   '7D' → 'Económica'."""
    fila, letra = separar_asiento(asiento)
    if fila <= VUELO["filas_ejecutiva"]:
        return "Ejecutiva"
    return "Económica"


def asiento_existe(asiento):
    """True si el asiento existe en ESTE avión ('13A' o '1B' no existen)."""
    fila, letra = separar_asiento(asiento)
    if fila < 1 or fila > VUELO["filas"]:            # la fila está fuera del avión
        return False
    if fila <= VUELO["filas_ejecutiva"]:             # en Ejecutiva no hay B ni E
        return letra in VUELO["letras_ejecutiva"]
    return letra in VUELO["letras_economica"]
```

---

<a id="archivo-pasajero-py"></a>
#### Archivo: `pasajero.py`
- **Ruta relativa:** `pasajero.py`
- **Lenguaje:** `python`

```python
# ==============================================================
#  pasajero.py  ·  La clase Pasajero
# --------------------------------------------------------------
#  Cada reserva del vuelo es un objeto Pasajero.
#  La lista enlazada guarda un Pasajero dentro de cada Nodo
#  (en nodo.data).
# ==============================================================

from vuelo import clase_de_asiento     # función que dice si un asiento es Ejecutiva o Económica


class Pasajero:
    """Un pasajero con su reserva en el vuelo AL-204."""

    # El CONSTRUCTOR se ejecuta al crear un objeto: Pasajero(1005, "Diana", ...)
    # self es "el objeto que se está creando"; guardamos cada dato en él.
    def __init__(self, numero, nombre, cedula, edad, asiento):
        self.numero = numero      # int → 1005          (número de reserva: ÚNICO)
        self.nombre = nombre      # str → "Diana"
        self.cedula = cedula      # str → "0923456789"  (texto: así no se pierde el 0 inicial)
        self.edad = edad          # int → 22
        self.asiento = asiento    # str → "3C"          (fila + letra)

    def clase(self):
        """Retorna 'Ejecutiva' (filas 1-2) o 'Económica' (filas 3-12)."""
        return clase_de_asiento(self.asiento)

    def es_menor_de_edad(self):
        """True si tiene menos de 18 años."""
        return self.edad < 18

    def a_diccionario(self):
        """Convierte el objeto en un dict.
        Así se puede guardar en el JSON y enviar al navegador
        (un objeto no se puede convertir a JSON; un dict sí)."""
        return {
            "numero": self.numero,
            "nombre": self.nombre,
            "cedula": self.cedula,
            "edad": self.edad,
            "asiento": self.asiento,
        }

    def __str__(self):
        """Lo que se muestra al hacer print(pasajero).
        {self.nombre:<8} = alineado a la izquierda en 8 espacios (para que quede en columna)."""
        return (f"#{self.numero} | {self.nombre:<8} | {self.cedula} | "
                f"{self.edad:>2} años | asiento {self.asiento:<3} | {self.clase()}")
```

---

<a id="archivo-lista-pasajeros-py"></a>
#### Archivo: `lista_pasajeros.py`
- **Ruta relativa:** `lista_pasajeros.py`
- **Lenguaje:** `python`

```python
# ==============================================================
#  lista_pasajeros.py  ·  Lista enlazada de pasajeros
#  ✍ ESTE ES EL ARCHIVO DONDE TRABAJAS TÚ
# --------------------------------------------------------------
#        head
#         |
#         v
#    +----------+---+    +----------+---+    +----------+---+
#    | #1005    | o-+--->| #1002    | o-+--->| #1009    | o-+---> None
#    | Diana 3C |   |    | Ana 1A   |   |    | Luis 7D  |   |
#    +----------+---+    +----------+---+    +----------+---+
#       data    next        data    next        data    next
#
#  Los pasajeros quedan en ORDEN DE LLEGADA (el orden en que reservaron).
#
#  ✅ Ya funcionan: la clase Nodo, el constructor __init__ y ingresar.
#  ✍ Tú implementas los otros 9 métodos. Reglas:
#     1. Recorre SIEMPRE con el patrón de la Unidad 3:
#            actual = self.head
#            while actual is not None:
#                ...
#                actual = actual.next
#     2. NO conviertas la lista enlazada en una list de Python para
#        buscar, modificar o eliminar (nada de list(), in, index, sort).
#     3. La ÚNICA list de Python permitida es el RESULTADO de listar()
#        y de filtrar_por_clase(): es la "caja" que Flask envía al navegador.
#     4. La clase NUNCA hace print() ni input(): RETORNA valores.
#     5. Para empezar un método, BORRA su línea  raise NotImplementedError(...)
#        y escribe tu código en su lugar.
# ==============================================================

from pasajero import Pasajero


class Nodo:
    """Un vagón del tren: lleva un pasajero (data) y el enganche al siguiente (next)."""

    def __init__(self, data):
        self.data = data          # el Pasajero que viaja en este vagón
        self.next = None          # al nacer no está enganchado a nadie


class ListaPasajeros:
    """Lista enlazada simple con todas las reservas del vuelo."""

    # ----------------------------------------------------------
    # ✅ EJEMPLO RESUELTO 1: el constructor
    # ----------------------------------------------------------
    def __init__(self):
        self.head = None          # la cabeza: el primer nodo (None = avión vacío)
        self.cantidad = 0         # cuántas reservas (nodos) hay

    # ----------------------------------------------------------
    # ✅ EJEMPLO RESUELTO 2: ingresar (= insertar_final de la Unidad 3)
    #    Engancha un pasajero NUEVO al final del tren.
    # ----------------------------------------------------------
    def ingresar(self, pasajero):
        nuevo = Nodo(pasajero)                # 1. creo el vagón con el pasajero

        if self.head is None:                 # 2a. Lista vacía:
            self.head = nuevo                 #     el nuevo es la cabeza
        else:                                 # 2b. Lista con nodos:
            actual = self.head                #     pongo el dedo en la cabeza
            while actual.next is not None:    #     mientras NO sea el último...
                actual = actual.next          #     ...avanzo un vagón
            actual.next = nuevo               #     el último ahora apunta al nuevo

        self.cantidad = self.cantidad + 1     # 3. hay una reserva más
        return True

    # ==========================================================
    # ✍ TAREA · implementa estos métodos (en este orden te será más fácil)
    # ==========================================================

    def esta_vacia(self):
        """Retorna True si no hay ninguna reserva; False si hay al menos una.
        Pista: mira self.head."""
        return self.head is None

    def longitud(self):
        """Retorna cuántas reservas hay.
        Pista: no recorras nada, ya tienes self.cantidad."""
        return self.cantidad

    def listar(self):
        """Retorna una list de Python con TODOS los Pasajero, en orden:
        desde head hasta el último. Si está vacía retorna [].
        ⚠ Guarda el pasajero (actual.data), no el nodo.
        👉 Cuando funcione, ¡el avión se pinta con los asientos ocupados!"""
        resultado = []
        actual = self.head
        while actual is not None:
            resultado.append(actual.data)
            actual = actual.next
        return resultado

    def buscar(self, numero):
        """Búsqueda lineal por número de reserva (int).
        Retorna el Pasajero encontrado o None si no existe."""
        actual = self.head
        while actual is not None:
            if actual.data.numero == numero:
                return actual.data
            actual = actual.next
        return None

    def buscar_por_asiento(self, asiento):
        """Retorna el Pasajero sentado en ese asiento ("7D") o None si está libre.
        👉 Flask lo usa al hacer clic en un asiento rojo y para no
           vender dos veces el mismo asiento."""
        actual = self.head
        while actual is not None:
            if actual.data.asiento == asiento:
                return actual.data
            actual = actual.next
        return None

    def buscar_por_cedula(self, cedula):
        """Retorna el Pasajero con esa cédula (str) o None.
        👉 Flask lo usa para que una persona no reserve dos asientos."""
        actual = self.head
        while actual is not None:
            if actual.data.cedula == cedula:
                return actual.data
            actual = actual.next
        return None

    def modificar(self, numero, nuevo_nombre, nueva_edad, nuevo_asiento):
        """Cambia nombre, edad y asiento del pasajero con ese número.
        Usa buscar(). El nodo NO cambia de lugar en la lista: solo cambia su carga.
        Retorna True si lo modificó, False si no existe."""
        pasajero = self.buscar(numero)
        if pasajero is not None:
            pasajero.nombre = nuevo_nombre
            pasajero.edad = nueva_edad
            pasajero.asiento = nuevo_asiento
            return True
        return False

    def eliminar(self, numero):
        """Cancela la reserva: desengancha el nodo que tiene ese número.
        Piensa en los casos: lista vacía · es la cabeza · está en medio o al
        final · no existe. ¡No olvides restar self.cantidad!
        Retorna el Pasajero eliminado, o None si no existía."""
        if self.head is None:
            return None

        if self.head.data.numero == numero:
            eliminado = self.head.data
            self.head = self.head.next
            self.cantidad = self.cantidad - 1
            return eliminado

        anterior = self.head
        while anterior.next is not None and anterior.next.data.numero != numero:
            anterior = anterior.next

        if anterior.next is None:
            return None

        eliminado = anterior.next.data
        anterior.next = anterior.next.next
        self.cantidad = self.cantidad - 1
        return eliminado

    def filtrar_por_clase(self, clase):
        """Retorna una list con los pasajeros de esa clase ("Ejecutiva" o
        "Económica"), en el mismo orden de la lista.
        Pista: cada pasajero sabe su clase con  pasajero.clase()"""
        resultado = []
        actual = self.head
        while actual is not None:
            if actual.data.clase() == clase:
                resultado.append(actual.data)
            actual = actual.next
        return resultado
```

---

<a id="archivo-persistencia-py"></a>
#### Archivo: `persistencia.py`
- **Ruta relativa:** `persistencia.py`
- **Lenguaje:** `python`

```python
# ==============================================================
#  persistencia.py  ·  Guardar y leer las reservas en JSON
# --------------------------------------------------------------
#    RAM (se borra)                           DISCO (permanece)
#    head -> [1005] -> [1002] -> None   --guardar-->   data/reservas.json
#                                       <--cargar---
# ==============================================================
import json      # librería de Python para leer y escribir JSON
import os        # librería para trabajar con rutas de archivos y carpetas

from pasajero import Pasajero

# Rutas de los archivos. os.path.join une partes de una ruta con la barra correcta
# (\ en Windows, / en Mac). __file__ es la ruta de ESTE archivo.
CARPETA = os.path.join(os.path.dirname(os.path.abspath(__file__)), "data")
ARCHIVO = os.path.join(CARPETA, "reservas.json")     # las reservas reales
EJEMPLO = os.path.join(CARPETA, "ejemplo.json")      # copia de seguridad (datos de prueba)

siguiente_numero = 1001          # el próximo número de reserva que se entregará


def cargar(lista, ruta=ARCHIVO):
    """Lee el JSON y mete cada reserva al FINAL de la lista con ingresar()."""
    global siguiente_numero      # global: vamos a cambiar la variable de afuera de la función
    if not os.path.exists(ruta):                 # la primera vez aún no existe reservas.json...
        ruta = EJEMPLO                           # ...entonces usamos los datos de prueba
    # with open(...) abre el archivo y lo cierra solo al terminar. "r" = leer
    with open(ruta, "r", encoding="utf-8") as archivo:
        contenido = json.load(archivo)           # texto JSON → dict de Python

    for d in contenido["reservas"]:              # cada d es un dict con los 5 datos
        pasajero = Pasajero(d["numero"], d["nombre"], d["cedula"], d["edad"], d["asiento"])
        lista.ingresar(pasajero)                 # ✅ el ingresar de tu lista enlazada

    siguiente_numero = contenido["siguiente_numero"]


def guardar(lista):
    """Recorre la lista enlazada (head → None) y escribe todo en el JSON."""
    reservas = []
    actual = lista.head                          # el dedo en la cabeza
    while actual is not None:                    # mientras haya vagón...
        reservas.append(actual.data.a_diccionario())   # ...guardo su pasajero como dict
        actual = actual.next                     # ...y avanzo
        if len(reservas) > 1000:                 # seguro: un avión no tiene 1000 asientos
            raise RuntimeError("La lista no termina nunca: ¿un nodo se apunta a sí mismo?")

    contenido = {"siguiente_numero": siguiente_numero, "reservas": reservas}
    os.makedirs(CARPETA, exist_ok=True)          # crea data/ si no existe
    # "w" = escribir (borra lo anterior). indent=2 = bonito y legible.
    # ensure_ascii=False = guarda las tildes tal cual (Sofía y no Sofía)
    with open(ARCHIVO, "w", encoding="utf-8") as archivo:
        json.dump(contenido, archivo, ensure_ascii=False, indent=2)   # dict → texto JSON


def nuevo_numero():
    """Entrega un número de reserva que nunca se ha usado: 1013, 1014, ..."""
    global siguiente_numero
    numero = siguiente_numero
    siguiente_numero = siguiente_numero + 1
    return numero


def restaurar(lista):
    """Vuelve a los 12 pasajeros de data/ejemplo.json."""
    lista.head = None             # soltar la cabeza = soltar todo el tren
    lista.cantidad = 0            # (Python borra solito los vagones que nadie apunta)
    cargar(lista, EJEMPLO)
    guardar(lista)
```

---

<a id="archivo-verificador-py"></a>
#### Archivo: `verificador.py`
- **Ruta relativa:** `verificador.py`
- **Lenguaje:** `python`

```python
# ==============================================================
#  verificador.py  ·  Pruebas automáticas de tus métodos
# --------------------------------------------------------------
#  Corre pruebas sobre una lista de PRÁCTICA (no toca tus reservas
#  reales) y arma el panel "Progreso del backend" de la página.
#  Estados: ok ✅ · pendiente 🚧 · fallo ❌ · error 💥 · lento ⏳
# ==============================================================
import threading      # para correr cada prueba en un "hilo" aparte con tiempo límite

from lista_pasajeros import ListaPasajeros
from pasajero import Pasajero
from revisar import ErrorEnTuMetodo, booleano, lista_de_pasajeros, pasajero_o_none


class Fallo(Exception):
    """La prueba corrió, pero el resultado no es el esperado."""


def esperar(condicion, mensaje):
    """Si la condición es falsa, la prueba falla con ese mensaje."""
    if not condicion:
        raise Fallo(mensaje)


def lista_de_prueba():
    """head → [1 Ana 1A] → [2 Beto 3C] → [3 Carla 5D] → [4 Dani 2C] → None"""
    lista = ListaPasajeros()
    lista.ingresar(Pasajero(1, "Ana", "0000000001", 30, "1A"))
    lista.ingresar(Pasajero(2, "Beto", "0000000002", 15, "3C"))
    lista.ingresar(Pasajero(3, "Carla", "0000000003", 40, "5D"))
    lista.ingresar(Pasajero(4, "Dani", "0000000004", 22, "2C"))
    return lista


def cadena(lista):
    """Lee los números caminando los nodos (sin usar tus métodos)."""
    numeros = []
    actual = lista.head
    while actual is not None:
        numeros.append(actual.data.numero)
        actual = actual.next
        if len(numeros) > 50:
            raise Fallo("La cadena no termina: algún nodo quedó apuntándose en círculo")
    return numeros


def numeros_de(pasajeros):
    resultado = []
    for p in pasajeros:
        resultado.append(p.numero)
    return resultado


# ---------- Una prueba por método ----------
def probar_esta_vacia():
    esperar(ListaPasajeros().esta_vacia() is True, "Con la lista vacía debería retornar True")
    esperar(lista_de_prueba().esta_vacia() is False, "Con 4 pasajeros debería retornar False")


def probar_longitud():
    esperar(ListaPasajeros().longitud() == 0, "Con la lista vacía debería retornar 0")
    valor = lista_de_prueba().longitud()
    esperar(valor == 4, f"Con 4 pasajeros debería retornar 4, retornó {valor!r}")


def probar_listar():
    vacia = ListaPasajeros().listar()
    esperar(vacia == [], f"Con la lista vacía debería retornar [], retornó {vacia!r}")
    lista = lista_de_prueba()
    numeros = numeros_de(lista_de_pasajeros(lista.listar(), "listar"))
    esperar(numeros == [1, 2, 3, 4], f"Esperaba los números [1, 2, 3, 4] (de head al último), obtuve {numeros}")
    esperar(cadena(lista) == [1, 2, 3, 4], "listar() no debe cambiar la lista enlazada")


def probar_busqueda(metodo, casos, clave_inexistente):
    """casos = [(clave, número esperado), ...] → uno en medio, la cabeza y el último."""
    lista = lista_de_prueba()
    for clave, esperado in casos:
        encontrado = pasajero_o_none(getattr(lista, metodo)(clave), metodo)   # getattr: llama al método por su nombre
        esperar(encontrado is not None, f"{metodo}({clave!r}) debería encontrar al pasajero #{esperado}, retornó None")
        esperar(encontrado.numero == esperado,
                f"{metodo}({clave!r}) retornó al pasajero #{encontrado.numero}, esperaba al #{esperado}")
    nada = getattr(lista, metodo)(clave_inexistente)
    esperar(nada is None, f"{metodo}({clave_inexistente!r}) no existe: debería retornar None, retornó {nada!r}")
    esperar(getattr(ListaPasajeros(), metodo)(casos[0][0]) is None, f"En una lista vacía {metodo}() debería retornar None")
    esperar(cadena(lista) == [1, 2, 3, 4], f"{metodo}() no debe cambiar la lista enlazada")


def probar_buscar():
    probar_busqueda("buscar", [(3, 3), (1, 1), (4, 4)], 99)


def probar_buscar_por_asiento():
    probar_busqueda("buscar_por_asiento", [("5D", 3), ("1A", 1), ("2C", 4)], "9F")


def probar_buscar_por_cedula():
    probar_busqueda("buscar_por_cedula", [("0000000003", 3), ("0000000001", 1), ("0000000004", 4)], "0999999999")


def probar_modificar():
    lista = lista_de_prueba()
    esperar(booleano(lista.modificar(3, "Carla M", 41, "6A"), "modificar") is True, "modificar(3, ...) existe: debería retornar True")
    nodo = lista.head.next.next                    # el tercer vagón
    esperar(nodo.data.nombre == "Carla M" and nodo.data.edad == 41 and nodo.data.asiento == "6A",
            "El pasajero #3 no quedó con nombre 'Carla M', edad 41 y asiento '6A'")
    esperar(cadena(lista) == [1, 2, 3, 4], "modificar() no debe mover nodos: el orden cambió")
    esperar(lista.modificar(99, "X", 1, "1A") is False, "modificar(99, ...) no existe: debería retornar False")
    esperar(lista.head.data.asiento == "1A", "modificar(99, ...) cambió a otro pasajero")


def probar_eliminar():
    lista = lista_de_prueba()
    quitado = pasajero_o_none(lista.eliminar(1), "eliminar")
    esperar(quitado is not None and quitado.numero == 1, "eliminar(1) (la cabeza) debería retornar al pasajero #1")
    esperar(cadena(lista) == [2, 3, 4], f"Tras eliminar la cabeza quedó {cadena(lista)}, esperaba [2, 3, 4]")
    esperar(lista.cantidad == 3, "Tras eliminar, self.cantidad debe bajar en 1")
    quitado = lista.eliminar(3)
    esperar(quitado is not None and quitado.numero == 3, "eliminar(3) (en medio) debería retornar al pasajero #3")
    esperar(cadena(lista) == [2, 4], f"Tras eliminar el del medio quedó {cadena(lista)}, esperaba [2, 4]")
    quitado = lista.eliminar(4)
    esperar(quitado is not None and quitado.numero == 4, "eliminar(4) (el último) debería retornar al pasajero #4")
    esperar(cadena(lista) == [2], f"Tras eliminar el último quedó {cadena(lista)}, esperaba [2]")
    esperar(lista.eliminar(99) is None, "eliminar(99) no existe: debería retornar None")
    esperar(cadena(lista) == [2], "eliminar(99) no debería quitar a nadie")
    lista.eliminar(2)
    esperar(lista.head is None and lista.cantidad == 0, "Al eliminar el único nodo, head debe quedar en None y cantidad en 0")
    esperar(lista.eliminar(2) is None, "En una lista vacía eliminar() debería retornar None")


def probar_filtrar_por_clase():
    lista = lista_de_prueba()
    ejecutiva = numeros_de(lista_de_pasajeros(lista.filtrar_por_clase("Ejecutiva"), "filtrar_por_clase"))
    esperar(ejecutiva == [1, 4], f"filtrar_por_clase('Ejecutiva') debería dar [1, 4], dio {ejecutiva}")
    economica = numeros_de(lista.filtrar_por_clase("Económica"))
    esperar(economica == [2, 3], f"filtrar_por_clase('Económica') debería dar [2, 3], dio {economica}")
    esperar(ListaPasajeros().filtrar_por_clase("Ejecutiva") == [], "En una lista vacía debería retornar []")
    esperar(cadena(lista) == [1, 2, 3, 4], "filtrar_por_clase() no debe cambiar la lista enlazada")


# Lista de (nombre del método, función que lo prueba)
PRUEBAS = [
    ("esta_vacia", probar_esta_vacia),
    ("longitud", probar_longitud),
    ("listar", probar_listar),
    ("buscar", probar_buscar),
    ("buscar_por_asiento", probar_buscar_por_asiento),
    ("buscar_por_cedula", probar_buscar_por_cedula),
    ("modificar", probar_modificar),
    ("eliminar", probar_eliminar),
    ("filtrar_por_clase", probar_filtrar_por_clase),
]


def correr_con_tiempo(prueba, segundos=1.0):
    """Corre la prueba en un hilo aparte; si tarda más de 1 s, es un bucle infinito."""
    resultado = {"estado": "lento",
                 "detalle": "Tardó demasiado: ¿bucle infinito? Revisa que avances con actual = actual.next"}

    def tarea():
        try:
            prueba()
            resultado["estado"] = "ok"
            resultado["detalle"] = "Todas las pruebas pasaron"
        except NotImplementedError:
            resultado["estado"] = "pendiente"
            resultado["detalle"] = "Aún no implementado"
        except (Fallo, ErrorEnTuMetodo) as e:
            resultado["estado"] = "fallo"
            resultado["detalle"] = str(e)
        except Exception as e:                   # cualquier otro error de Python
            resultado["estado"] = "error"
            resultado["detalle"] = f"{type(e).__name__}: {e}"

    hilo = threading.Thread(target=tarea, daemon=True)
    hilo.start()
    hilo.join(segundos)                          # espera como máximo 1 segundo
    return resultado


def revisar_todo():
    """Corre las 9 pruebas y retorna una lista de diccionarios para el panel."""
    informe = []
    for nombre, prueba in PRUEBAS:
        resultado = correr_con_tiempo(prueba)
        informe.append({"metodo": nombre, "estado": resultado["estado"], "detalle": resultado["detalle"]})
    return informe
```

---

<a id="archivo-revisar-py"></a>
#### Archivo: `revisar.py`
- **Ruta relativa:** `revisar.py`
- **Lenguaje:** `python`

```python
# ==============================================================
#  revisar.py  ·  Revisa lo que RETORNAN tus métodos
# --------------------------------------------------------------
#  Antes de enviar un resultado al navegador, app.py lo pasa por
#  estas funciones. Si algo no cuadra, lanzan ErrorEnTuMetodo con
#  un mensaje que te dice qué corregir.
# ==============================================================
from lista_pasajeros import Nodo
from pasajero import Pasajero


class ErrorEnTuMetodo(Exception):
    """Un error 'nuestro': un método retornó algo de un tipo que no corresponde."""


def pasajero_o_none(valor, metodo):
    """Para buscar, buscar_por_asiento, buscar_por_cedula y eliminar."""
    if valor is None or isinstance(valor, Pasajero):    # isinstance: ¿es de esta clase?
        return valor                                    # ✅ está bien
    if isinstance(valor, Nodo):
        raise ErrorEnTuMetodo(f"{metodo}() retornó un Nodo. Retorna el pasajero: actual.data")
    raise ErrorEnTuMetodo(f"{metodo}() debe retornar un Pasajero o None, "
                          f"pero retornó {type(valor).__name__}: {valor!r}")


def lista_de_pasajeros(valor, metodo):
    """Para listar y filtrar_por_clase: debe ser una list de Pasajero."""
    if valor is None:
        raise ErrorEnTuMetodo(f"{metodo}() retornó None. ¿Olvidaste el  return resultado ?")
    if not isinstance(valor, list):
        raise ErrorEnTuMetodo(f"{metodo}() debe retornar una list, pero retornó {type(valor).__name__}")
    for elemento in valor:
        if isinstance(elemento, Nodo):
            raise ErrorEnTuMetodo(f"{metodo}() guardó Nodos en la lista. Guarda el pasajero: actual.data")
        if not isinstance(elemento, Pasajero):
            raise ErrorEnTuMetodo(f"{metodo}() debe guardar objetos Pasajero, pero encontré "
                                  f"{type(elemento).__name__}: {elemento!r}")
    return valor


def booleano(valor, metodo):
    """Para esta_vacia y modificar: debe ser True o False."""
    if valor is True or valor is False:
        return valor
    raise ErrorEnTuMetodo(f"{metodo}() debe retornar True o False, pero retornó {valor!r}")


def entero(valor, metodo):
    """Para longitud: debe ser un número entero."""
    if isinstance(valor, int) and not isinstance(valor, bool):
        return valor
    raise ErrorEnTuMetodo(f"{metodo}() debe retornar un número entero, pero retornó {valor!r}")
```

---

<a id="archivo-prueba-consola-py"></a>
#### Archivo: `prueba_consola.py`
- **Ruta relativa:** `prueba_consola.py`
- **Lenguaje:** `python`

```python
# ==============================================================
#  prueba_consola.py  ·  Prueba tus métodos SIN Flask
#      python prueba_consola.py
#  Cada vez que termines un método, quita el # de sus líneas
#  y vuelve a ejecutar este archivo.
# ==============================================================
from lista_pasajeros import ListaPasajeros
from pasajero import Pasajero

lista = ListaPasajeros()                  # una lista vacía: head = None
lista.ingresar(Pasajero(1005, "Diana", "0923456789", 22, "3C"))   # ✅ ingresar ya funciona
lista.ingresar(Pasajero(1002, "Ana", "0912345678", 28, "1A"))
lista.ingresar(Pasajero(1009, "Luis", "0987654321", 35, "7D"))
lista.ingresar(Pasajero(1004, "Beto", "0934567890", 16, "7E"))
print("ingresar OK: 4 pasajeros ingresados, cantidad =", lista.cantidad)

# --- descomenta a medida que avances (lo que debería salir está a la derecha) ---
print("esta_vacia:", lista.esta_vacia())                 # False
print("longitud:", lista.longitud())                     # 4
for pasajero in lista.listar():                          # Diana, Ana, Luis, Beto
    print("   ", pasajero)
print("buscar 1009:", lista.buscar(1009))                # Luis
print("buscar 9999:", lista.buscar(9999))                # None
print("asiento 7E:", lista.buscar_por_asiento("7E"))     # Beto
print("cedula de Ana:", lista.buscar_por_cedula("0912345678"))
print("modificar:", lista.modificar(1009, "Luis A.", 36, "9F"))   # True
print("eliminar cabeza:", lista.eliminar(1005))          # Diana
print("eliminar 9999:", lista.eliminar(9999))            # None
print("ejecutiva:", [str(p) for p in lista.filtrar_por_clase("Ejecutiva")])
```

---

### Categoría: Vistas e Interfaz (HTML)

<a id="archivo-templates-index-html"></a>
#### Archivo: `templates/index.html`
- **Ruta relativa:** `templates/index.html`
- **Lenguaje:** `html`

```html
<!DOCTYPE html>
<!-- La línea de arriba le dice al navegador: "esto es HTML moderno". -->
<html lang="es">
<head>
  <!-- ========== CONFIGURACIÓN (no se ve en la página) ========== -->
  <meta charset="UTF-8">                                                  <!-- para que se vean bien las tildes y la ñ -->
  <meta name="viewport" content="width=device-width, initial-scale=1.0">  <!-- para que se adapte a celulares -->
  <title>AeroLista ✈ Vuelo {{ vuelo.codigo }}</title>                     <!-- texto de la pestaña; Jinja pone "AL-204" -->
  <!-- El iconito de la pestaña: un avión dibujado con SVG (así no hace falta una imagen) -->
  <link rel="icon" href="data:image/svg+xml,<svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 100 100'><text y='.9em' font-size='90'>✈</text></svg>">

  <!-- Dos tipos de letra de Google: Inter (textos) y Fira Code (números de asiento) -->
  <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700;800&family=Fira+Code:wght@400;500;600&display=swap" rel="stylesheet">

  <!-- Nuestra hoja de estilos (la creamos en el paso 6).
       url_for('static', filename=...) arma la dirección correcta hacia la carpeta static/ -->
  <link rel="stylesheet" href="{{ url_for('static', filename='css/estilos.css') }}">
</head>
<body>

  <!-- ========== 1. ENCABEZADO: marca, ruta del vuelo y datos ========== -->
  <header class="barra">
    <div class="marca">
      <span class="marca-logo">✈</span>
      <div><b>AeroLista</b><small>Reservas con listas enlazadas</small></div>
    </div>
    <div class="ruta">
      <!-- {{ vuelo.codigo_origen }} se reemplaza por "GYE" (viene de vuelo.py) -->
      <div class="ruta-punto"><b>{{ vuelo.codigo_origen }}</b><small>{{ vuelo.origen }}</small></div>
      <div class="ruta-linea"><span>✈</span></div>
      <div class="ruta-punto"><b>{{ vuelo.codigo_destino }}</b><small>{{ vuelo.destino }}</small></div>
    </div>
    <div class="vuelo-datos">
      <span>Vuelo <b>{{ vuelo.codigo }}</b></span>
      <span>📅 {{ vuelo.fecha }}</span>
      <span>🕖 {{ vuelo.hora }}</span>
    </div>
  </header>

  <!-- ========== 2. CONTENIDO: el avión (izquierda) y el panel (derecha) ========== -->
  <main class="contenedor">
    <section class="zona-avion">
      <!-- Tablero: 4 tarjetas con los números del vuelo -->
      <div class="tablero">
        <div class="dato"><small>Reservados</small><b id="dato-reservados">—</b></div>
        <div class="dato"><small>Libres</small><b id="dato-libres">—</b></div>
        <div class="dato"><small>Asientos</small><b id="dato-total">—</b></div>
        <div class="dato">
          <small>Ocupación</small>
          <div class="ocupacion"><span class="luz" id="luz-ocupacion"></span><b id="dato-ocupacion">—</b></div>
          <div class="barra-ocupacion"><span id="barra-ocupacion"></span></div>   <!-- barrita de progreso -->
        </div>
      </div>
      <p class="aviso-pendiente" id="aviso-tablero" hidden></p>
      <!-- Leyenda del semáforo y botones para elegir la vista -->
      <div class="controles">
        <ul class="leyenda">
          <li><span class="muestra libre"></span>Libre</li>
          <li><span class="muestra seleccionado"></span>Seleccionado</li>
          <li><span class="muestra ocupado"></span>Reservado</li>
          <li><span class="muestra encontrado"></span>Encontrado</li>
        </ul>
        <div class="vistas">
          <!-- data-vista guarda qué vista activa cada botón -->
          <button type="button" class="btn-vista activo" data-vista="3d">🧊 3D</button>
          <button type="button" class="btn-vista" data-vista="2d">🗺️ 2D</button>
        </div>
      </div>
      <!-- Deslizadores (input type="range"): solo se ven en 3D -->
      <div class="ajustes-3d" id="ajustes-3d">
        <label>Inclinación <input type="range" id="rango-inclinacion" min="0" max="65" value="42"></label>
        <label>Giro <input type="range" id="rango-giro" min="-180" max="180" value="0"></label>
      </div>
      <!-- Aviso que usaremos en la Etapa E si Python no puede listar las reservas -->
      <p class="aviso-pendiente" id="aviso-avion" hidden></p>

      <!-- LA ESCENA: el "cielo" donde flota el avión -->
      <div class="escena" id="escena">
        <div class="avion vista-3d" id="avion">
          <div class="sombra"></div>                 <!-- sombra en el suelo (se ve en 3D) -->
          <div class="ala ala-izq"></div>            <!-- alas: rectángulos recortados -->
          <div class="ala ala-der"></div>
          <div class="motor motor-izq"></div>        <!-- motores debajo de las alas -->
          <div class="motor motor-der"></div>
          <div class="timon timon-izq"></div>        <!-- alas pequeñas de la cola -->
          <div class="timon timon-der"></div>
          <div class="fuselaje">                     <!-- el cuerpo del avión -->
            <div class="nariz">
              <div class="parabrisas"><span></span><span></span></div>   <!-- ventanas del piloto -->
            </div>
            <div class="cabina" id="cabina">
              <!-- VACÍA: app.js crea aquí las 12 filas y los 68 asientos (paso 12) -->
            </div>
            <div class="cola"><span>AL</span></div>   <!-- la cola, con la marca -->
          </div>
        </div>
      </div>
    </section>
    <!-- ========== PANEL DE OPERACIONES ========== -->
    <aside class="panel">
      <!-- Los 5 botones de las pestañas. data-pestana dice cuál abre cada uno -->
      <nav class="pestanas">
        <button type="button" class="pestana activa" data-pestana="reservar">➕ Reservar</button>
        <button type="button" class="pestana" data-pestana="buscar">🔍 Buscar</button>
        <button type="button" class="pestana" data-pestana="modificar">✏️ Modificar</button>
        <button type="button" class="pestana" data-pestana="eliminar">🗑️ Eliminar</button>
        <button type="button" class="pestana" data-pestana="listar">📋 Listar</button>
      </nav>

      <!-- El contenido de cada pestaña. El id es "pestana-" + el nombre -->
      <section class="contenido-pestana activa" id="pestana-reservar">
        <h2>Nueva reserva</h2>
        <p class="ayuda">① Haz clic en un asiento <b class="txt-verde">verde</b> · ② llena los datos · ③ reserva.</p>
        <form id="form-reservar" class="formulario" autocomplete="off">
          <!-- El asiento elegido (lo escribe JavaScript al hacer clic en el avión) -->
          <div class="asiento-elegido">
            <span>Asiento</span>
            <output id="reservar-asiento" class="chip-asiento vacio">—</output>
            <small id="reservar-clase">Elige uno en el avión</small>
          </div>
          <!-- name="nombre" permite leerlo después como formulario.nombre.value -->
          <label>Nombre
            <input name="nombre" required minlength="2" maxlength="30" placeholder="Ej: Carla Mendoza">
          </label>
          <div class="dos-campos">
            <label>Cédula   <!-- pattern="[0-9]{10}": exactamente 10 dígitos -->
              <input name="cedula" required pattern="[0-9]{10}" maxlength="10" inputmode="numeric" placeholder="10 dígitos">
            </label>
            <label>Edad
              <input name="edad" type="number" required min="0" max="120" placeholder="Ej: 25">
            </label>
          </div>
          <button type="submit" class="btn btn-primario">✈ Reservar asiento</button>
        </form>
      </section>
      <section class="contenido-pestana" id="pestana-buscar">
        <h2>Buscar reserva</h2>
        <form id="form-buscar" class="formulario" autocomplete="off">
          <!-- Tres opciones (radio): solo se puede marcar UNA porque comparten name="tipo" -->
          <div class="segmentado">
            <label><input type="radio" name="tipo" value="numero" checked><span>N.º reserva</span></label>
            <label><input type="radio" name="tipo" value="cedula"><span>Cédula</span></label>
            <label><input type="radio" name="tipo" value="asiento"><span>Asiento</span></label>
          </div>
          <div class="fila-busqueda">
            <input id="buscar-valor" required placeholder="Ej: 1002">
            <button type="submit" class="btn btn-primario">🔍 Buscar</button>
          </div>
        </form>
        <div id="resultado-buscar" class="resultado"></div>   <!-- aquí aparece el pase -->
      </section>
      <section class="contenido-pestana" id="pestana-modificar">
        <h2>Modificar reserva</h2>
        <!-- Primer formulario: qué reserva quieres modificar -->
        <form id="form-cargar" class="formulario" autocomplete="off">
          <div class="fila-busqueda">
            <input id="modificar-numero" type="number" required placeholder="N.º de reserva, ej: 1009">
            <button type="submit" class="btn btn-suave">Cargar</button>
          </div>
        </form>
        <!-- Segundo formulario: los datos (empieza escondido con "hidden") -->
        <form id="form-modificar" class="formulario" autocomplete="off" hidden>
          <div class="asiento-elegido">
            <span>Asiento</span>
            <output id="modificar-asiento" class="chip-asiento">—</output>
            <small>Clic en un asiento verde para cambiarlo</small>
          </div>
          <label>Nombre <input name="nombre" required minlength="2" maxlength="30"></label>
          <div class="dos-campos">
            <label>Cédula <input name="cedula" disabled></label>   <!-- disabled: no se puede cambiar -->
            <label>Edad <input name="edad" type="number" required min="0" max="120"></label>
          </div>
          <div class="botones">
            <button type="submit" class="btn btn-primario">💾 Guardar cambios</button>
            <button type="button" class="btn btn-suave" id="btn-cancelar-edicion">Cancelar</button>
          </div>
        </form>
      </section>
      <section class="contenido-pestana" id="pestana-eliminar">
        <h2>Cancelar una reserva</h2>
        <p class="ayuda">El asiento vuelve a quedar <b class="txt-verde">libre</b>.</p>
        <form id="form-eliminar" class="formulario" autocomplete="off">
          <div class="fila-busqueda">
            <input id="eliminar-numero" type="number" required placeholder="N.º de reserva, ej: 1004">
            <button type="submit" class="btn btn-peligro">🗑️ Cancelar</button>
          </div>
        </form>
        <!-- Ventana de confirmación. method="dialog" hace que los botones la cierren
             y guarden su value ("si" o "no") en modal.returnValue -->
        <dialog class="modal" id="modal">
          <form method="dialog">
            <h3 id="modal-titulo">¿Seguro?</h3>
            <p id="modal-texto"></p>
            <div class="botones">
              <button value="no" class="btn btn-suave">No, volver</button>
              <button value="si" class="btn btn-peligro">Sí, cancelar reserva</button>
            </div>
          </form>
        </dialog>
      </section>
      <section class="contenido-pestana" id="pestana-listar">
        <h2>Pasajeros del vuelo</h2>
        <!-- Filtros: data-clase="" significa "todas" -->
        <div class="filtros">
          <button type="button" class="chip activo" data-clase="">Todos</button>
          <button type="button" class="chip" data-clase="Ejecutiva">Ejecutiva</button>
          <button type="button" class="chip" data-clase="Económica">Económica</button>
        </div>
        <div class="tabla-caja">
          <table>
            <thead><tr><th>#</th><th>N.º</th><th>Nombre</th><th>Edad</th><th>Asiento</th></tr></thead>
            <tbody id="tabla-reservas"></tbody>   <!-- las filas las crea JavaScript -->
          </table>
        </div>
      </section>

      <!-- ========== PROGRESO DEL BACKEND ========== -->
      <section class="progreso" id="progreso">
        <div class="progreso-titulo">
          <h3>🧪 Progreso del backend</h3>
          <b id="progreso-cuenta">—</b>
        </div>
        <div class="progreso-barra"><span id="progreso-barra"></span></div>
        <ul id="progreso-lista"></ul>   <!-- un renglón por método (lo llena progreso.js) -->
        <div class="botones">
          <button type="button" class="btn btn-suave" id="btn-revisar">↻ Revisar otra vez</button>
          <button type="button" class="btn btn-suave" id="btn-restaurar">↺ Datos de ejemplo</button>
        </div>
      </section>
    </aside>
  </main>

  <!-- ========== 3. EL TREN: la lista enlazada dibujada ========== -->
  <section class="tren">
    <div class="tren-titulo">
      <h2>🚂 Tu lista enlazada en la memoria del servidor</h2>
      <code id="tren-cantidad">nodos = —</code>
    </div>
    <div class="tren-vagones" id="tren-vagones"></div>   <!-- los vagones los crea tren.js -->
  </section>

  <!-- ========== 4. PIE DE PÁGINA ========== -->
  <footer class="pie" id="pie">AeroLista · Proyecto de Estructuras de Datos · Flask + listas enlazadas</footer>

  <!-- ========== 5. ELEMENTOS FLOTANTES ========== -->
  <div class="toasts" id="toasts"></div>   <!-- aquí aparecen los mensajes flotantes -->
  <div class="tooltip" id="tooltip" hidden></div>   <!-- el globo de ayuda (empieza escondido) -->

  <!-- ========== 6. SCRIPTS (el orden importa: se cargan de arriba hacia abajo) ========== -->
  <script src="{{ url_for('static', filename='js/utilidades.js') }}"></script>  <!-- atajos: $, crear... -->
  <script src="{{ url_for('static', filename='js/api.js') }}"></script>         <!-- hablar con Flask -->
  <script src="{{ url_for('static', filename='js/mensajes.js') }}"></script>    <!-- toasts y avisos -->
  <script src="{{ url_for('static', filename='js/avion.js') }}"></script>       <!-- dibuja filas y asientos -->
  <script src="{{ url_for('static', filename='js/clic.js') }}"></script>        <!-- clic en un asiento -->
  <script src="{{ url_for('static', filename='js/vista3d.js') }}"></script>     <!-- vista 3D -->
  <!-- datos-demo.js ya no se usa: ahora los datos vienen de Flask y de tu lista enlazada -->
  <script src="{{ url_for('static', filename='js/semaforo.js') }}"></script>    <!-- 🚦 colores de los asientos -->
  <script src="{{ url_for('static', filename='js/tooltip.js') }}"></script>     <!-- globo de ayuda -->
  <script src="{{ url_for('static', filename='js/tablero.js') }}"></script>     <!-- números del vuelo -->
  <script src="{{ url_for('static', filename='js/pestanas.js') }}"></script>    <!-- pestañas del panel -->
  <script src="{{ url_for('static', filename='js/reservar.js') }}"></script>    <!-- ➕ reservar -->
  <script src="{{ url_for('static', filename='js/modificar.js') }}"></script>   <!-- ✏️ modificar -->
  <script src="{{ url_for('static', filename='js/eliminar.js') }}"></script>    <!-- 🗑️ eliminar -->
  <script src="{{ url_for('static', filename='js/buscar.js') }}"></script>      <!-- 🔍 buscar -->
  <script src="{{ url_for('static', filename='js/listar.js') }}"></script>      <!-- 📋 listar -->
  <script src="{{ url_for('static', filename='js/tren.js') }}"></script>        <!-- 🚂 el tren -->
  <script src="{{ url_for('static', filename='js/progreso.js') }}"></script>    <!-- 🧪 progreso -->
  <!-- 📌 SCRIPTS · cada paso agrega su archivo .js ENCIMA de esta línea -->
  <script src="{{ url_for('static', filename='js/main.js') }}"></script>   <!-- lo creamos en el paso 7 -->
</body>
</html>
```

---

### Categoría: Hojas de Estilo (CSS)

<a id="archivo-static-css-estilos-css"></a>
#### Archivo: `static/css/estilos.css`
- **Ruta relativa:** `static/css/estilos.css`
- **Lenguaje:** `css`

```css
/* ================================================================
   AeroLista ✈ · estilos.css
   Cada paso del proyecto agrega una sección nueva AL FINAL.
   ================================================================ */

/* ---------- PASO 6 · VARIABLES: los colores de toda la app ----------
   :root es la raíz de la página. Las variables empiezan con --
   y se usan así:  color: var(--azul);                                */
:root {
  --noche: #0B1F3A;        /* azul muy oscuro: encabezado */
  --noche-2: #13305A;
  --azul: #2563EB;         /* azul de los botones principales */
  --azul-claro: #DBEAFE;
  --fondo: #EEF2F7;        /* gris azulado del fondo */
  --tarjeta: #FFFFFF;      /* blanco de las tarjetas */
  --texto: #0F172A;        /* texto principal */
  --texto-2: #64748B;      /* texto secundario (gris) */
  --borde: #E2E8F0;

  /* el SEMÁFORO de los asientos: color normal y color oscuro (respaldo) */
  --verde: #22C55E;   --verde-osc: #15803D;   /* libre */
  --ambar: #F59E0B;   --ambar-osc: #B45309;   /* seleccionado */
  --rojo: #EF4444;    --rojo-osc: #B91C1C;    /* reservado */
  --celeste: #3B82F6;                         /* encontrado en una búsqueda */

  --morado: #7C3AED;       /* clase Ejecutiva */
  --sombra: 0 1px 2px rgba(15, 23, 42, .06), 0 4px 16px rgba(15, 23, 42, .06);
  --radio: 14px;           /* redondeo de las tarjetas */
}

/* ---------- BASE: reglas para toda la página ---------- */
* {                          /* * = TODOS los elementos */
  box-sizing: border-box;    /* el ancho incluye relleno y borde (más fácil de calcular) */
  margin: 0;                 /* quitamos los márgenes que el navegador pone solo */
  padding: 0;
}
body {
  font-family: Inter, system-ui, sans-serif;   /* tipo de letra (y respaldos) */
  background: var(--fondo);
  color: var(--texto);
  font-size: 15px;
  line-height: 1.5;                            /* espacio entre líneas */
}
button, input { font: inherit; }               /* botones y campos usan la misma letra */
code { font-family: "Fira Code", monospace; }
[hidden] { display: none !important; }         /* lo que tenga el atributo hidden no se ve */

/* Los marcadores 📌 de los próximos pasos: una caja punteada */
.pronto {
  border: 2px dashed #CBD5E1;
  border-radius: 12px;
  padding: 14px 18px;
  margin: 10px 0;
  color: var(--texto-2);
  font-size: 14px;
  background: rgba(255, 255, 255, .6);
}

/* ---------- ENCABEZADO ---------- */
.barra {
  display: flex;                    /* sus hijos se ponen en FILA */
  align-items: center;              /* centrados verticalmente */
  justify-content: space-between;   /* uno a la izquierda, uno al centro, uno a la derecha */
  gap: 24px;                        /* espacio entre ellos */
  flex-wrap: wrap;                  /* si no caben, bajan a otra línea */
  padding: 14px 28px;
  background: linear-gradient(100deg, var(--noche), var(--noche-2));   /* degradado */
  color: #fff;
}
.marca { display: flex; align-items: center; gap: 12px; }
.marca-logo {
  display: grid; place-items: center;          /* centra el avioncito */
  width: 42px; height: 42px;
  border-radius: 12px;                         /* esquinas redondas */
  background: linear-gradient(135deg, #38BDF8, var(--azul));
  font-size: 22px;
  transform: rotate(-45deg);                   /* gira el ✈ para que "despegue" */
}
.marca b { display: block; font-size: 20px; }
.marca small { color: #93C5FD; font-size: 12px; }
.ruta { display: flex; align-items: center; gap: 14px; }
.ruta-punto { text-align: center; }
.ruta-punto b { display: block; font-size: 24px; letter-spacing: .04em; }
.ruta-punto small { color: #93C5FD; font-size: 12px; }
.ruta-linea { position: relative; width: 120px; border-top: 2px dashed #60A5FA; }   /* la línea punteada */
.ruta-linea span {                             /* el avioncito encima de la línea */
  position: absolute; left: 50%; top: -14px;
  transform: translateX(-50%);
}
.vuelo-datos { display: flex; gap: 10px; flex-wrap: wrap; font-size: 13px; }
.vuelo-datos span { background: rgba(255, 255, 255, .1); padding: 5px 12px; border-radius: 99px; }

/* ---------- DISTRIBUCIÓN: avión a la izquierda, panel a la derecha ----------
   display: grid divide el espacio en COLUMNAS:
   la primera ocupa lo que sobre (1fr) y la segunda mide 420 px fijos.    */
.contenedor {
  display: grid;
  grid-template-columns: minmax(0, 1fr) 420px;
  gap: 24px;
  max-width: 1380px;          /* en pantallas enormes no se estira más */
  margin: 24px auto;          /* auto a los lados = centrado */
  padding: 0 24px;
  align-items: start;
}
.zona-avion {                 /* la tarjeta blanca del avión */
  background: var(--tarjeta);
  border-radius: var(--radio);
  box-shadow: var(--sombra);
  padding: 20px;
  min-width: 0;
}
.pie { text-align: center; color: var(--texto-2); font-size: 12px; padding: 10px 0 30px; }

/* En pantallas angostas (celulares/tablets) todo va en UNA columna */
@media (max-width: 1080px) {
  .contenedor { grid-template-columns: 1fr; }
}

/* ---------- PASO 10 · MENSAJES FLOTANTES (toasts) ---------- */
.toasts {
  position: fixed;                 /* fijo en la pantalla, aunque hagas scroll */
  right: 20px;                     /* pegado a la derecha... */
  bottom: 20px;                    /* ...y abajo */
  display: grid;                   /* los mensajes se apilan uno debajo del otro */
  gap: 10px;
  z-index: 60;                     /* por encima de todo lo demás */
  width: min(380px, calc(100vw - 40px));   /* 380 px, o menos si la pantalla es angosta */
}
.toast {
  display: flex; gap: 10px; align-items: flex-start;
  background: #fff;
  border-left: 5px solid var(--verde);     /* franja de color a la izquierda */
  border-radius: 12px;
  padding: 12px 14px;
  box-shadow: 0 12px 30px rgba(15, 23, 42, .18);
  font-size: 14px;
  animation: entrar .3s ease-out;          /* aparece deslizándose (ver @keyframes abajo) */
}
.toast-aviso, .toast-pendiente { border-color: var(--ambar); }   /* ⚠ y 🚧 en ámbar */
.toast-error, .toast-codigo { border-color: var(--rojo); }       /* ❌ y 💥 en rojo */
.toast-codigo { background: #FEF2F2; }
.toast.saliendo { opacity: 0; transform: translateX(30px); transition: .4s; }   /* se va hacia la derecha */

/* @keyframes define una ANIMACIÓN: aquí, "aparecer subiendo un poquito" */
@keyframes entrar { from { opacity: 0; transform: translateY(6px); } }

/* ---------- PASO 11 · AVISOS FIJOS (para la Etapa E) ---------- */
.aviso-pendiente {
  margin: 0 0 12px; padding: 10px 14px; border-radius: 10px;
  background: #FFF7E6; border: 1px solid #FCD34D; color: #7A4F00; font-size: 14px;   /* ámbar */
}
.aviso-pendiente.es-error { background: #FEF2F2; border-color: #FCA5A5; color: var(--rojo-osc); }  /* rojo */
.aviso-pendiente code { background: rgba(255, 255, 255, .8); padding: 1px 6px; border-radius: 5px; }

/* ---------- PASO 11 · EL AVIÓN ----------
   .avion mide 780 px de ancho (de punta de ala a punta de ala).
   El fuselaje (el cuerpo) mide 380 px y va centrado.            */
.escena {
  position: relative;            /* para que el avión se ubique respecto a la escena */
  overflow: hidden;              /* lo que se salga de la escena no se ve */
  height: 1100px;                /* el paso 13 la ajusta con JavaScript */
  border-radius: 12px;
  background:                    /* el cielo: dos "nubes" (círculos blancos) + degradado */
    radial-gradient(circle at 20% 15%, rgba(255, 255, 255, .9) 0 40px, transparent 41px),
    radial-gradient(circle at 80% 70%, rgba(255, 255, 255, .7) 0 30px, transparent 31px),
    linear-gradient(180deg, #DBEAFE, #EFF6FF 60%, #F8FAFC);
}
.avion {
  --escala: 1;                   /* variables que cambiará JavaScript en el paso 13 */
  --inclinacion: 0deg;
  --giro: 0deg;
  --ajuste-y: 0px;
  position: absolute;            /* lo colocamos en el centro de la escena: */
  left: 50%; top: 50%;
  translate: -50% calc(-50% + var(--ajuste-y));
  width: 780px;
  transform-style: preserve-3d;  /* sus hijos viven en 3D (paso 13) */
  transform: scale(var(--escala)) perspective(1800px) rotateX(var(--inclinacion)) rotateZ(var(--giro));
  transition: transform .8s cubic-bezier(.2, .7, .2, 1), translate .8s;   /* cambios suaves */
}

/* El cuerpo: nariz + cabina + cola, uno debajo del otro */
.fuselaje { position: relative; width: 380px; margin: 0 auto; transform-style: preserve-3d; z-index: 2; }
.nariz {
  position: relative;
  height: 210px;
  background: linear-gradient(90deg, #CBD5E1, #F8FAFC 30%, #FFFFFF 50%, #F8FAFC 70%, #CBD5E1);  /* brillo metálico */
  border-radius: 190px 190px 0 0 / 230px 230px 0 0;     /* esquinas de arriba MUY redondas = media elipse */
}
.parabrisas { position: absolute; top: 62px; left: 50%; translate: -50% 0; display: flex; gap: 6px; }
.parabrisas span { width: 40px; height: 18px; background: linear-gradient(#1E3A5F, #334E72);
                   clip-path: polygon(22% 0, 100% 0, 100% 100%, 0 100%); }   /* ventana inclinada */
.parabrisas span:last-child { clip-path: polygon(0 0, 78% 0, 100% 100%, 0 100%); }   /* su espejo */
.cabina {
  min-height: 600px;             /* alto mínimo mientras no hay filas */
  transform-style: preserve-3d;
  background: linear-gradient(90deg, #CBD5E1 0 6px, #F8FAFC 6px calc(100% - 6px), #CBD5E1 calc(100% - 6px));  /* piel gris a los lados */
  padding: 4px 14px 10px;
}
.cola {
  position: relative;
  height: 230px;
  background: linear-gradient(90deg, #CBD5E1, #F8FAFC 30%, #FFFFFF 50%, #F8FAFC 70%, #CBD5E1);
  clip-path: polygon(0 0, 100% 0, 60% 100%, 40% 100%);   /* TRAPECIO: la cola se angosta */
}
.cola span {                     /* el timón vertical visto desde arriba, con la marca "AL" */
  position: absolute; left: 50%; top: 60px; translate: -50% 0;
  width: 26px; height: 160px; border-radius: 13px 13px 4px 4px;
  background: linear-gradient(var(--azul), var(--noche));
  color: #fff; font: 800 11px Inter; writing-mode: vertical-rl; text-align: center; padding-top: 10px;
}

/* Las alas: rectángulos de 230 x 250 recortados con 4 puntos (x y) */
.ala {
  position: absolute; top: 520px;
  width: 230px; height: 250px;
  background: linear-gradient(180deg, #E2E8F0, #CBD5E1);
  transform: translateZ(-2px);   /* un poquito DEBAJO del cuerpo */
}
.ala-izq { left: 0;  clip-path: polygon(100% 0, 0 72%, 3% 86%, 100% 58%); }
.ala-der { right: 0; clip-path: polygon(0 0, 100% 72%, 97% 86%, 0 58%); }   /* espejo */
.motor {
  position: absolute; top: 560px;
  width: 36px; height: 74px; border-radius: 14px 14px 8px 8px;
  background: linear-gradient(90deg, #94A3B8, #E2E8F0 50%, #94A3B8);
  transform: translateZ(-1px);
}
.motor-izq { left: 110px; }
.motor-der { right: 110px; }
.timon {                         /* alas chiquitas de la cola */
  position: absolute; bottom: 40px;
  width: 140px; height: 120px;
  background: linear-gradient(180deg, #E2E8F0, #CBD5E1);
  transform: translateZ(-2px);
}
.timon-izq { left: 260px;  clip-path: polygon(100% 0, 0 70%, 4% 92%, 100% 62%); }
.timon-der { right: 260px; clip-path: polygon(0 0, 100% 70%, 96% 92%, 0 62%); }
.sombra {                        /* mancha borrosa debajo del avión (solo se ve en 3D) */
  position: absolute; left: 22%; right: 22%; top: 8%; bottom: 8%;
  border-radius: 50%;
  background: rgba(15, 23, 42, .28);
  filter: blur(26px);            /* la desenfoca */
  opacity: 0;                    /* invisible por ahora */
  transform: translateZ(-120px); /* 120 px debajo del avión */
  transition: opacity .8s;
}

/* ---------- PASO 12 · FILAS Y ASIENTOS ---------- */
.servicio { display: flex; justify-content: space-between; padding: 8px 6px; font-size: 11px; color: var(--texto-2); }
.servicio span { background: #E2E8F0; border-radius: 6px; padding: 3px 8px; }
.separador {                     /* el letrero EJECUTIVA / ECONÓMICA con líneas a los lados */
  display: flex; align-items: center; gap: 8px; margin: 6px 0 4px;
  font-size: 10px; font-weight: 800; letter-spacing: .14em; color: var(--texto-2);
}
.separador::before, .separador::after { content: ""; flex: 1; border-top: 1px dashed #CBD5E1; }   /* las líneas */
.separador.ejecutiva { color: var(--morado); }

.fila {
  position: relative;
  display: grid;                                    /* la fila es una cuadrícula de 5 columnas: */
  grid-template-columns: 10px 1fr 30px 1fr 10px;   /* ventana · lado izq · pasillo · lado der · ventana */
  align-items: center;
  gap: 6px;
  height: 46px;
  transform-style: preserve-3d;
}
.fila-ejecutiva { height: 56px; }                   /* las filas ejecutivas son más altas */
.ventana { width: 7px; height: 20px; border-radius: 4px; background: linear-gradient(#93C5FD, #60A5FA); justify-self: center; }
.lado { display: flex; justify-content: center; gap: 6px; transform-style: preserve-3d; }   /* asientos en fila */
.numero-fila { text-align: center; font: 600 11px "Fira Code", monospace; color: #94A3B8; }
/* ::before y ::after crean "cajitas" extra sin tocar el HTML: el letrero SALIDA a cada lado */
.fila-salida::before, .fila-salida::after {
  content: "SALIDA"; position: absolute; top: 50%; translate: 0 -50%;
  font-size: 8px; font-weight: 800; letter-spacing: .08em; color: #fff;
  background: var(--rojo-osc); padding: 2px 4px; border-radius: 4px;
}
.fila-salida::before { left: -54px; }
.fila-salida::after { right: -54px; }

/* EL ASIENTO. Usa dos variables: --color (el asiento) y --color-osc (el respaldo).
   Cada estado del semáforo solo cambia esas dos variables.                        */
.asiento {
  --color: var(--verde);
  --color-osc: var(--verde-osc);
  position: relative;
  width: 36px; height: 38px;
  border: 0;
  border-radius: 10px 10px 6px 6px;
  background: var(--color);
  color: rgba(255, 255, 255, .95);
  font-size: 11px; font-weight: 700;
  cursor: pointer;                                  /* manito al pasar el mouse */
  /* franja oscura abajo = el respaldo (los pasajeros miran hacia la nariz) + sombrita */
  box-shadow: inset 0 -9px 0 var(--color-osc), 0 2px 3px rgba(15, 23, 42, .25);
  transform-style: preserve-3d;
  transition: background .35s, box-shadow .35s, transform .15s;   /* el color cambia suave */
  display: grid; place-items: center;
}
.fila-ejecutiva .asiento { width: 46px; height: 46px; border-radius: 12px 12px 7px 7px; }   /* más anchos */
.asiento:hover { transform: translateY(-3px) scale(1.08); }   /* :hover = cuando el mouse está encima */

/* 🚦 EL SEMÁFORO (parte 1): libre y seleccionado */
.asiento.libre        { --color: var(--verde); --color-osc: var(--verde-osc); }
.asiento.seleccionado { --color: var(--ambar); --color-osc: var(--ambar-osc); animation: latido 1.1s ease-in-out infinite; }
@keyframes latido { 50% { transform: translateY(-3px) scale(1.1); } }   /* "late" para llamar la atención */

/* ---------- PASO 13 · LEYENDA, BOTONES DE VISTA Y 3D ---------- */
.controles { display: flex; justify-content: space-between; align-items: center; gap: 12px; margin: 4px 0 8px; flex-wrap: wrap; }
.leyenda { display: flex; gap: 14px; list-style: none; flex-wrap: wrap; font-size: 13px; color: var(--texto-2); }  /* sin viñetas */
.leyenda li { display: flex; align-items: center; gap: 6px; }
.muestra { width: 16px; height: 16px; border-radius: 5px; }          /* el cuadradito de color */
.muestra.libre { background: var(--verde); }
.muestra.seleccionado { background: var(--ambar); }
.muestra.ocupado { background: var(--rojo); }
.muestra.encontrado { background: var(--rojo); outline: 3px solid var(--celeste); outline-offset: 1px; }
.vistas { display: flex; background: var(--fondo); padding: 4px; border-radius: 10px; }
.btn-vista { border: 0; background: none; padding: 6px 14px; border-radius: 8px; cursor: pointer; font-weight: 600; font-size: 13px; color: var(--texto-2); }
.btn-vista.activo { background: #fff; color: var(--texto); box-shadow: var(--sombra); }   /* el botón elegido */
.ajustes-3d { display: flex; gap: 22px; font-size: 13px; color: var(--texto-2); flex-wrap: wrap; margin-bottom: 10px; }
.ajustes-3d label { display: flex; align-items: center; gap: 8px; }
.ajustes-3d input { accent-color: var(--azul); width: 140px; }

/* En 3D: se ve la sombra y los asientos "flotan" 3 px sobre el piso */
.avion.vista-3d .sombra { opacity: 1; }
.avion.vista-3d .asiento { transform: translateZ(3px); }
.avion.vista-3d .asiento:hover { transform: translateZ(14px) scale(1.1); }

/* ---------- PASO 15 · EL SEMÁFORO (parte 2): reservado, encontrado, editando ---------- */
.asiento.ocupado    { --color: var(--rojo); --color-osc: var(--rojo-osc); }           /* 🔴 */
.asiento.encontrado { outline: 3px solid var(--celeste); outline-offset: 2px; animation: pulso 1.2s ease-out infinite; }  /* 🔵 */
.asiento.editando   { outline: 3px dashed var(--ambar); outline-offset: 2px; }        /* el que se está modificando */

/* El icono del pasajero (el SVG) */
.icono-pasajero { width: 22px; height: 22px; margin-top: -6px; fill: #fff; transform-origin: 50% 100%; transition: transform .8s; }
.asiento.menor .icono-pasajero { width: 17px; height: 17px; }        /* menores de edad: más pequeño */
.fila-ejecutiva .icono-pasajero { width: 26px; height: 26px; }

/* En 3D el icono hace el giro CONTRARIO al avión → queda "de pie" mirándote */
.avion.vista-3d .asiento.ocupado .icono-pasajero {
  transform: translateZ(2px) rotateZ(calc(-1 * var(--giro))) rotateX(calc(-1 * var(--inclinacion)));
}

/* Animaciones: al reservar el asiento "salta"; al liberarlo, "brilla" */
.asiento.recien { animation: aparecer .7s cubic-bezier(.2, 1.6, .4, 1); }
.asiento.liberado { animation: liberar .9s ease-out; }
@keyframes pulso {      /* una onda azul que se expande alrededor del asiento encontrado */
  0%   { box-shadow: inset 0 -9px 0 var(--color-osc), 0 0 0 0 rgba(59, 130, 246, .7); }
  100% { box-shadow: inset 0 -9px 0 var(--color-osc), 0 0 0 14px rgba(59, 130, 246, 0); }
}
@keyframes aparecer { 0% { transform: scale(.3); } 100% { transform: scale(1); } }
@keyframes liberar { 0% { transform: scale(1.3); filter: brightness(1.6); } 100% { transform: scale(1); } }

/* ---------- PASO 16 · GLOBO DE AYUDA (tooltip) ---------- */
.tooltip {
  position: fixed;                 /* se ubica según la pantalla (lo mueve JavaScript) */
  z-index: 50;
  pointer-events: none;            /* el mouse lo "atraviesa": no tapa los clics */
  background: var(--noche); color: #fff;
  padding: 8px 12px; border-radius: 10px;
  font-size: 12.5px; line-height: 1.4;
  box-shadow: 0 8px 20px rgba(0, 0, 0, .25);
  max-width: 240px;
}
.tooltip b { font-size: 14px; }
.tooltip small { color: #93C5FD; }

/* ---------- PASO 17 · TABLERO ---------- */
.tablero { display: grid; grid-template-columns: repeat(4, 1fr); gap: 12px; margin-bottom: 14px; }  /* 4 columnas iguales */
.dato { background: var(--fondo); border-radius: 12px; padding: 10px 14px; }
.dato small { display: block; color: var(--texto-2); font-size: 12px; font-weight: 600; text-transform: uppercase; letter-spacing: .04em; }
.dato b { font-size: 26px; letter-spacing: -.02em; }
.ocupacion { display: flex; align-items: center; gap: 8px; }
.luz { width: 14px; height: 14px; border-radius: 50%; background: #CBD5E1; }      /* el foquito apagado */
.luz.verde { background: var(--verde); box-shadow: 0 0 10px var(--verde); }        /* encendido: brilla */
.luz.ambar { background: var(--ambar); box-shadow: 0 0 10px var(--ambar); }
.luz.rojo  { background: var(--rojo);  box-shadow: 0 0 10px var(--rojo); }
.barra-ocupacion { height: 6px; background: var(--borde); border-radius: 9px; margin-top: 4px; overflow: hidden; }
.barra-ocupacion span { display: block; height: 100%; width: 0; background: var(--verde); transition: width .6s, background .6s; }
@media (max-width: 640px) { .tablero { grid-template-columns: repeat(2, 1fr); } }   /* celulares: 2 columnas */

/* ---------- PASO 18 · PANEL Y PESTAÑAS ---------- */
.panel {
  background: var(--tarjeta); border-radius: var(--radio); box-shadow: var(--sombra);
  position: sticky; top: 16px;        /* sticky: se queda "pegado" arriba al hacer scroll */
  overflow: hidden;
}
.pestanas { display: grid; grid-template-columns: repeat(5, 1fr); background: var(--fondo); padding: 6px; gap: 4px; }
.pestana {
  border: 0; background: none; padding: 9px 4px; border-radius: 9px; cursor: pointer;
  font-size: 12px; font-weight: 600; color: var(--texto-2); white-space: nowrap;   /* nowrap: sin saltos de línea */
}
.pestana:hover { color: var(--texto); }
.pestana.activa { background: #fff; color: var(--azul); box-shadow: var(--sombra); }   /* la pestaña abierta */
.contenido-pestana { display: none; padding: 20px; }                   /* TODAS escondidas... */
.contenido-pestana.activa { display: block; animation: entrar .3s ease-out; }   /* ...menos la activa */
.contenido-pestana h2 { font-size: 18px; margin-bottom: 6px; }
.ayuda { font-size: 13px; color: var(--texto-2); margin-bottom: 14px; }
.panel > .pronto { margin: 0 20px 20px; }
@media (max-width: 1080px) { .panel { position: static; } }   /* en pantallas angostas no se queda pegado */
@media (max-width: 640px) { .pestana { font-size: 11px; } }

/* ---------- PASO 19 · FORMULARIOS Y BOTONES ---------- */
.txt-verde { color: var(--verde-osc); }
.formulario { display: grid; gap: 12px; margin-bottom: 14px; }            /* un campo debajo del otro */
.formulario label { display: grid; gap: 4px; font-size: 13px; font-weight: 600; color: var(--texto-2); }
.formulario input:not([type="radio"]) {                                  /* todos los campos menos los radio */
  width: 100%; padding: 10px 12px;
  border: 1.5px solid var(--borde); border-radius: 10px;
  background: #fff; color: var(--texto); font-size: 15px;
  outline: none; transition: border-color .2s, box-shadow .2s;
}
.formulario input:focus { border-color: var(--azul); box-shadow: 0 0 0 4px var(--azul-claro); }   /* :focus = mientras escribes */
.formulario input:disabled { background: var(--fondo); color: var(--texto-2); }
.dos-campos { display: grid; grid-template-columns: 1fr 110px; gap: 10px; }   /* cédula ancha + edad angosta */
.fila-busqueda { display: flex; gap: 8px; }
.fila-busqueda input { flex: 1; }                                          /* el campo ocupa lo que sobre */
.botones { display: flex; gap: 8px; flex-wrap: wrap; }

.btn { border: 0; border-radius: 10px; padding: 10px 16px; font-weight: 700; font-size: 14px; cursor: pointer; transition: transform .1s, filter .2s; }
.btn:hover { filter: brightness(1.06); }
.btn:active { transform: scale(.97); }                                     /* se "hunde" al hacer clic */
.btn-primario { background: linear-gradient(135deg, #3B82F6, var(--azul)); color: #fff; }
.btn-peligro { background: linear-gradient(135deg, #F87171, var(--rojo)); color: #fff; }
.btn-suave { background: var(--fondo); color: var(--texto); }

.asiento-elegido {
  display: flex; align-items: center; gap: 12px;
  background: var(--fondo); padding: 10px 12px; border-radius: 12px;
  font-size: 13px; font-weight: 600; color: var(--texto-2);
}
.chip-asiento {                                                            /* el asiento elegido, en grande */
  min-width: 52px; text-align: center; padding: 6px 10px; border-radius: 9px;
  background: var(--ambar); color: #fff; font: 700 17px "Fira Code", monospace;
  box-shadow: inset 0 -5px 0 var(--ambar-osc);
}
.chip-asiento.vacio { background: #CBD5E1; box-shadow: inset 0 -5px 0 #94A3B8; }   /* gris si no hay ninguno */

/* ---------- PASO 21 · VENTANA DE CONFIRMACIÓN (dialog) ---------- */
.modal { margin: auto; border: 0; border-radius: 16px; padding: 22px; width: min(420px, 92vw); box-shadow: 0 30px 60px rgba(0, 0, 0, .3); }
.modal::backdrop { background: rgba(11, 31, 58, .55); backdrop-filter: blur(2px); }   /* el fondo oscuro y borroso */
.modal h3 { margin-bottom: 8px; }
.modal p { color: var(--texto-2); margin-bottom: 18px; }
.modal .botones { justify-content: flex-end; }                                         /* botones a la derecha */

/* ---------- PASO 22 · BUSCADOR Y PASE DE ABORDAR ---------- */
/* Botones de opción "segmentados": escondemos el círculo del radio y pintamos el texto */
.segmentado { display: grid; grid-template-columns: repeat(3, 1fr); background: var(--fondo); padding: 4px; border-radius: 10px; }
.segmentado label { display: block; }
.segmentado input { position: absolute; opacity: 0; pointer-events: none; }   /* el círculo, invisible */
.segmentado span { display: block; text-align: center; padding: 7px; border-radius: 8px; cursor: pointer; font-size: 13px; }
.segmentado input:checked + span { background: #fff; color: var(--azul); box-shadow: var(--sombra); }   /* + = el hermano de al lado */
.resultado .vacio { text-align: center; color: var(--texto-2); padding: 18px; background: var(--fondo); border-radius: 12px; }

/* El pase de abordar: cuerpo + talón */
.pase {
  display: grid; grid-template-columns: 1fr 92px;
  border-radius: 14px; overflow: hidden;
  background: #fff; border: 1px solid var(--borde);
  box-shadow: 0 10px 24px rgba(15, 23, 42, .1);
  animation: entrar .35s ease-out;
}
.pase-cuerpo { padding: 14px 16px; position: relative; }
.pase-cuerpo::after {                     /* la línea perforada entre el pase y el talón */
  content: ""; position: absolute; right: -1px; top: 10px; bottom: 10px;
  border-right: 2px dashed var(--borde);
}
.pase-cabecera { display: flex; justify-content: space-between; font-size: 11px; font-weight: 800; letter-spacing: .08em; color: var(--azul); margin-bottom: 8px; }
.pase-nombre { font-size: 20px; font-weight: 800; }
.pase-ruta { display: flex; align-items: center; gap: 8px; margin: 6px 0; font: 700 15px "Fira Code", monospace; }
.pase-ruta i { flex: 1; border-top: 2px dotted #CBD5E1; }
.pase-detalle { display: grid; grid-template-columns: repeat(3, auto); gap: 2px 14px; font-size: 12px; color: var(--texto-2); }
.pase-detalle b { display: block; color: var(--texto); font-size: 13px; }
.pase-talon { background: linear-gradient(160deg, var(--noche), var(--azul)); color: #fff; display: grid; place-content: center; text-align: center; gap: 2px; }
.pase-talon small { font-size: 10px; letter-spacing: .1em; opacity: .8; }
.pase-talon b { font: 800 28px "Fira Code", monospace; }
.pase-talon em { font-style: normal; font-size: 11px; background: rgba(255, 255, 255, .18); border-radius: 99px; padding: 1px 8px; }
.pase.ejecutiva .pase-talon { background: linear-gradient(160deg, #2E1065, var(--morado)); }   /* morado en Ejecutiva */
.pase-acciones { display: flex; gap: 8px; margin-top: 12px; }
.pase-acciones .btn { flex: 1; }

/* ---------- PASO 23 · TABLA Y FILTROS ---------- */
.filtros { display: flex; gap: 6px; margin: 4px 0 12px; }
.chip { border: 1.5px solid var(--borde); background: #fff; border-radius: 99px; padding: 5px 14px; font-size: 13px; font-weight: 600; cursor: pointer; color: var(--texto-2); }
.chip.activo { background: var(--azul); border-color: var(--azul); color: #fff; }
.tabla-caja { max-height: 330px; overflow: auto; border: 1px solid var(--borde); border-radius: 12px; }   /* scroll si es larga */
table { width: 100%; border-collapse: collapse; font-size: 13px; }
th { position: sticky; top: 0; background: var(--fondo); text-align: left; padding: 8px 10px;   /* el encabezado no se mueve */
     font-size: 11px; text-transform: uppercase; letter-spacing: .05em; color: var(--texto-2); }
td { padding: 8px 10px; border-top: 1px solid var(--borde); }
tbody tr { cursor: pointer; }
tbody tr:hover td { background: #F8FAFC; }
.etiqueta-asiento { font: 700 12px "Fira Code", monospace; background: var(--rojo); color: #fff; padding: 2px 7px; border-radius: 6px; }
.etiqueta-asiento.ejecutiva { background: var(--morado); }
td.sin-datos { text-align: center; color: var(--texto-2); padding: 20px; cursor: default; }

/* ---------- PASO 24 · EL TREN (la lista enlazada) ---------- */
.tren { background: var(--tarjeta); border-radius: var(--radio); box-shadow: var(--sombra);
        max-width: 1332px; margin: 0 auto 24px; padding: 18px 22px 22px; }
.tren-titulo { display: flex; justify-content: space-between; align-items: center; gap: 12px; margin-bottom: 14px; flex-wrap: wrap; }
.tren-titulo h2 { font-size: 17px; }
.tren-titulo code { background: var(--azul-claro); color: var(--azul); padding: 3px 10px; border-radius: 8px; font-size: 13px; }
.tren-vagones { display: flex; flex-wrap: wrap; align-items: center; gap: 8px 6px; }   /* si no caben, bajan de línea */
.etiqueta-head { font: 700 13px "Fira Code", monospace; color: #fff; background: var(--morado); padding: 6px 10px; border-radius: 8px; }
.vagon {                              /* un nodo: [ data | next ] */
  display: grid; grid-template-columns: auto 26px;
  border: 2px solid var(--noche); border-radius: 10px;
  overflow: hidden; background: #fff; cursor: pointer; transition: transform .15s;
}
.vagon:hover { transform: translateY(-3px); }
.vagon-data { padding: 4px 10px; line-height: 1.25; }
.vagon-data small { display: block; font: 600 10px "Fira Code", monospace; color: var(--texto-2); }
.vagon-data b { font-size: 13px; }
.vagon-data span { display: block; font-size: 11px; color: var(--texto-2); }
.vagon-next { background: var(--noche); color: #fff; display: grid; place-items: center; font-size: 13px; }   /* la parte "next" */
.vagon.nuevo { animation: aparecer .7s cubic-bezier(.2, 1.6, .4, 1); border-color: var(--verde); }   /* el recién llegado */
.flecha { color: var(--noche); font-weight: 800; }
.nodo-none { font: 700 13px "Fira Code", monospace; color: var(--texto-2); border: 2px dashed #CBD5E1; padding: 6px 10px; border-radius: 8px; }
.tren-vacio { color: #7A4F00; background: #FFF7E6; border: 1px solid #FCD34D; padding: 12px 16px; border-radius: 10px; }
@media (max-width: 1380px) { .tren { margin: 0 24px 24px; } }

/* ---------- PASO 30 · PROGRESO DEL BACKEND ---------- */
.progreso { border-top: 1px solid var(--borde); padding: 16px 20px 20px; background: #FBFCFE; }
.progreso-titulo { display: flex; justify-content: space-between; align-items: center; }
.progreso-titulo h3 { font-size: 15px; }
.progreso-titulo b { font: 700 14px "Fira Code", monospace; color: var(--azul); }
.progreso-barra { height: 8px; background: var(--borde); border-radius: 9px; overflow: hidden; margin: 8px 0 10px; }
.progreso-barra span { display: block; height: 100%; width: 0; background: linear-gradient(90deg, #34D399, var(--verde)); transition: width .6s; }
#progreso-lista { list-style: none; display: grid; gap: 4px; margin-bottom: 12px; }
#progreso-lista li { display: grid; grid-template-columns: 22px 1fr; gap: 2px 6px; font-size: 13px; }   /* icono | texto */
#progreso-lista li small { grid-column: 2; color: var(--texto-2); font-size: 12px; }
#progreso-lista li.fallo small, #progreso-lista li.error small, #progreso-lista li.lento small { color: var(--rojo-osc); }
```

---

### Categoría: Frontend y Módulos de Cliente (JavaScript)

<a id="archivo-static-js-api-js"></a>
#### Archivo: `static/js/api.js`
- **Ruta relativa:** `static/js/api.js`
- **Lenguaje:** `javascript`

```javascript
// ================================================================
//  AeroLista ✈ · api.js — cómo HABLA la página con Flask
//  Cada función del objeto API hace un pedido (fetch) a una ruta
//  /api/... de app.py y devuelve el JSON que contesta Flask:
//     { ok: true,  datos: ..., mensaje: "...", avisos: [...] }
//     { ok: false, mensaje: "..." }
// ================================================================

const SEGUNDOS_DE_ESPERA = 6;      // si Flask tarda más que esto, nos rendimos

// async: esta función tarda (viaja por la red), así que trabaja "en segundo plano".
// Dentro de una función async podemos usar await = "espera aquí hasta que llegue".
async function pedir(url, metodo = "GET", cuerpo = null, espera = SEGUNDOS_DE_ESPERA) {
  // Un "controlador" nos permite CANCELAR el pedido si tarda demasiado
  const controlador = new AbortController();
  const reloj = setTimeout(() => controlador.abort(), espera * 1000);   // alarma en 6 s

  // Opciones del pedido: método (GET, POST, PUT, DELETE) y encabezados
  const opciones = { method: metodo, signal: controlador.signal, headers: {} };
  if (cuerpo !== null) {                                 // si hay datos para ENVIAR...
    opciones.headers["Content-Type"] = "application/json";   // ...avisamos que van en JSON
    opciones.body = JSON.stringify(cuerpo);              // objeto JS → texto JSON
  }

  try {                                                  // try: "intenta esto..."
    const respuesta = await fetch(url, opciones);       // 1. viaja a Flask y espera la respuesta
    return await respuesta.json();                       // 2. texto JSON → objeto de JavaScript
  } catch (error) {                                      // catch: "...y si algo falla, haz esto"
    if (error.name === "AbortError") {                   // sonó la alarma: tardó demasiado
      return { ok: false, error_codigo: true,
               mensaje: `Flask no respondió en ${espera} s. ¿Hay un bucle infinito? ` +
                        "Revisa que avances con actual = actual.next y reinicia Flask (Ctrl+C)." };
    }
    return { ok: false, error_codigo: true,              // no hubo conexión
             mensaje: "No hay conexión con Flask. ¿Está encendido? → python app.py" };
  } finally {                                            // finally: se ejecuta SIEMPRE al final
    clearTimeout(reloj);                                 // apagamos la alarma
  }
}

// El objeto API empieza vacío. Cada función se agrega como API.nombre = ...
const API = {};

// GET /api/vuelo → los datos del avión (filas, letras...)
API.vuelo = () => pedir("/api/vuelo");

// ---- 🆕 Paso 29: las funciones de verdad (reemplazan a las de datos-demo.js) ----
// encodeURIComponent protege los textos para que viajen bien en la dirección (tildes, espacios...)
API.resumen       = ()              => pedir("/api/resumen");                          // longitud, esta_vacia
API.listar        = (clase = "")    => pedir("/api/reservas" + (clase ? "?clase=" + encodeURIComponent(clase) : ""));
API.buscarNumero  = (numero)        => pedir("/api/reservas/" + numero);               // buscar
API.buscarAsiento = (asiento)       => pedir("/api/asientos/" + encodeURIComponent(asiento));   // buscar_por_asiento
API.buscarCedula  = (cedula)        => pedir("/api/cedulas/" + encodeURIComponent(cedula));     // buscar_por_cedula
API.reservar      = (datos)         => pedir("/api/reservas", "POST", datos);          // ingresar ✅
API.modificar     = (numero, datos) => pedir("/api/reservas/" + numero, "PUT", datos); // modificar
API.eliminar      = (numero)        => pedir("/api/reservas/" + numero, "DELETE");     // eliminar
API.restaurar     = ()              => pedir("/api/restaurar", "POST");

// ---- 🆕 Paso 30: el panel de progreso (le damos 20 s porque corre 9 pruebas) ----
API.progreso = () => pedir("/api/progreso", "GET", null, 20);
```

---

<a id="archivo-static-js-main-js"></a>
#### Archivo: `static/js/main.js`
- **Ruta relativa:** `static/js/main.js`
- **Lenguaje:** `javascript`

```javascript
// ================================================================
//  AeroLista ✈ · main.js  —  el "director de orquesta"
//  Versión 8 (final): avión + tablero + tren + progreso.
// ================================================================

// ESTADO: la "memoria" de la página
const estado = {
  vuelo: null,          // los datos del avión (de /api/vuelo)
  vista: "3d",          // "3d" o "2d"
  reservas: [],         // la lista de pasajeros (en orden de llegada)
  porAsiento: {},       // atajo para pintar: { "7D": pasajero, ... }
  seleccionado: null,   // asiento libre elegido con clic (🟡)
  encontrado: null,     // asiento resaltado por una búsqueda (🔵)
  enEdicion: null,      // pasajero que se está modificando
  pestana: "reservar",  // pestaña abierta del panel
  filtroClase: "",      // filtro de la pestaña Listar
};

// Vuelve a pedir todo lo que puede cambiar después de reservar, modificar o eliminar
async function refrescar(numeroNuevo = null) {
  const r = await cargarReservas();       // lista → avión pintado (semaforo.js)
  dibujarTren(r, numeroNuevo);            // la misma lista → tren (tren.js)
  await cargarResumen();                  // tablero (tablero.js)
}

// Arranque de la aplicación
async function iniciar() {
  const r = await API.vuelo();            // 1. datos del avión
  if (!r.ok) {
    avisar(r.mensaje, "codigo");
    return;
  }
  estado.vuelo = r.datos;
  dibujarAvion();                         // 2. filas y asientos (avion.js)
  aplicarAngulos();                       // 3. vista 3D (vista3d.js)
  await refrescar();                      // 4. reservas, tren y tablero
  avisar(`Vuelo ${estado.vuelo.codigo} listo · haz clic en un asiento`);
  await cargarProgreso();                 // 5. 🆕 pruebas de tus métodos (progreso.js)
}

iniciar();
```

---

<a id="archivo-static-js-avion-js"></a>
#### Archivo: `static/js/avion.js`
- **Ruta relativa:** `static/js/avion.js`
- **Lenguaje:** `javascript`

```javascript
// ================================================================
//  AeroLista ✈ · avion.js — crea las filas y los asientos
//  Usa estado.vuelo (filas, letras...) que llegó de Flask.
// ================================================================

// Dibuja TODO el interior del avión dentro de <div id="cabina">
function dibujarAvion() {
  const cabina = $("#cabina");
  const v = estado.vuelo;                    // atajo: v.filas, v.letras_economica, ...
  cabina.innerHTML = "";                     // empezamos con la cabina vacía

  cabina.append(crearServicio("🍽 Cocina", "🚻 Baño"));   // zona de servicio adelante

  // Recorrido de las filas: 1, 2, 3, ..., 12
  for (let fila = 1; fila <= v.filas; fila++) {
    // Antes de la fila 1 ponemos el letrero "EJECUTIVA"
    if (fila === 1) cabina.append(crear("div", "separador ejecutiva", "EJECUTIVA"));
    // Antes de la primera fila económica (la 3) ponemos "ECONÓMICA"
    if (fila === v.filas_ejecutiva + 1) cabina.append(crear("div", "separador", "ECONÓMICA"));
    cabina.append(crearFila(fila));          // creamos la fila y la metemos en la cabina
  }

  cabina.append(crearServicio("🚻 Baño", "🍽 Cocina"));   // zona de servicio atrás
}

// Una franja con dos letreros (cocina y baño)
function crearServicio(izquierda, derecha) {
  const zona = crear("div", "servicio");
  zona.append(crear("span", "", izquierda), crear("span", "", derecha));
  return zona;
}

// Crea UNA fila:   ventana | A B C | 7 | D E F | ventana
function crearFila(fila) {
  const v = estado.vuelo;
  const esEjecutiva = fila <= v.filas_ejecutiva;                         // ¿fila 1 o 2?
  const letras = esEjecutiva ? v.letras_ejecutiva : v.letras_economica;  // ["A","C","D","F"] o ["A",...,"F"]

  // La fila es un div. Las filas ejecutivas llevan una clase extra (asientos más grandes)
  const divFila = crear("div", esEjecutiva ? "fila fila-ejecutiva" : "fila");
  // includes() pregunta si la lista contiene ese valor: ¿es fila de salida de emergencia?
  if (v.filas_salida.includes(fila)) divFila.classList.add("fila-salida");

  const izquierda = crear("div", "lado");    // lado izquierdo del pasillo
  const derecha = crear("div", "lado");      // lado derecho del pasillo

  // Recorrido de las letras de esta fila
  for (const letra of letras) {
    const boton = crearAsiento(fila + letra);   // 7 + "D" → "7D"
    if (letra < "D") izquierda.append(boton);   // A, B, C van a la izquierda ("A" < "D" es true)
    else derecha.append(boton);                 // D, E, F van a la derecha
  }

  // Armamos la fila en orden: ventana, lado izq, número, lado der, ventana
  divFila.append(crear("span", "ventana"), izquierda,
                 crear("span", "numero-fila", String(fila)),
                 derecha, crear("span", "ventana"));
  return divFila;
}

// Crea UN asiento: <button class="asiento libre" data-asiento="7D">D</button>
function crearAsiento(codigo) {
  const boton = crear("button", "asiento libre", codigo.slice(-1));   // slice(-1) = última letra
  boton.type = "button";                     // un botón normal (no envía formularios)
  boton.dataset.asiento = codigo;            // guarda el código en data-asiento="7D"
  // Cuando le hagan clic, llama a clicEnAsiento con SU código (está en clic.js)
  boton.addEventListener("click", () => clicEnAsiento(codigo));
  return boton;
}
```

---

<a id="archivo-static-js-reservar-js"></a>
#### Archivo: `static/js/reservar.js`
- **Ruta relativa:** `static/js/reservar.js`
- **Lenguaje:** `javascript`

```javascript
// ================================================================
//  AeroLista ✈ · reservar.js — crear una reserva nueva
// ================================================================

// Escribe en el formulario el asiento elegido (o "—" si no hay ninguno)
function actualizarAsientoElegido() {
  const salida = $("#reservar-asiento");
  if (estado.seleccionado) {
    salida.textContent = estado.seleccionado;              // "9C"
    salida.classList.remove("vacio");
    $("#reservar-clase").textContent = "Clase " + claseDeAsiento(estado.seleccionado);
  } else {
    salida.textContent = "—";
    salida.classList.add("vacio");
    $("#reservar-clase").textContent = "Elige uno en el avión";
  }
}

// Se ejecuta al enviar el formulario. "evento" trae información del envío.
async function reservar(evento) {
  evento.preventDefault();          // ⚠ MUY IMPORTANTE: evita que la página se recargue

  if (!estado.seleccionado) {       // sin asiento no se puede reservar
    avisar("Primero haz clic en un asiento verde del avión", "aviso");
    return;
  }

  // 1. LEER: evento.target es el formulario; .nombre es el input con name="nombre"
  const formulario = evento.target;
  const datos = {
    nombre: formulario.nombre.value.trim(),     // trim() quita espacios al inicio y al final
    cedula: formulario.cedula.value.trim(),
    edad: formulario.edad.value,
    asiento: estado.seleccionado,
  };

  // 2. LLAMAR: pedimos la reserva (por ahora al servidor de mentira; en el paso 29, a Flask)
  const r = await API.reservar(datos);

  // 3. MOSTRAR
  if (!r.ok) return mostrarProblema(r);         // ❌ algo salió mal: avisamos y salimos
  avisar(r.mensaje, "ok");                      // ✅
  for (const aviso of r.avisos) avisar(aviso, "aviso");   // ⚠ advertencias (si las hay)
  formulario.reset();                           // limpia los campos
  estado.seleccionado = null;                   // ya no hay asiento elegido
  actualizarAsientoElegido();
  await refrescar(r.datos.numero);              // vuelve a pintar todo
  animarAsiento(r.datos.asiento, "recien");     // el asiento nuevo "salta"
}

// ---- Conexiones ----
// "submit" se dispara al pulsar el botón del formulario (o Enter en un campo)
$("#form-reservar").addEventListener("submit", reservar);

// Si te vas a otra pestaña, el asiento amarillo se suelta
for (const boton of $$(".pestana")) {
  boton.addEventListener("click", () => {
    if (boton.dataset.pestana !== "reservar" && estado.seleccionado && !estado.enEdicion) {
      estado.seleccionado = null;
      actualizarAsientoElegido();
      pintarAsientos();
    }
  });
}
```

---

<a id="archivo-static-js-modificar-js"></a>
#### Archivo: `static/js/modificar.js`
- **Ruta relativa:** `static/js/modificar.js`
- **Lenguaje:** `javascript`

```javascript
// ================================================================
//  AeroLista ✈ · modificar.js — cambiar nombre, edad o asiento
// ================================================================

// Al enviar el primer formulario (Cargar)
async function enviarCargar(evento) {
  evento.preventDefault();
  await cargarParaModificar($("#modificar-numero").value);
}

// Busca la reserva y llena el segundo formulario con sus datos
async function cargarParaModificar(numero) {
  const r = await API.buscarNumero(numero);     // en Flask usará buscar() de tu lista
  if (!r.ok) return mostrarProblema(r);
  const p = r.datos;
  estado.enEdicion = p;                         // la página recuerda a quién modificamos
  estado.seleccionado = null;
  const formulario = $("#form-modificar");
  formulario.nombre.value = p.nombre;           // llenamos los campos
  formulario.cedula.value = p.cedula;
  formulario.edad.value = p.edad;
  $("#modificar-asiento").textContent = p.asiento;
  formulario.hidden = false;                    // mostramos el formulario
  resaltar(p.asiento);                          // 🔵 y borde punteado en su asiento
}

// Al enviar el segundo formulario (Guardar cambios)
async function guardarCambios(evento) {
  evento.preventDefault();
  const p = estado.enEdicion;
  const formulario = evento.target;
  const datos = {                               // 1. LEER
    nombre: formulario.nombre.value.trim(),
    edad: formulario.edad.value,
    // || significa "o": si no eligió otro asiento, se queda en el suyo
    asiento: estado.seleccionado || p.asiento,
  };
  const r = await API.modificar(p.numero, datos);   // 2. LLAMAR
  if (!r.ok) return mostrarProblema(r);             // 3. MOSTRAR
  avisar(r.mensaje, "ok");
  for (const aviso of r.avisos) avisar(aviso, "aviso");
  cancelarEdicion();
  await refrescar();
  animarAsiento(r.datos.asiento, "recien");
  resaltar(r.datos.asiento);
}

// Sale del modo edición y esconde el formulario
function cancelarEdicion() {
  estado.enEdicion = null;
  estado.seleccionado = null;
  $("#form-modificar").hidden = true;
  pintarAsientos();
}

// ---- Conexiones ----
$("#form-cargar").addEventListener("submit", enviarCargar);
$("#form-modificar").addEventListener("submit", guardarCambios);
$("#btn-cancelar-edicion").addEventListener("click", cancelarEdicion);
// Si te vas a otra pestaña en medio de una edición, se cancela
for (const boton of $$(".pestana")) {
  boton.addEventListener("click", () => {
    if (boton.dataset.pestana !== "modificar" && estado.enEdicion) cancelarEdicion();
  });
}
```

---

<a id="archivo-static-js-eliminar-js"></a>
#### Archivo: `static/js/eliminar.js`
- **Ruta relativa:** `static/js/eliminar.js`
- **Lenguaje:** `javascript`

```javascript
// ================================================================
//  AeroLista ✈ · eliminar.js — cancelar una reserva
// ================================================================

// Abre la ventana <dialog> y ESPERA la respuesta del usuario.
// Retorna una promesa que se cumple con true (Sí) o false (No).
function preguntar(titulo, texto) {
  const modal = $("#modal");
  $("#modal-titulo").textContent = titulo;
  $("#modal-texto").textContent = texto;
  modal.showModal();                          // muestra la ventana
  return new Promise((resolver) => {
    // "close" se dispara cuando se cierra; once: true = escuchar solo una vez
    modal.addEventListener("close", () => resolver(modal.returnValue === "si"), { once: true });
  });
}

// Pregunta y, si el usuario acepta, elimina. Retorna true si se eliminó.
// pasajero es opcional: si lo tenemos, mostramos su nombre en la pregunta.
async function confirmarEliminar(numero, pasajero = null) {
  const quien = pasajero ? `${pasajero.nombre} (asiento ${pasajero.asiento})` : `la reserva #${numero}`;
  const acepta = await preguntar("¿Cancelar la reserva?",
    `Se quitará a ${quien} y el asiento quedará libre.`);
  if (!acepta) return false;                  // dijo que no

  const r = await API.eliminar(numero);       // en Flask usará eliminar() de tu lista
  if (!r.ok) {
    mostrarProblema(r);
    return false;
  }
  avisar(r.mensaje, "ok");
  $("#eliminar-numero").value = "";
  // Si justo estábamos modificando a ese pasajero, salimos de la edición
  if (estado.enEdicion && estado.enEdicion.numero === r.datos.numero) cancelarEdicion();
  estado.encontrado = null;
  await refrescar();
  animarAsiento(r.datos.asiento, "liberado"); // el asiento "brilla" al quedar libre
  return true;
}

// Al enviar el formulario de la pestaña Eliminar
async function enviarEliminar(evento) {
  evento.preventDefault();
  await confirmarEliminar($("#eliminar-numero").value);
}

$("#form-eliminar").addEventListener("submit", enviarEliminar);
```

---

<a id="archivo-static-js-buscar-js"></a>
#### Archivo: `static/js/buscar.js`
- **Ruta relativa:** `static/js/buscar.js`
- **Lenguaje:** `javascript`

```javascript
// ================================================================
//  AeroLista ✈ · buscar.js — buscar y mostrar el pase de abordar
// ================================================================

// Al enviar el formulario de búsqueda
async function enviarBusqueda(evento) {
  evento.preventDefault();
  const tipo = $('input[name="tipo"]:checked').value;     // :checked = el radio marcado
  const valor = $("#buscar-valor").value.trim();
  // /^\d+$/ = "solo dígitos". test() dice si el texto cumple ese patrón
  if (tipo === "numero" && !/^\d+$/.test(valor)) {
    avisar("El número de reserva solo lleva dígitos (ej: 1002)", "error");
    return;
  }
  await buscar(tipo, valor);
}

// Busca según el tipo y muestra el resultado
async function buscar(tipo, valor) {
  let r;
  if (tipo === "numero") r = await API.buscarNumero(valor);          // en Flask: buscar()
  else if (tipo === "cedula") r = await API.buscarCedula(valor);     // en Flask: buscar_por_cedula()
  else r = await API.buscarAsiento(valor.toUpperCase());             // en Flask: buscar_por_asiento()

  const caja = $("#resultado-buscar");
  caja.innerHTML = "";                                    // borramos el resultado anterior

  if (!r.ok) {                                            // no se encontró (o hubo un error)
    estado.encontrado = null;
    pintarAsientos();
    caja.append(crear("p", "vacio", (r.pendiente ? "🚧 " : "🔎 ") + r.mensaje));
    if (r.pendiente || r.error_codigo) mostrarProblema(r);
    return;
  }
  if (r.datos === null) {                                 // buscó un asiento y está libre
    caja.append(crear("p", "vacio", "💺 " + r.mensaje));
    resaltar(valor.toUpperCase());
    return;
  }
  caja.append(crearPase(r.datos));                        // ¡encontrado!
  resaltar(r.datos.asiento);                              // 🔵 en el avión
}

// Arma el pase de abordar de un pasajero con sus dos botones
function crearPase(p) {
  const clase = claseDeAsiento(p.asiento);
  const v = estado.vuelo;
  const contenedor = crear("div");
  // Plantilla con comillas invertidas: HTML con ${variables} adentro
  contenedor.innerHTML = `
    <article class="pase ${clase === "Ejecutiva" ? "ejecutiva" : ""}">
      <div class="pase-cuerpo">
        <div class="pase-cabecera"><span>✈ AEROLISTA</span><span>PASE DE ABORDAR</span></div>
        <div class="pase-nombre">${escapar(p.nombre)}</div>
        <div class="pase-ruta"><span>${v.codigo_origen}</span><i></i><span>✈</span><i></i><span>${v.codigo_destino}</span></div>
        <div class="pase-detalle">
          <span>Reserva<b>#${p.numero}</b></span>
          <span>Cédula<b>${escapar(p.cedula)}</b></span>
          <span>Edad<b>${p.edad} años</b></span>
        </div>
      </div>
      <div class="pase-talon"><small>ASIENTO</small><b>${escapar(p.asiento)}</b><em>${clase}</em></div>
    </article>
    <div class="pase-acciones">
      <button type="button" class="btn btn-suave" data-accion="modificar">✏️ Modificar</button>
      <button type="button" class="btn btn-peligro" data-accion="eliminar">🗑️ Cancelar reserva</button>
    </div>`;

  // Botón Modificar: va a la pestaña Modificar y carga a este pasajero (paso 20)
  contenedor.querySelector('[data-accion="modificar"]').addEventListener("click", () => {
    cambiarPestana("modificar");
    $("#modificar-numero").value = p.numero;
    cargarParaModificar(p.numero);
  });
  // Botón Cancelar reserva: reutiliza confirmarEliminar (paso 21)
  contenedor.querySelector('[data-accion="eliminar"]').addEventListener("click", async () => {
    const eliminado = await confirmarEliminar(p.numero, p);
    if (eliminado) $("#resultado-buscar").innerHTML = "";   // el pase ya no tiene sentido
  });
  return contenedor;
}

// ---- Conexiones ----
$("#form-buscar").addEventListener("submit", enviarBusqueda);
// Al cambiar de opción, cambia el texto de ejemplo del campo
for (const radio of $$('input[name="tipo"]')) {
  radio.addEventListener("change", () => {
    const ejemplos = { numero: "Ej: 1002", cedula: "Ej: 0912345678", asiento: "Ej: 7D" };
    $("#buscar-valor").placeholder = ejemplos[radio.value];
  });
}
```

---

<a id="archivo-static-js-listar-js"></a>
#### Archivo: `static/js/listar.js`
- **Ruta relativa:** `static/js/listar.js`
- **Lenguaje:** `javascript`

```javascript
// ================================================================
//  AeroLista ✈ · listar.js — la tabla de pasajeros
// ================================================================

async function cargarTabla() {
  // "" → todos (en Flask: listar())  ·  "Ejecutiva" → solo esa clase (filtrar_por_clase())
  const r = await API.listar(estado.filtroClase);
  const cuerpo = $("#tabla-reservas");
  cuerpo.innerHTML = "";                       // vaciamos la tabla

  if (!r.ok) {                                 // colspan="5": una celda que ocupa las 5 columnas
    cuerpo.innerHTML = `<tr><td colspan="5" class="sin-datos">${r.pendiente ? "🚧" : "💥"} ${escapar(r.mensaje)}</td></tr>`;
    return;
  }
  if (r.datos.length === 0) {
    cuerpo.innerHTML = `<tr><td colspan="5" class="sin-datos">No hay pasajeros en esta clase</td></tr>`;
    return;
  }
  // forEach recorre la lista y nos da cada pasajero y su posición (i)
  r.datos.forEach((p, i) => {
    const fila = crear("tr");
    const ejecutiva = claseDeAsiento(p.asiento) === "Ejecutiva" ? "ejecutiva" : "";
    fila.innerHTML = `<td>${i}</td><td>${p.numero}</td><td>${escapar(p.nombre)}${p.edad < 18 ? " 🧒" : ""}</td>
                      <td>${p.edad}</td><td><span class="etiqueta-asiento ${ejecutiva}">${escapar(p.asiento)}</span></td>`;
    fila.addEventListener("click", () => resaltar(p.asiento));   // clic en la fila → 🔵 en el avión
    cuerpo.append(fila);
  });
}

// Clic en un filtro. Delegación: un oyente en la caja de filtros
function elegirFiltro(evento) {
  const chip = evento.target.closest(".chip");
  if (!chip) return;                           // hizo clic fuera de un botón
  estado.filtroClase = chip.dataset.clase;
  for (const otro of $$(".chip")) otro.classList.toggle("activo", otro === chip);
  cargarTabla();
}

// ---- Conexiones ----
$(".filtros").addEventListener("click", elegirFiltro);
// Cada vez que abres la pestaña Listar, la tabla se vuelve a cargar
$('.pestana[data-pestana="listar"]').addEventListener("click", cargarTabla);
```

---

<a id="archivo-static-js-tablero-js"></a>
#### Archivo: `static/js/tablero.js`
- **Ruta relativa:** `static/js/tablero.js`
- **Lenguaje:** `javascript`

```javascript
// ================================================================
//  AeroLista ✈ · tablero.js — los números de arriba del avión
// ================================================================

async function cargarResumen() {
  const r = await API.resumen();          // { cantidad, vacia, total_asientos }
  const totalAsientos = $$(".asiento").length;      // 68 (contamos los botones dibujados)
  $("#dato-total").textContent = totalAsientos;

  if (!r.ok) {                            // si falla, dejamos guiones y mostramos el aviso
    for (const id of ["#dato-reservados", "#dato-libres", "#dato-ocupacion"]) $(id).textContent = "—";
    $("#luz-ocupacion").className = "luz";
    $("#barra-ocupacion").style.width = "0";
    mostrarAviso("#aviso-tablero", r, "El tablero no puede contar las reservas.");
    return;
  }
  ocultarAviso("#aviso-tablero");

  const cantidad = r.datos.cantidad;
  const porcentaje = Math.round((cantidad * 100) / totalAsientos);   // regla de tres

  // El semáforo de ocupación
  let color = "verde";                    // 🟢 menos de la mitad
  if (porcentaje >= 80) color = "rojo";   // 🔴 casi lleno
  else if (porcentaje >= 50) color = "ambar";   // 🟡 más de la mitad

  $("#dato-reservados").textContent = cantidad;
  $("#dato-libres").textContent = totalAsientos - cantidad;
  $("#dato-ocupacion").textContent = porcentaje + "%";
  $("#luz-ocupacion").className = "luz " + color;            // className reemplaza TODAS las clases
  $("#barra-ocupacion").style.width = porcentaje + "%";      // la barrita crece
  $("#barra-ocupacion").style.background = `var(--${color})`;
}
```

---

<a id="archivo-static-js-progreso-js"></a>
#### Archivo: `static/js/progreso.js`
- **Ruta relativa:** `static/js/progreso.js`
- **Lenguaje:** `javascript`

```javascript
// ================================================================
//  AeroLista ✈ · progreso.js — el panel "Progreso del backend"
// ================================================================

// Un icono para cada estado que manda verificador.py
const ICONOS_PROGRESO = { ok: "✅", pendiente: "🚧", fallo: "❌", error: "💥", lento: "⏳" };

// Pide las pruebas a Flask y dibuja un renglón por método.
// Retorna true si los 9 pasaron.
async function cargarProgreso() {
  const lista = $("#progreso-lista");
  const r = await API.progreso();
  lista.innerHTML = "";
  if (API.modo === "demo") {                       // con datos de mentira no hay nada que probar
    $("#progreso-cuenta").textContent = "demo";
    lista.innerHTML = `<li><span>🧪</span><span>Modo de prueba: los datos vienen de <code>datos-demo.js</code>.</span></li>`;
    return false;
  }
  if (!r.ok) {
    lista.innerHTML = `<li class="error"><span>💥</span><span>${escapar(r.mensaje)}</span></li>`;
    return false;
  }
  // filter deja solo los "ok"; length cuenta cuántos son
  const listos = r.datos.filter((m) => m.estado === "ok").length;
  $("#progreso-cuenta").textContent = `${listos}/${r.datos.length}`;          // "3/9"
  $("#progreso-barra").style.width = (listos * 100) / r.datos.length + "%";
  for (const m of r.datos) {
    const item = crear("li", m.estado);
    item.innerHTML = `<span>${ICONOS_PROGRESO[m.estado]}</span><code>${m.metodo}()</code>`;
    // Si falló, mostramos el detalle (qué esperaba y qué obtuvo)
    if (m.estado !== "ok" && m.estado !== "pendiente") item.append(crear("small", "", m.detalle));
    lista.append(item);
  }
  return listos === r.datos.length;
}

// Botón "Revisar otra vez": vuelve a probar y refresca toda la página
async function revisarProgreso() {
  const completo = await cargarProgreso();
  if (completo) avisar("¡Backend completo! Todos tus métodos pasaron las pruebas 🎉", "ok");
  await refrescar();
}

// Botón "Datos de ejemplo": vuelve a los 12 pasajeros (pregunta antes con el dialog del paso 21)
async function restaurarDatos() {
  const acepta = await preguntar("¿Restaurar los datos de ejemplo?",
    "Se borrarán las reservas actuales y volverán las 12 reservas de prueba.");
  if (!acepta) return;
  const r = await API.restaurar();
  if (!r.ok) return mostrarProblema(r);
  avisar(r.mensaje, "ok");
  estado.encontrado = null;
  await refrescar();
}

$("#btn-revisar").addEventListener("click", revisarProgreso);
$("#btn-restaurar").addEventListener("click", restaurarDatos);
```

---

<a id="archivo-static-js-semaforo-js"></a>
#### Archivo: `static/js/semaforo.js`
- **Ruta relativa:** `static/js/semaforo.js`
- **Lenguaje:** `javascript`

```javascript
// ================================================================
//  AeroLista ✈ · semaforo.js — pinta cada asiento según su estado
// ================================================================

// El dibujo del pasajero: un SVG con un círculo (cabeza) y una curva (hombros)
function iconoPasajero() {
  return `<svg class="icono-pasajero" viewBox="0 0 24 24" aria-hidden="true">
            <circle cx="12" cy="6.5" r="4.5"/>
            <path d="M3 23c0-5.5 4-9.5 9-9.5s9 4 9 9.5z"/>
          </svg>`;
}

// Pide la lista de pasajeros (API.listar) y pinta el avión.
// Retorna la respuesta para que otros (el tren, paso 24) también la usen.
async function cargarReservas() {
  const r = await API.listar();
  estado.porAsiento = {};                  // empezamos el "mapa" vacío
  if (r.ok) {
    estado.reservas = r.datos;             // la lista completa, en orden
    // Armamos un diccionario asiento → pasajero:  { "7D": {...Luis}, "1A": {...Ana} }
    // así, para saber quién está en el 7D basta con estado.porAsiento["7D"]
    for (const pasajero of r.datos) {
      estado.porAsiento[pasajero.asiento] = pasajero;
    }
    ocultarAviso("#aviso-avion");
  } else {
    estado.reservas = [];
    mostrarAviso("#aviso-avion", r, "El avión todavía no puede ver las reservas.");
  }
  pintarAsientos();
  return r;
}

// Recorre los 68 botones y le pone a cada uno la clase correcta
function pintarAsientos() {
  for (const boton of $$(".asiento")) {
    const codigo = boton.dataset.asiento;                  // "7D"
    const pasajero = estado.porAsiento[codigo];            // el pasajero, o undefined si está libre
    // Primero le quitamos TODAS las clases de estado...
    boton.classList.remove("libre", "seleccionado", "ocupado", "encontrado", "editando", "menor");

    // ...y luego le ponemos la que corresponde
    if (pasajero) {                                        // 🔴 hay alguien sentado
      boton.classList.add("ocupado");
      if (pasajero.edad < 18) boton.classList.add("menor");   // los menores tienen icono más pequeño
      boton.innerHTML = iconoPasajero();                   // innerHTML mete HTML (el dibujo SVG)
      boton.setAttribute("aria-label", `Asiento ${codigo}: ${pasajero.nombre}`);   // para lectores de pantalla
    } else {                                               // 🟡 o 🟢
      boton.classList.add(codigo === estado.seleccionado ? "seleccionado" : "libre");
      boton.textContent = codigo.slice(-1);                // vuelve a mostrar la letra
      boton.setAttribute("aria-label", `Asiento ${codigo}: libre`);
    }
    if (codigo === estado.encontrado) boton.classList.add("encontrado");   // 🔵 resultado de búsqueda
    // El asiento del pasajero que se está modificando (paso 20) lleva borde punteado
    if (estado.enEdicion && codigo === estado.enEdicion.asiento) boton.classList.add("editando");
  }
}

// Reproduce una animación CSS sobre un asiento ("recien" o "liberado")
function animarAsiento(asiento, animacion) {
  const boton = botonDeAsiento(asiento);
  if (!boton) return;
  boton.classList.remove(animacion);
  void boton.offsetWidth;                  // truco: obliga al navegador a "reiniciar" la animación
  boton.classList.add(animacion);
  setTimeout(() => boton.classList.remove(animacion), 1000);
}

// Hace scroll suave hasta un asiento
function enfocarAsiento(asiento) {
  const boton = botonDeAsiento(asiento);
  if (boton) boton.scrollIntoView({ behavior: "smooth", block: "center" });
}

// Marca un asiento con el borde azul 🔵 y lo enfoca
function resaltar(asiento) {
  estado.encontrado = asiento;
  pintarAsientos();
  enfocarAsiento(asiento);
}
```

---

<a id="archivo-static-js-pestanas-js"></a>
#### Archivo: `static/js/pestanas.js`
- **Ruta relativa:** `static/js/pestanas.js`
- **Lenguaje:** `javascript`

```javascript
// ================================================================
//  AeroLista ✈ · pestanas.js — cambiar de pestaña en el panel
// ================================================================

// Muestra la pestaña "nombre" y esconde las demás
function mostrarPestana(nombre) {
  estado.pestana = nombre;                         // la página recuerda cuál está abierta
  // A cada botón: le pone "activa" solo si es el de esta pestaña
  for (const boton of $$(".pestana")) {
    boton.classList.toggle("activa", boton.dataset.pestana === nombre);
  }
  // A cada sección: le pone "activa" solo si su id es "pestana-" + nombre
  for (const seccion of $$(".contenido-pestana")) {
    seccion.classList.toggle("activa", seccion.id === "pestana-" + nombre);
  }
}

// Para cambiar de pestaña DESDE EL CÓDIGO, "hacemos clic" en su botón.
// Así se ejecutan también los oyentes que otros archivos agregan a esos botones.
function cambiarPestana(nombre) {
  $(`.pestana[data-pestana="${nombre}"]`).click();
}

// Cada botón, al hacer clic, muestra su pestaña
for (const boton of $$(".pestana")) {
  boton.addEventListener("click", () => mostrarPestana(boton.dataset.pestana));
}
```

---

<a id="archivo-static-js-clic-js"></a>
#### Archivo: `static/js/clic.js`
- **Ruta relativa:** `static/js/clic.js`
- **Lenguaje:** `javascript`

```javascript
// ================================================================
//  AeroLista ✈ · clic.js — qué pasa al hacer clic en un asiento
//  Versión 3 (final): rojo → pase de abordar · verde → elegir.
// ================================================================

async function clicEnAsiento(codigo) {
  // a) Asiento ROJO: buscamos quién está ahí y mostramos su pase  🆕
  if (estado.porAsiento[codigo]) {
    cambiarPestana("buscar");                                    // abre la pestaña Buscar
    $('input[name="tipo"][value="asiento"]').checked = true;     // marca la opción "Asiento"
    $("#buscar-valor").value = codigo;                           // escribe el asiento
    await buscar("asiento", codigo);                             // y busca (buscar.js)
    return;
  }

  // b) Asiento VERDE mientras se modifica una reserva: será su asiento nuevo
  if (estado.enEdicion) {
    estado.seleccionado = codigo;
    $("#modificar-asiento").textContent = codigo;
    pintarAsientos();
    return;
  }

  // c) Asiento VERDE: lo elegimos para una reserva nueva (otro clic lo suelta)
  if (estado.pestana !== "reservar") cambiarPestana("reservar");
  estado.seleccionado = estado.seleccionado === codigo ? null : codigo;
  estado.encontrado = null;
  actualizarAsientoElegido();
  pintarAsientos();
  if (estado.seleccionado) $('#form-reservar [name="nombre"]').focus({ preventScroll: true });
}
```

---

<a id="archivo-static-js-tooltip-js"></a>
#### Archivo: `static/js/tooltip.js`
- **Ruta relativa:** `static/js/tooltip.js`
- **Lenguaje:** `javascript`

```javascript
// ================================================================
//  AeroLista ✈ · tooltip.js — el globo de ayuda de los asientos
// ================================================================

// Llena el globo con los datos del asiento y lo muestra
function mostrarTooltip(codigo, evento) {
  const tooltip = $("#tooltip");
  const p = estado.porAsiento[codigo];          // ¿hay alguien sentado ahí?
  const clase = claseDeAsiento(codigo);
  tooltip.innerHTML = p
    ? `<b>${escapar(p.nombre)}</b> ${p.edad < 18 ? "🧒" : ""}<br>Asiento ${codigo} · ${clase}<br>` +
      `<small>Reserva #${p.numero} · clic para ver su pase</small>`
    : `<b>Asiento ${codigo}</b><br>${clase} · <span style="color:#86EFAC">libre</span><br><small>clic para elegirlo</small>`;
  tooltip.hidden = false;
  moverTooltip(evento);
}

// Pone el globo un poquito abajo y a la derecha del mouse.
// clientX / clientY = posición del mouse en la pantalla
function moverTooltip(evento) {
  const tooltip = $("#tooltip");
  tooltip.style.left = evento.clientX + 16 + "px";
  tooltip.style.top = evento.clientY + 16 + "px";
}

function ocultarTooltip() {
  $("#tooltip").hidden = true;
}

// ---- Delegación: UN solo oyente en la cabina vigila los 68 asientos ----
// "mouseover" se dispara cuando el mouse entra a cualquier cosa dentro de la cabina.
// evento.target es el elemento exacto; closest(".asiento") sube hasta encontrar el botón.
$("#cabina").addEventListener("mouseover", (evento) => {
  const boton = evento.target.closest(".asiento");
  if (boton) mostrarTooltip(boton.dataset.asiento, evento);
});
$("#cabina").addEventListener("mousemove", (evento) => {
  if (!$("#tooltip").hidden) moverTooltip(evento);       // el globo sigue al mouse
});
$("#cabina").addEventListener("mouseout", (evento) => {
  if (evento.target.closest(".asiento")) ocultarTooltip();   // salió de un asiento
});
```

---

<a id="archivo-static-js-tren-js"></a>
#### Archivo: `static/js/tren.js`
- **Ruta relativa:** `static/js/tren.js`
- **Lenguaje:** `javascript`

```javascript
// ================================================================
//  AeroLista ✈ · tren.js — dibuja la lista como vagones de tren
//     head → [vagón] → [vagón] → ... → None
// ================================================================

// r: la respuesta de API.listar() · numeroNuevo: la reserva recién creada (se resalta)
function dibujarTren(r, numeroNuevo = null) {
  const tren = $("#tren-vagones");
  tren.innerHTML = "";

  if (!r.ok) {                                  // la lista no se pudo leer
    $("#tren-cantidad").textContent = "nodos = ?";
    const aviso = crear("p", "tren-vacio");
    aviso.innerHTML = r.pendiente
      ? `🚧 Aquí verás los vagones de tu lista cuando implementes <code>listar()</code>.`
      : `💥 ${escapar(r.mensaje)}`;
    tren.append(aviso);
    return;
  }

  $("#tren-cantidad").textContent = `nodos = ${r.datos.length}`;
  tren.append(crear("span", "etiqueta-head", "head"), crear("span", "flecha", "→"));   // la cabeza

  // Un vagón por pasajero, en el MISMO orden de la lista
  r.datos.forEach((pasajero, posicion) => {
    const vagon = crear("div", "vagon");
    if (pasajero.numero === numeroNuevo) vagon.classList.add("nuevo");   // el recién llegado, en verde
    vagon.title = "Clic para ver su asiento";
    vagon.innerHTML = `
      <div class="vagon-data">
        <small>[${posicion}] #${pasajero.numero}</small>
        <b>${escapar(pasajero.nombre)}</b>
        <span>asiento ${escapar(pasajero.asiento)}</span>
      </div>
      <div class="vagon-next">●</div>`;
    vagon.addEventListener("click", () => resaltar(pasajero.asiento));
    tren.append(vagon, crear("span", "flecha", "→"));
  });

  tren.append(crear("span", "nodo-none", "None"));   // el final de la lista
}
```

---

<a id="archivo-static-js-vista3d-js"></a>
#### Archivo: `static/js/vista3d.js`
- **Ruta relativa:** `static/js/vista3d.js`
- **Lenguaje:** `javascript`

```javascript
// ================================================================
//  AeroLista ✈ · vista3d.js — vista 3D: inclinar y girar el avión
// ================================================================

// Cambia entre "3d" y "2d"
function cambiarVista(vista) {
  estado.vista = vista;                                       // la página recuerda la vista
  $("#avion").classList.toggle("vista-3d", vista === "3d");   // toggle(clase, condición): la pone si es true
  $("#ajustes-3d").hidden = vista !== "3d";                   // deslizadores solo en 3D
  for (const boton of $$(".btn-vista")) {                     // marca el botón elegido
    boton.classList.toggle("activo", boton.dataset.vista === vista);
  }
  aplicarAngulos();
}

// Pasa los valores de los deslizadores a las variables CSS del avión
function aplicarAngulos() {
  const avion = $("#avion");
  const en3D = estado.vista === "3d";
  // En 2D los dos ángulos valen 0: el avión queda plano (y la transición lo anima)
  avion.style.setProperty("--inclinacion", (en3D ? $("#rango-inclinacion").value : 0) + "deg");
  avion.style.setProperty("--giro", (en3D ? $("#rango-giro").value : 0) + "deg");
  ajustarEscena();
}

// Calcula DÓNDE se ve en pantalla un punto (x, y) del avión después de girarlo.
// Es el mismo cálculo que hace el navegador con rotateZ → rotateX → perspective.
function proyectar(x, y, giro, incl) {
  const x1 = x * Math.cos(giro) - y * Math.sin(giro);    // rotateZ (girar sobre la mesa)
  const y1 = x * Math.sin(giro) + y * Math.cos(giro);
  const y2 = y1 * Math.cos(incl);                        // rotateX: se acorta...
  const z2 = y1 * Math.sin(incl);                        // ...y gana profundidad
  const factor = 1800 / (1800 - z2);                     // perspective(1800px): lo cercano crece
  return [x1 * factor, y2 * factor];
}

// Elige la escala y el alto de la escena para que el avión quepa COMPLETO
function ajustarEscena() {
  const avion = $("#avion");
  const escena = $("#escena");
  const mitadAncho = avion.offsetWidth / 2;              // offsetWidth = ancho real en px
  const mitadAlto = avion.offsetHeight / 2;
  const en3D = estado.vista === "3d";
  // Math.PI / 180 convierte grados en radianes (lo que usan Math.cos y Math.sin)
  const giro = en3D ? ($("#rango-giro").value * Math.PI) / 180 : 0;
  const incl = en3D ? ($("#rango-inclinacion").value * Math.PI) / 180 : 0;

  // Proyectamos las 4 esquinas del avión y buscamos la caja que las encierra
  let minX = Infinity, maxX = -Infinity, minY = Infinity, maxY = -Infinity;
  const esquinas = [[-mitadAncho, -mitadAlto], [mitadAncho, -mitadAlto],
                    [mitadAncho, mitadAlto], [-mitadAncho, mitadAlto]];
  for (const [x, y] of esquinas) {
    const [px, py] = proyectar(x, y, giro, incl);
    minX = Math.min(minX, px); maxX = Math.max(maxX, px);
    minY = Math.min(minY, py); maxY = Math.max(maxY, py);
  }
  // Si no cabe a lo ancho, lo achicamos (escala menor a 1); nunca lo agrandamos
  const escala = Math.min(1, (escena.clientWidth - 24) / (maxX - minX));
  avion.style.setProperty("--escala", escala.toFixed(3));
  avion.style.setProperty("--ajuste-y", (-(minY + maxY) / 2 * escala).toFixed(1) + "px");   // lo centra
  escena.style.height = Math.round((maxY - minY) * escala + 50) + "px";                     // alto justo
}

// ---- Conectamos los controles (esto se ejecuta apenas se carga el archivo) ----
for (const boton of $$(".btn-vista")) {
  boton.addEventListener("click", () => cambiarVista(boton.dataset.vista));
}
// "input" se dispara MIENTRAS mueves el deslizador
$("#rango-inclinacion").addEventListener("input", aplicarAngulos);
$("#rango-giro").addEventListener("input", aplicarAngulos);
// "resize" se dispara cuando cambia el tamaño de la ventana
window.addEventListener("resize", ajustarEscena);
```

---

<a id="archivo-static-js-utilidades-js"></a>
#### Archivo: `static/js/utilidades.js`
- **Ruta relativa:** `static/js/utilidades.js`
- **Lenguaje:** `javascript`

```javascript
// ================================================================
//  AeroLista ✈ · utilidades.js — atajos que usan todos los archivos
// ================================================================

// $("#cabina") es un atajo de document.querySelector("#cabina"):
// devuelve el PRIMER elemento que cumple el selector CSS.
// (selector) => ... es una "función flecha": una forma corta de escribir una función.
const $ = (selector) => document.querySelector(selector);

// $$(".asiento") devuelve TODOS los elementos que cumplen el selector.
const $$ = (selector) => document.querySelectorAll(selector);

// Crea un elemento HTML nuevo de una sola vez.
// Ejemplo: crear("span", "ventana")  →  <span class="ventana"></span>
// Los parámetros con = "" son OPCIONALES: si no los pasas, valen "".
function crear(etiqueta, clase = "", texto = "") {
  const elemento = document.createElement(etiqueta);   // 1. crea la etiqueta
  if (clase) elemento.className = clase;               // 2. le pone su(s) clase(s)
  if (texto) elemento.textContent = texto;             // 3. le pone el texto
  return elemento;                                     // todavía NO se ve: falta meterlo en la página
}

// Protege el texto que escribe el usuario antes de meterlo con innerHTML.
// Si alguien escribe <b>Ana</b> como nombre, se mostrará tal cual y no como HTML.
function escapar(texto) {
  const div = document.createElement("div");
  div.textContent = String(texto);       // textContent nunca interpreta etiquetas
  return div.innerHTML;                  // devuelve el texto ya "escapado" (&lt;b&gt;...)
}

// Dice la clase de un asiento: "1A" → "Ejecutiva", "7D" → "Económica".
function claseDeAsiento(asiento) {
  const fila = parseInt(asiento);        // parseInt("12A") lee el número del inicio → 12
  return fila <= estado.vuelo.filas_ejecutiva ? "Ejecutiva" : "Económica";
  //     condición ? valor si es verdad : valor si es mentira   (operador ternario)
}

// Busca el botón de un asiento por su código: botonDeAsiento("7D")
function botonDeAsiento(asiento) {
  return document.querySelector(`.asiento[data-asiento="${asiento}"]`);
}
```

---

<a id="archivo-static-js-mensajes-js"></a>
#### Archivo: `static/js/mensajes.js`
- **Ruta relativa:** `static/js/mensajes.js`
- **Lenguaje:** `javascript`

```javascript
// ================================================================
//  AeroLista ✈ · mensajes.js — mensajes para el usuario
// ================================================================

// Muestra un mensaje flotante (toast) que desaparece solo.
//   texto: lo que dice el mensaje
//   tipo:  "ok" ✅ · "aviso" ⚠️ · "pendiente" 🚧 · "error" ❌ · "codigo" 💥
function avisar(texto, tipo = "ok") {
  // Un objeto como "diccionario": a cada tipo le corresponde un icono
  const iconos = { ok: "✅", aviso: "⚠️", pendiente: "🚧", error: "❌", codigo: "💥" };

  // Los mensajes buenos duran 3.5 s; los problemas, 7 s (para que alcances a leerlos)
  const duracion = tipo === "ok" ? 3500 : 7000;

  // Armamos:  <div class="toast toast-ok"> <span>✅</span> <p>texto</p> </div>
  const toast = crear("div", "toast toast-" + tipo);
  toast.append(crear("span", "", iconos[tipo]), crear("p", "", texto));
  $("#toasts").append(toast);                                   // ahora SÍ se ve

  // setTimeout(función, milisegundos) = "ejecuta esto dentro de X milisegundos"
  setTimeout(() => toast.classList.add("saliendo"), duracion);  // empieza a irse...
  setTimeout(() => toast.remove(), duracion + 400);             // ...y se borra de la página
}

// Recibe una respuesta de Flask que salió mal (r.ok === false) y elige el toast:
function mostrarProblema(r) {
  if (r.pendiente) avisar(r.mensaje, "pendiente");                        // 🚧 método sin programar
  else if (r.error_codigo) avisar(r.mensaje + "  (revisa la terminal de Flask)", "codigo");   // 💥 error de Python
  else avisar(r.mensaje, "error");                                        // ❌ dato incorrecto
}

// Muestra un aviso FIJO (no se va solo) dentro de una caja de la página.
// Lo usaremos para decir qué método de Python falta. selector = "#aviso-avion", etc.
function mostrarAviso(selector, r, titulo) {
  const caja = $(selector);
  caja.hidden = false;                                  // la hacemos visible
  caja.classList.toggle("es-error", !r.pendiente);      // rojo si es error, ámbar si es pendiente
  caja.innerHTML = r.pendiente
    ? `🚧 ${titulo} Implementa <code>${r.metodo}()</code> en <code>lista_pasajeros.py</code>, guarda y recarga la página.`
    : `💥 ${escapar(r.mensaje)}`;
}

// Esconde un aviso fijo
function ocultarAviso(selector) {
  $(selector).hidden = true;
}
```

---

<a id="archivo-static-js-datos-demo-js"></a>
#### Archivo: `static/js/datos-demo.js`
- **Ruta relativa:** `static/js/datos-demo.js`
- **Lenguaje:** `javascript`

```javascript
// ================================================================
//  AeroLista ✈ · datos-demo.js — un SERVIDOR DE MENTIRA
//  Finge ser Flask mientras construimos la interfaz (Etapa D).
//  En el paso 29 lo desconectamos y los datos vendrán de Python.
//  ⚠ Si recargas la página, todo vuelve a como estaba (no se guarda).
// ================================================================

API.modo = "demo";      // una marca para saber que estamos usando datos falsos

// Los 12 pasajeros de prueba, en ORDEN DE LLEGADA (Diana reservó primero).
// Cada pasajero es un objeto con 5 datos. Son los mismos que tendrá data/ejemplo.json
const RESERVAS_INICIALES = [
  { numero: 1005, nombre: "Diana",   cedula: "0923456789", edad: 22, asiento: "3C" },
  { numero: 1002, nombre: "Ana",     cedula: "0912345678", edad: 28, asiento: "1A" },
  { numero: 1009, nombre: "Luis",    cedula: "0987654321", edad: 35, asiento: "7D" },
  { numero: 1004, nombre: "Beto",    cedula: "0934567890", edad: 16, asiento: "7E" },
  { numero: 1001, nombre: "Elena",   cedula: "0901234567", edad: 41, asiento: "2F" },
  { numero: 1008, nombre: "Carla",   cedula: "0945678901", edad: 33, asiento: "5A" },
  { numero: 1003, nombre: "Jorge",   cedula: "0956789012", edad: 52, asiento: "10B" },
  { numero: 1010, nombre: "Sofía",   cedula: "0967890123", edad: 9,  asiento: "10C" },
  { numero: 1007, nombre: "Mateo",   cedula: "0978901234", edad: 27, asiento: "12F" },
  { numero: 1006, nombre: "Valeria", cedula: "0989012345", edad: 30, asiento: "4D" },
  { numero: 1011, nombre: "Pedro",   cedula: "0990123456", edad: 45, asiento: "1D" },
  { numero: 1012, nombre: "Lucía",   cedula: "0911223344", edad: 19, asiento: "8B" },
];

// Copia de trabajo: { ...p } copia cada objeto, para no dañar los originales
let reservasDemo = RESERVAS_INICIALES.map((p) => ({ ...p }));
let siguienteNumero = 1013;          // el próximo número de reserva

// ---- Ayudantes: responden igual que Flask, con una pequeña espera (120 ms) ----
// Una Promise es "un valor que llegará después": así se comporta igual que fetch.
function responder(datos = null, mensaje = "") {
  return new Promise((resolver) =>
    setTimeout(() => resolver({ ok: true, datos, mensaje, avisos: [] }), 120));
}
function fallar(mensaje) {
  return new Promise((resolver) => setTimeout(() => resolver({ ok: false, mensaje }), 120));
}

// Revisa los datos de un formulario. Retorna el problema o "" si todo está bien.
function problemaEnDatos(datos) {
  if (!datos.nombre || datos.nombre.length < 2) return "Escribe el nombre del pasajero";
  if (!/^\d{10}$/.test(datos.cedula)) return "La cédula debe tener 10 dígitos";   // /^\d{10}$/ = exactamente 10 dígitos
  const edad = Number(datos.edad);
  if (datos.edad === "" || !Number.isInteger(edad) || edad < 0 || edad > 120) return "La edad debe ser un número entre 0 y 120";
  return "";
}

// ---- Las funciones del "servidor" ----

// Resumen para el tablero: cuántos hay y cuántos asientos tiene el avión
API.resumen = () => responder({
  cantidad: reservasDemo.length,
  vacia: reservasDemo.length === 0,
  total_asientos: $$(".asiento").length,
});

// Listar todos (o solo una clase). filter() deja pasar solo los que cumplen la condición
API.listar = (clase = "") => {
  if (clase === "") return responder(reservasDemo.map((p) => ({ ...p })));
  return responder(reservasDemo.filter((p) => claseDeAsiento(p.asiento) === clase));
};

// Buscar por número. find() devuelve el PRIMERO que cumple la condición (o undefined)
API.buscarNumero = (numero) => {
  const p = reservasDemo.find((r) => r.numero === Number(numero));
  return p ? responder({ ...p }, `Reserva #${numero} encontrada`) : fallar(`No existe la reserva #${numero}`);
};

// Buscar por asiento: si nadie está sentado, responde datos = null (asiento libre)
API.buscarAsiento = (asiento) => {
  const p = reservasDemo.find((r) => r.asiento === asiento);
  return p ? responder({ ...p }, `En el ${asiento} viaja ${p.nombre}`)
           : responder(null, `El asiento ${asiento} está libre`);
};

// Buscar por cédula
API.buscarCedula = (cedula) => {
  const p = reservasDemo.find((r) => r.cedula === cedula);
  return p ? responder({ ...p }, "Pasajero encontrado") : fallar(`Ningún pasajero tiene la cédula ${cedula}`);
};

// Reservar: valida y agrega AL FINAL (en Python será lista.ingresar)
API.reservar = (datos) => {
  const problema = problemaEnDatos(datos);
  if (problema) return fallar(problema);
  // some() pregunta: ¿ALGUNO cumple la condición?
  if (reservasDemo.some((r) => r.asiento === datos.asiento)) return fallar(`El asiento ${datos.asiento} ya está reservado`);
  if (reservasDemo.some((r) => r.cedula === datos.cedula)) return fallar(`La cédula ${datos.cedula} ya tiene una reserva en este vuelo`);
  const nuevo = { numero: siguienteNumero, nombre: datos.nombre, cedula: datos.cedula,
                  edad: Number(datos.edad), asiento: datos.asiento };
  siguienteNumero = siguienteNumero + 1;
  reservasDemo.push(nuevo);                          // push() = append() de Python
  return responder({ ...nuevo }, `Reserva #${nuevo.numero} creada: ${nuevo.nombre} en el asiento ${nuevo.asiento}`);
};

// Modificar nombre, edad y asiento
API.modificar = (numero, datos) => {
  const p = reservasDemo.find((r) => r.numero === Number(numero));
  if (!p) return fallar(`No existe la reserva #${numero}`);
  const problema = problemaEnDatos({ ...datos, cedula: p.cedula });
  if (problema) return fallar(problema);
  const ocupante = reservasDemo.find((r) => r.asiento === datos.asiento);
  if (ocupante && ocupante.numero !== p.numero) return fallar(`El asiento ${datos.asiento} ya es de ${ocupante.nombre}`);
  p.nombre = datos.nombre;
  p.edad = Number(datos.edad);
  p.asiento = datos.asiento;
  return responder({ ...p }, `Reserva #${numero} actualizada`);
};

// Eliminar: findIndex da la posición; splice(posición, 1) quita 1 elemento
API.eliminar = (numero) => {
  const posicion = reservasDemo.findIndex((r) => r.numero === Number(numero));
  if (posicion === -1) return fallar(`No existe la reserva #${numero}`);
  const [quitado] = reservasDemo.splice(posicion, 1);
  return responder(quitado, `Reserva #${numero} cancelada: el asiento ${quitado.asiento} quedó libre`);
};

// Volver a los 12 pasajeros iniciales
API.restaurar = () => {
  reservasDemo = RESERVAS_INICIALES.map((p) => ({ ...p }));
  siguienteNumero = 1013;
  return responder(null, "Se restauraron las 12 reservas de ejemplo");
};

// El panel de progreso no existe en modo demo
API.progreso = () => responder(null);
```

---
