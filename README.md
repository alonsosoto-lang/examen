# examen
exameeen


# base.py — NO MODIFICAR


print("EXAMEN 2DO PRACTICO - POO    ")
print("EQUIPO 7: ")
print("Alonso Soto Barrera - 39882")
print("Salvador Flores Cazares - 39883")
print("Bruno Isaias Macias Guerrero - ")
print("Jose Alberto Magana Torres- 39435")
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
#PASO 1 -------------------------------------------------------------------------------------------------------

class ISSClient(RequestHandler, TelemetrySensor):
   
    #Hereda de RequestHandler para reutilizar el método get() sin repetir código.
    #Hereda de TelemetrySensor para cumplir el contrato de sensores del sistema.



    def __init__(self):
        
        #HERENCIA (Paso 1):
        #El constructor NO recibe la URL como parámetro; la fija internamente.
      
    
        super().__init__("https://api.wheretheiss.at/v1")
        # La URL queda encapsulada aquí; el usuario de esta clase no necesita
        # conocerla ni escribirla manualmente.

    def get_position(self):
     
        #Llama al método heredado get() con el endpoint de la ISS.        
        return self.get("/satellites/25544")

    def get_coordinates(self):
 
        position = self.get_position()
        return position["latitude"], position["longitude"]

    def acquire(self):
        
        return self.get_position()

    def describe(self):
      
        return "[SENSOR-ORB] ISS Tracker — NORAD ID: 25544 | Altitud: ~408 km"
