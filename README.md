# examen
exameeen


# base.py — NO MODIFICAR


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
import requests
from abc import ABC, abstractmethod



def get(self, endpoint=""):
    url = self.base_url + endpoint
    response = requests.get(url)
    return response.json()


def describe(self):
    return f"RequestHandler apuntando a: {self.base_url}"

class TelemetrySensor(ABC):
    def __init__(self):

    @abstractmethod
    def aquire(self):
        pass
        
class ISSClient(RequestHandler, TelemetrySensor):
    def __init__(self, base_url):
        
        def acquire(self):
            return self.get("/positions")   
        
