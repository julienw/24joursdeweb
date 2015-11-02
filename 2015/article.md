Une nouvelle architecture pour nos applications web mobiles
===========================================================

Le Web mobile existe depuis quelques années maintenant. Si on voulait prendre un
raccourci simpliste, on dirait qu'il est arrivé (du moins massivement) avec le
premier iPhone.

De même, nous créons des applications Web depuis quelques années. Si on voulait
prendre un raccourci simpliste, on dirait que cette mouvance est arrivée avec
GMail et Google Maps. Ah oui, je vais appeler "applications web" ces
sites web qui fonctionnent surtout en JavaScript et qui s'exécutent sur le poste
client de l'utilisateur.

Depuis ces quelques années, la communauté Web a créé des bibliothèques pour se
simplifier la vie et retrouver les automatismes d'autres environnements:
programmation suivant le paradigme MVC, concept nouveau (ou pas) de voir les
interactions utilisateurs comme un cycle, coupler les vues HTML et leur
comportement en JavaScript. Liste non exhaustive, vous en conviendrez.

Dans Firefox OS, nous avons fait le choix dès le départ de ne pas utiliser de
telles bibliothèques toutes faites. En effet, bien qu'elles soient utiles pour
le développement multi-navigateurs, elles le sont moins dès lors qu'on développe
pour un unique moteur (en l'occurrence, Gecko). De plus elles cachent trop
souvent les problèmes de la plate-forme Web alors que notre but est bien de
trouver ceux-ci afin de les corriger.


Des "Single-Page-Apps" classiques
---------------------------------

Les applications développées pour Firefox OS utilisent néanmoins des mécanismes
de Single-Page-Apps classiques: chargement d'une URL unique, systèmes qui
permettent d'afficher ou cacher des vues en fonction des actions utilisateurs,
éventuellement en modifiant l'URL, des objets en mémoire qui contiennent le
modèle.

Cela fonctionne correctement bien puisqu'on arrive à avoir des chiffres de
performance proches des mêmes applications pour Android tournant sur les mêmes
téléphones.

Cependant on voit aussi que l'on arrive à la limite de cette méthode.

Les problèmes de la méthode classique
-------------------------------------
### Utilisation d'application _packagées_

Toutes les applications préinstallées, ainsi qu'une partie des applications
installées sur le Marketplace, sont sous forme _packagées_, c'est-à-dire
qu'elles se présentent sous forme d'une archive ZIP. Cela avait été rendu
nécessaire pour la volonté de signer une application afin de lui accorder des
privilèges supplémentaires.

Mais il faut avouer que ce n'est pas très «Web», cette archive !

### Gestion des mises à jour

Cette utilisation d'une archive ZIP a plusieurs désavantages. Le moindre n'est
pas le souci des mises à jour. En effet, on est alors obligé de mettre à jour
l'ensemble de l'application même si un seul fichier change.

Par ailleurs, puisque les applications préinstallées ne proviennent pas du
Marketplace, elles ne peuvent être mises à jour qu'avec l'ensemble du système.

### Vitesse du lancement et performance ressentie

C'est évidemment une métrique très importante pour l'utilisateur. Or dans une
Single-Page App on doit forcément contrôler minutieusement le chargement initial
pour permettre un affichage rapide du premier rendu, et ce n'est pas trivial.

### Fluidité des animations

On le sait, sur mobile, les animations doivent absolument tourner sur le GPU. Il
n'empêche que même ainsi il est trop facile de rendre une animation hachée,
et surtout on y est très sensible en tant qu'humain. Il suffit de faire tourner
un peu trop de JavaScript ou déclencher un reflow au mauvais moment, sur le
thread principal.

### Gestion de la mémoire

Avec le modèle de la Single-Page App, on va garder dans une seule page
l'ensemble des données nécessaires à l'exécution de l'application, ainsi que
l'ensemble du markup nécessaire. Évidemment il y a des techniques pour améliorer
cela, mais là encore, ce n'est pas trivial.

Par ailleurs, il est très compliqué de libérer la mémoire proprement une fois
qu'elle a été allouée. Et au contraire il est très facile de conserver une
petite référence dans un coin `\_°<` qui empêche le *garbage collector* de faire
son boulot correctement. Par exemple, on oublie trop facilement d'enlever des
gestionnaire d'événements.

### Utilisation des cœurs multiples

Intrinsèquement le langage JavaScript utilise un modèle à thread unique, ce qui
signifie qu'en général un seul code JavaScript ne peut tourner à un instant
donné, même si le matériel qui fait tourner le code contient plusieurs
processeurs. C'est d'autant plus important sur mobile où les processeurs ne sont
pas particulièrement véloces __mais__ multiples.

### Adaptation de l'interface

Il faut savoir que Firefox OS tourne aujourd'hui tant sur des téléphones
d'entrée de gamme que sur des télévisions 4K. Or il  est aujourd'hui difficile
d'adapter l'interface à différents contextes. Quand
bien même les vues seraient bien délimitées par rapport au code JavaScript, il
est difficile de changer simplement son apparence ou son agencement.

Pour aller plus loin, il est extrêmement difficile de changer d'apparence à la
volée, comme pourrait le faire un système de thèmabilité.

Des outils d'ores et déjà disponibles
-------------------------------------

La plate-forme Web nous apporte des choses dès maintenant: les (Shared) Workers,
les détestées IFrames, les canaux de communication comme BroadcastChannel ou
MessageChannel, et les Service Workers.

### Les Workers pour exécuter du code en parallèle

Ah les Workers ! On sait que ça existe, mais on ne les utilise pas. Il faut
avouer que c'est un peu pénible. Notamment on n'a pas accès à la `window`, et
donc ni au DOM ni aux APIs. Jusqu'à récemment on ne pouvait même pas utiliser
indexedDB, un comble !

C'est néanmoins un outil fondamental, puisque c'est lui qui nous extraie du
modèle «thread unique» du JavaScript.

Il faut savoir qu'il en existe deux types: les *Worker*s simples, attachés à une
page Web particulière, et les *SharedWorker*s qui, comme leur nom l'indique,
sont partagés entre plusieurs pages d'une même origine.

TODO: parler des APIs disponibles

### Les IFrames pour séparer facilement des parties de l'application

Les plus vieux d'entre nous se rappellent bien de l'utilisation des frames dans
les sites web des années 90, et depuis leur simple évocation nous provoque des
frissons dans le dos.

Cependant l'IFrame reste un outil très utile: elle permet l'encapsulation totale
d'un document, tant au niveau du DOM qu'au niveau JavaScript. Cela permet de
réduire les impacts des reflows (puisque le DOM est divisé en plusieurs arbres),
et de libérer très facilement la mémoire prise par ces sous-documents (il suffit
de supprimer l'IFrame du DOM principal).

D'ailleurs, dans Firefox OS, chaque application vit dans son IFrame.

### `BroadcastChannel` et `MessageChannel` pour communiquer facilement

Ces deux objets font partie de nouvelles spécifications pour permettre aux
applications de communiquer plus facilement entre leurs différentes pages ou
scripts.

Comme son nom l'indique, avec `BroadcastChannel` une partie de l'application va
pouvoir s'abonner à un certain canal alors qu'une autre partie va pouvoir
y publier des messages. On n'a pas besoin d'avoir une référence vers la fenêtre
ou le script destination comme avec `postMessage`. Et cela fonctionne tant dans
la page principale que dans les IFrames ou les Workers. On peut voir ça comme un
gestionnaire d'événements global à une application et qui traverserait les
couches d'IFrames et de Workers.

`MessageChannel` est assez différent: il permet une communication
bidirectionnelle entre deux points d'une application. En ce sens c'est beaucoup
plus proche de `postMessage`, que l'on va d'ailleurs devoir utiliser pour passer
le port de communication à l'une des parties.

### Les Service Workers: une gestion programmatique du cache, et plus

On ne va pas trop rentrer dans les détails ici. C'est en effet un sujet très
riche.

En gros, ce qu'il faut savoir:

* Ça se place en coupure de toutes les requêtes HTTP et HTTPS. En clair, ça
  signifie qu'on va pouvoir intercepter toutes les requêtes et exécuter du code
  avant et/ou après la requête, voire la remplacer entièrement par autre chose.

  Ça signifie qu'on contrôle totalement le fonctionnement réseau de notre
  application. On peut même implémenter des URLs qui n'existent pas vraiment.

* Au sein d'un Service Worker on a accès à une API spécialisée dans le stockage
  de contenu avec une clé (l'API Cache). Cette API de cache est complètement
  sous le contrôle du code JavaScript du Service Worker.

* Un Service Worker va être installé par une application au premier accès, ou
  bien à son installation avec l'utilisation d'un manifest. Après cette
  installation, il pourra être actif quand bien même aucune fenêtre de
  l'application n'est active ou même exécutée en fond.

  Un Service Worker suit néanmoins un cycle de vie bien spécifié. Il n'est donc
  pas constamment actif, mais le moteur peut le «réveiller» lorsqu'il en a
  besoin.

TODO: prélocalisation
TODO: caching

TODO: web components
TODO: page transition


