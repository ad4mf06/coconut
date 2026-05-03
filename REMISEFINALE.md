# Remise Finale

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

3. **Coconut**

```bash
pip install coconut
```

4. **Espace disque** — Il faut au minimum **2 Go** d'espace libre sur le disque de stockage
(inclut Python, Git, le dépôt et l'environnement virtuel avec ses dépendances).

### Étapes

1. **Créer un dossier à l'endroit de votre choix**

   - Prendre un endroit logique, par exemple le bureau
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

```bash
coconut mon_fichier.coco
```

### Lancer l'interpréteur interactif

```bash
coconut
```

### Exécuter les tests du projet

```bash
make test
```

---

## Issues réalisées

### Issue #863 – `where` block syntax — Terminée

**Lien vers la PR :** https://github.com/evhub/coconut/pull/900

Ajout d'une nouvelle syntaxe `block_where_stmt` permettant d'assigner le résultat d'un bloc `where:` à une variable. La dernière expression du bloc sans `=` devient la valeur assignée.

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
- Support du mot-clé `where:`
- Gestion du scope des variables avec le gestionnaire de noms
- Tests ajoutés dans `coconut/tests/src/cocotest/agnostic/primary_1.coco`

**Problèmes rencontrés et solutions**

- *Compréhension du code existant* — Le langage et la structure du projet n'étaient pas familiers.
  **Solution :** Recherches sur le fonctionnement de PyParsing et de l'architecture du compilateur.

- *Gestion de l'indentation* — Le handler existant pour `where` dans `compiler.py` ne prenait que les lignes indentées après `where:`.
  **Solution :** Documenter que toutes les lignes après `where:` doivent être indentées.

- *Identification de la valeur de retour* — Il fallait distinguer l'expression finale (sans `=`) des assignations intermédiaires.
  **Solution :** Ajout du `final_where_statement` — la dernière ligne sans `=` est traitée comme la valeur de retour du bloc. Par exemple, dans le bloc ci-dessus, `x + y` est le `final_where_statement`.

**Réponse du mainteneur du projet :**
> This looks pretty good! We should pick only one of these two syntaxes, though, and you should make sure to test that the variables in the where clause are properly hidden from the enclosing scope. I think I probably prefer the where: syntax over the do: syntax since do has a specific meaning in many functional programming languages that is not this.

**Ajustements suite à la revue :**
1. Suppression de la syntaxe `do:` — seul `where:` est conservé.
2. Ajout d'un test vérifiant que les variables du bloc `where:` ne fuient pas dans le scope englobant.

**Tester l'issue :**

```bash
git checkout feature/where-final-result-863
coconut
```

Dans la console interactive :

```coconut
out = where:
      x = 1 + 2 + 3
      y = 4 + 5 + 6
      x + y

print(out)
```

Résultat attendu : `21`

---

### Issue #887 – Pattern-matching comprehensions — Terminée

**Lien vers la PR :** https://github.com/evhub/coconut/pull/901

Ajout du support des clauses `if` et `for` optionnelles après un `for` avec pattern matching dans les compréhensions. L'implémentation supporte le mélange arbitraire de clauses standard et de clauses avec pattern matching.

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
- Modification de `match_comp_expr_handle` dans `compiler.py` (ligne 4597) pour gérer les clauses supplémentaires
- Tests complets dans `primary_2.coco` : list/dict/set/generator comprehensions, conditions multiples, types `CompPair` et `CompTriple`

**Problèmes rencontrés et solutions**

- *Absence de gestion d'erreur dans le comp existant* — Les éléments non correspondants au pattern causaient une erreur.
  **Solution :** Ajout d'un filtre silencieux ignorant les éléments non correspondants.

- *Détermination du type de clause* — Dans `match_comp_expr_handle`, il n'y avait pas de vérification de la longueur du token pour savoir si l'expression contenait plusieurs `for`/`if`.
  **Solution :** Condition `len(match_for_group) == 2` pour le cas simple (comportement existant) et `len(match_for_group) > 2` pour le cas avec clauses supplémentaires.

**Réponse du mainteneur du projet :**
> I think it's a bit confusing for only the first comprehension to have the ability to be a pattern-matching comprehension. My preference here is to either stick with the current implementation that only allows one comprehension, or to support arbitrary mixing and matching of standard and pattern-matching comprehensions (the ideal, but annoying to implement).

**Ajustement suite à la revue :**
Implémentation du support arbitraire des clauses standard et pattern-matching dans une même compréhension (deuxième option proposée par le mainteneur).

**Tester l'issue :**

```bash
git checkout feature/pattern-matching-comprehensions-887
coconut
```

Dans la console interactive :

```coconut
data CompPair(a, b)
pairs = [CompPair(1, 2), CompPair(3, 4), CompPair(5, 6)]
print(sum(a + b for CompPair(a, b) in pairs))
```

Résultat attendu : `21`

---

### Issue #888 – Pytest with automatic compilation — Terminée

**Lien vers la PR :** https://github.com/evhub/coconut/pull/902

Ajout du support de l'exécution automatique des fichiers `.coco` via pytest sans compilation préalable. Un plugin pytest est configuré pour collecter et compiler les fichiers `.coco` à la volée, évitant ainsi les erreurs de module mismatch.

**Changements effectués :**
- Création du fichier `pytest_plugin.py` contenant trois fonctions :
  1. `pytest_configure` — installé au démarrage de la session pytest. Installe `_PytestCoconutImporter` en tête de `sys.meta_path` pour intercepter les imports de modules `.coco`.
  2. `pytest_collect_file` — appelé pour chaque fichier trouvé. Si le fichier est `test_*.coco`, il le compile en `.py` à côté du source (pas dans un cache), puis demande à pytest de le collecter. Cela résout l'erreur `import file mismatch`.
  3. `pytest_ignore_collect` — évite la double collecte en ignorant `__coconut_cache__/` et les `.py` qui ont un `.coco` source à côté.
- Override de deux fonctions héritées de `CoconutImporter` :
  1. `compile()` — remplace `sys.stdin` pendant la compilation (pytest capture stdin, ce qui causerait une `OSError`).
  2. `find_spec()` — retourne un spec pointant vers le `.py` en place, empêchant l'importer global de rediriger ailleurs.
- Ajout d'une entrée `pytest11` dans `setup.py` pour l'installation automatique du plugin.
- Ajout d'une fonction dans `api.py` pour l'installation manuelle du plugin.

**Problèmes rencontrés et solutions**

- *Double collecte* — Les fichiers de test ayant déjà une version compilée (`.py`) causaient une erreur de double collecte.
  **Solution :** Implémentation de `pytest_ignore_collect` pour filtrer les `.py` qui ont un `.coco` source correspondant.

**Tester l'issue :**

```bash
git checkout feature/pytest-auto-compilation-888
```

Créer un dossier `test_demo` et y créer un fichier `test_pipeline.coco` avec le contenu suivant :

```coconut
def test_pipeline():
    result = 1 |> (+ 1) |> (+ 1)
    assert result == 3

def test_pattern_match():
    match [1, 2, 3]:
        case [head, *tail]:
            assert head == 1
            assert tail == [2, 3]
```

Installer les dépendances et lancer les tests :

```bash
pip install -e .
pytest test_demo/ -v
```

---

### Issue #899 – Match rest keyword arguments of a data class — Terminée

**Lien vers la branche :** `issue-899-match-rest-kwargs`

Ajout du support de la syntaxe `**rest` dans les patterns de matching sur les data classes, permettant de capturer les arguments nommés restants.

**Exemple :**

```coconut
data Point(x, y, z=0)

match Point(x=1, **rest) = Point(1, 2, z=3):
    assert x == 1
    assert rest == {"y": 2, "z": 3}
```

**Problème identifié :**
Le compilateur reconnaissait la syntaxe `*_` (args positionnels restants) mais lançait une erreur lorsque l'utilisateur utilisait `**` pour capturer les arguments nommés restants.

**Changements effectués :**
- Modification de `coconut/compiler/grammar.py` pour reconnaître la syntaxe `**rest` dans les patterns de data class
- Modification de `coconut/compiler/compiler.py` pour générer le code Python correspondant
- Tests ajoutés dans `primary_2.coco` vérifiant que la capture des kwargs restants fonctionne correctement

**Problèmes rencontrés et solutions**

- *Localisation du problème dans la grammaire* — La syntaxe `**` n'était pas définie dans les règles de pattern matching des data classes.
  **Solution :** Analyse de la grammaire existante pour `*_` puis ajout d'une règle analogue pour `**`.

- *Génération du code Python* — La transformation en Python nécessitait de récupérer les kwargs non capturés par le pattern.
  **Solution :** Génération d'un dictionnaire des arguments nommés résiduels dans le handler de `compiler.py`.

---

## Tests unitaires

Les tests sont répartis sur les différentes branches de fonctionnalités et dans `test_demo/`.

### Tests de l'issue #863 (`feature/where-final-result-863`)

Fichier : `coconut/tests/src/cocotest/agnostic/primary_1.coco`

| Test | Description |
|------|-------------|
| `assert where == 10 where: ten = 10` | Assignation simple dans un bloc where |
| `assert true where: true = True` | Assignation booléenne |
| `assert a == 5 where: {"a": a} = {"a": 5}` | Destructuration de dictionnaire |
| `assert a == 3 where: (1, 2, a) = (1, 2, 3)` | Destructuration de tuple |
| `assert a == 2 == b where: a = 2; b = 2` | Assignations multiples |
| `assert a == 3 where: a = 2; a = a + 1` | Assignations chaînées |
| `assert a == 5 where: def six() = 6; a = six()` | Définition de fonction dans where |

### Tests de l'issue #887 (`feature/pattern-matching-comprehensions-887`)

Fichier : `coconut/tests/src/cocotest/agnostic/primary_2.coco`

| Test | Description |
|------|-------------|
| `[a + b for CompPair(a, b) in pairs]` | Compréhension de liste avec pattern |
| `[a + b for CompPair(a, b) in pairs for _ in range(2)]` | Clause `for` supplémentaire |
| `[a + b for CompPair(a, b) in pairs if a > 1]` | Clause `if` de filtrage |
| Compréhension de dict avec pattern | Vérification du type de résultat |
| Compréhension de set avec pattern | Vérification de la déduplication |
| Expression génératrice avec pattern | Évaluation paresseuse |
| Mélange de clauses standard et pattern | Mélange arbitraire de `for` standard et pattern |

### Tests de l'issue #888 (`feature/pytest-auto-compilation-888`)

Fichier : `test_demo/test_pipeline.coco`

| Test | Description |
|------|-------------|
| `test_pipeline()` | Opérateur pipeline `\|>` avec fonctions partielles |
| `test_pattern_match()` | Pattern matching sur liste avec capture `*tail` |

### Tests de l'issue #899 (`issue-899-match-rest-kwargs`)

Fichier : `coconut/tests/src/cocotest/agnostic/primary_2.coco`

| Test | Description |
|------|-------------|
| Match avec `**rest` sur data class | Capture des kwargs restants dans un dictionnaire |

---

## Structure du projet

```
coconut/
├── compiler/
│   ├── grammar.py      # Règles PyParsing (modifié pour #863, #887, #899)
│   └── compiler.py     # Handlers de compilation (modifié pour #863, #887, #888, #899)
├── tests/src/cocotest/agnostic/
│   ├── primary_1.coco  # Tests pour #863
│   └── primary_2.coco  # Tests pour #887, #899
├── api.py              # API publique (modifié pour #888)
└── pytest_plugin.py    # Plugin pytest (créé pour #888)
```
