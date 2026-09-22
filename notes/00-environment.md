Environnement de développement

Dual boot vs machine virtuelle vs WSL2

Une VM ou WSL2 ajoute une couche entre Linux et le matériel. Pour de la ligne de commande ou des tâches CPU, cette couche est invisible. Pour de la 3D temps réel (Gazebo, RViz) ou un accès matériel direct (port série USB), elle devient pénalisante.

Pourquoi le dual boot pour ce projet :

- Une VM suffit pour les premiers paliers (terminal, quelques programmes), mais devient vite limitante en performance dès qu'on utilise des logiciels de modélisation 3D ou d'autres outils gourmands en ressources graphiques. Sur la durée d'un projet, cette limite se paie.
- Palier 5 (Gazebo) : la simulation physique a besoin d'une vraie accélération graphique. Une VM offre une 3D très limitée (le passthrough GPU complet est complexe à mettre en place), alors qu'en dual boot les pilotes NVIDIA de la RTX 3070 sont utilisés nativement.
- Palier 7 (ESP32 en USB) : ici ce n'est pas un problème de performance mais d'accès matériel direct. Le port série (/dev/ttyUSB0) doit être redirigé depuis l'hôte vers la VM, et cette redirection est instable : le périphérique peut se déconnecter/reconnecter à chaque reset de l'ESP32. En dual boot natif, ce problème n'existe pas.

Contrepartie acceptée : redémarrer pour changer d'OS. Ce n'est pas gênant dans mon cas car je dédie une session complète au travail sur le projet, puis je rebascule sur Windows une fois le travail terminé. Le dual boot ne deviendrait pénalisant que si je devais changer d'OS en permanence au cours d'une même session, ce qui n'est pas mon usage.

Situation personnelle : dual boot Ubuntu 24.04 / Windows déjà configuré depuis mon stage de fin d'études au BRL — je pars donc avec de l'avance sur la mission 0.1.

Distribution Linux LTS

LTS = Long Term Support. Une version LTS d'Ubuntu est maintenue pendant 5 ans (contre 9 mois pour une version standard), avec des mises à jour de sécurité et de maintenance, sans changement brutal du système. Cela donne un environnement stable et reproductible sur plusieurs années.

La robotique s'y accroche pour deux raisons liées :

- Un projet robotique dépend de nombreuses bibliothèques bas niveau (drivers, pilotes graphiques, outils de simulation). Si l'une d'elles change brutalement de version, cela peut casser tout le reste.
- ROS 2 amplifie ce besoin : chaque distribution ROS 2 est elle-même versionnée et alignée sur une version Ubuntu précise (Jazzy ↔ 24.04, Humble ↔ 22.04). ROS 2 n'est pas une simple bibliothèque ajoutée par-dessus n'importe quel Ubuntu : c'est un écosystème entier de paquets Debian compilés et testés ensemble pour ce couple (ROS, Ubuntu) précis. Sortir de ce couple expose à des incompatibilités de dépendances difficiles à diagnostiquer.

LTS assure donc la stabilité du système ; l'alignement version ROS / version Ubuntu assure la stabilité de l'écosystème robotique par-dessus.

Système de fichiers, chemins, permissions, sudo

Chemin absolu vs relatif :

- Un chemin absolu part toujours de la racine du système (/) et est valide depuis n'importe où (ex. /home/utilisateur/projet/notes/00-git.md).
- Un chemin relatif part du répertoire de travail actuel, donc le même chemin peut pointer vers des fichiers différents selon d'où on le tape (ex. notes/00-git.md, ou ../projet/notes/00-git.md pour remonter puis redescendre).

Permissions rwx :

- r (read) : lire le contenu
- w (write) : modifier/écrire
- x (execute) : exécuter (ou entrer dans un dossier)

Elles s'appliquent à trois catégories, toujours dans le même ordre mais avec des droits qui varient selon le fichier : propriétaire, groupe, autres. ls -l (liste détaillée) affiche une chaîne de 10 caractères en début de ligne, ex. -rwxr-xr-- :

- 1er caractère : type (- fichier normal, d dossier)
- caractères 2-4 : droits du propriétaire (rwx)
- caractères 5-7 : droits du groupe (r-x)
- caractères 8-10 : droits des autres (r--)

chmod sert à modifier ces droits.

sudo (superuser do) : exécute une commande avec les droits administrateur. À manier avec précaution : une erreur avec sudo (mauvais chemin dans un rm -rf, par exemple) peut endommager le système sans confirmation ni possibilité de récupération facile. On l'utilise seulement quand c'est nécessaire, jamais par réflexe.

Gestionnaire de paquets apt, dépôt, clé GPG

apt : gestionnaire de paquets qui installe, met à jour et désinstalle des logiciels ainsi que leurs dépendances. Par rapport à une installation manuelle :

- il résout les dépendances automatiquement, y compris les conflits de version entre logiciels ;
- il garde un registre de ce qui est installé, ce qui permet une désinstallation propre (sans fichiers orphelins).

Dépôt (repository) : serveur qui héberge un index des paquets disponibles (liste, versions, dépendances). apt update télécharge cet index ; apt install ne télécharge le paquet lui-même qu'ensuite.

Clé GPG : signature numérique qui garantit qu'un paquet vient bien de la source annoncée et n'a pas été modifié en chemin. Le mainteneur du dépôt (ex. l'équipe ROS) signe ses paquets avec sa clé privée ; on ajoute sa clé publique à son système, ce qui permet à apt de vérifier la signature avant installation. Cela protège contre un paquet corrompu ou une attaque de type "homme du milieu" sur un miroir compromis. C'est pourquoi la mission 0.1 demande d'ajouter la clé GPG du dépôt ROS avant d'installer Jazzy.