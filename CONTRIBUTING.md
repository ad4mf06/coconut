# Contribuer au projet

Merci de l'intérêt pour ce fork de Coconut ! Ce guide explique comment configurer l'environnement de développement et soumettre des contributions.

## Prérequis

- Python 3.8 ou plus récent
- Git
- Environ 2 Go d'espace disque disponible

## Configurer l'environnement de développement

1. **Forker ou cloner ce dépôt**

```bash
git clone https://github.com/ad4mf06/coconut.git
cd coconut
```

2. **Créer un environnement virtuel et l'activer**

```bash
python -m venv .venv
```

- Windows : `.venv\Scripts\activate`
- macOS/Linux : `source .venv/bin/activate`

3. **Installer le projet en mode développement**

```bash
pip install -e .
```

Cela installe Coconut et toutes ses dépendances en mode éditable, ce qui signifie que vos modifications sont prises en compte sans réinstaller.

## Travailler sur une fonctionnalité

1. **Créer une branche à partir de `develop`**

```bash
git checkout develop
git pull
git checkout -b feature/ma-fonctionnalite
```

2. **Effectuer vos modifications**

   Les fichiers principaux à modifier sont :
   - `coconut/compiler/grammar.py` — règles PyParsing
   - `coconut/compiler/compiler.py` — handlers de compilation
   - `coconut/compiler/matching.py` — pattern matching

3. **Ajouter des tests**

   Les tests sont écrits en Coconut (`.coco`) dans `coconut/tests/src/cocotest/agnostic/`. Ajoutez des `assert` dans le fichier approprié :
   - `primary_1.coco` ou `primary_2.coco` pour les tests généraux
   - `specific.coco` pour les tests spécifiques à une version Python

4. **Lancer les tests**

```bash
make test
```

   Pour un cycle plus rapide (si vous ne modifiez que les tests) :

```bash
make test-tests
```

5. **Vérifier que votre code fonctionne dans l'interpréteur interactif**

```bash
coconut
```

## Soumettre une contribution

1. **Pousser votre branche**

```bash
git push origin feature/ma-fonctionnalite
```

2. **Ouvrir une Pull Request** vers la branche `develop` de ce fork

   Dans la description de la PR, incluez :
   - Ce que la fonctionnalité fait
   - Comment la tester
   - Les problèmes rencontrés et les solutions trouvées

## Style de code

- Respecter le style du code existant (indentation, nommage)
- Ajouter `from coconut.root import *` en haut de tout nouveau fichier Python
- Utiliser `logger` de `coconut.terminal` pour les messages de log
- Ajouter votre nom dans la section `Authors:` des docstrings des fichiers modifiés

## Ressources utiles

- [Documentation Coconut](https://coconut.readthedocs.io/)
- [Issues ouvertes du projet upstream](https://github.com/evhub/coconut/issues)
- [Guide de contribution upstream](https://coconut.readthedocs.io/en/develop/CONTRIBUTING.html)
