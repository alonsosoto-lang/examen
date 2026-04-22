# examen
exameeen


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
