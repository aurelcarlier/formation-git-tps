<img src="Git-Logo-1788C-small.png" />

TP4 : Synchronisation en équipe
===============================

Objectifs du TP :

- ajouter un dépôt distant
- intégrer des modifications à plusieurs

Pour ce TP, vous allez travailler en parallèle avec un de vos voisins :

- développer une première fonctionnalité vous-même
- intégrer une seconde fonctionnalité, développée en parallèle par votre collègue

<pre>
$ <add>git checkout -b tp4 tp4-start</add>
Switched to a new branch 'tp4'
</pre>

1. Ajouter un dépôt distant et récupérer son historique
-------------------------------------------------------

* Ajouter le dépôt GitHub du collègue comme remote au dépôt local :

  <div class="script">
  Remplacer `sogiuserY` par le vrai nom d'utilisateur de votre voisin dans le
  code suivant. Le nom `sogivoisinY` peut également être remplacé par un
  identifiant plus descriptif.

  <pre>
  $ <add>git remote add sogivoisinY git@github.com:sogiuserY/Forth.git</add>
  $ <add>git remote</add>
  origin
  sogivoisinY
  </pre>

  Demandez à votre voisin l'addresse de son dépôt Forth sur GitHub, en particulier son login <code>sogiuser</code>.
  </div>

* Récupérer son historique de façon à visualiser ses commits et ses branches (`git l --all` ou `gitk --all`) :

  <pre class="script">
  $ <add>git fetch sogivoisinY</add>
  remote: Counting objects: 95, done.
  remote: Compressing objects: 100% (67/67), done.
  remote: Total 95 (delta 54), reused 48 (delta 21)
  Unpacking objects: 100% (95/95), done.
  From github.com:sogilis/Forth
  &nbsp;* [new branch]      tp3        -> voisinY/tp3
  </pre>

<div class='page-breaker'></div>

Schéma de principe :

                                     SogiUserA:
           ,------------------------   - origin/*
           |          ,------------>   - sogivoisinB/*
    ,------|- GitHub -|-------.        - master
    |      v          |       |        - ...
    |      _          _       |
    |    /   \      /   \     |
    |   |\ _ /|    |\ _ /|    |
    |   |     |    |     |    |
    |   |  A  |    |  B  |    |
    |   |     |    |     |    |
    |   `_____'    `_____'    |
    |      |          ^       |
    `------|----------|-------'      SogiUserB:
           |          `-------------   - origin/*
           `----------------------->   - sogivoisinA/*
                                       - master
                                       - ...


Pour la suite, vous devez vous répartir le développement des fonctionnalités A et B avec votre collègue :

- fonctionnalité A : remplacer la division flottante par une division entière ;
- fonctionnalité B : implémenter une garde contre les divisions par zéro.

2A. Fonctionnalité A : remplacer la division flottante par une division entière
-------------------------------------------------------------------------------

* Modifier le code de l'opération `/` pour définir une division entière au lieu d'une division flottante (utiliser la fonction `math.floor` de `lua`) :

  <div class="script">
  Modifier dans <code>forth</code> :
  <pre>
  symbol_table['/'] = function(...)
    local a, b = pop(2)
    <add>push(math.floor(a / b))</add>
  end -- '/'
  </pre>

  Modifier dans <code>features/simple-operations.feature</code> :
  <pre>
  &nbsp; Scenario: Division
  &nbsp;    When I execute "5 2 / ."
  &nbsp;    <add>Then I should get "2 ok"</add>
  </pre>
  </div>

* Relancer les tests avec `cucumber`

* Faire le commit et pousser sur le dépôt `origin` :

  <pre class="script">
  $ <add>git commit -am 'Redefine division as quotient'</add>
  $ <add>git push -u origin tp4</add>
  </pre>

Passer ensuite directement à la section 3.

2B. Fonctionnalité B : implémenter une garde contre les divisions par zéro
--------------------------------------------------------------------------

* Modifier le code de l'opération `/` pour ajouter une garde contre les divisions par zéro :

        assert(b > 0, "divide by zero")

  <div class="script">
  Modifier dans <code>forth</code> :
  <pre>
  symbol_table['/'] = function(...)
    local a, b = pop(2)
    <add>assert(b > 0, "divide by zero")</add>
    push(a / b)
  end -- '/'
  </pre>

  Ajouter dans <code>features/simple-operations.feature</code> :
  <pre>
  &nbsp; Scenario: Division
  &nbsp;    When I execute "5 2 / ."
  &nbsp;    Then I should get "2.5 ok"
  <add>
  &nbsp; Scenario: Division by zero
  &nbsp;    When I execute "1 0 /"
  &nbsp;    Then I should receive the error "divide by zero"
  </add>
  </pre>
  </div>

* Relancer les tests avec `cucumber`

* Faire le commit et pousser sur le dépôt `origin` :

  <pre class="script">
  $ <add>git commit -am 'Add guard for division by zero'</add>
  $ <add>git push -u origin tp4</add>
  </pre>

3. Intégrer la fonctionnalité du dépôt voisin
---------------------------------------------

* Synchroniser l'historique du dépôt voisin :

  <div class="script">
  <pre>
  $ <add>git fetch sogivoisinY</add>
  From github.com:sogilis/Forth
  &nbsp;* [new branch]      tp4        -> sogivoisinY/tp4
  </pre>

  Ceci permet de récupérer et de voir les modifications du dépôt distant sans les intégrer de suite.
  </div>

* Intégrer la branche `tp4` du dépôt voisin avec la branche courante `tp4` :

  <div class="script">
  <pre>
  &#35; git pull commence par un git fetch
  $ <add>git pull sogivoisinY tp4</add>
  From github.com:sogilis/Forth
  &nbsp;* branch            tp4        -> FETCH_HEAD
  Auto-merging forth
  CONFLICT (content): Merge conflict in forth
  Auto-merging features/simple-operations.feature
  Automatic merge failed; fix conflicts and then commit the result.

  &#35; ou, si on a déjà fait un git fetch
  $ <add>git merge sogivoisinY/tp4</add>
  </pre>

  Résoudre le conflit sans oublier le commit de merge après avoir testé avec
  cucumber.
  </div>

* Pousser l'intégration sur le dépôt `origin` :

  <div class="script">
  <pre>
  $ <add>git push origin</add>
  </pre>
  </div>
