print("EXAMEN 2DO PRACTICO - POO    ")
print("EQUIPO 7: ")
print("Alonso Soto Barrera - 39882")
print("Salvador Flores Cazares - 39883")
print("Bruno Isaias Macias Guerrero - ")
print("Jose Alberto Magana Torres")
print("\n\n - Aplicacion de APIs  -\n\n ")

import requests
from abc import ABC, abstractmethod

class RequestHandler:
    def __init__(self, base_url):
        self.base_url = base_url
 
    def get(self, endpoint=""):
        url = self.base_url + endpoint
        response = requests.get(url)
        return response.json()
 
    def describe(self):
        return f"RequestHandler apuntando a: {self.base_url}"


class TelemetrySensor(ABC):
    @abstractmethod
    def acquire(self): 
        pass
        
#PASO 1 ------------------------------------------------------------------------

class ISSClient(RequestHandler, TelemetrySensor):
    
    def __init__(self):
      
        super().__init__("https://api.wheretheiss.at/v1")

    def get_position(self):
        return self.get("/satellites/25544")

    def get_coordinates(self):
        position = self.get_position()
        return position["latitude"], position["longitude"]

    def acquire(self):
        return self.get_position()

    def describe(self):
        return "[SENSOR-ORB] ISS Tracker — NORAD ID: 25544 | Altitud: ~408 km"

# Paso 2 - Encapsulamiento -----------------------------------------------------

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
        
        endpoint= f"/forecast?latitude={self.__lat}&longitude={self.__lon}&current_weather=true"
        data = self.get(endpoint)
        return data["current_weather"]

    def acquire(self):
        return self.get_weather()

    def describe(self):
        return f"[SENSOR-ATM] Open Meteo - Coordenadas: {self.__lat}, {self.__lon}"





Estacion = ISSClient()
print(Estacion.describe())
print("Datos ISS:", Estacion.acquire())

print("")


Atmosfera = AtmosphereClient(32.5149, -117.0382)
print(Atmosfera.describe())
print("Datos del Clima:", Atmosfera.acquire())
