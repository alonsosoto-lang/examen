# examen



# base.py — NO MODIFICAR


print("EXAMEN 2DO PRACTICO - POO    ")
print("EQUIPO 7: ")
print("Alonso Soto Barrera - 39882")
print("Salvador Flores Cazares - 39883")
print("Bruno Isaias Macias Guerrero - ")
print("Jose Alberto Magana Torres")
print("\n\n - Aplicacion de APIs  -\n\n ")

import requests

class RequestHandler:
    def __init__(self, base_url):
        self.base_url = base_url
 
 
    def get(self, endpoint=""):
        url = self.base_url + endpoint
        response = requests.get(url)
        return response.json()
 
 
    def describe(self):
        return f"RequestHandler apuntando a: {self.base_url}"

#----------------------------------------
# Paso 2 - Encapsulamiento
#----------------------------------------

class AtmosphereClient(RequestHandler):

    def __init__(self, lat, lon):

        super().__init__("https://api.open-meteo.com/v1")

        self.__lat = lat
        self.__lon = lon

    def get_lat(self)

        return self.__lat

    def get_lon(self)

        return self.__lon

    def get_weather(self):

        endpoint= f"/forecast?latitude={self.__lat}&longitude={self.__lon}&current_weather=true"

        data = self.get(endpoint)

        return data["current_weather"]
    
    
