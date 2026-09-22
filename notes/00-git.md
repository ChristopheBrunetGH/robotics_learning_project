Socle Git

Commit

Un commit est un instantané (snapshot) de l'état complet du projet à un instant donné — pas seulement "la modification" faite. Il contient :

les fichiers tels qu'ils sont à ce moment,
un message,
l'auteur, la date,
une référence vers son (ou ses) commit(s) parent(s), ce qui forme la chaîne de l'historique.

Chaque commit est identifié par un hash unique, calculé à partir de son contenu.

Branche

Une branche n'est pas une copie du projet. C'est un simple pointeur mobile vers un commit (un petit fichier contenant un hash). Créer une branche est donc instantané, sans copie de fichiers. Quand on commite sur une branche, son pointeur avance automatiquement vers le nouveau commit.

Merge : intègre les commits d'une branche dans l'historique d'une autre (en général main), via un commit de fusion. main avance pour inclure ces changements. La branche peut ensuite être supprimée : ce n'est qu'un pointeur en moins, les commits restent dans l'historique de main.

Pourquoi travailler sur une branche séparée est sûr : tant qu'on n'a pas mergé, main n'a pas bougé. Si le travail ne convient pas, il suffit de retourner sur main (git switch main) et de supprimer la branche ratée (git branch -D nom-branche) — rien sur main n'a été touché. C'est un bac à sable jetable.

Remote

Un remote est l'URL d'un autre dépôt Git — pas forcément GitHub, ça peut être n'importe quel serveur (privé, un dossier sur une autre machine...). GitHub est un service qui héberge des remotes. origin est le nom donné par défaut au remote principal. Un remote sert à la fois de sauvegarde en ligne et de point de collaboration entre plusieurs personnes.

HEAD détaché

Normalement HEAD pointe vers une branche, qui elle-même pointe vers un commit (HEAD → main → commit X). Un HEAD détaché survient quand HEAD pointe directement sur un commit (ou un tag), sans passer par une branche — typiquement via git checkout <hash> (ou git switch <hash>) pour inspecter un ancien commit.

Piège : si on commite dans cet état, les nouveaux commits existent mais aucune branche ne pointe vers eux. En rebasculant sur une autre branche, ils deviennent orphelins (invisibles, puis supprimés par le garbage collector), sauf si on a créé une branche à cet endroit avant de continuer à travailler (git switch -c nouvelle-branche).

Règle pratique : checkout/switch avec un nom de branche pour se déplacer normalement ; en HEAD détaché (consultation d'un vieux commit), ne jamais commiter sans d'abord créer une branche à cet endroit.

Merge vs rebase

Merge : crée un commit de fusion avec deux parents (le dernier commit de la branche et le dernier commit de main). Le graphe montre la fourche puis le point de recollement. L'historique reflète fidèlement ce qui s'est passé, y compris en parallèle.

Rebase : Git ne déplace pas les commits tels quels, il les rejoue un par un au-dessus du dernier commit de main, en créant à chaque fois un nouveau commit (même changement, mais parent différent donc nouveau hash). Les anciens commits de la branche disparaissent, remplacés par leurs équivalents rejoués. Résultat : un historique linéaire, sans fourche.

Quand utiliser l'un ou l'autre :

Rebase : pour nettoyer son propre historique local, tant que personne d'autre n'a récupéré ces commits. Donne un historique plus lisible.
Merge : dès qu'une branche est partagée ou déjà poussée pour d'autres.

Pourquoi ne jamais rebaser une branche partagée : le rebase change les hash des commits. Si quelqu'un a déjà pull les anciens commits avant le rebase, il garde en local les anciens hash pendant que le remote reçoit les nouveaux. Git ne voit aucun lien entre les deux séries : historique divergent, commits dupliqués, conflits confus à résoudre côté collègue.