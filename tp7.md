<img src="Git-Logo-1788C-small.png" />

TP7 : Mise en forme de l'historique
===================================

Objectifs du TP :

- utiliser rebase pour transplanter une branche dans l'historique
- utiliser rebase pour linéariser un historique
- utiliser interactive rebase pour modifier l'historique

<pre>
$ <add>git checkout -b tp7 tp7-start</add>
Switched to a new branch 'tp7'
</pre>

1. Définir l'opération CLEAR
----------------------------

* Définir l'opération CLEAR qui vide la pile :

  <div class="script">
    Ajouter dans <code>forth</code>:

    <pre class="context">
    symbol_table.EXIT = function(...)
      print "bye"
      os.exit()
    end -- EXIT
    <add>
    symbol_table.CLEAR = function(...)
      stack = {}
    end -- CLEAR
    </add>
    symbol_table['<='] = function(...)
      local a, b = pop(2)
      push(a <= b)
    end -- less/equal
    </pre>

    Ajouter dans <code>features/stack.feature</code>:

    <pre class="context">
    &nbsp; Scenario: Swap Top Two Elements
    &nbsp;    When I execute "1 2 SWAP . ."
    &nbsp;    Then I should get "1 2 ok"
    <add>
    &nbsp; Scenario: CLEAR empties the stack
    &nbsp;   When I execute "1 2 3 CLEAR"
    &nbsp;   Then I should get "ok"
    &nbsp;   When I execute "."
    &nbsp;   Then I should receive the error "stack is empty"
    </add></pre>
  </div>

* Créer le commit :

  <pre class="script">
    $ <add>git ci -am 'Implement CLEAR operator'</add>
    [tp7 9741d83] Implement CLEAR operator
     2 files changed, 10 insertions(+)
  </pre>


2. Définir l'opération MIN
--------------------------

MIN prend les deux derniers éléments de la pile et pousse le plus petit sur la pile.

    2 3 MIN .
    > 2 ok

* Implémenter l'opération MIN, sans oublier les tests :

  <div class="script">
    Ajouter dans <code>forth</code>:

    <pre class="context">
    symbol_table.SWAP = function(...)
      local a, b = pop(2)
      push(b, a)
    end -- SWAP
    <add>
    symbol_table.MIN = function(...)
      local a, b = pop(2)
      if a < b then
        push(a)
      else
        push(b)
      end
    end -- 'MIN'
    </add>
    symbol_table['*'] = function(...)
      local a, b = pop(2)
      push(a * b)
    end -- '*'
  </pre>

  Ajouter dans <code>features/simple-operations.feature</code>:

  <pre class="context">
  &nbsp; Scenario: Negation
  &nbsp;    When I execute "1 NEG ."
  &nbsp;    Then I should get "-1 ok"
  <add>
  &nbsp; Scenario: Minimum
  &nbsp;    When I execute "0 1 3 MIN . ."
  &nbsp;    Then I should get "1 0 ok"
  &nbsp;    When I execute "0 3 1 MIN . ."
  &nbsp;    Then I should get "1 0 ok"
  </add></pre>
  </div>

* Créer le commit :

  <pre class="script">
    $ <add>git ci -am 'Implement MIN operator'</add>
    [tp7 976f300] Implement MIN operator
     2 files changed, 15 insertions(+)
  </pre>


3. Définir l'opération MAX
--------------------------

* Implémenter l'opération MAX en vous inspirant de MIN :

  <div class="script">
    Ajouter dans <code>forth</code>:

    <pre class="context">
    symbol_table.MIN = function(...)
      local a, b = pop(2)
      if a < b then
        push(a)
      else
        push(b)
      end
    end -- 'MIN'
    <add>
    symbol_table.MAX = function(...)
      local a, b = pop(2)
      if a > b then
        push(a)
      else
        push(b)
      end
    end -- 'MAX'
    </add>
    symbol_table['*'] = function(...)
      local a, b = pop(2)
      push(a * b)
    end -- '*'
  </pre>

  Ajouter dans <code>features/simple-operations.feature</code>:

  <pre class="context">
  &nbsp; Scenario: Minimum
  &nbsp;    When I execute "0 1 3 MIN . ."
  &nbsp;    Then I should get "1 0 ok"
  &nbsp;    When I execute "0 3 1 MIN . ."
  &nbsp;    Then I should get "1 0 ok"
  <add>
  &nbsp; Scenario: Maximum
  &nbsp;    When I execute "0 1 3 MAX . ."
  &nbsp;    Then I should get "3 0 ok"
  &nbsp;    When I execute "0 3 1 MAX . ."
  &nbsp;    Then I should get "3 0 ok"
  </add></pre>
  </div>

* Créer le commit :

  <pre class="script">
    $ <add>git ci -am 'Implement MAX operator'</add>
    [tp7 c3da000] Implement MAX operator
     2 files changed, 15 insertions(+)
  </pre>


4. Transplanter une partie de la branche courante sur une autre branche
-----------------------------------------------------------------------

Si vous regardez votre historique récent avec <code>git l --all</code>, vous pouvez voir vos trois derniers commits mais aussi une branche distante <code>origin/functions-and-more</code>, qui a divergé à partir de <code>tp7-start</code>. Cette branche contient des fonctionnalités intéressantes (une commande WORDS pour afficher tous les mots connus par l'interpréteur Forth, la syntaxe pour les commentaires et les fonctions...). Elle contient aussi déjà une définition de l'opération CLEAR, en doublon de la nôtre.

      * c3da000 (HEAD, tp7) Implement Maximum operator
      * 212f8d8 Implement Minimum operator
      * 74d8b2b Implement CLEAR operator
      | * a16cac4 (origin/functions-and-more) Define Forth comments & functions def
      | * 01735fd Dispatch for Forth functions
      | * 9741d83 Implement CLEAR operator
      | * 40038e0 Define system commands WORDS
      |/
      * d3fecf8 (tag: tp7-start) Fix SWAP operation


Nous <strong>ne pouvons pas</strong> faire un merge des deux branches sans introduire une duplication de l'implémentation de CLEAR (car les deux ne sont pas définis au même endroit dans le fichier). Par contre, il est possible de transplanter simplement notre branche avec les deux derniers commits (MIN et MAX) au-dessus de la branche <code>origin/functions-and-more</code>. Nous abandonnons donc notre commit avec l'implémentation de CLEAR au profit de l'autre implémentation.

* Transplanter les deux derniers commits de la branche courante (MIN/MAX) sur la branche `origin/functions-and-more` :

  <div class="script">
    <pre>
    $ <add>git rebase HEAD~2 --onto origin/functions-and-more</add>
    First, rewinding head to replay your work on top of it...
    Applying: Implement Minimum operator
    Applying: Implement Maximum operator
    </pre>
    HEAD~2 indique à rebase qu'il faut exclure les commits à partir du 2ème parent de HEAD, ce qui correspond au commit de CLEAR.
  </div>

* Relancer les tests :

  <pre class="script">
    $ <add>cucumber</add>
    ...
    22 scenarios (22 passed)
    74 steps (74 passed)
    0m0.451s
  </pre>

* Visualiser l'historique :

        * fff0ce3 (HEAD, tp7) Implement Maximum operator
        * 8e07284 Implement Minimum operator
        * a16cac4 (origin/functions-and-more) Define Forth comments & functions def
        * 01735fd Dispatch for Forth functions
        * 9741d83 Implement CLEAR operator
        * 40038e0 Define system commands WORDS
        * d3fecf8 (tag: tp7-start) Fix SWAP operation

Nous avons fini d'implémenter des fonctionnalités pour l'interpréteur Forth. Il est maintenant possible de définir des fonctions récursives sur la pile, par exemple avec cette implémentation de factoriel :

        > ( les commentaires sont entoures par des parentheses )
        ok
        > ( Definition recursive de factoriel )
        > : FAC DUP 1 > IF DUP 1 - FAC * ELSE 1 * THEN NOP ;
        ok
        > 5 FAC .
        120 ok
        > ( WORDS liste les mots connus par l'interpreteur, y compris les fonctions
        de l'utilisateur comme FAC )
        > WORDS
        NOP MIN % // SWAP EXIT + * - / . STACK FAC WORDS DROP ( IF NEG >= DUP <= :
        CLEAR < MAX > ok


5. Linéariser l'historique
--------------------------

Le but de la linéarisation de l'historique est de supprimer les commits de merge pour présenter une forme plus simple à revoir pour l'intégrateur, sans les merges liés au développement distribué. La linéarisation (comme toute mise en forme de l'historique) est à faire avant de partager son historique.

* [Optionnel] Créer un tag `tp7-before-rebase` sur le dernier commit :

  <div>
    <pre>
      $ <add>git tag tp7-before-rebase</add>
    </pre>
    Cette étape facultative permet d'avoir un point de retour facile à trouver dans le cas où on voudrait annuler la linéarisation et repartir de l'état courant (avec `git reset`).
  </div>

* Linéariser l'historique depuis l'étape tp2-start (utiliser `git rebase`)

  <div class="script">
    <pre>
      $ <add>git rebase tp2-start</add>
      First, rewinding head to replay your work on top of it...
      Applying: Add STACK system command
      Applying: Add test and implementation for duplication op
      Applying: Implement pop(n), indefinite n argument
      Applying: Basic check for empty stack
      Using index info to reconstruct a base tree...
      M	forth
      Falling back to patching base and 3-way merge...
      Auto-merging forth
      CONFLICT (content): Merge conflict in forth
      Failed to merge in the changes.
      Patch failed at 0004 Basic check for empty stack
      ...
    </pre>

     Si la linéarisation supprime les merges, elle reproduit aussi les conflits qui ont pu être déjà résolus et qu'il faudra à nouveau corriger (à moins que l'option `rerere` ait été activée auparavant).
  </div>

* Résoudre les conflits en cours de rebase puis continuer le rebase :

  <div class="script">
    Une fois un conflit résolu (avec `git mergetool`), on peut vérifier l'état de la copie courante avant de continuer.

    <pre>
      $ <add>git st</add>
      &#35; Not currently on any branch.
      &#35; You are currently rebasing.
      &#35;   (all conflicts fixed: run "git rebase --continue")
      &#35;
      &#35; Changes to be committed:
      &#35;   (use "git reset HEAD <file>..." to unstage)
      &#35;
      &#35;	new file:   features/errors.feature
      &#35;	modified:   forth
      &#35;
      &#35; Untracked files:
      &#35;   (use "git add <file>..." to include in what will be committed)
      &#35;
      &#35;	forth.orig
      $ <add>git rebase --continue</add>
      Applying: Basic check for empty stack
      Applying: Dispatch functions for conditions and comments
      Applying: Define Forth conditions
      ...
      Applying: Implement Maximum operator
    </pre>
  </div>

* Relancer les tests `cucumber`

* Visualiser l'historique de la branche courante, maintenant linéaire :

        * dfe2158 (HEAD, tp7) Implement Maximum operator
        * 207e3cd Implement Minimum operator
        * b4a7968 Define Forth comments & functions definitions
        * 4e4c4a5 Dispatch for Forth functions
        * c261266 Implement CLEAR operator
        * 3d2eaf1 Define system commands WORDS
        * 0fe511c Fix SWAP operation
        * 1de12b3 Revert "Implement SWAP operation"
        * a3f4ef9 Implement DROP operation
        * 5008f3e Implement EXIT system command
        * 386eed8 Implement SWAP operation
        * 7ec13aa Implement NOP operation
        * c8f1f20 Implement quotient and modulo operators

  <div>
    Avec <code>gitk --all</code>, vous pouvez voir la plupart des anciens commits. En effet, ceux-ci sont toujours référencés par d'autres branches ou tags.
  </div>

* Supprimer le tag `tp7-before-rebase` si le résultat est ok :

  <pre>
    $ <add>git tag -d tp7-before-rebase</add>
  </pre>

6. Mettre en forme l'historique
-------------------------------

Un rebase interactif permet de mettre en forme de façon fine l'historique de la branche courante. Nous allons voir différentes options possibles, comme changer l'ordre des commits, merger et supprimer des commits, ou encore éditer un commit pour le séparer en deux.

* Pour lancer un rebase interactif remontant jusqu'à tp2-start :

  <pre>
    $ <add>git rebase -i tp2-start</add>
  </pre>

  <div>
    Cette commande indique que l'on veut voir (et possiblement modifier) la liste des commits depuis le tag tp2-start. L'éditeur de texte s'ouvre avec la liste des commits dans l'ordre descendant et une action en début de ligne pour chaque commit.
  </div>

* Modifier l'ordre des commits

  <pre>
  ...
  pick b61e84d Dispatch functions for conditions and comments
  pick e3319e9 Define Forth conditions
  pick 7f8b833 Implement missing integer comparison operations
  ...
  </pre>

  Le commit des opérateurs de comparaison a malencontreusement été introduit après les conditions (cf le mot clé missing dans le message de commit). On veut remplacer par un ordre plus logique, avec les opérateurs de comparaison avant les conditions (et supprimer au passage le "missing" du commentaire).

  <div class="script">
    <pre>
    ...
    pick b61e84d Dispatch functions for conditions and comments
    <add>reword 7f8b833 Implement missing integer comparison operations
    pick e3319e9 Define Forth conditions</add>
    ...
    </pre>
    Inverser les deux lignes et choisir <code>reword</code> comme action sur le commit pour éditer le commentaire lors du rebase.

    Sous vi, utiliser dd pour couper une ligne dans le buffer et p pour insérer le buffer après la ligne courante.

    Notez que ce changement produit deux conflits coup sur coup puisque le contexte d'application des deux commits change avec l'ordre. Il faut résoudre les conflits puis continuer le rebase avec <code>git rebase --continue</code>.
  </div>

* Supprimer des commits devenus inutiles

  <pre>
  ...
  pick 2036146 Implement SWAP operation         # Buggé
  pick 8830337 Implement EXIT system command
  pick ad18943 Implement DROP operation
  pick d81125d Revert "Implement SWAP operation"
  pick 9142b61 Fix SWAP operation
  ...
  </pre>

  L'implémentation buggée de SWAP a donné lieu à plusieurs commits. On veut ne garder que le dernier bon commit et supprimer les autres (premier essai et revert).

  <div class="script">
    <pre>
    ...
    pick 8830337 Implement EXIT system command
    pick ad18943 Implement DROP operation
    <add>reword</add> 9142b61 Fix SWAP operation
    ...
    </pre>
    Supprimer les lignes des commits visés et utiliser <code>reword</code> sur le dernier commit de SWAP pour éditer le commentaire.
  </div>

* Fusionner des commits

  Pour simplifier l'historique, on peut aussi fusionner des commits voisins, comme par exemple les deux commits pour MIN et MAX.

  <div class="script">
    <pre>
    ...
    pick 3128169 Implement Minimum operator
    <add>squash</add> 8611d88 Implement Maximum operator
    </pre>
    <code>squash</code> permet de fusionner le commit visé dans le parent, tout en permettant d'éditer le commentaire. L'autre option est <code>fixup</code>.
  </div>

* Editer un commit

  L'opération la plus complexe est l'édition d'un commit, qui permet de complètement modifier son contenu et aussi de créer plusieurs commits à partir du commit original. Par exemple le commit de définition des fonctions contient aussi la définition des commentaires : on peut les séparer dans deux commits indépendants.

  <pre class="script">
  <add>edit</add> 385da0e Define Forth comments & functions definitions
  </pre>

  Git stoppe le rebase dans l'état où se trouve le commit visé. Il faut annuler le commit et l'index pour retrouver les modifications avant commit.

  <pre class="script">
  $ <add>git reset HEAD^</add>
  Unstaged changes after reset:
  M	forth
  </pre>

  La manipulation est ensuite la même que pour la création de commits partiels (avec git gui par exemple). Créer un commit avec la définition des fonctions et le fichier <code>features/functions.feature</code> :

  <pre>
  -- Define new symbol
  symbol_table[':'] = function(original_dispatcher)
      ...
  end -- define
  </pre>

  <pre class="script">
    $ <add>git commit -m 'Forth functions definition'</add>
  </pre>

  Créer un commit avec la définition des commentaires et le fichier <code>features/comments.feature</code> :

  <pre>
  -- Comment
  symbol_table['('] = function(original_dispatcher)
    return make_dispatch_skip_until(")", original_dispatcher)
  end -- comment
  </pre>

  <pre class="script">
    $ <add>git add -u</add>
    $ <add>git add features/comments.feature</add>
    $ <add>git commit -m 'Define Forth comments'</add>
  </pre>

  Finir le rebase :

  <pre class="script">
    $ <add>git rebase --continue</add>
    Successfully rebased and updated refs/heads/tp7.
  </pre>

* Itérer sur la mise en forme et visualiser l'historique

 Il est possible d'itérer plusieurs fois avec un rebase interactif. On peut appliquer une action, voir son effet, puis réitérer avec un autre rebase interactif, jusqu'à obtenir un historique simple et clair pour la revue de code.

        * 96d2174 (HEAD, tp7) Implement MIN/MAX operators
        * bcaa0a9 Implement Forth functions definition
        * 6f3a810 Define WORDS system command
        * 9f304dd Define EXIT system command
        * 841573b Implement CLEAR operator
        * bfcdd31 Implement SWAP operation
        * 938af44 Implement DROP operation
        * 1540890 Implement NOP operation
        * c61f9b7 Implement NEG operation
        * fa2ca5b Implement quotient and modulo operators
        * 5f6987f Implement division operator
        * fe1a475 Interpreter accepts float as input
        * 58f3c9f Define Forth comments
        * c543a5a Define Forth conditions
        * d9a8112 Dispatch functions for conditions and comments
        * 85514f5 Implement integer comparison operations
        * 4ae05a9 Basic check for empty stack
