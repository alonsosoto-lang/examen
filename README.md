import requests
from abc import ABC, abstractmethod

# Respuestas a las preguntas de reflexión:
# 1. El método get construye la URL completa uniendo base_url y endpoint. Devuelve un diccionario (JSON).
# 2. Tiene "" por defecto para permitir peticiones a la URL base sin argumentos adicionales.
# 3. Se construye: https://api.wheretheiss.at/v1/satellites/25544

# --- Clase Base (NO MODIFICAR) ---
class RequestHandler:
    def __init__(self, base_url):
        self.base_url = base_url
 
    def get(self, endpoint=""):
        url = self.base_url + endpoint
        response = requests.get(url)
        return response.json()
 
    def describe(self):
        return f"RequestHandler apuntando a: {self.base_url}"

# --- Paso 3: Abstracción ---
class TelemetrySensor(ABC):
    @abstractmethod
    def acquire(self):
        pass

# --- Paso 1: Herencia e ISSClient ---
class ISSClient(RequestHandler, TelemetrySensor):
    def __init__(self):
        super().__init__("https://api.wheretheiss.at/v1")

    def get_position(self):
        return self.get("/satellites/25544")

    def get_coordinates(self):
        data = self.get_position()
        return (data['latitude'], data['longitude'])

    def acquire(self):
        return self.get_position()

    def describe(self):
        data = self.get_position()
        alt = round(data.get('altitude', 0), 2)
        return f"[SENSOR-ORB] ISS Tracker — NORAD ID: 25544 | Altitud: ~{alt} km"

# --- Paso 2 & 6A: Encapsulamiento y AtmosphereClient ---
class AtmosphereClient(RequestHandler, TelemetrySensor):
    def __init__(self, lat, lon):
        super().__init__("https://api.open-meteo.com/v1")
        self.__lat = lat
        self.__lon = lon

    def get_lat(self):
        return self.__lat

    def get_lon(self):
        return self.__lon

    def get_weather(self):
        endpoint = f"/forecast?latitude={self.__lat}&longitude={self.__lon}&current_weather=true"
        response = self.get(endpoint)
        return response.get('current_weather', {})

    def acquire(self):
        return self.get_weather()

    def evaluate_conditions(self):
        data = self.get_weather()
        code = data.get('weathercode', 0)
        wind = data.get('windspeed', 0)
        if code in [0, 1] and wind < 30:
            return "CONDICIONES ÓPTIMAS para observación"
        elif 2 <= code <= 45:
            return "CONDICIONES MODERADAS — nubosidad presente"
        else:
            return "CONDICIONES ADVERSAS — precipitación o viento intenso"

    def describe(self):
        return f"[SENSOR-ATM] Open-Meteo — Coordenadas: ({self.__lat}, {self.__lon})"

# --- Paso 6B: Clase MissionReport ---
class MissionReport:
    def __init__(self, iss_client, atm_client):
        self.__iss = iss_client
        self.__atm = atm_client

    def generate(self):
        return {
            "iss": self.__iss.acquire(),
            "weather": self.__atm.acquire(),
            "desc_iss": self.__iss.describe(),
            "desc_atm": self.__atm.describe(),
            "evaluation": self.__atm.evaluate_conditions()
        }

    def print_report(self):
        data = self.generate()
        print(" REPORTE DE TELEMETRÍA — ISS")
        print("=" * 40)
        print("\n[ DATOS ORBITALES ]")
        print(f"  Posición   : lat {data['iss']['latitude']}°  lon {data['iss']['longitude']}°")
        print(f"  Altitud    : {data['iss']['altitude']} km")
        print(f"  Velocidad  : {data['iss']['velocity']} km/h")
        print(f"  Visibilidad: {data['iss']['visibility']}")
        print("\n[ CONDICIONES ATMOSFÉRICAS BAJO LA ISS ]")
        print(f"  Temperatura: {data['weather']['temperature']} °C")
        print(f"  Viento     : {data['weather']['windspeed']} km/h")
        print(f"  ANÁLISIS   : {data['evaluation']}")
        print("\n[ MÓDULOS DEL SISTEMA ]")
        print(f"  {data['desc_iss']}")
        print(f"  {data['desc_atm']}")

# --- Paso 5: Integración ---
if __name__ == "__main__":
    client_iss = ISSClient()
    lat, lon = client_iss.get_coordinates()
    client_atm = AtmosphereClient(lat, lon)
    
    reporte = MissionReport(client_iss, client_atm)
    reporte.print_report()
