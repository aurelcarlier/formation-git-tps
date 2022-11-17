<img src="Git-Logo-1788C-small.png" />

TP5 : Commits partiels, amender un commit
=========================================

Objectifs du TP :

- utiliser la staging area pour faire des commits partiels
- amender le dernier commit

<pre>
$ <add>git checkout -b tp5 tp5-start</add>
Switched to a new branch 'tp5'
</pre>

1. Implémenter les flottants et la division entière
---------------------------------------------------

Notre interpréteur Forth n'accepte que la saisie de nombres entiers mais supporte la division flottante (post TP3). Pour ce TP, nous allons redéfinir un peu les spécifications :

- permettre la saisie de nombres flottant par l'utilisateur
- garder la division flottante (post TP3)
- implémenter le quotient comme une opération à part entière (et non redéfinir la division comme au TP4)

Les étapes sont les suivantes.

* Implémenter la saisie des flottants en changeant la regexp de filtrage des nombres dans la fonction `dispatch` :

  <pre>
    elseif word:find(<add>"^%d+%.?%d*$"</add>) then  -- Si c'est un nombre, empiler `wo
  </pre>

  <div class="script">
    Modifier dans <code>forth</code>
    <pre>
    function dispatch(disp, word)
      ...
      elseif word:find(<add>"^%d+%.?%d*$"</add>) then  -- Si c'est un nombre, empiler `wo
        push(tonumber(word))
      ...
    end -- dispatch
    </pre>

    Ajouter dans <code>features/simple-operations.feature</code>
    <pre>
    &nbsp; Background:
    &nbsp;   Given a forth interpreter
    &nbsp; <add>
    &nbsp; Scenario: Float Input
    &nbsp;    When I execute "5.5 4 . ."
    &nbsp;    Then I should get "4 5.5 ok"
    &nbsp; </add>
    &nbsp; Scenario: Addition
    &nbsp;    When I execute "3 4 + ."
    &nbsp;    Then I should get "7 ok"
    </pre>
  </div>

* Implémenter la division entière avec l'opération `//`

  <div class="script">
  Ajouter dans <code>forth</code>

  <pre>
  symbol_table['/'] = function(...)
    local a, b = pop(2)
    push(a / b)
  end -- '/'
  <add>
  symbol_table['//'] = function(...)
    local a, b = pop(2)
    push(math.floor(a / b))
  end -- '//'
  </add>
  symbol_table.NEG = function(...)
    local a = pop()
    push(a * -1)
  end -- DUP
  </pre>

  Ajouter dans <code>features/simple-operations.feature</code>
  <pre>
  &nbsp; Scenario: Division
  &nbsp;    When I execute "5 2 / ."
  &nbsp;    Then I should get "2.5 ok"
  &nbsp; <add>
  &nbsp; Scenario: Quotient
  &nbsp;    When I execute "5 2 // ."
  &nbsp;    Then I should get "2 ok"
  &nbsp; </add>
  &nbsp; Scenario: Negation
  &nbsp;    When I execute "1 NEG ."
  &nbsp;    Then I should get "-1 ok"
  </pre>
  </div>

2. Faire un commit partiel avec la saisie des flottants
-------------------------------------------------------

* Ajouter dans la staging area uniquement les modifications pour la saisie des flottants

  <div class="script">
    On peut utiliser la console en mode interactif avec <code>git add -i</code> ou directement <code>git add -p</code>, ou bien l'interface graphique <code>git gui</code>, plus intuitive.

    1. Lancer l'interface graphique avec <code>git gui</code>
    1. Sélectionner le fichier <code>forth</code>
    1. Sélectionner les lignes modifiées dans l'interpréteur pour la saisie des flottants
    1. Clic droit sur la sélection puis <em>"Indexer les lignes"</em>
    1. Faire de même pour le test dans <code>features/simple-operations.feature</code>
  </div>

* Vérifier l'état de la staging area

  <pre>
  $ <add>git status</add>
  &#35; On branch tp5
  &#35; Changes to be committed:
  &#35;   (use "git reset HEAD <file>..." to unstage)
  &#35;
  &#35;	modified:   features/simple-operations.feature
  &#35;	modified:   forth
  &#35;
  &#35; Changes not staged for commit:
  &#35;   (use "git add <file>..." to update what will be committed)
  &#35;   (use "git checkout -- <file>..." to discard changes in working directory)
  &#35;
  &#35;	modified:   features/simple-operations.feature
  &#35;	modified:   forth
  </pre>

  Les fichiers apparaissent à la fois dans la staging area et dans la copie locale car les modifications concernant la division entière sont toujours dans la copie locale mais n'ont pas été ajouté dans staging. Utiliser <code>git diff</code> et <code>git diff --staged</code> pour voir les différences.

* Créer le commit

  <div class="script">
  Utiliser le bouton <em>Commiter</em> de git gui ou bien <code>git commit -m 'Interpreter accepts float as input'</code>.
  </div>

3. Faire un commit avec la division entière
-------------------------------------------

* Créer le commit avec les modifications de la division entière

  <pre class="script">
  $ <add>git commit -am 'Implement quotient operator'</add>
  [tp5 fb0f67f] Implement quotient operator
  2 files changed, 9 insertions(+)
  </pre>

* Visualiser l'historique

        * fb0f67f (HEAD, tp5) Implement quotient operator
        * 2b823d6 Interpreter accepts float as input

4. Implémenter l'opération modulo
---------------------------------

On décide d'ajouter l'opération complémentaire modulo (sans créer de commit pour l'instant).

* Implémenter l'opération modulo

  <div class="script">
    Ajouter dans <code>forth</code>
    <pre>
    symbol_table['//'] = function(...)
      local a, b = pop(2)
      push(math.floor(a / b))
    end -- '//'
    <add>
    symbol_table['%'] = function(...)
      local a, b = pop(2)
      push(a % b)
    end -- '%'
    </add>
    symbol_table.NEG = function(...)
      local a = pop()
      push(a * -1)
    end -- DUP
    </pre>

    Ajouter dans <code>features/simple-operations.feature</code>
    <pre>
    &nbsp; Scenario: Quotient
    &nbsp;    When I execute "5 2 // ."
    &nbsp;    Then I should get "2 ok"
    &nbsp; <add>
    &nbsp; Scenario: Modulo
    &nbsp;    When I execute "5 2 % ."
    &nbsp;    Then I should get "1 ok"
    &nbsp; </add>
    &nbsp; Scenario: Negation
    &nbsp;    When I execute "1 NEG ."
    &nbsp;    Then I should get "-1 ok"
    </pre>
  </div>

5. Amender le dernier commit avec modulo
----------------------------------------

On va amender (modifier) le dernier commit pour avoir un seul commit avec quotient et modulo, plutôt que d'ajouter un nouveau commit.

* Amender le dernier commit pour ajouter l'opération modulo (ne pas oublier de changer le message du commit)

  <div class="script">
  On peut utiliser la ligne de commande :
  <pre>
  $ <add>git add -u</add>
  $ <add>git commit --amend -m 'Implement quotient and modulo operators'</add>
  [tp5 8c98774] Implement quotient and modulo operators
   2 files changed, 18 insertions(+)
  </pre>

  Ou l'interface graphique :
  1. Lancer <code>git gui</code>
  1. Cliquer sur <em>Corriger dernier commit</em>
  1. Mettre à jour la staging area avec les dernières modifications
  1. Changer le message et cliquer sur <em>Commiter</em>
  </div>

* Visualiser l'historique. Le commit précédent a été "remplacé" par le commit avec les deux opérations

        * 8c98774 (HEAD, tp5) Implement quotient and modulo operators
        * 2b823d6 Interpreter accepts float as input
