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
un peu trop de JavaScript au mauvais moment, sur le thread principal.

### Gestion de la mémoire

Avec le modèle de la Single-Page App, on va garder dans une seule page
l'ensemble des données nécessaires à l'exécution de l'application, ainsi que
l'ensemble du markup nécessaire. Évidemment il y a des techniques pour améliorer
cela, mais là encore, ce n'est pas trivial.

### Utilisation des cœurs multiples

Intrinsèquement le langage JavaScript utilise un modèle à thread unique, ce qui
signifie qu'en général un seul code JavaScript ne peut tourner à un instant
donné, même si le matériel qui fait tourner le code contient plusieurs
processeurs. C'est d'autant plus important sur mobile où les processeurs ne sont
pas particulièrement véloces __mais__ multiples.

