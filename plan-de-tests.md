# Projet VV - Sudoku Solver

## 1. Présentation du projet

### 1.1 Objet et intérêt

Sudoku Solver est une application en ligne de commande qui permet de saisir une grille de Sudoku, puis de demander à un solveur de la compléter. L’utilisateur entre les valeurs sous forme de coordonnées et peut corriger une case en y remettant zéro. Le programme vérifie la cohérence de la grille, tente d’abord des déductions à partir des valeurs possibles, puis utilise une recherche récursive lorsque ces déductions ne suffisent pas.

Le projet expose des difficultés  de vérification : respecter les règles de lignes, colonnes et blocs, traiter les entrées invalides, explorer des choix sans corrompre l’état initial et distinguer une grille invalide d’une grille impossible à résoudre. L’application est assez petite et est donc adaptée à l’étude de tests unitaires comme à l’ajout de tests de mutation.

### 1.2 Structure et technologies

Le code est écrit en Java. Il est organisé autour de quatre responsabilités :

- `com.sudoku`: point d’entrée de l’application (`SudokuApplication`) ;
- `com.sudoku.UI` et `com.sudoku.UI.controller`: affichage console, saisie et dialogue avec l’utilisateur ;
- `com.sudoku.engine`: représentation du plateau, éléments et lignes, déroulement d’une partie et solveur ;
- `com.sudoku.model`: objets de données et énumérations liés aux coordonnées

Les tests sont dans `src/test/java`. Gradle gère la compilation, les tests et la création du JAR exécutable. Le projet utilise JUnit 4.13 pour ses tests.

## 2. Méthodes de vérification existantes

### 2.1 Tests unitaires automatisés

La principale méthode de vérification est la classe `SudokuBoardTestSuite`, qui contient 16 tests JUnit. Elle couvre plusieurs opérations du plateau :

- lecture et écriture d’une case ;
- détection de doublons dans une ligne, une colonne ou un bloc ;
- acceptation d’un plateau sans doublons ;
- vérification de la possibilité de placer une valeur ;
- détection d’une case vide et calcul initial des valeurs possibles ;
- différence entre copie superficielle et copie profonde.
- rejet d’une grille invalide ;
- résolution d’une grille par déductions ;
- résolution récursive de deux grilles de difficulté différente.

Les grilles de test sont regroupées dans `DataFixture`, ce qui permet de les réutiliser sans répéter leur initialisation.

### 2.2 Compilation et exécution des tests avec Gradle

La tâche Gradle `test` compile les sources de test, les tests sont alors executables via l'onglet "testing" de VS Code.

### 2.3 Vérifications annoncées et limites des dispositifs en place

Les scripts `run.bat` et `run.sh` facilitent le lancement, mais ne garantissent pas la validité de l'application. En effet, ils enchaînent la construction et le lancement sans s’assurer explicitement que la construction a réussi. Un ancien JAR pourrait ainsi être lancé après un échec du build, ce qui donne une impression trompeuse de réussite.

| Aspect | Vérification présente | Ce qu’elle couvre | Limites constatées |
|---|---|---|---|
| Manipulation du plateau | Tests unitaires JUnit | Lecture et écriture des cases, détection des cases vides et copies superficielles ou profondes | Mal organisé, auncun tests vérifient les coordonnées hors grille ou les valeurs hors de l’intervalle 1 à 9 |
| Règles du Sudoku | Tests unitaires JUnit | Détection des doublons dans une ligne, une colonne ou un bloc ; vérification des placements possibles | Mal organisé, quelques cas sont testés, mais pas toutes les combinaisons de conflits |
| Solveur | Tests unitaires JUnit sur quelques grilles | Grille invalide, résolution par déductions et résolution récursive | Mal organisé, seulement quelques cas sont testés. |
| Interface console | Aucune suite de tests automatisés dédiée | Lancement manuel de l’application | Les entrées mal formées, les messages et les transitions du jeu ne sont pas testés automatiquement |
| Qualité du code et couverture | Compilation et tests avec Gradle | Vérification que le code compile et que les tests passent | Aucun outil de couverture, d’analyse statique ou de test par mutation n’est configuré ; aucune intégration continue n’est visible |


## 3. Propositions pour étendre la vérification

### 3.1 Priorité 1 : vérifier la validité réelle des solutions

Validation des grilles :

1. chaque case contient une valeur de 1 à 9 ;
2. chaque ligne contient exactement une fois les valeurs de 1 à 9 ;
3. chaque colonne contient exactement une fois les valeurs de 1 à 9 ;
4. chaque bloc 3 × 3 contient exactement une fois les valeurs de 1 à 9 ;
5. chaque valeur préremplie est identique à celle de la grille d’entrée.

Ajouter également des cas pour une grille déjà complète et correcte, et une grille complète mais incorrecte.

### 3.2 Priorité 2 : compléter les tests unitaires déjà présents

Compléter les tests du plateau et du solveur avec des cas ciblés :

- coordonnées minimales et maximales, puis coordonnées hors plateau ;
- valeurs 0, 1, 9 et valeurs hors domaine ;
- cellule vide sans valeur candidate possible ;
- branche récursive qui échoue avant de trouver une autre branche valide ;
- absence de modification de la grille d’origine pendant la recherche ;
- comportement de `deepCopy` pour les cases et leurs listes de candidats ;
- plusieurs exécutions successives du solveur sur des instances distinctes.

### 3.3 Priorité 3 : tester la saisie et le déroulement du jeu

La lecture directe depuis un scanner partagé et l’affichage statique rendent les tests de l’interface plus difficiles. Une évolution simple serait d’injecter les sources d’entrée et de sortie (par exemple un `Scanner` et un `PrintStream`) dans les contrôleurs. Les tests pourraient alors simuler les séquences saisies sans intervention humaine.

Cas de test suggérés :

- saisie d’une coordonnée valide et saisie d’une valeur nulle pour effacer une case ;
- ligne trop courte ou trop longue, caractères non numériques, puis saisie valide ;
- choix de résolution, sortie de partie et réponse de confirmation invalide ;
- scénario complet qui saisit une grille puis vérifie le résultat et les principaux messages ;
- tester la décision « nouvelle partie » et « quitter » avec des valeurs nulles/aléatoires etc.

### 3.4 Priorité 4 : Autres tests et automatisation de la qualité

- **Test par mutation** avec PIT : modifier artificiellement des conditions ou opérations pour vérifier que les tests échouent lorsqu’un défaut est introduit. Un mutant survivant peut révéler une assertion manquante.
- **Test d'intégration continue** : configurer GitHub Actions pour lancer `gradlew test` à chaque pull request et vérifier la création du JAR. Le JDK de CI doit être 17 ou supérieur pour exécuter le wrapper actuel.
- **Tests combinatoires** : tester toutes les combinaisons des paramètres


### 3.5 Priorité 5 : Mesure de la qualité et de la couverture des tests

- **Code coverage** : 100 x nb_lignes_exec/nb_lignes_totales
- **Couverture de code avec JaCoCo (En bonus)** : Utiliser JaCoCo  pour produire un rapport indiquant les lignes et les branches exécutées par les tests. Analyse le solveur et les décisions récursives, ainsi que la validation du plateau et la saisie.
- **Analyse statique (En bonus)** : utiliser un outil comme SpotBugs pour rechercher des constructions susceptibles de provoquer des erreurs.