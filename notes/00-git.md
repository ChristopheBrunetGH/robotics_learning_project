# Socle Git

## Commit

Un commit est un **instantané** (snapshot) de l'état complet du projet à un
instant donné — pas seulement "la modification" faite. Il contient :
- les fichiers tels qu'ils sont à ce moment,
- un message,
- l'auteur, la date,
- une référence vers son (ou ses) commit(s) parent(s), ce qui forme la
  chaîne de l'historique.

Chaque commit est identifié par un **hash** unique, calculé à partir de son
contenu.

## Branche

Une branche n'est **pas une copie du projet**. C'est un simple **pointeur
mobile** vers un commit (un petit fichier contenant un hash). Créer une
branche est donc instantané, sans copie de fichiers. Quand on commite sur
une branche, son pointeur avance automatiquement vers le nouveau commit.

**Merge** : intègre les commits d'une branche dans l'historique d'une autre
(en général `main`), via un commit de fusion. `main` avance pour inclure ces
changements. La branche peut ensuite être supprimée : ce n'est qu'un
pointeur en moins, les commits restent dans l'historique de `main`.

**Pourquoi travailler sur une branche séparée est sûr :** tant qu'on n'a pas
mergé, `main` n'a pas bougé. Si le travail ne convient pas, il suffit de
retourner sur `main` (`git switch main`) et de supprimer la branche ratée
(`git branch -D nom-branche`) — rien sur `main` n'a été touché. C'est un
bac à sable jetable.

## Remote

Un remote est l'URL d'un autre dépôt Git — pas forcément GitHub, ça peut
être n'importe quel serveur (privé, un dossier sur une autre machine...).
GitHub est un service qui héberge des remotes. `origin` est le nom donné
par défaut au remote principal. Un remote sert à la fois de sauvegarde
en ligne et de point de collaboration entre plusieurs personnes.

## HEAD détaché

Normalement `HEAD` pointe vers une **branche**, qui elle-même pointe vers un
commit (HEAD → main → commit X). Un HEAD détaché survient quand `HEAD`
pointe **directement** sur un commit (ou un tag), sans passer par une
branche — typiquement via `git checkout <hash>` (ou `git switch <hash>`)
pour inspecter un ancien commit.

**Piège :** si on commite dans cet état, les nouveaux commits existent mais
aucune branche ne pointe vers eux. En rebasculant sur une autre branche, ils
deviennent orphelins (invisibles, puis supprimés par le garbage collector),
sauf si on a créé une branche à cet endroit avant de continuer à travailler
(`git switch -c nouvelle-branche`).

Règle pratique : `checkout`/`switch` avec un **nom de branche** pour se
déplacer normalement ; en HEAD détaché (consultation d'un vieux commit), ne
jamais commiter sans d'abord créer une branche à cet endroit.

## Merge vs rebase

**Merge** : crée un **commit de fusion** avec deux parents (le dernier
commit de la branche et le dernier commit de `main`). Le graphe montre la
fourche puis le point de recollement. L'historique reflète fidèlement ce qui
s'est passé, y compris en parallèle.

**Rebase** : Git ne déplace pas les commits tels quels, il les **rejoue un
par un** au-dessus du dernier commit de `main`, en créant à chaque fois un
**nouveau commit** (même changement, mais parent différent donc nouveau
hash). Les anciens commits de la branche disparaissent, remplacés par leurs
équivalents rejoués. Résultat : un historique linéaire, sans fourche.

**Quand utiliser l'un ou l'autre :**
- **Rebase** : pour nettoyer son propre historique local, tant que
  **personne d'autre n'a récupéré** ces commits. Donne un historique plus
  lisible.
- **Merge** : dès qu'une branche est partagée ou déjà poussée pour d'autres.

**Pourquoi ne jamais rebaser une branche partagée :** le rebase change les
hash des commits. Si quelqu'un a déjà `pull` les anciens commits avant le
rebase, il garde en local les anciens hash pendant que le remote reçoit les
nouveaux. Git ne voit aucun lien entre les deux séries : historique
divergent, commits dupliqués, conflits confus à résoudre côté collègue.

---

# Pratique — Mission 0.3

## La zone d'index (staging area)

`git add` place le contenu dans la **staging area**, une zone intermédiaire
entre le dossier de travail et l'historique. Ce n'est pas juste "une liste
de fichiers sélectionnés" : c'est un instantané exact de ce que sera le
prochain commit, jusqu'au niveau de la ligne (`git add -p` permet de ne
stager que certaines lignes d'un fichier modifié, pour committer séparément
le reste).

Enchaînement observé : `git status` (fichier modifié, non indexé) → `git
add` (fichier indexé, "staged", prêt) → `git commit` (le contenu indexé
devient un commit permanent et immuable).

## Exercice 1 — Branche, fusion, suppression

Séquence testée : `git switch -c experiment/readme-style`, modification du
README, `git add` / `git commit -m "style: rework README formatting"`,
retour sur `main` avec `git switch main`, puis `git merge
experiment/readme-style`.

**Observation :** `git log --oneline --graph` a montré une **ligne droite**,
pas de commit de fusion à deux parents. C'est un **fast-forward** : comme
`main` n'avait pas bougé pendant le travail sur la branche, Git n'a eu qu'à
avancer le pointeur de `main` tout droit jusqu'au dernier commit de la
branche — pas besoin de réconcilier deux historiques divergents. Un vrai
commit de merge (à deux parents) n'apparaît que si `main` a aussi progressé
en parallèle.

`git branch -d experiment/readme-style` vérifie si la branche est fusionnée
dans la branche locale courante ; un message d'avertissement peut apparaître
si une branche de suivi (upstream) existe encore côté GitHub. Bonne
pratique : pousser d'abord (`git push`), supprimer la branche ensuite. Pour
supprimer aussi côté GitHub : `git push origin --delete <branche>`.

## Exercice 2 — Conflit de fusion

Provoqué en modifiant la même ligne du tableau de suivi des paliers sur deux
branches différentes (`branche_a` → 1 jour, `branche_b` → 4 jours), fusionnées
toutes les deux dans `main`. Résultat du deuxième merge :

```
<<<<<<< HEAD
| 0 | Mise en place professionnelle          | 🟡 En cours | 1 jours |
=======
| 0 | Mise en place professionnelle          | 🟡 En cours | 4 jours |
>>>>>>> branche_b
```

**Sens des marqueurs :**
- `<<<<<<< HEAD` : début du bloc, et le contenu jusqu'à `=======` est la
  version de la branche **sur laquelle on se trouve** au moment du merge
  (ici `main`, qui contenait déjà le commit de `branche_a`).
- `=======` : séparateur entre les deux versions.
- `>>>>>>> branche_b` : fin du bloc ; le contenu juste au-dessus vient de la
  branche qu'on est en train de **fusionner**.

Ce n'est donc pas "ancien vs nouveau" au sens chronologique, mais **"où je
suis" (HEAD) vs "ce que j'intègre"** (nom après `>>>>>>>`).

Résolution : supprimer les 3 lignes de marqueurs, garder le contenu voulu,
`git add`, puis `git commit` (message de merge pré-rempli par Git).

**Point important :** un conflit résolu ne détruit rien du passé. Les deux
commits d'origine (`branche_a` et `branche_b`) restent intacts et consultables
dans l'historique (`git show <hash>`) avec leur contenu respectif. Le commit
de merge a deux parents, et le choix fait en résolvant le conflit ne décide
que du contenu **du fichier à partir de ce point**, pas de ce qui a existé
avant. **L'historique Git est cumulatif, jamais destructif par un merge.**
Les couleurs de `git log --graph` servent uniquement à distinguer
visuellement les chemins qui divergent puis se rejoignent — elles n'indiquent
en rien un contenu "retenu" ou "perdu".

## Exercice 3 — Annuler un mauvais commit : reset vs revert

Test : commit d'un gros fichier binaire inutile (`fake_large_file.bin`,
10 Mo), poussé sur GitHub par erreur (hash `63233dc`).

| Commande | Effet | Réécrit l'historique ? |
|---|---|---|
| `git reset --soft <commit>` | Ramène la branche en arrière ; les changements annulés reviennent en staging, rien n'est perdu | Oui, localement |
| `git reset --hard <commit>` | Ramène la branche en arrière ; les changements annulés sont détruits (working directory compris) | Oui, localement |
| `git revert <commit>` | Crée un **nouveau commit** qui applique l'inverse exact du commit ciblé | Non — l'historique est complété, pas réécrit |

`reset` déplace le pointeur de branche **vers le passé** (comme si le commit
n'avait jamais existé sur cette branche) ; `revert` avance en ajoutant un
commit correctif, le commit fautif restant visible dans l'historique.

**Pourquoi c'est dangereux sur une branche déjà poussée :** `reset --hard`
réécrit l'historique local. Forcer le push ensuite (`git push --force`) casse
la synchronisation de quiconque a déjà récupéré l'ancien historique — même
danger que le rebase d'une branche partagée. Sur une branche
publique/partagée, on utilise **`revert`**, jamais `reset --hard` +
force-push.

Cas testé (fichier déjà poussé) : `git revert 63233dc`, puis `git push`. Le
fichier disparaît du dossier de travail, mais son ajout reste visible dans
l'historique via un commit qui l'annule.

**Cas d'un vrai secret (mot de passe) déjà poussé — question de validation :**
`revert` ne suffit PAS : le secret reste lisible dans l'ancien commit
(`git show <hash>`) même après le revert, puisque revert ne supprime rien du
passé. `reset --soft`/`--hard` ne suffisent pas non plus une fois poussé : le
commit fautif existe toujours côté remote tant qu'on n'a pas forcé le push,
et un clone/fetch antérieur peut déjà en contenir une copie. La bonne
procédure a deux volets :
1. Réécrire l'historique pour purger le secret de **tous les commits
   passés** qui le contiennent, avec un outil dédié (`git filter-repo`, BFG
   Repo-Cleaner) — pas un simple `reset` — puis `git push --force` en
   prévenant les autres qu'ils doivent re-cloner.
2. **Révoquer/changer le secret immédiatement**, indépendamment du nettoyage
   Git. Un secret poussé sur un dépôt public doit être considéré comme
   compromis définitivement dès l'instant du push (des bots scannent GitHub
   en continu) : nettoyer l'historique limite les dégâts futurs, ça n'annule
   pas l'exposition déjà survenue.

## Exercice 4 — `.gitignore` pour Python + C++ + ROS 2

Contenu retenu : `build/`, `install/`, `log/`, `__pycache__/`, `*.pyc`,
`.venv/`, `*.so`, `*.o`, `.vscode/`.

**Pourquoi `build/` et `install/` sont le piège classique d'un débutant sur
un workspace ROS 2 (compilé avec `colcon`) :**
- `build/` contient les fichiers intermédiaires de compilation (cache
  CMake, fichiers objets, code généré) — le "chantier" de la compilation.
- `install/` contient les binaires compilés, avec des chemins absolus codés
  en dur propres à la machine (`/home/utilisateur/...`), inutilisables tels
  quels ailleurs.
- `log/` contient les journaux de build/run, utiles seulement en local.

Ces dossiers sont entièrement **régénérables** à partir du code source
(`colcon build`), **liés à une machine précise**, **volumineux**, et leur
suivi génère des conflits de merge sur du contenu généré et sans intérêt.
Règle générale, valable dans n'importe quel langage : **Git suit le code
source, jamais ce que la compilation produit à partir de lui.**

## Exercice 5 — Convention de message de commit

Adoption de **Conventional Commits** (`type(scope): description`), avec les
types utilisés dans ce dépôt : `docs:`, `style:`, `chore:`, `fix:`, `test:`
(déjà appliqués par exemple dans `docs: add palier 0 theoretical notes`,
`style: rework README formatting`, `chore: add .gitignore for python/c++/ros2
workspace`).

**Pourquoi je l'adopte :**
- Lisibilité immédiate de l'historique pour moi-même dans plusieurs mois ou
  pour un collaborateur.
- Filtrage rapide : `git log --grep="^fix"` retrouve instantanément tous les
  commits de correction, sans lire tout l'historique.
- Base pour de l'automatisation (changelog généré, versionnage sémantique
  selon les types de commits).
- Signal de rigueur pour un recruteur qui consulte le dépôt.

## Commandes pratiquées durant cette session

`init` (via GitHub + `git init` local), `status`, `add`, `commit`, `push`,
`log` (`--oneline`, `--graph`), `diff`, `branch` (`-c`, `-d`), `switch`,
`merge`, `revert`. Restent à pratiquer en conditions réelles : `clone`,
`pull`, `fetch`, `restore`, `stash`, `tag`, `remote` (au-delà de `origin`
déjà configuré) — à l'occasion des prochains paliers.

