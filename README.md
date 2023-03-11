# Projet-de-programmation:Mégarama
# Les membres du groupe: 
Diomandé Vatogba Idrisa,
Amy Pouye,
Peijun GE

## Mégarama présenration :
Le Groupe Mégarama compte 30 cinémas représentant 226 écrans, implantés en France, en Espagne et Maroc.
Mégarama a débuté ses activités en 1950 grâce à son fondateur Jean-Pierre Lemoine, amoureux inconditionnel du spectacle cinématographique.
## Dans le cadre notre projet,nous avons travaillé avec le site de Mégarama afin d'automatiser tous les films diffusés du jour,de tous les cinémas Mégarama en France,les heures de début et de fin pour chaque film et leur version.



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


## fonction pour déterminer toutes les séances pour chaque film de la journée 
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
    
    ## Liens des cinémas Mégarama
   
   from bs4 import BeautifulSoup
import urllib3
import re

req = urllib3.PoolManager()
#res = req.request('GET', 'https://beaux-arts.megarama.fr/FR/43/horaires-cinema-beaux-arts-besancon.html')
res = req.request('GET', 'https://megarama.fr/index-france.html')

soup = BeautifulSoup(res.data, 'html.parser')
urlpages = re.findall('<span class="cache-trait">-</span> <a href=(.*?)</a>',str(soup))
links = re.findall('"(.*?)"',str(urlpages)) # ["https://beaux-arts.megarama.fr/", ....]

def get_url(links):
    cleaned_links = []
    urls = []
    for link in links:
        res = req.request('GET', link)
        soup = BeautifulSoup(res.data, 'html.parser')
        urls += [re.findall('<a class="nav-link" href="(.*?)">Horaires',str(soup))]
    for url in urls:
        if url != [] and 'horaires-cinema' in url[0]:
            cleaned_links += [url[0]]
    return cleaned_links
cleaned_links = get_url(links)
print(cleaned_links)


# code final
from bs4 import BeautifulSoup
import urllib3
import re

req = urllib3.PoolManager()
res = req.request('GET', 'https://megarama.fr/index-france.html')
soup = BeautifulSoup(res.data, 'html.parser')
urlpages = re.findall('<span class="cache-trait">-</span> <a href=(.*?)</a>',str(soup))
links = re.findall('"(.*?)"',str(urlpages)) 
# Page d'accueil de tous les sites de cinéma de la ville
# Par exemple, ["https://beaux-arts.megarama.fr/", ...]
print(links)

def get_url(links):
    '''
        C'est une fonction qui crée une liste contenant les URL des horaires 
        de cinéma de toutes les villes en fonction des pages d'accueil obtenus
        précédents. 
    '''
    cleaned_links = []
    urls = []
    for link in links:
        res = req.request('GET', link)
        soup = BeautifulSoup(res.data, 'html.parser')
        urls += [re.findall('<a class="nav-link" href="(.*?)">Horaires',str(soup))]
    for url in urls:
        if url != [] and 'megarama' in url[0]:
            # Pour filtrer les sites qui ne sont pas vides et qui contiennent 'megarama'
            cleaned_links += [url[0]]
    return cleaned_links
cleaned_links = get_url(links)
print(cleaned_links) 
# une liste contenant les URL des horaires de cinéma de toutes les villes



def get_movies(cleaned_link): 
    # cleaned_link: l'URL des horaires de cinéma de chaque ville, càd un élément de la liste obtenue précédente
    '''
        Il s'agit d'une fonction qui divise l'intervalle de code pour tous les films 
        dans chaque ville en fonction du code source du site.
    '''
    res = req.request('GET', cleaned_link)
    soup = BeautifulSoup(res.data, 'html.parser')
    content = soup.find_all('div' ,class_= 'container-horaires')
    movies = str(content).split('<!-- fin picto-->')
    movies = movies[1:]
    return movies


def get_debut_fin_version(movie, cleaned_link):
    '''
        Il s'agit d'une fonction qui donnera la ville du cinéma, la date et 
        l'horaire de sortie et la version à partir de l'intervalle de code 
        du film et URL obtenus par les fonctions précédentes.
    '''
    ville = re.findall('https://(.*?).megarama', str(cleaned_link))
    date = re.findall('<a categorie="(.*?) ', str(movie))[0]
    film = re.findall('<div class="afficheTitre fittext1">(.*?)</div>',str(movie))
    debut = re.findall('<div class="heure">(.*?)</div>',str(movie))
    fin = re.findall('<div class="heureFin"><i>(.*?)</i></div>',str(movie))
    
    version = re.findall('class="version">(.*?)</div>',str(movie))
    if len(str(version)) > 40: 
        # les codes source de site annecy est différent que d'autres
        version= re.findall('<div class="col-3 col-sm-3 col-lg-2 BTH (.*?)"', str(movie))

    return ville, date, film, debut, fin, version
        
   
   ##imprimer les outputs de la fonction.

for cleaned_link in cleaned_links:
    for movie in get_movies(cleaned_link):
        print(get_debut_fin_version(movie, cleaned_link))
        
   

    
    
