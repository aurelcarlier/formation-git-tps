<img src="Git-Logo-1788C-small.png" />

TP8 : Import et export de patches
=================================

Objectifs du TP :

- exporter des commits sous forme de patches
- importer une série de patches pour recréer un historique
- résoudre des conflits à l'import de patches

*Pour ce TP, nous repartons sur une ancienne version de l'interpréteur, correspondant à la fin du tp2.*

<pre>
$ <add>git checkout -b tp8 tp8-start</add>
Switched to a new branch 'tp8'
</pre>


1. Exporter des modifications au format patch
---------------------------------------------

Git a été conçu autour du format patch unifié pour s'intégrer aux outils de partage de code standards. Par exemple, la commande `git diff` affiche par défaut un contenu au format patch.

* Exporter toutes les modifications entre le dernier commit et le commit taggé `tp2-start` dans un fichier `forth-tp2.patch` :

  <div class="script">
    <pre>
    $ <add>git diff tp2-start HEAD > forth-tp2.patch</add>
    </pre>
    Le patch généré contient les modifications des fichiers sources mais ne retient aucune méta-donnée sur les commits ayant enregistrés ces modifications.
  </div>


2. Exporter des commits comme une série de patches
--------------------------------------------------

Le format patch, tout en restant compatible, est aussi étendu par Git pour contenir toutes les informations nécessaires à la recréation de commits (commentaire, auteur, date). C'est une méthode très simple pour partager l'historique du code : chaque commit est exporté comme un fichier patch, qui peut alors être envoyé par mail et réimporter dans un autre dépôt, sans remote.

* Exporter les cinq derniers commits de la branche courante comme des patches (utiliser `format-patch`) :

  <div class="script">
    <pre>
    $ <add>git format-patch -5</add>
    0001-Add-STACK-system-command.patch
    0002-Add-test-and-implementation-for-duplication-op.patch
    0003-Implement-pop-n-indefinite-n-argument.patch
    0004-Basic-check-for-empty-stack.patch
    0005-Add-patch-files-for-import-exercise.patch
    </pre>
    Les fichiers de patch sont numérotés dans leur ordre d'application. Les commits de merge sont ignorés par l'export. L'import des patches risque alors de regénérer des conflits. Il est donc préférable de linéariser son historique avant de créer un export.
  </div>

* Comparer avec l'historique de la branche courante

        * cbd4606 (HEAD, tp8) Add patch files for import exercise
        *   600373e (tag: tp3-start) Merge branch 'tp2' into popn
        |\
        | * d457bef Basic check for empty stack
        * | 568e46f Implement pop(n), indefinite n argument
        |/
        *   1ebdac3 Merge branch 'stack_dup' into tp2
        |\
        | * 6c99ba4 Add test and implementation for duplication op
        * | 30a8af7 Add STACK system command
        |/
        * 514ec60 (tag: tp2-start) Added implementation for multiplication as usual

Les fichiers patch générés n'étant pas utiles pour la suite du TP, vous pouvez faire le ménage avec `git clean -f` ou `rm *.patch`.


3. Lancer l'import d'une série de patches
-----------------------------------------

Le dossier `tp-patches` contient une série de trois patches pour l'implémentation des opérateurs de comparaison et des conditions en Forth.

* Lancer l'application de ces patches (utiliser `git am`) :

  <div class="script">
    <pre>
    $ <add>git am tp-patches/000[123]-*.patch</add>
    Applying: Dispatch functions for conditions and comments
    Applying: Define Forth conditions
    /Users/deniersi/Documents/Sogilis/formations/git/Forth/.git/rebase-apply/p
    +
    error: patch failed: forth:49
    error: forth: patch does not apply
    Patch failed at 0002 Define Forth conditions
    The copy of the patch that failed is found in:
       /Users/deniersi/Documents/Sogilis/formations/git/Forth/.git/rebase-appl
    When you have resolved this problem, run "git am --resolved".
    If you prefer to skip this patch, run "git am --skip" instead.
    To restore the original branch and stop patching, run "git am --abort".
    </pre>
    Le premier patch (<code>Dispatch functions...</code>) est bien appliqué, mais il y a conflit sur l'application du second.
  </div>

* Vérifier l'application du premier patch et l'état courant

        * fd365f7 (HEAD, tp8) Dispatch functions for conditions and comments
        * cbd4606 (tag: tp8-start) Add patch files for import exercise

  <pre>
  $ <add>git st</add>
  &#35; On branch tp8
  &#35; You are in the middle of an am session.
  &#35;   (fix conflicts and then run "git am --resolved")
  &#35;   (use "git am --skip" to skip this patch)
  &#35;   (use "git am --abort" to restore the original branch)
  &#35;
  nothing to commit, working directory clean
  </pre>
  En cas de conflit, <code>git am</code> laisse la copie de travail propre par défaut, contrairement à <code>git merge</code>. Git détecte cependant que nous sommes dans le cadre d'une session <code>am</code>.


4. Résoudre le conflit à l'import du second patch
-------------------------------------------------

Git n'a pas pu appliqué le second patch car il entre en conflit avec la copie locale. Nous allons demander à Git d'utiliser les informations historiques contenues dans le patch pour appliquer un merge *3-way*.

* Réappliquer le second patch en mode *3-way merge* :

  <div class="script">
    <pre>$ <add>git am -3</add>
    Applying: Define Forth conditions
    Using index info to reconstruct a base tree...
    M	forth
    +
    warning: 1 line adds whitespace errors.
    Falling back to patching base and 3-way merge...
    Auto-merging forth
    CONFLICT (content): Merge conflict in forth
    Failed to merge in the changes.
    Patch failed at 0002 Define Forth conditions
    The copy of the patch that failed is found in:
       /Users/deniersi/Documents/Sogilis/formations/git/Forth/.git/rebase-appl
    When you have resolved this problem, run "git am --resolved".
    If you prefer to skip this patch, run "git am --skip" instead.
    To restore the original branch and stop patching, run "git am --abort".
    </pre>
  </div>

* Résoudre le conflit (utiliser `git mergetool`)

  <div>
    Vérifiez bien que la staging area est sans conflit avant de continuer.
    <pre>
    $ <add>git st</add>
    &#35; On branch tp8
    &#35; You are in the middle of an am session.
    &#35;   (fix conflicts and then run "git am --resolved")
    &#35;   (use "git am --skip" to skip this patch)
    &#35;   (use "git am --abort" to restore the original branch)
    &#35;
    &#35; Changes to be committed:
    &#35;   (use "git reset HEAD <file>..." to unstage)
    &#35;
    &#35;  new file:   features/conditions.feature
    &#35;  modified:   forth
    </pre>
  </div>

* Continuer le process `am` pour finaliser l'application du patch :

  <pre class="script">
  $ <add>git am --resolved</add>
  Applying: Define Forth conditions
  Applying: Implement missing integer comparison operations
  error: patch failed: forth:49
  error: forth: patch does not apply
  ...
  </pre>


5. Résoudre le conflit à l'import du troisième patch
----------------------------------------------------

Nous pourrions procéder de la même manière pour le troisième patch. Mais nous pouvons aussi utiliser les outils unix standard pour appliquer manuellement le fichier patch, sans passer par Git.

* Utiliser la commande `patch` pour appliquer le dernier fichier patch :

  <div>
    <pre>
    $ <add>patch -p1 < tp-patches/0003-Implement-missing-integer-comparison-
      operations.patch</add>
    patching file forth
    Hunk #1 succeeded at 60 with fuzz 2 (offset 11 lines).
    </pre>
    La commande <code>patch</code> est plus permissive que Git pour l'application d'un patch.
  </div>

* Relancer les tests :

  <pre>
    $ <add>cucumber</add>
    ...
    9 scenarios (9 passed)
    29 steps (29 passed)
    0m0.232s
  </pre>

* Ajouter les modifications locales dans la staging area :

  <pre class="script">
  $ <add>git add -u</add>
  </pre>

* Continuer le process `am` pour finaliser l'application du patch :

  <pre class="script">
  $ <add>git am --resolved</add>
  Applying: Implement missing integer comparison operations
  </pre>

* Visualiser l'historique :

        * 2c43799 (HEAD, tp8) Implement missing integer comparison operations
        * bce8c1e Define Forth conditions
        * fd365f7 Dispatch functions for conditions and comments
        * cbd4606 (tag: tp8-start) Add patch files for import exercise
        *   600373e (tag: tp3-start) Merge branch 'tp2' into popn
