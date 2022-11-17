<img src="Git-Logo-1788C-small.png" />

TP9: Submodules 
===============

Clone du projet
---------------

    git clone git@github.com:sogilis/Fortunes.git
    cd Fortunes
    git checkout -b tp tp-start

Initialiser les submodules
--------------------------

    git submodule update --init

Ajouter un submodule
--------------------

Ajouter le submodule libaction (branche libaction) :

    git submodule add -b libaction git@github.com:sogilis/Fortunes.git libaction

Modifier `fortune.lua` pour qu'il l'utilise (enlever les commentaires).

Faire un commit :

    git commit -am "Added action submodule"

Visualiser les modifications avec `git show` :

    diff --git a/.gitmodules b/.gitmodules
    index e192c65..cb03f32 100644
    --- a/.gitmodules
    +++ b/.gitmodules
    @@ -6,3 +6,7 @@
            path = libplace
            url = git@github.com:sogilis/Fortunes.git
            branch = libplace
    +[submodule "libaction"]
    +       path = libaction
    +       url = git@github.com:sogilis/Fortunes.git
    +       branch = libaction
    diff --git a/fortune.lua b/fortune.lua
    index a6eafa7..078d2c3 100755
    --- a/fortune.lua
    +++ b/fortune.lua
    @@ -1,12 +1,12 @@
     #!/usr/bin/env lua
     
     people = require("libpeople/people")
    --- action = require("libaction/action")
    +action = require("libaction/action")
     place  = require("libplace/place")
     
     local function fortune()
       return people() .. " " ..
    -    -- action() .. " " ..
    +    action() .. " " ..
         place()
     end
     
    diff --git a/libaction b/libaction
    new file mode 160000
    index 0000000..a901732
    --- /dev/null
    +++ b/libaction
    @@ -0,0 +1 @@
    +Subproject commit a90173200bcfa09f330211fac01c100a48a3e7b3


Mettre à jour un submodule via submodule update
-----------------------------------------------

Mettre à jour le submodule `libplace` en utilisant l'option `--remote` de `git submodule update`

    git submodule update --remote libplace
    Submodule path 'libplace': checked out 'ed7361f11abd6c59084d33fa2e76046be3b4adff'
    
    git status
    git diff
    git diff --staged

    git commit -am "Updated libplace to new version"


Mettre à jour un submodule manuellement
---------------------------------------

Mettre à jour le submodule libpeople avec un `git pull origin libpeople-v1`

    pushd libpeople
    git pull origin libpeople-v1
    popd

Commiter le changement de version du submodule dans le dépôt principal :

    git status
    git diff
    git add libpeople
    git diff --staged

    git commit -m "Updated libpeople"

Attention, detached HEAD
------------------------

En mode detached HEAD, modifier des fichiers est dangereux. Vos submodules sont
probablement en mode detached HEAD, changer leur version pour vous placer sur
une branche. Sinon, vous devrez le faire plus tard et vous aurez peut-être des
conflits.

Visualiser les branches des submodules:

    $ git submodule foreach git branch
    Entering 'libaction'
    * libaction
      master
    Entering 'libpeople'
    * (detached from 3d7aafc)
      master
    Entering 'libplace'
    * (detached from ed7361f)
      master

Mettre à jour les submodules et choisir pour chaque submodule la branche dans laquelle on souhaite travailler :

    pushd libpeople
    git checkout libpeople
    popd

    pushd libplace
    git checkout libplace
    popd

Ajouter de nouvelles actions, lieux et personnes
------------------------------------------------

Dans chaque submodule, ajouter des éléments à la liste. Exemple d'éléments :

Personnes :

    "une tortue";
    "un chat";
    "mon ami";
    "un ver de terre";

Actions :

    "est en train de boire";
    "rêve à son chien";
    "essuie sa vaisselle";
    "va au cinéma";

Lieux :

    "dans sa chambre";
    "au sommet d'un arbre";
    "dans le placard";
    "au sommet de la tour Eiffel";

Faire un commit pour chaque module :

    git submodule foreach git citool

Regardez bien si il y a des modules en mode detached HEAD, si c'est le cas, il faut les replacer sur une branche car vous venez de créer une branche anonyme, et votre commit pourrait disparaître au prochain `git submodule update`. Si vous êtes en mode detached HEAD, la commande `git rebase <branch>; git rebase HEAD <branch>` devrait vous remettre sur la branche `<branch>`.


Charger une nouvelle version des dépôts
---------------------------------------

    git pull origin next

Il y a un conflit sur libaction (que vous avez ajouté manuellement et qui
apparaît dans next). Le résoudre en prenant la version next (et non pas la
version locale) :

    git mergetool
    git commit (ne pas mettre -a)

Pour `git commit`, ne pas mettre `-a`. La résolution du merge est en staging
area, si vous faites un `git add` (équivalent du flag `-a`), la résolution va
changer par rapport à ce que vous avez indiqué dans git mergetool pour prendre
la version que vous aviez des submodules avant le pull). En effet, lorsque
mergetool a résolu les conflits de versions, il n'a pas mis à jour les
submodules car c'est toujours une opération manuelle.

Les submodules ont été mis à jour dans le commit, mais pas dans la copie locale.
C'est le comportement par défaut de Git. Les mettre à jour avec la version de
next (avec différentes stratégie pour traiter les divergences) :
  
    git submodule update --checkout libpeople
    git submodule update --merge libaction
    git submodule update --rebase libplace

Il peut ensuite être nécessaire de faire un update recursif pour exécuter
`fortune.lua` car `libaction` dépend lui même d'un sous module:

    git submodule update --init --recursive --merge
    
(on utilise `--merge` pour ne pas perdre les modifications locales de libaction et libplace)

Retrouver les personnes perdues
-------------------------------

Avec `git submodule update --checkout`, les personnes ajoutées à libpeople ont
disparu. On peut les retrouver avec un cherry-pick:

    pushd libpeople
    git cherry-pick libpeople
    popd

Commit des versions dans le dépôt principal
-------------------------------------------

    git ci -am "Fin du TP"



