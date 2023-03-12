# Projet-de-programmation:Mégarama
# Les membres du groupe: 
Diomandé Vatogba Idrisa,
Amy Pouye,
Peijun GE

## Mégarama présenration :
Le Groupe Mégarama compte 30 cinémas représentant 226 écrans, implantés en France, en Espagne et Maroc.
Mégarama a débuté ses activités en 1950 grâce à son fondateur Jean-Pierre Lemoine, amoureux inconditionnel du spectacle cinématographique.
## Dans le cadre notre projet,nous avons travaillé avec le site de Mégarama afin d'automatiser tous les films diffusés du jour,de tous les cinémas Mégarama en France,les heures de début et de fin pour chaque film et leur version.

## la structure de notre codes: 
#   1. Capture tous les liens des horaires de cinéma de toutes les villes et les stocker dans un dict. 
#   2. diviser l'intervalle de code source pour tous les films dans chaque ville 
#   3. Collecter tous les information du film (la date, l'horaire de sortie et la versio)
#   4. Stocker tous les info dans un dict générale à l'aide de deux boucles
#   5. Affcher les résultats

## fonction qui crée un dictionnaire contenant les URL des horaires de cinéma de toutes les villes en fonction de leurs pages d'accueil. 

from bs4 import BeautifulSoup
import urllib3
import re
import tqdm


def get_url_horaire():
    '''
        C'est une fonction qui crée un dictionnaire contenant les URL des horaires 
        de cinéma de toutes les villes en fonction de leurs pages d'accueil. 
    '''
    req = urllib3.PoolManager()
    res = req.request('GET', 'https://megarama.fr/index-france.html')
    soup = BeautifulSoup(res.data, 'html.parser')
    urlpages = re.findall('<span class="cache-trait">-</span> <a href=(.*?)</a>',str(soup))
    links_acceuil = re.findall('"(.*?)"',str(urlpages)) 
    # une liste contenue le page d'accueil de tous les sites de cinéma de la ville
    # Par exemple, ["https://beaux-arts.megarama.fr/", ...]

    links_horaire = {}
    for link in links_acceuil:
        res = req.request('GET', link)
        soup = BeautifulSoup(res.data, 'html.parser')
        url = re.search('<a class="nav-link" href="(.*?)">Horaires',str(soup))
        if url != None and 'megarama' in url.group(1):
            # Pour filtrer les sites qui ne sont pas de type None et qui contiennent 'megarama'
            ville = re.search('https://(.*?).megarama', str(url.group(1))).group(1)
            link_horaire = url.group(1)
            links_horaire.update({ville:link_horaire})
    return links_horaire
links_horaire = get_url_horaire()
links_horaire
# un dict contenant les URLs des horaires de cinéma de toutes les villes


## fonction pour diviser l'intervalle de code source pour tous les films dans chaque ville en fonction de leur code source du site.

def get_movies(link_horaire): 
    # link_horaire: l'URL des horaires de cinéma de chaque ville, càd un élément du dict obtenu précédent.
    '''
        Il s'agit d'une fonction qui divise l'intervalle de code source pour tous les films 
        dans chaque ville en fonction de leaur code source du site.
    '''
    req = urllib3.PoolManager()
    res = req.request('GET', link_horaire)
    soup = BeautifulSoup(res.data, 'html.parser')
    content = soup.find_all('div' ,class_= 'container-horaires')
    movies = str(content).split('<!-- fin picto-->')
    movies = movies[1:]
    return movies


## fonction pour déterminer toutes les séances pour chaque film de la journée 
def get_debut_fin_version(movie):
    '''
        Il s'agit d'une fonction qui donnera la date, l'horaire de sortie et la version à 
        partir de l'intervalle de code source du film.
    '''
    date = re.findall('<a categorie="(.*?) ', str(movie))[0]
    film = re.findall('<div class="afficheTitre fittext1">(.*?)</div>',str(movie))
    debut = re.findall('<div class="heure">(.*?)</div>',str(movie))
    fin = re.findall('<div class="heureFin"><i>(.*?)</i></div>',str(movie))
    version = re.findall('class="version">(.*?)</div>',str(movie))
    if len(str(version)) > 40:
        # les codes sources de site annecy est différent que d'autres
        version= re.findall('<div class="col-3 col-sm-3 col-lg-2 BTH (.*?)"', str(movie))


    return  date, film, debut, fin, version

   
    
    
    ##imprimer les outputs de la fonction.
 # on crée une liste qui contient tous les info du film, et les stocker dans un dict à la fin
tous_villes_tous_movies = dict()
for ville in tqdm.tqdm(links_horaire):
    tous_info = [] # Cette liste est vidée après chaque boucle 
    link_horaire = links_horaire[ville]
    for movie in get_movies(link_horaire):
        date ,film, debut, fin, version = get_debut_fin_version(movie)
        info = {'film':film, 'heure debut':debut, 'heure fin':fin, 'version du film':version}
        # on stocke l'information d'un film dans un dict 
        tous_info.append(info)
        # on met le dict d'info dans la liste, c'est une liste contient tous les films de chaque ville
    tous_villes_tous_movies.update({ville:tous_info})
    # on stocke les info du film pour chaque ville      

tous = dict()
tous.update({date:tous_villes_tous_movies})
tous

##résultats        
print(tous[date]['garat']) # tous les films du mégarama à Garat aujourd'hui
print(tous[date]['garat'][0]) # les séances du premier film du mégarama à Garat aujourd'hui





