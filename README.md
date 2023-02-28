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

movies = str(content).split('<!-- fin picto-->')
for movie in movies:
    print(movie)
  
 movies = movies[1:]
for movie in movies:
    print(movie)


## fonction pour déterminer toutes les séances pour cahque film de la journée 
import re

def get_debut_fin_version(movie):
    position_str = str(movie).find('<a categorie=') # où se trouve la chaîne de caractères '<a categorie='
    position_date = position_str + len('<a categorie=') + 1 # Où se trouve la date
    date = str(movie)[position_date : position_date + 10] 

    film = re.findall('<div class="afficheTitre fittext1">(.*?)</div>',str(movie))
    debut = re.findall('<div class="heure">(.*?)</div>',str(movie))
    fin = re.findall('<div class="heureFin"><i>(.*?)</i></div>',str(movie))
    version = re.findall('class="version">(.*?)</div>',str(movie))
    
    
    return date,film, debut, fin, version
    
    
##imprimer les outputs de la fonction. 
for movie in movies:
    print(get_debut_fin_version(movie))
