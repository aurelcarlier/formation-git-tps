<img src="Git-Logo-1788C-small.png" />

TP6 : Cherry-pick, stash, revert...
===================================

Objectifs du TP :

- utiliser cherry-pick pour récupérer n'importe quel commit d'une autre branche
- utiliser le stash pour sauvegarder temporairement des modifications
- utiliser revert pour annuler un commit erroné et checkout pour récupérer un fichier de l'historique

<pre>
$ <add>git checkout -b tp6 tp6-start</add>
Switched to a new branch 'tp6'
</pre>

1. Faire un cherry-pick pour récupérer l'opération SWAP
-------------------------------------------------------

La branche distante `origin/more_ops` contient plusieurs commits avec de nouvelles opérations. Nous sommes ici intéressés par l'opération SWAP, qui échange les deux premiers éléments de la pile.

        * ee7d4bf (origin/more_ops) Implement EXIT system command
        * 2af61dd Implement SWAP operation
        * 5ff05fe Implement NOP operation

* Récupérer le commit implémentant SWAP avec un cherry-pick

  <div class="script">
  <pre>
  &#35; En utilisant une référence relative (parent du dernier commit de more_ops)
  $ <add>git cherry-pick origin/more_ops~1</add>
  &#35; Ou, en utilisant le SHA1 du commit
  $ <add>git cherry-pick 2af61dd</add>
  [tp6 3b92224] Implement SWAP operation
   1 file changed, 5 insertions(+)
  </pre>

  Dans ce cas, un merge ou un rebase n'aurait pas été pratique car on ne veut pas récupérer les commits autour de SWAP dans la branche <code>more_ops</code>.
  </div>

* Visualiser l'historique

        * 3b92224 (HEAD, tp6) Implement SWAP operation
        * 8c98774 Implement quotient and modulo operators

2. Implémenter l'opération DROP
-------------------------------

* Créer un nouveau fichier `features/stack.feature` avec un test pour l'opération DROP, qui enlève le premier élément de la pile

  <pre class="script"><add>
  Feature: The Forth interpreter shall understand stack operations
  &nbsp;
  &nbsp; Background:
  &nbsp;   Given a forth interpreter
  &nbsp;
  &nbsp; Scenario: Drop Top Element
  &nbsp;    When I execute "1 2 DROP ."
  &nbsp;    Then I should get "1 ok"
  </add></pre>

* Implémenter l'opération DROP dans `forth`

  <pre class="script">
  &#45;-
  &#45;-  Implémentation des fonctions de base de Forth
  &#45;----------------------------------------------------
  <add>
  symbol_table.DROP = function(...)
  &nbsp; pop()
  end -- DROP
  </add>
  symbol_table.DUP = function(...)
  &nbsp; local a = pop()
  &nbsp; push(a, a)
  end -- DUP
  </pre>

* Avant de créer le commit pour DROP, on décide finalement d'intégrer la branche `origin/more_ops`

3. Stasher des modifications en cours pour intégrer une branche
---------------------------------------------------------------

Intégrer la branche `origin/more_ops` avec un merge échoue à ce stade car notre fichier `forth` est non sauvegardé dans la copie locale.

  <pre>
  $ <add>git merge origin/more_ops </add>
  error: Your local changes to the following files would be overwritten by merg
  	forth
  Please, commit your changes or stash them before you can merge.
  Aborting
  </pre>

* Mettre de côté les modifications locales dans le stash

  <div class="script">
  <pre>
  $ <add>git stash save 'Implement DROP op'</add>
  Saved working directory and index state On tp6: Implement DROP op
  HEAD is now at 3b92224 Implement SWAP operation
  $ <add>git st</add>
  &#35; On branch tp6
  &#35; Untracked files:
  &#35;   (use "git add <file>..." to include in what will be committed)
  &#35;
  &#35;	features/stack.feature
  nothing added to commit but untracked files present (use "git add" to trac
  </pre>
  Le nouveau fichier <code>stack.feature</code> n'est pas encore versionné. Il n'a pas été pris en compte par le stash (il l'aurait été si ajouté dans staging).
  </div>

* Intégrer la branche `origin/more_ops` avec un merge

  <pre class="script">
  $ <add>git merge origin/more_ops</add>
  Auto-merging forth
  Merge made by the 'recursive' strategy.
  &nbsp;features/system.feature | 4 ++++
  &nbsp;forth                   | 8 ++++++++
  &nbsp;2 files changed, 12 insertions(+)
  </pre>

4. Récupérer des modifications du stash
---------------------------------------

* Récupérer les modifications pour DROP à partir du stash

  <pre class="script">
  $ <add>git stash apply</add>
  Auto-merging forth
  &#35; On branch tp6
  &#35; Changes not staged for commit:
  &#35;   (use "git add <file>..." to update what will be committed)
  &#35;   (use "git checkout -- <file>..." to discard changes in working directory)
  &#35;
  &#35;	modified:   forth
  &#35;
  &#35; Untracked files:
  &#35;   (use "git add <file>..." to include in what will be committed)
  &#35;
  &#35;	features/stack.feature
  no changes added to commit (use "git add" and/or "git commit -a")
  Dropped refs/stash@{0} (fd644558958781b256559dfef45051d195127acd)
  </pre>

* Relancer les tests et créer le commit (sans oublier le nouveau fichier de test)

  <pre class="script">
  $ <add>cucumber</add>
  ...
  16 scenarios (16 passed)
  50 steps (50 passed)
  0m0.325s
  $ <add>git add -A</add>
  $ <add>git ci -m 'Implement DROP operation'</add>
  [tp6 0e8e291] Implement DROP operation
   2 files changed, 12 insertions(+)
   create mode 100644 features/stack.feature
  </pre>

5. Ecrire un test pour SWAP
---------------------------

Il n'y a pas de test pour l'opération SWAP récupérée avec le cherry-pick.

* Ecrire le test suivant dans `features/stack.feature`

  <pre>
  Scenario: Drop Top Element
     When I execute "1 2 DROP ."
     Then I should get "1 ok"
  <add>
  Scenario: Swap Top Two Elements
     When I execute "1 2 SWAP . ."
     Then I should get "1 2 ok"
  </add></pre>

* Lancer les tests. Ceux-ci doivent échouer : il y a un bug dans l'implémentation de SWAP !

  <pre>
  $ <add>cucumber</add>
  ...
  Failing Scenarios:
  cucumber features/stack.feature:10 # Scenario: Swap Top Two Elements
  17 scenarios (1 failed, 16 passed)
  53 steps (1 failed, 52 passed)
  0m0.427s
  </pre>

6. Annuler le commit buggé de SWAP
----------------------------------

* Mettre de côté le test de SWAP

  <pre class="script">
  $ <add>git stash</add>
  </pre>

* Annuler le commit de cherry-pick avec `git revert`

  <pre class="script">
  $ <add>git revert 3b922244621077e325e762d42aa92c73815d4d8d</add>
  [tp6 af03fc6] Revert "Implement SWAP operation"
   1 file changed, 5 deletions(-)
  </pre>

* Lancer les tests qui doivent être verts

  <pre>
  $ <add>cucumber</add>
  ...
  16 scenarios (16 passed)
  50 steps (50 passed)
  0m0.289s
  </pre>

7. Corriger SWAP
----------------

La branche `tp6` étant maintenant stable, on peut s'appliquer à corriger le bug de SWAP.

* Récupérer le test pour SWAP

  <pre class="script">
  $ <add>git stash apply</add>
  </pre>

* Récupérer la version précédente du fichier `forth` avec l'implémentation de SWAP (utiliser `git checkout`)

  <pre class="script">
  $ <add>git checkout tp6~1 -- forth</add>
  </pre>

* Corriger SWAP

  <pre>
  symbol_table.SWAP = function(...)
  &nbsp; local a, b = pop(2)
  &nbsp; <add>push(b, a)</add>
  end -- SWAP  
  </pre>

* Relancer les tests et créer le commit avec la correction

  <pre class="script">
  $ <add>cucumber</add>
  17 scenarios (17 passed)
  53 steps (53 passed)
  0m0.323s
  ...
  $ <add>git add -u</add>
  $ <add>git commit -m 'Fix SWAP operation'</add>
  </pre>

* Visualiser l'historique

        * d3fecf8 (HEAD, tp6) Fix SWAP operation
        * af03fc6 Revert "Implement SWAP operation"
        * 0e8e291 Implement DROP operation
        *   d4a52a4 Merge branch 'more_ops' into tp6
        |\  
        | * ee7d4bf Implement EXIT system command
        | * 2af61dd Implement SWAP operation
        | * 5ff05fe Implement NOP operation
        * | 3b92224 Implement SWAP operation
        |/  
        * 8c98774 Implement quotient and modulo operators
