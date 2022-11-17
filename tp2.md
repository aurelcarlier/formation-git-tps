<img src="Git-Logo-1788C-small.png" />

TP2 : Branches locales
======================

Objectifs du TP :

- créer une nouvelle branche
- changer de branche
- supprimer une branche
- fusionner (merge) deux branches et résoudre des conflits de merge


1. Créer des branches
---------------------

* Créer une branche `tp2` qui démarre du tag `tp2-start` et se placer sur cette branche (utiliser `git checkout`).

  <pre class="script">
  $ <add>git checkout -b tp2 tp2-start</add>
  Switched to a new branch 'tp2'
  </pre>

* Créer une branche `dup_op` au même endroit que `tp2` en utilisant `git branch`.

  <div class="script">
  <pre>$ <add>git branch dup_op</add></pre>
  ou
  <pre>$ <add>git branch dup_op tp2</add></pre>
  </div>

* Utiliser `git branch` pour vérifier la branche en cours et vérifier que les autres branches ont été créées. Vous devriez être sur `tp2` et voir la branche `dup_op` créée.

  <pre class="script">
  $ <add>git branch</add>
  &nbsp; dup_op
  &nbsp; tp1
  &#42; tp2
  </pre>

* Changer la branche en cours pour `dup_op` et vérifier avec `git branch`.

  <pre class="script">
  $ <add>git checkout dup_op</add>
  Switched to branch 'dup_op'
  $ <add>git branch</add>
  &#42; dup_op
  &nbsp; tp1
  &nbsp; tp2
  </pre>

2. Supprimer une branche
------------------------

* Créer une branche `nouvelle-branche` en utilisant `git checkout -b`, vérifiez que vous êtes dessus avec `git branch` ou `git status`.

* Tenter de supprimer `nouvelle-branche` (`git branch -d`).

* Changer la branche en cours pour `dup_op`.

* Supprimer `nouvelle-branche` et vérifier que cela a fonctionné.

<pre class=script>
$ <add>git checkout -b nouvelle-branche</add>
Switched to a new branch 'nouvelle-branche'
$ <add>git branch</add>
&#42; nouvelle-branche
  dup_op
  tp1
  tp2
$ <add>git status</add>
# On branch nouvelle-branche
nothing to commit, working directory clean
$ <add>git branch -d nouvelle-branche</add>
error: Cannot delete the branch 'nouvelle-branche' which you are currently on.
$ <add>git co dup_op</add>
Switched to branch 'dup_op'
$ <add>git branch -d nouvelle-branche</add>
Deleted branch nouvelle-branche (was 2b746c6).
</pre>

3. Définir DUP dans la branche dup_op
-------------------------------------

DUP est une commande pour dupliquer le dernier élément de la pile Forth.

* Implémenter cette commande (faire un pop du dernier élément et deux push) et ajouter un scénario de test.

  <div class="script">
  Ajouter dans <code>forth</code>:

  <pre class="context">
  &#45;-
  &#45;-  Implémentation des fonctions de base de Forth
  &#45;---------------------------------------------------
  <add>
  symbol_table.DUP = function(...)
  &nbsp; local a = pop()
  &nbsp; push(a, a)
  end -- DUP
  </add>
  symbol_table['*'] = function(...)
  &nbsp; local a, b = pop(), pop()
  &nbsp; push(a * b)
  end -- '*'
  </pre>

  Ajouter dans <code>features/simple-operations.feature</code>:

  <pre class="context">
  &nbsp; Scenario: Multiplication
  &nbsp;    When I execute "3 4 * ."
  &nbsp;    Then I should get "12 ok"
  <add>
  &nbsp; Scenario: Duplication
  &nbsp;    When I execute "1 DUP . ."
  &nbsp;    Then I should get "1 1 ok"
  </add>
  </pre>
  </div>

* Tester avec cucumber.

  <pre class=script>
  $ <add>cucumber</add>
  Feature: The Forth interpreter shall understand basic operations
  ...
  4 scenarios (4 passed)
  12 steps (12 passed)
  0m0.770s
  </pre>

* Mettre les modifications en staging et les commiter.

  <pre class="script">
  $ <add>git add -u</add>
  $ <add>git commit -m "Add test and implementation for duplication op"</add>
  [dup_op d0bbf38] Add test and implementation for duplication op
  &nbsp;2 files changed, 10 insertions(+)
  </pre>

* Vérifier que vous avez l'historique suivant avec `git log` ou `gitk` :

        * d0bbf38 (HEAD, dup_op) Add test and implementation for duplication op
        * 2b746c6 (tag: tp2-start, tp2) Added implementation for multiplication

  (ce log est obtenu grâce à l'alias `git l`.)


4. Définir STACK dans la branche principale
-------------------------------------------

* Changer de branche pour la branche `tp2` (vérifier la branche en cours).

  <pre class=script>
  $ <add>git co tp2</add>
  Switched to branch 'tp2'
  $ <add>git st</add>
  &#35; On branch tp2
  nothing to commit, working directory clean
  $ <add>git br</add>
  &nbsp; dup_op
  &nbsp; tp1
  &#42; tp2
  </pre>

* Ajouter la commande `STACK` définie avec le code suivant :

          for i = 1, #stack do
            io.write(tostring(stack[i]), " ")
          end

  <b>Note importante :</b> faites attention à ne pas implémenter la commande STACK au même endroit que la commande DUP dans la branche dup_op, afin de ne pas provoquer de conflits. Vérifiez l'endroit où vous avez inséré DUP en utilisant par exemple <code>gitk dup_op</code> ou <code>git log -p dup_op</code>.
  
  <div class=script>
  Ajouter dans <code>forth</code>:

  <pre class="context">
  symbol_table['.'] = function(...)
  &nbsp; io.write(tostring(pop()), " ")
  end -- '.'
  <add>
  symbol_table.STACK = function(...)
  &nbsp; for i = 1, #stack do
  &nbsp;   io.write(tostring(stack[i]), " ")
  &nbsp; end
  end -- STACK
  </add>
  &#45;-
  &#45;-  Dispatch
  </pre>
  </div>

* Implémenter le test suivant dans un *nouveau* fichier `features/system.feature` :

  <pre class="context">
  <add>
  Feature: The Forth interpreter shall understand system commands

    Background:
      Given a forth interpreter

    Scenario: STACK command displays the content of the stack
      When I execute "3 1 2 STACK"
      Then I should get "3 1 2 ok"

    Scenario: STACK command does not change the stack
      When I execute "3 1 2 STACK"
      Then I should get "3 1 2 ok"
      When I execute "STACK"
      Then I should get "3 1 2 ok"
  </add>
  </pre>

* Tester avec `cucumber` puis faire un commit, sans oublier le nouveau fichier de *features* :

  <pre class=script>
  $ <add>git add features/system.feature</add>
  $ <add>git ci -am "Add STACK system command"</add>
  [tp2 739687e] Add STACK system command
  &nbsp;1 file changed, 14 insertions(+)
  &nbsp;create mode 100644 features/system.feature
  </pre>

  Vous pouvez vérifier l'historique de toutes les branches avec `git l --all` ou
  `git l --branches`:

        * 739687e (HEAD, tp2) Add STACK system command
        | * d0bbf38 (dup_op) Add test and implementation for duplication op
        |/  
        * 2b746c6 (tag: tp2-start) Added implementation for multiplication as usual


5. Intégrer les modifications de dup_op dans la branche principale
------------------------------------------------------------------

* Utiliser `git merge` pour faire un merge de la branche `dup_op` dans la branche courante `tp2`.

  <div class=script>
  <pre>$ <add>git merge dup_op</add>
  Auto-merging forth
  Merge made by the 'recursive' strategy.
  &nbsp;features/simple-operations.feature | 5 +++++
  &nbsp;forth                              | 5 +++++
  &nbsp;2 files changed, 10 insertions(+)
  </pre>
  
  Un éditeur de texte s'ouvre pour que vous puissiez modifier le message du commit de merge. Le message n'ayant pas besoin d'être modifié, vous pouvez simplement fermer la fenêtre.
  </div>

* Vérifier l'absence de conflits dans les messages du terminal et avec `git status`. Vérifier l'historique :

        *   0425423 (HEAD, tp2) Merge branch 'dup_op' into tp2
        |\  
        | * d0bbf38 (dup_op) Add test and implementation for duplication op
        * | 739687e Add STACK system command
        |/  
        * 2b746c6 (tag: tp2-start) Added implementation for multiplication as usual

* Tester avec cucumber. Vérifier que les tests passent sur DUP et STACK.

  <pre class=script>
  $ <add>cucumber</add>
  Feature: The Forth interpreter shall understand basic operations
    ...
  Feature: The Forth interpreter shall understand system commands
    ...
  6 scenarios (6 passed)
  20 steps (20 passed)
  0m0.196s
  </pre>

* Supprimer dup_op qui est maintenant mergée dans la branche principale.

  <pre class=script>
  $ <add>git br -d dup_op</add>
  Deleted branch dup_op (was d0bbf38).
  </pre>

6. Créer une branche popn
-------------------------

* Créer une branche `popn` à partir de la branche `tp2`. N'oubliez pas d'utiliser `git checkout` afin que les nouveaux développements se fassent sur `popn`.

  <pre class=script>
  $ <add>git co -b popn</add>
  Switched to a new branch 'popn'
  </pre>

* Modifier le code de pop pour accepter un paramètre `n` qui permet d'indiquer le nombre d'éléments à retourner. S'inspirer du code suivant:

        function pop(n)
          if n == nil or n == 1 then
            return table.remove(stack)
          else
            local last    = pop(1)
            local res     = { pop(n-1) }
            res[#res + 1] = last
            return table.unpack(res)
          end
        end -- pop

  Vous pouvez aussi modifier le commentaire au début du fichier (ligne 21):

        -- pop(n)        --> dépile les n dernières valeurs de la pile `stack`

  Ainsi que les différents appels à pop pour utiliser ce nouveau paramètre. Par exemple dans l'implémentation de l'opérateur `-` :

          local a, b = pop(2)

  <pre class="script context">
  local push, pop, dispatch, main_loop

  <add>-- pop(n)        --> dépile les n dernières valeurs de la pile `stack`</add>
  -- push(a, ...)  --> empile toutes les valeurs passées en paramètre dans
  -- dispatch(...) --> procédure appelée pour dispatcher les symboles lus
  -- main_loop()   --> exécute la boucle principale
  ...

  symbol_table['*'] = function(...)
  <add>  local a, b = pop(2)</add>
  &nbsp; push(a * b)
  end -- '*'

  symbol_table['-'] = function(...)
  <add>  local a, b = pop(2)</add>
  &nbsp; push(a - b)
  end -- '-'

  symbol_table['+'] = function(...)
  <add>  local a, b = pop(2)</add>
  &nbsp; push(a + b)
  end -- '+'
  ...

  &#45;-
  &#45;-  Fonctions concernant la pile
  &#45;-----------------------------------
  <add>
  function pop(n)
  &nbsp; if n == nil or n == 1 then
  &nbsp;   return table.remove(stack)
  &nbsp; else
  &nbsp;   local last    = pop(1)
  &nbsp;   local res     = { pop(n-1) }
  &nbsp;   res[#res + 1] = last
  &nbsp;   return table.unpack(res)
  &nbsp; end</add>
  end -- pop
  </pre>

* Vérifier que le code fonctionne grâce à cucumber, puis faire un commit.

  <pre class=script>
  $ <add>cucumber</add>
  ...
  $ <add>git commit -am "Implement pop(n), indefinite n argument"</add>
  [popn 433c4af] Implement pop(n), indefinite n argument
  &nbsp;1 file changed, 12 insertions(+), 5 deletions(-)
  </pre>

  Votre historique devrait correspondre à :

        * 568e46f (HEAD, popn) Implement pop(n), indefinite n argument
        *   1ebdac3 (tp2) Merge branch 'dup_op' into tp2
        |\  
        | * 6c99ba4 Add test and implementation for duplication op
        * | 30a8af7 Add STACK system command
        |/  
        * 514ec60 (tag: tp2-start) Added implementation for multiplication as usual


7. Corriger une erreur dans la branche principale
-------------------------------------------------

Le but de cette modification est de modifier le message d'erreur en cas d'opération sur un pile vide pour le rendre plus explicite. Actuellement, le comportement suivant est observé :

<pre>
> .
./forth:49: bad argument #1 to 'tostring' (value expected)
> 1 +
./forth:35: attempt to perform arithmetic on local 'a' (a nil value)
</pre>

Nous allons laisser de côté la branche `popn` pour l'instant et faire ces modifications dans la branche principale `tp2`.

* Revenir sur la branche principale `tp2` :

  <pre class=script>
  $ <add>git co tp2</add>
  Switched to branch 'tp2'
  </pre>

* Ajouter le test pour cette fonctionnalité dans un nouveau fichier `features/errors.feature` :

  <pre class="context">
  <add>
  Feature: The Forth interpreter shall throw meaningfull errors

    Background:
      Given a forth interpreter

    Scenario: Stack is empty
       When I execute "."
       Then I should receive the error "stack is empty"
  </add>
  </pre>

* Implémenter cette fonctionnalité en ajoutant le code suivant au début de `function pop()`:

          assert(#stack > 0, "stack is empty")

  <pre class="context script">
  &#45;-
  &#45;-  Fonctions concernant la pile
  &#45;-----------------------------------

  function pop()
  <add>&nbsp; assert(#stack > 0, "stack is empty")</add>
  &nbsp; return table.remove(stack)
  end -- pop
  </pre>

* Tester avec cucumber puis créer le commit :

  <pre class="script">
  $ <add>cucumber</add>
  ...
  $ <add>git add features/errors.feature</add>
  $ <add>git commit -am "Basic check for empty stack"</add>
  [tp2 d457bef] Basic check for empty stack
  &nbsp;2 files changed, 9 insertions(+)
  &nbsp;create mode 100644 features/errors.feature
  </pre>

* Vérifier l'historique :

        * d457bef (HEAD, tp2) Basic check for empty stack
        | * 568e46f (popn) Implement pop(n), indefinite n argument
        |/  
        *   1ebdac3 Merge branch 'dup_op' into tp2
        |\  
        | * 6c99ba4 Add test and implementation for duplication op
        * | 30a8af7 Add STACK system command
        |/  
        * 514ec60 (tag: tp2-start) Added implementation for multiplication as usual


8. Intégrer les modifications de la branche principale dans popn
----------------------------------------------------------------

* Revenir sur la branche `popn` :

  <pre class="script">
  $ <add>git co popn</add>
  Switched to branch 'popn'
  </pre>

* Faire un merge de la branche `tp2` dans `popn` :

  <pre class="script">
  $ <add>git merge tp2</add>
  Auto-merging forth
  CONFLICT (content): Merge conflict in forth
  Automatic merge failed; fix conflicts and then commit the result.
  </pre>

* Vous devriez avoir un conflit : vérifiez avec `git status`. Résoudre le conflit de la manière qui vous convient (utiliser `git mergetool`, résoudre à la main dans l'éditeur de texte...).

  <div class=script>
  <pre>
  $ <add>git st</add>
  &#35; On branch popn
  &#35; You have unmerged paths.
  &#35;   (fix conflicts and run "git commit")
  &#35;
  &#35; Changes to be committed:
  &#35;
  &#35;	new file:   features/errors.feature
  &#35;
  &#35; Unmerged paths:
  &#35;   (use "git add <file>..." to mark resolution)
  &#35;
  &#35;	both modified:      forth
  &#35;
  </pre>
  
  Dans <code>forth</code>, vous devriez voir :

  <pre class=context>
  function pop(n)
  <<<<<<< HEAD
  &nbsp; if n == nil or n == 1 then
  &nbsp;   return table.remove(stack)
  &nbsp; else
  &nbsp;   local last    = pop(1)
  &nbsp;   local res     = { pop(n-1) }
  &nbsp;   res[#res + 1] = last
  &nbsp;   return table.unpack(res)
  &nbsp; end
  ||||||| merged common ancestors
  &nbsp; return table.remove(stack)
  &#61;======
  &nbsp; assert(#stack > 0, "stack is empty")
  &nbsp; return table.remove(stack)
  &gt;>>>>>> tp2
  end -- pop
  </pre>

  Modifier manuellement <code>forth</code> :

  <pre class=context>
  function pop(n)<add>
  &nbsp; assert(#stack > 0, "stack is empty")
  &nbsp; if n == nil or n == 1 then
  &nbsp;   return table.remove(stack)
  &nbsp; else
  &nbsp;   local last    = pop(1)
  &nbsp;   local res     = { pop(n-1) }
  &nbsp;   res[#res + 1] = last
  &nbsp;   return table.unpack(res)
  &nbsp; end</add>
  end -- pop
  </pre>
  
  À tout moment, si vous souhaitez refaire la résolution de merge d'une autre manière, vous pouvez revenir à l'état initial en utilisant <code>git checkout --merge</code> :
  
  <pre>
  $ <add>git co --merge forth</add>
  </pre>
  </div>

* Si nécessaire, marquer le conflit de fichier comme résolu (suivre les indications de `git status`).

  <pre class=script>
  $ <add>git add forth</add>
  </pre>

* Tester avec `cucumber` et faire le commit pour terminer le merge :

  <pre class=script>
  $ <add>cucumber</add>
  ...
  $ <add>git commit</add>
  [popn 600373e] Merge branch 'tp2' into popn
  </pre>

* Supprimer les fichiers non versionnés laissés par la résolution du conflit (utiliser `git clean`) :

  <pre class=script>
  $ <add>git st</add>
  &#35; On branch popn
  &#35; Untracked files:
  &#35;   (use "git add <file>..." to include in what will be committed)
  &#35;
  &#35;	forth.orig
  nothing added to commit but untracked files present (use "git add" to track)
  $ <add>git clean -n</add>
  Would remove forth.orig
  $ <add>git clean -f</add>
  Removing forth.orig
  $ <add>git st</add>
  &#35; On branch popn
  nothing to commit, working directory clean
  </pre>

* Voici l'historique après résolution du conflit :

        *   600373e (HEAD, popn) Merge branch 'tp2' into popn
        |\  
        | * d457bef (tp2) Basic check for empty stack
        * | 568e46f Implement pop(n), indefinite n argument
        |/  
        *   1ebdac3 Merge branch 'dup_op' into tp2
        |\  
        | * 6c99ba4 Add test and implementation for duplication op
        * | 30a8af7 Add STACK system command
        |/  
        * 514ec60 (tag: tp2-start) Added implementation for multiplication as usual


9. Intégrer les modifications de la branche popn dans la branche principale
---------------------------------------------------------------------------

Nous allons maintenant faire un merge dans l'autre sens.

* Revenir sur la branche `tp2` :

  <pre class="script">
  $ <add>git co tp2</add>
  Switched to branch 'tp2'
  </pre>

* Faire un merge de la branche `popn` - vous devriez remarquer que le merge est un fast-forward.

  <pre class="script">
  $ <add>git merge popn</add>
  Updating d457bef..600373e
  Fast-forward
  &nbsp;forth | 17 ++++++++++++-----
  &nbsp;1 file changed, 12 insertions(+), 5 deletions(-)
  </pre>

* Supprimer la branche `popn` maintenant inutile :

  <pre class=script>
  $ <add>git branch -d popn</add>
  Deleted branch popn (was 600373e).
  </pre>

* Vérifier l'historique :

        *   600373e (HEAD, tp2) Merge branch 'tp2' into popn
        |\  
        | * d457bef Basic check for empty stack
        * | 568e46f Implement pop(n), indefinite n argument
        |/  
        *   1ebdac3 Merge branch 'dup_op' into tp2
        |\  
        | * 6c99ba4 Add test and implementation for duplication op
        * | 30a8af7 Add STACK system command
        |/  
        * 514ec60 (tag: tp2-start) Added implementation for multiplication as usual

