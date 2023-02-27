# Projet-de-programmation
# Les membres du groupe: 
Diomandé Vatogba Idrisa,
Amy Pouye,
Peijun GE

from bs4 import BeautifulSoup
import urllib3

req = urllib3.PoolManager()
res = req.request('GET', 'https://beaux-arts.megarama.fr/FR/43/horaires-cinema-beaux-arts-besancon.html')
soup = BeautifulSoup(res.data, 'html.parser')
content = soup.find_all('div' ,class_= 'container-horaires')
content
