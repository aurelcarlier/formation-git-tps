<img src="Git-Logo-1788C-small.png" />

TP3 : Synchronisation avec un dépôt distant
===========================================

Objectifs du TP :

- intégrer son travail avec les commits et les branches d'un dépôt distant
- pousser des commits et des nouvelles branches vers un dépôt distant

Pour commencer ce TP, créer une branche `tp3` à partir du tag `tp3-start` :

<pre>
$ <add>git checkout -b tp3 tp3-start</add>
Switched to a new branch 'tp3'
</pre>

Si git vous demande de vous authentifier à chaque fois que vous lancez une commande `fetch/pull/push`, tapez la commande suivante pour mémoriser votre identification pour la session courante:

<pre>
git config --global credential.helper cache
</pre>

1. Intégrer la branche distante div_op dans la branche tp3
----------------------------------------------------------

Sur le dépôt distant `origin`, il existe une branche `div_op` avec une implémentation de la division (flottante).

<pre>
> 5 2 / .
2.5 ok
</pre>

* Faire un merge de cette branche distante dans la branche courante `tp3` (avec `git pull`) :

  <div class=script>
  <pre>$ <add>git pull origin div_op</add>
  From github.com:sogilis/Forth
  &nbsp;* branch            div_op     -> FETCH_HEAD
  Updating 600373e..5f3ebbc
  Fast-forward
  &nbsp;features/simple-operations.feature | 4 +++-
  &nbsp;forth                              | 5 +++++
  &nbsp;2 files changed, 8 insertions(+), 1 deletion(-)
  </pre>

  La branche <code>div_op</code> descendant directement de la branche <code>tp3</code>, le merge se résoud en <em>fast forward</em>.
  </div>

2. Implémenter l'opération NEG dans la branche tp3
--------------------------------------------------

NEG dépile le dernier nombre et pousse son opposé sur la pile :

<pre>
> 5 NEG .
-5 ok
</pre>

* Définir l'opération NEG dans l'interpréteur :

  <div class="script">
  Ajouter dans <code>forth</code>:

  <pre class="context">
  symbol_table['/'] = function(...)
  &nbsp; local a, b = pop(2)
  &nbsp; push(a / b)
  end -- '/'
  <add>
  symbol_table.NEG = function(...)
  &nbsp; local a = pop()
  &nbsp; push(a * -1)
  end -- NEG
  </add>
  symbol_table['.'] = function(...)
  &nbsp; io.write(tostring(pop()), " ")
  end -- '.'
  </pre>

  Ajouter dans <code>features/simple-operations.feature</code>:

  <pre class="context">
  &nbsp; Scenario: Division
  &nbsp;    When I execute "5 2 / ."
  &nbsp;    Then I should get "2.5 ok"
  <add>
  &nbsp; Scenario: Negation
  &nbsp;    When I execute "1 NEG ."
  &nbsp;    Then I should get "-1 ok"
  </add>
  </pre>
  </div>

* Créer le commit :

  <pre class="script">
  $ <add>git add -u</add>
  $ <add>git commit -m 'Implement NEG operation'</add>
  [tp3 4939065] Implement NEG operation
   2 files changed, 9 insertions(+)
  </pre>

3. Pousser la branche locale tp3 sur un dépôt distant
-----------------------------------------------------

* Pousser les modifications sur une nouvelle branche distante `tp3` :

  <div class="script">
  <pre>
    $ <add>git push -u origin tp3</add>
    Counting objects: 9, done.
    Delta compression using up to 2 threads.
    Compressing objects: 100% (5/5), done.
    Writing objects: 100% (5/5), 514 bytes, done.
    Total 5 (delta 4), reused 0 (delta 0)
    To git@github.com:sogilis/Forth.git
    &nbsp;* [new branch]      tp3 -> tp3
    Branch tp3 set up to track remote branch tp3 from origin.
  </pre>

  Une nouvelle branche distante <code>tp3</code> est créée sur le dépôt <code>origin</code>. On utilise l'option <code>-u</code> pour enregistrer l'information de tracking entre la branche locale <code>tp3</code> et la branche distante <code>tp3</code>.
  </div>

* Vérifier l'historique :

        * 4939065 (HEAD, origin/tp3, tp3) Implement NEG operation
        * 5f3ebbc (origin/div_op) Implement division operator
        *   600373e (tag: tp3-start) Merge branch 'tp2' into popn

4. Intégrer la branche locale tp3 avec la branche distante tp3_master
---------------------------------------------------------------------

* Sur le dépôt `origin`, une branche `tp3_master` existe déjà. On chercher maintenant à intégrer la branche `tp3` dans cette branche `tp3_master`.

  <pre class="script">
  $ <add>git push origin tp3:tp3_master</add>
  To git@github.com:sogilis/Forth.git
   ! [rejected]        tp3 -> tp3_master (non-fast-forward)
  error: failed to push some refs to 'git@github.com:sogilis/Forth.git'
  hint: Updates were rejected because a pushed branch tip is behind its remot
  hint: counterpart. Check out this branch and merge the remote changes
  hint: (e.g. 'git pull') before pushing again.
  hint: See the 'Note about fast-forwards' in 'git push --help' for details.  
  </pre>

* Le push est rejeté car les deux branches `tp3_master` et `tp3` ont divergé, ce que l'on peut voir avec `git l tp3 origin/tp3_master` ou `gitk tp3 origin/tp3_master`.

        * 4939065 (HEAD, origin/tp3, tp3) Implement NEG operation
        * 5f3ebbc (origin/div_op) Implement division operator
        | * a2978ae (origin/tp3_master) Implement missing integer comparison opera
        | * e2f65d1 Define Forth conditions
        | * 660b8f4 Dispatch functions for conditions and comments
        |/  
        *   600373e (tag: tp3-start) Merge branch 'tp2' into popn

* Intégrer la branche distante `tp3_master` dans la branche `tp3` (utiliser `git pull`) :

  <div class="script">
  <pre>
  $ <add>git st</add>
  &#35; On branch tp3
  nothing to commit, working directory clean
  $ <add>git pull origin tp3_master</add>
  From github.com:sogilis/Forth
  &nbsp;* branch            tp3_master -> FETCH_HEAD
  Auto-merging forth
  Merge made by the 'recursive' strategy.
  &nbsp;features/conditions.feature | 13 +++++++++++++
  &nbsp;forth                       | 57 +++++++++++++++++++++++++++++++++++++++
  &nbsp;2 files changed, 70 insertions(+)
  &nbsp;create mode 100644 features/conditions.feature
  </pre>

  Dans ce cas, le merge se passe bien. En cas de conflits, il faudra les résoudre (<code>mergetool</code>...) puis finaliser le merge avec <code>git commit</code>.
  </div>

* Réessayer le push de `tp3` vers `tp3_master` :

  <pre class="script">
  $ <add>git push origin tp3:tp3_master</add>
  Counting objects: 10, done.
  Delta compression using up to 2 threads.
  Compressing objects: 100% (4/4), done.
  Writing objects: 100% (4/4), 508 bytes, done.
  Total 4 (delta 3), reused 0 (delta 0)
  To git@github.com:sogilis/Forth.git
     a2978ae..debccf0  tp3 -> tp3_master
  </pre>

* Mettre aussi à jour `tp3` sur `origin` en utilisant la forme courte de `git push` (si l'information de tracking est en place) :

  <div class="script">
  <pre>
  $ <add>git push</add>
  Total 0 (delta 0), reused 0 (delta 0)
  To git@github.com:sogilis/Forth.git
     4939065..debccf0  tp3 -> tp3
  </pre>

  Dans ce cas, seule la branche distante <code>origin/tp3</code> est mise à jour, puisque les commits ont déjà été poussés via <code>origin/tp3_master</code>.
  </div>

* Vérifier l'historique :

        *   debccf0 (HEAD, origin/tp3_master, origin/tp3, tp3) Merge branch 'tp3_m
        |\  
        | * a2978ae Implement missing integer comparison operations
        | * e2f65d1 Define Forth conditions
        | * 660b8f4 Dispatch functions for conditions and comments
        * | 4939065 Implement NEG operation
        * | 5f3ebbc (origin/div_op) Implement division operator
        |/  
        *   600373e (tag: tp3-start) Merge branch 'tp2' into popn


5. Utiliser l'option rebase de git pull
---------------------------------------

* Créer une nouvelle branche `tp3_rebase` à partir du commit d'implémentation de la négation :

  <pre class="script">
  $ <add>git checkout -b tp3_rebase HEAD^</add>
  Switched to a new branch 'tp3_rebase'
  $ <add>git l</add>
  &#35; 4939065 (HEAD, tp3_rebase) Implement NEG operation
  &#35; 5f3ebbc (origin/div_op) Implement division operator
  &#35;   600373e (tag: tp3-start) Merge branch 'tp2' into popn
  </pre>

* Intégrer la branche distante `tp3_base` avec l'option `rebase` :

  <pre class="script">
  $ <add>git pull --rebase origin tp3_base</add>
  From github.com:sogilis/Forth
  &nbsp;* branch            tp3_base   -> FETCH_HEAD
  First, rewinding head to replay your work on top of it...
  Applying: Implement division operator
  Applying: Implement NEG operation
  </pre>

* Vérifier l'historique (et comparer avec le `pull` classique) :

        * 789ba88 (HEAD, tp3_rebase) Implement NEG operation
        * 0b33e5c Implement division operator
        * a2978ae (origin/tp3_base) Implement missing integer comparison operations
        * e2f65d1 Define Forth conditions
        * 660b8f4 Dispatch functions for conditions and comments
        *   600373e (tag: tp3-start) Merge branch 'tp2' into popn
