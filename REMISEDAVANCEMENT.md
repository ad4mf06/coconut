# Remise d'avancement

## Instructions d'installation

### Prérequis


1. **Python 3.8 ou plus récent** — Copier la ligne suivante dans un terminal :

```bash
winget install Python.Python.3
```

2. **Git** — Copier la ligne suivante dans un terminal :

```bash
winget install --id Git.Git -e --source winget
```

3. **Espace disque** — Il faut au minimum **2 Go** d'espace libre sur le disque de stockage (inclut Python, Git, le dépôt et l'environnement virtuel avec ses dépendances).

### Étapes

1. **Créer un dossier à l'endroit de votre choix**

- Prener un endroit logique par exemple le bureau
- Donner un nom significatif au dossier
- Ouvrir le terminal à partir de ce dossier

2. **Cloner le fork du projet coconut**

```bash
git clone https://github.com/ad4mf06/coconut.git
cd coconut
```

3. **Créer un environnement virtuel**

```bash
python -m venv .venv
```

Activer l'environnement :

- Windows : `.venv\Scripts\activate`
- macOS/Linux : `source .venv/bin/activate`

4. **Installer les dépendances**

```bash
pip install -e .
```

---

## Instructions d'utilisation

### Compiler un fichier `.coco`

```bashg
coconut mon_fichier.coco
```

### Lancer l'interpréteur interactif

```bash
coconut
```

### Exécuter les tests

```bash
coconut --test
```

---

## Issues travaillées

### Issue #863 – `where` / `do` block syntax - Terminer

Ajout d'une nouvelle syntaxe `block_where_stmt` permettant d'assigner le résultat d'un bloc `where:` ou `do:` à une variable. La dernière expression du bloc devient la valeur assignée.

**Exemple :**

```coconut
out = where:
    x = 1 + 2 + 3
    y = 4 + 5 + 6
    x + y
assert out == 21
```

**Changements effectués :**
- Ajout de la règle de grammaire `block_where_stmt` dans `grammar.py`
- Support des mots-clés `where:` et `do:`
- Gestion du scope des variables avec le gestionnaire de noms
- Tests ajoutés dans `coconut/tests/src/cocotest/agnostic/primary_1.coco` couvrant : addition, multiplication, division, calculs chaînés, concaténation de chaînes et blocs complexes


**Problèmes principaux rencontrés**

- Il s'agissait de la première issue donc il m'a fallu du temps pour bien comprendre le code.
- Le langage de programmation ne m'était pas familier.  
Solution : Rechercher sur Google et avec l'IA le fonctionnement du langage Python et du projet actuel.
- Le handler existant du fichier `compiler.py` pour la fonction where ne prenait que les lignes 
  avec une indentation après le **where :** 
Solution : toujours indenter les lignes après avoir fait un where : ou un do : 
- Le `block_where_stmt` qui m'a prit du temps à comprendre la logique de ceci.  
Solution : Ajouter le `final_where_statement` qui est une ligne sans `=` pour que le compilateur 
  comprenne que cette ligne est l'équation du résultat que l'on s'attend.  
Par exemple : Dans le bloque d'exemple en haut, le `final_where_statement` est x + y puisque 
  cette ligne n'a pas de `=`


Lien vers la PR -> https://github.com/evhub/coconut/pull/900


### Réponse du project owner

This looks pretty good! We should pick only one of these two syntaxes, though, and you should make sure to test that the variables in the where clause are properly hidden from the enclosing scope. I think I probably prefer the where: syntax over the do: syntax since do has a specific meaning in many functional programming languages that is not this.


### Ajustement 

1. J'ai enlevé le do: de la syntaxe possible pour le where statement.

2. J'ai ajouté un test qui vérifie que les variables de la fonction soient bien caché.

### Tester l'issue

Entrez la commande suivante : 

```bash
git checkout feature/where-final-result-863     
```
Ensuite en console, entrez :

```bash
coconut   
```

Une fois la console intéractive lancer, vous pouvez des where : par exemple :

```bash
out = where:
      x = 1 + 2 + 3
      y = 4 + 5 + 6
      x + y 
      
print(out)
```

Ceci devrait donner le nombre 21.



---

### Issue #887 – Pattern-matching comprehensions - Terminer

Ajout du support des clauses `if` et `for` optionnelles à la fin des compréhensions avec pattern matching.

**Exemple :**

```coconut
data CompPair(a, b)
pairs = [CompPair(1, 2), CompPair(3, 4)]

# Compréhension de base
assert [a + b for CompPair(a, b) in pairs] == [3, 7]

# Avec clause for supplémentaire
assert [a + b for CompPair(a, b) in pairs for _ in range(2)] == [3, 3, 7, 7]

# Avec filtre if sur les variables du pattern
assert [a + b for CompPair(a, b) in pairs if a > 1] == [7]
```

**Changements effectués :**
- Ajout de `.Optional(comp_iter)` dans `match_comp_for` dans `grammar.py` (ligne 1969)
- Modification de `match_comp_expr_handle` dans `compiler.py` (ligne 4597) pour gérer les clauses extra
- Tests complets ajoutés dans `primary_2.coco` : dict/set/generator comprehensions, conditions multiples, types `CompPair` et `CompTriple`

**Problèmes principaux rencontrés**

- Le comp actuel n'avait pas de gestion d'erreur  
Solution : Ajouter un filtre qui ignore les éléments non-correspondant
- Dans le `match_comp_expr_handle`il n'avait pas de vérification de la longueur du token pour 
  déterminer si l'expression contenait plusieurs for ou if  
Solution : Ajouter un if `len(match_for_group) == 2` pour continuer dans le fonctionnement 
  actuel du comp actuel et un if `len(match_for_group) > 2` pour embarquer dans la fonction 
  implémenter avec un comp contenant plusieurs for ou if


Lien vers la PR -> https://github.com/evhub/coconut/pull/901

### Message du project owner
I think it's a bit confusing for only the first comprehension to have the ability to be a pattern-matching comprehension. My preference here is to either stick with the current implementation that only allows one comprehension, or to support arbitrary mixing and matching of standard and pattern-matching comprehensions (the ideal, but annoying to implement).



### Ajustement

- J'ai implémenté la deuxième option de sa demande c'est-à-dire que j'ai implémenté le support 
  arbitraire des matching standard et des compréhensions avec pattern matching. 


### Tester l'issue

Commencer par aller sur la branche contenant l'implémentation

```bash
git checkout feature/pattern-matching-comprehensions-887 
```

Ensuite lancer la console intéractive

```bash
coconut
```

Puis exécuter ce test

```bash
data CompPair(a, b)
pairs = [CompPair(1, 2), CompPair(3, 4), CompPair(5, 6)]
print(sum(a + b for CompPair(a, b) in pairs))
```

Ceci devrait donner le nombre 21


---


### Issue #888 – Pytest with automatic compilation - Terminer 

Cette issue propose d'ajouter le support de l'exécution automatique des fichiers `.coco` via pytest sans compilation préalable. L'objectif est de configurer un hook pytest pour collecter et compiler les fichiers `.coco` à la volée, évitant ainsi les erreurs de module mismatch.

**Changements effectués:**
- Analyse de l'erreur `import file mismatch` causée par la différence entre `.coco` et `.py`
- Exploration des hooks pytest (`pytest_collect_file`) pour gérer la compilation automatique
- Ajouté pytest11 comme plugin pour qu'il soit directement installé
- Création du fichier pytest_plugin.py qui contient ces 3 fonctions :
  1.  pytest_configure — appelé au démarrage de la session pytest. Installe _PytestCoconutImporter en tête de sys.meta_path pour intercepter les imports de modules .coco.
  2.  pytest_collect_file — appelé pour chaque fichier trouvé. Si le fichier est test_*.coco, il le compile en .py à côté du source (pas dans un cache), puis dit à pytest de collecter ce .py. C'est ce qui règle le import file
  3.  pytest_ignore_collect — évite la double-collecte en ignorant le dossier __coconut_cache__/ et les .py qui ont un .coco source à côté
- Override 2 function hérité de CoconutImporter
  1. La function compile() pour remplacer sys.stdin pendant la compilation (pytest capture stdin, ce qui causerait une OSError)
  2. La function find_spec() pour retourner un spec pointant vers le .py en place, empêchant l'importer global de rediriger ailleurs
- Ajouté une fonction dans api.py qui permet d'installer manuellement pytest11

**Problèmes principaux rencontrés**

- Problème de double collect. Les fichiers test ayant déja une version complilés lançaient une 
  erreur.


Lien vers la PR -> https://github.com/evhub/coconut/pull/902


### Tester l'issue

Commencer par aller sur la branche contenant l'implémentation

```bash
git checkout feature/pytest-auto-compilation-888 
```

Créer un dossier nommé ``test_demo``

Créer un fichier à l'intérieur du nouveau dossier ``test_pipeline.coco``

Copier ceci dans le fichier créé 

```bash
def test_pipeline():
    result = 1 |> (+ 1) |> (+ 1)
    assert result == 3

def test_pattern_match():
    match [1, 2, 3]:
        case [head, *tail]:
            assert head == 1
            assert tail == [2, 3]

```

Ensuite pour voir si l'importation de test fonctionne, 

Installer les dépendances

```bash
pip install -e 
```

Lancer le test

```bash
pytest test_demo/ -v     
```

### Issue #899 - Match rest keyword arguments of a data class - Commencé le 03/03/2026

Lien vers l'issue -> https://github.com/evhub/coconut/issues/899


Planification de l'issue : 
1. Trouver le problème :   
  Première hypothèse, le match peut reconnaitre la synthaxe  (*_) ou toute forme d'utilisation 
    d'un * comme argument, mais le compileur ne reconnait pas l'argument (**) etlance une erreur 
    lorsque l'utilisateur entre  **
2. Implémenter la correction  
  Implémenter les fichiers coconut/compiler.py et coconut/grammar.py
3. Tester que tout fonctionne  
   Ajouter des tests dans primary_2.coco pour vérifier que l'implémentation fonctionne
4. Finaliser l'issue préférablement à la semaine 9 pour laiser 3 semaines pour la dernière issue 
   et une semaine pour la remise finale


### Autre issue disponible intéressante que j'aimerais faire avant la remise finale
Xontrib: Allow using coconut in rc files and @(...) syntax -> https://github.com/evhub/coconut/issues/883

