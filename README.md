# examen
exameeen


# base.py — NO MODIFICAR


print("EXAMEN 2DO PRACTICO - POO    ")
print("EQUIPO 7: ")
print("Alonso Soto Barrera - 39882")
print("Salvador Flores Cazares - 39883")
print("Bruno Isaias Macias Guerrero -39901 ")
print("Jose Alberto Magana Torres-39435")
print("\n\n - Aplicacion de APIs  -\n\n ")

import requests

class RequestHandler:
    def __init__(self, base_url):
        self.base_url = base_url




 #----------------------------------------------------------------------------------------------\



#
# PREGUNTAS INICIALES (antes del Paso 1):
#
# 1. ¿Qué construye el método get antes de hacer la petición?
#    Construye la URL completa concatenando base_url + endpoint.
#    Devuelve el JSON de la respuesta ya convertido a diccionario de Python.
#
# 2. ¿Por qué el parámetro endpoint tiene valor por defecto ""?
#    Para poder llamar a get() sin argumentos cuando el endpoint
#    es la raíz, sin que el programa falle por falta de parámetro.
#
# 3. Si llamas a get("/satellites/25544"), ¿qué URL se construye?
#    "https://api.wheretheiss.at/v1/satellites/25544"
#
# ============================================================


import requests
from abc import ABC, abstractmethod

#Esta es la clase base, no se modifica
class RequestHandler:
    def __init__(self, base_url):
        self.base_url = base_url

    def get(self, endpoint=""):
        url = self.base_url + endpoint
        response = requests.get(url)
        return response.json()

    def describe(self):
        return f"RequestHandler apuntando a: {self.base_url}"


# Paso 3
# TelemetrySensor es una clase abstracta que obliga a todas las clases hijas a implementar el metodo acquire()
class TelemetrySensor(ABC):
    """
    Interfaz común para todos los sensores del sistema.
    Cualquier clase que herede de esta DEBE implementar acquire().
    """

    @abstractmethod
    def acquire(self):
        pass




# ============================================================
# PASO 1: HERENCIA — Cliente de Telemetría Orbital
# Pilar POO: HERENCIA
# ISSClient hereda de RequestHandler y TelemetrySensor.
# Reutiliza get() del padre y agrega comportamiento propio.
# ============================================================

class ISSClient(RequestHandler, TelemetrySensor):
    """
    Sensor orbital que obtiene la posición de la ISS en tiempo real.
    Hereda de RequestHandler para hacer peticiones HTTP
    y de TelemetrySensor para cumplir el contrato de sensores.
    """

    def __init__(self):
        # HERENCIA: fijamos la URL base sin que el usuario la pase
        super().__init__("https://api.wheretheiss.at/v1")

    def get_position(self):
        """Devuelve el diccionario completo con todos los datos de la ISS."""
        # Usamos el método heredado get() de RequestHandler
        return self.get("/satellites/25544")

    def get_coordinates(self):
        """Devuelve únicamente (latitude, longitude) de la ISS."""
        position = self.get_position()
        return position["latitude"], position["longitude"]

    # PASO 3: ABSTRACCIÓN — implementación obligatoria de acquire()
    def acquire(self):
        """Cumple el contrato de TelemetrySensor devolviendo la posición."""
        return self.get_position()

    # PASO 4: POLIMORFISMO — sobreescritura de describe()
    # Pilar POO: POLIMORFISMO
    # Mismo nombre de método, comportamiento distinto al del padre.
    def describe(self):
        return "[SENSOR-ORB] ISS Tracker — NORAD ID: 25544 | Altitud: ~408 km"


# ============================================================
# PASO 2: ENCAPSULAMIENTO — Cliente Atmosférico
# Pilar POO: ENCAPSULAMIENTO
# Los atributos __lat y __lon son privados y solo se acceden
# mediante getters públicos.
# ============================================================

class AtmosphereClient(RequestHandler, TelemetrySensor):
    """
    Sensor atmosférico que consulta el clima de un punto del planeta
    usando la API Open-Meteo con las coordenadas recibidas.
    """

    def __init__(self, lat, lon):
        # HERENCIA: URL base de Open-Meteo
        super().__init__("https://api.open-meteo.com/v1")

        # ENCAPSULAMIENTO: atributos privados, no accesibles desde fuera
        self.__lat = lat
        self.__lon = lon

    # --- Getters públicos (ENCAPSULAMIENTO) ---

    def get_lat(self):
        """Devuelve la latitud almacenada de forma privada."""
        return self.__lat

    def get_lon(self):
        """Devuelve la longitud almacenada de forma privada."""
        return self.__lon

    def get_weather(self):
        """
        Construye el endpoint con atributos privados y devuelve
        únicamente el diccionario current_weather del JSON.
        """
        endpoint = f"/forecast?latitude={self.__lat}&longitude={self.__lon}&current_weather=true"
        data = self.get(endpoint)
        return data["current_weather"]

    # PASO 3: ABSTRACCIÓN — implementación obligatoria de acquire()
    def acquire(self):
        """Cumple el contrato de TelemetrySensor devolviendo el clima."""
        return self.get_weather()

    # PASO 4: POLIMORFISMO — sobreescritura de describe()
    # Pilar POO: POLIMORFISMO
    def describe(self):
        return f"[SENSOR-ATM] Open-Meteo — Coordenadas: ({self.__lat}, {self.__lon})"

    # ============================================================
    # PASO 6A: ANÁLISIS DE CONDICIONES DE OPERACIÓN
    # (Equipos de 2 personas)
    # Este método no hace peticiones HTTP, solo interpreta datos.
    # ============================================================

    def evaluate_conditions(self):
        """
        Analiza los datos del clima y devuelve una evaluación
        de las condiciones de operación.
        """
        weather = self.get_weather()
        code = weather["weathercode"]
        wind = weather["windspeed"]

        if code in (0, 1) and wind < 30:
            return "CONDICIONES ÓPTIMAS para observación"
        elif 2 <= code <= 45:
            return "CONDICIONES MODERADAS — nubosidad presente"
        else:
            return "CONDICIONES ADVERSAS — precipitación o viento intenso"

    # REFLEXIÓN PASO 6A:
    # evaluate_conditions() no hace ninguna petición HTTP, solo procesa
    # datos que ya obtuvo get_weather(). Esto respeta el principio de
    # responsabilidad única: get_weather() se encarga de obtener datos
    # y evaluate_conditions() se encarga de interpretarlos. Cada método
    # tiene una sola razón para cambiar.


# ============================================================
# PASO 6B: CLASE MissionReport
# (Equipos de 3 personas)
# Capa de orquestación que coordina múltiples sensores.
# No hereda de ninguna clase anterior.
# ============================================================

class MissionReport:
    """
    Orquestador del sistema: coordina los sensores y genera
    un reporte unificado con todos los datos consolidados.
    """

    def __init__(self, iss_client, atmosphere_client):
        # Guardamos las instancias de los sensores como atributos privados
        self.__iss = iss_client
        self.__atm = atmosphere_client

    def generate(self):
        """
        Llama a acquire() y describe() en ambos sensores y devuelve
        un diccionario con todos los datos consolidados.
        """
        return {
            "orbital_data":   self.__iss.acquire(),
            "weather_data":   self.__atm.acquire(),
            "iss_description": self.__iss.describe(),
            "atm_description": self.__atm.describe(),
        }

    def print_report(self):
        """Llama a generate() e imprime el reporte completo."""
        data = self.generate()
        orbital  = data["orbital_data"]
        weather  = data["weather_data"]

        print("=" * 40)
        print("   REPORTE DE TELEMETRÍA — ISS")
        print("=" * 40)

        print("\n[ DATOS ORBITALES ]")
        print(f"  Posición   : lat {orbital['latitude']:.2f}°  lon {orbital['longitude']:.2f}°")
        print(f"  Altitud    : {orbital['altitude']:.2f} km")
        print(f"  Velocidad  : {orbital['velocity']:.2f} km/h")
        print(f"  Visibilidad: {orbital['visibility']}")

        print("\n[ CONDICIONES ATMOSFÉRICAS BAJO LA ISS ]")
        print(f"  Temperatura: {weather['temperature']} °C")
        print(f"  Viento     : {weather['windspeed']} km/h")
        print(f"  Dirección  : {weather['winddirection']}°")
        print(f"  Cód. clima : {weather['weathercode']}")

        print("\n[ MÓDULOS DEL SISTEMA ]")
        print(f"  {data['iss_description']}")
        print(f"  {data['atm_description']}")

    # REFLEXIÓN PASO 6B:
    # MissionReport llama a acquire() y describe() sin importar qué sensor
    # recibe. Si agregaras un tercer sensor que implemente TelemetrySensor,
    # podrías extender MissionReport sin modificarlo. Esto ilustra el
    # principio Abierto/Cerrado: abierto para extensión, cerrado para
    # modificación.


# ============================================================
# PASO 5: INTEGRACIÓN — Sistema de telemetría completo
# ============================================================

if __name__ == "__main__":

    # 1. Obtenemos coordenadas reales de la ISS
    iss = ISSClient()
    lat, lon = iss.get_coordinates()

    # 2. Creamos el cliente atmosférico con esas coordenadas
    atm = AtmosphereClient(lat, lon)

    # 3. Obtenemos los datos completos de ambos sensores
    orbital_data = iss.acquire()
    weather_data  = atm.acquire()

    # 4. Imprimimos el reporte de misión
    print("=" * 40)
    print("   REPORTE DE TELEMETRÍA — ISS")
    print("=" * 40)

    print("\n[ DATOS ORBITALES ]")
    print(f"  Posición   : lat {orbital_data['latitude']:.2f}°  lon {orbital_data['longitude']:.2f}°")
    print(f"  Altitud    : {orbital_data['altitude']:.2f} km")
    print(f"  Velocidad  : {orbital_data['velocity']:.2f} km/h")
    print(f"  Visibilidad: {orbital_data['visibility']}")

    print("\n[ CONDICIONES ATMOSFÉRICAS BAJO LA ISS ]")
    print(f"  Temperatura: {weather_data['temperature']} °C")
    print(f"  Viento     : {weather_data['windspeed']} km/h")
    print(f"  Dirección  : {weather_data['winddirection']}°")
    print(f"  Cód. clima : {weather_data['weathercode']}")

    # PASO 6A: evaluación de condiciones
    print(f"\n[ EVALUACIÓN DE CONDICIONES ]")
    print(f"  {atm.evaluate_conditions()}")

    # 5. Panel de control con polimorfismo
    print("\n[ MÓDULOS DEL SISTEMA ]")
    sensores = [iss, atm]
    for s in sensores:
        print(f"  {s.describe()}")

    print("=" * 40)

    # PASO 6B: reporte con MissionReport (equipos de 3)
    # report = MissionReport(iss, atm)
    # report.print_report()
 
    def get(self, endpoint=""):
        url = self.base_url + endpoint
        response = requests.get(url)
        return response.json()
 
 
    def describe(self):
        return f"RequestHandler apuntando a: {self.base_url}"
