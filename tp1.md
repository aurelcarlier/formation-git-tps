<img src="Git-Logo-1788C-small.png" />

TP1 : Premiers pas avec Git
===========================

Objectifs du TP :

- manipuler la staging area
- faire des commits
- consulter l'historique
- annuler des modifications de la copie locale avant l'étape commit

Pour débuter le TP, vous pouvez cloner le dépôt Forth sur [https://github.com](https://github.com) ou directement utiliser le clone fourni dans le répertoire `~/formation-git/Forth` de vos postes de travail.

Introduction
------------

Le sujet du TP est un interpréteur Forth, le code est dans `forth` et les tests dans le dossier `features`. Pour exécuter l'interpréteur, il faut utiliser `./forth` (terminer par Ctrl-D) et pour exécuter les tests, il faut utiliser `cucumber`.

1. Configurer Git
-----------------

Configurez git et indiquez votre nom et e-mail pour vous identifier comme auteur des futurs commits. Utilisez la commande `git config` ou modifier manuellement les fichiers `~/.gitconfig` ou `.git/config`.

<pre class="script">
<add>git config --global user.name  "Firstname Lastname"
git config --global user.email "firstname@lastname.name"</add>
</pre>

2. Créer une branche tp1
------------------------

Pour chaque TP, nous allons créer une branche spécifique (`tp1`) en partant d'un état connu, identifié par un tag (ici `tp1-start`). Les branches seront vues en détail au TP2.

<pre>
$ <add>git checkout -b tp1 tp1-start</add>
Switched to a new branch 'tp1'
</pre>

3. Implémentation de la soustraction
------------------------------------

* Implémenter la soustraction en s'aidant de l'implémentation existante de l'addition. Pour cela, il faut modifier le code dans `forth` et les tests dans `features/simple-operations.feature`.

  <div class="script">
  Ajouter dans <code>forth</code> :

  <pre class="context">
  &#45;-
  &#45;-  Implémentation des fonctions de base de Forth
  &#45;---------------------------------------------------
  <add>
  symbol_table['-'] = function(...)
  &nbsp; local b, a = pop(), pop()
  &nbsp; push(a - b)
  end -- '-'
  </add>
  symbol_table['+'] = function(...)
  &nbsp; local a, b = pop(), pop()
  &nbsp; push(a + b)
  end -- '+'
  </pre>

  Ajouter dans <code>features/simple-operations.feature</code> :

  <pre class="context">
  &nbsp; Scenario: Addition
  &nbsp;    When I execute "3 4 + ."
  &nbsp;    Then I should get "7 ok"
  <add>
  &nbsp; Scenario: Substraction
  &nbsp;    When I execute "3 4 - ."
  &nbsp;    Then I should get "-1 ok"
  </add>
  </pre>
  </div>

* Tester les modifications

  <pre class=script>
  $ <add>cucumber</add>
  Feature: The Forth interpreter shall understand basic operations

  &nbsp; Background:                 # features/simple-operations.feature:3
  &nbsp;   Given a forth interpreter # features/step_definitions/steps.rb:5

  &nbsp; Scenario: Addition          # features/simple-operations.feature:6
  &nbsp;   When I execute "3 4 + ."  # features/step_definitions/steps.rb:9
  &nbsp;   Then I should get "7 ok"  # features/step_definitions/steps.rb:14

  &nbsp; Scenario: Substraction      # features/simple-operations.feature:10
  &nbsp;   When I execute "3 4 - ."  # features/step_definitions/steps.rb:9
  &nbsp;   Then I should get "-1 ok" # features/step_definitions/steps.rb:14

  2 scenarios (2 passed)
  6 steps (6 passed)
  0m0.049s
  </pre>

* Utiliser `git status` et `git diff` pour visualiser les modifications de la copie locale

  <pre class=script>
  $ <add>git status</add>
  &#35; On branch tp1
  &#35; Changes not staged for commit:
  &#35;   (use "git add <file>..." to update what will be committed)
  &#35;   (use "git checkout -- <file>..." to discard changes in working directory)
  &#35;
  &#35;	modified:   features/simple-operations.feature
  &#35;	modified:   forth
  &#35;
  no changes added to commit (use "git add" and/or "git commit -a")
  </pre>

* Mettez à jour la staging area en utilisant `git add <FILE>` ou `git add -u`. N'hésitez pas a tester les autres commandes pour mettre à jour la staging area (`git reset HEAD -- <FILE>` qui est expliquée dans `git status`).

  <pre class=script>
  $ <add>git add features/simple-operations.feature forth</add>
  $ <add>git status</add>
  &#35; On branch tp1
  &#35; Changes to be committed:
  &#35;   (use "git reset HEAD <file>..." to unstage)
  &#35;
  &#35;	modified:   features/simple-operations.feature
  &#35;	modified:   forth
  &#35;
  </pre>

* Faire un commit pour la soustraction, en donnant un bref commentaire sur le contenu du commit.

  <div class=script>
  <pre>
  $ <add>git commit</add>
  [tp1 e7a419c] Add implementation for substraction
  &nbsp;2 files changed, 11 insertions(+), 2 deletions(-)
  </pre>

  Exemple de message pour le commit :

  <pre class=context>
  <add>Add implementation for substraction

  The test scenario for substraction and the implementation were added. Test
  is in the feature directory as usual and the implementation is very similar
  to addition.
  </add>
  &#35; Please enter the commit message for your changes. Lines starting
  &#35; with '#' will be ignored, and an empty message aborts the commit.
  &#35; On branch tp1
  &#35; Changes to be committed:
  &#35;   (use "git reset HEAD <file>..." to unstage)
  &#35;
  &#35;	modified:   features/simple-operations.feature
  &#35;	modified:   forth
  &#35;
  </pre>
  </div>

4. Implémentation de la multiplication
--------------------------------------

* Implémenter la multiplication de manière similaire à la soustraction.

  <div class="script">
  Ajouter dans <code>forth</code> :

  <pre class="context">
  &#45;-
  &#45;-  Implémentation des fonctions de base de Forth
  &#45;---------------------------------------------------
  <add>
  symbol_table['*'] = function(...)
  &nbsp; local a, b = pop(), pop()
  &nbsp; push(a * b)
  end -- '*'
  </add>
  symbol_table['-'] = function(...)
  &nbsp; local b, a = pop(), pop()
  &nbsp; push(a - b)
  end -- '-'
  </pre>

  Ajouter dans <code>features/simple-operations.feature</code> :

  <pre class="context">
  &nbsp; Scenario: Substraction
  &nbsp;    When I execute "3 4 - ."
  &nbsp;    Then I should get "-1 ok"
  <add>
  &nbsp; Scenario: Multiplication
  &nbsp;    When I execute "3 4 * ."
  &nbsp;    Then I should get "12 ok"
  </add>
  </pre>
  </div>

* Retester puis utiliser `git commit -a` pour faire un commit sans avoir à initialiser manuellement la staging area.

  <div class="script">
  Tester :

  <pre>
  $ <add>cucumber</add>
  Feature: The Forth interpreter shall understand basic operations

  &nbsp; Background:                 # features/simple-operations.feature:3
  &nbsp;   Given a forth interpreter # features/step_definitions/steps.rb:5

  &nbsp; Scenario: Addition          # features/simple-operations.feature:6
  &nbsp;   When I execute "3 4 + ."  # features/step_definitions/steps.rb:9
  &nbsp;   Then I should get "7 ok"  # features/step_definitions/steps.rb:14

  &nbsp; Scenario: Substraction      # features/simple-operations.feature:10
  &nbsp;   When I execute "3 4 - ."  # features/step_definitions/steps.rb:9
  &nbsp;   Then I should get "-1 ok" # features/step_definitions/steps.rb:14

  &nbsp; Scenario: Multiplication    # features/simple-operations.feature:14
  &nbsp;   When I execute "3 4 * ."  # features/step_definitions/steps.rb:9
  &nbsp;   Then I should get "12 ok" # features/step_definitions/steps.rb:14

  3 scenarios (3 passed)
  9 steps (9 passed)
  0m0.292s
  </pre>

  Commiter :

  <pre>
  $ <add>git commit -a</add>
  [tp1 2b746c6] Added implementation for multiplication
  &nbsp;2 files changed, 9 insertions(+)
  </pre>

  Message de commit :

  <pre class=context>
  <add>Added implementation for multiplication
  </add>
  &#35; Please enter the commit message for your changes. Lines starting
  &#35; with '#' will be ignored, and an empty message aborts the commit.
  &#35; On branch tp1
  &#35; Changes to be committed:
  &#35;   (use "git reset HEAD <file>..." to unstage)
  &#35;
  &#35;	modified:   features/simple-operations.feature
  &#35;	modified:   forth
  &#35;
  </pre>
  </div>

5. Consulter l'historique
-------------------------

La commande `git log` permet d'afficher l'historique des commits. Par défaut, l'entête de chaque commit (numéro de révision, date, auteur, message) est affiché.

<pre>
git log
</pre>

Il existe de nombreuses options pour filtrer la sortie et formatter l'affichage. Par exemple, pour afficher les 3 derniers commits au format patch :

<pre>
git log -3 -p
</pre>

Pour voir le graphe des révisions pour la branche courante, en vue condensée :

<pre>
$ <add>git l</add>
* 9a4ef2f (HEAD, tp1) Added implementation for multiplication as usual
* aec33b2 Add implementation for substraction
* cd0049c (origin/tp1) Prepare the code for TP1
* a356b99 RETRIEVE: new implementation for pop function
* 5559556 Special step for testing error message
* 36f836b fix gems and env for cucumber
* ce7e307 Added minimal redo and Gemfile
* df7262d Fix output
* e387718 First release
</pre>

Note : `git l` est un alias pour la commande `git log --graph --decorate --pretty=oneline --abbrev-commit` et est défini dans le fichier `.gitconfig` du projet.

Alternativement, les interfaces graphiques permettent de consulter l'historique, voir les modifications apportées par chaque commit, rechercher de l'information... de manière simple et intuitive. Par défaut, `gitk` est fourni avec git et peut être lancé depuis la ligne de commande :

    gitk &

Revenons à la ligne de commande, avec les commandes basiques pour afficher un commit en particulier et comparer plusieurs commits.

La commande `git show` permet d'affiche le contenu de n'importe quelle révision. Par défaut, ce contenu est affiché au format patch.

    git show <SHA1>

Pour identifier une révision, on peut utiliser sa référence absolue (SHA1) ou bien une référence relative (HEAD, tag, branche). Pour afficher le commit parent :

    git show HEAD^

Pour comparer l'état courant avec une révision arbitraire, il suffit de donner l'identifiant unique de cette révision (identifiant SHA1).

    git diff <SHA1>

Vous pouvez comparer deux commits entre eux

    git diff HEAD^^ HEAD^

### Rappel sur les identifiants relatifs de révision ###
- `HEAD`: le dernier commit, et le commit parent du prochain commit
- `HEAD^`: le parent de HEAD
- `HEAD^2`: le deuxième parent de HEAD (commit de merge avec plusieurs parents)
- `HEAD~1`: l'ancêtre au premier degré de HEAD, le parent (identique à `HEAD^`)
- `HEAD~2`: le grand-parent
- `HEAD~3`: l'arrière-grand-parent
- et ainsi de suite

6. Annuler des modifications (avant commit)
-------------------------------------------

* Ajouter les tests suivant dans `features/simple-operations.feature` :

  <pre class="context">
  &nbsp; Scenario: Multiplication
  &nbsp;    When I execute "3 4 * ."
  &nbsp;    Then I should get "12 ok"
  <add>
  &nbsp; Scenario: If True
  &nbsp;    When I execute "3 5 < IF 1 . ELSE 2 . THEN 3 ."
  &nbsp;    Then I should get "1 3 ok"
  &nbsp;
  &nbsp; Scenario: If False
  &nbsp;    When I execute "6 5 < IF 1 . ELSE 2 . THEN 3 ."
  &nbsp;    Then I should get "2 3 ok"
  </add>
  </pre>

* Utilisez `git add` pour préparer la staging area avec ces modifications.

  <pre class=script>
  $ <add>git add -u</add>
  $ <add>git status</add>
  &#35; On branch tp1
  &#35; Changes to be committed:
  &#35;   (use "git reset HEAD <file>..." to unstage)
  &#35;
  &#35;	modified:   features/simple-operations.feature
  &#35;
  </pre>

  Cependant, cette fonctionnalité est prématurée car les opérateurs de comparaison `<` et `>` n'ont pas encore été implémentés dans l'interpréteur. On décide donc de revenir en arrière et *de ne pas créer le commit*.

* Utiliser `git reset` pour enlever ces modifications de la staging area, tous en les laissant dans la copie locale. Visualiser les différentes étapes avec `git status` (alias `git st`).

  <pre class=script>
  $ <add>git reset HEAD features/simple-operations.feature</add>
  Unstaged changes after reset:
  M	features/simple-operations.feature
  $ <add>git st</add>
  &#35; On branch tp1
  &#35; Changes not staged for commit:
  &#35;   (use "git add <file>..." to update what will be committed)
  &#35;   (use "git checkout -- <file>..." to discard changes in working directory)
  &#35;
  &#35;	modified:   features/simple-operations.feature
  &#35;
  no changes added to commit (use "git add" and/or "git commit -a")
  </pre>

* Utiliser `git checkout <FILE>` pour supprimer ces modifications de la copie locale.

  <pre class=script>
  $ <add>git checkout features/simple-operations.feature</add>
  $ <add>git st</add>
  &#35; On branch tp1
  nothing to commit, working directory clean
  </pre>

  D'autres solutions sont possibles pour remettre à zéro la copie locale et la staging area avec différentes granularités.

* Il est possible de réaliser en une seule étape l'équivalent de `git reset` et `git checkout <FILE>` en utilisant `git checkout HEAD <FILE>`.

  <pre class=script>
  $ <add>git add -u</add>
  $ <add>git st</add>
  &#35; On branch tp1
  &#35; Changes to be committed:
  &#35;   (use "git reset HEAD <file>..." to unstage)
  &#35;
  &#35;	modified:   features/simple-operations.feature
  &#35;
  $ <add>git checkout HEAD features/simple-operations.feature</add>
  $ <add>git st</add>
  &#35; On branch tp1
  nothing to commit, working directory clean
  </pre>

* Il est aussi possible d'utiliser `git reset --hard` qui supprime les modifications sur tous les fichiers sans avoir besoin de les indiquer sur la ligne de commande.

  <pre class=script>
  $ <add>git add -u</add>
  $ <add>git st</add>
  &#35; On branch tp1
  &#35; Changes to be committed:
  &#35;   (use "git reset HEAD <file>..." to unstage)
  &#35;
  &#35;	modified:   features/simple-operations.feature
  &#35;
  $ <add>git reset --hard</add>
  HEAD is now at 2b746c6 Added implementation for multiplication as usual
  $ <add>git st</add>
  &#35; On branch tp1
  nothing to commit, working directory clean
  </pre>

