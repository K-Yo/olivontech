---
date: '2024-11-11T18:39:08+01:00'
draft: false
title: 'Développement local pour XSOAR'
slug: 'local-development-on-xsoar'
categories:
  - Tuto
tags:
  - XSOAR
description: 'Apprenez comment développer du contenu pour XSOAR depuis votre IDE.'
---

XSOAR possède un IDE intégré permettant de développer des Scripts − composant essentiel à la personnalisation de l’orchestrateur.
Cependant celui-ci comporte un certain nombre de limites.
Vous apprendrez dans cet article comment utiliser votre IDE favori pour développer les Scripts.

<!--more-->

## Customiser XSOAR

Il y a principalement deux niveaux de développement dans XSOAR: les Playbooks et les Scripts (les Intégrations ne sont qu’un type de Script).

Les playbooks sont créés à travers l’éditeur visuel intégré à la solution. Il y a beaucoup à dire sur cet éditeur et ses spécificités, mais pour cet article, nous allons nous concentrer sur l’édition des scripts.

Les Scripts sont un aspect inévitable de XSOAR, dès que vous souhaitez faire des opérations sur-mesure à votre SOAR.
Ils sont principalement écrits en Python (ou Javascript, mais le monde de la sécurité informatique n’est pas très fana du JS); et lorsque vous souhaitez en modifier un, XSOAR intègre un IDE pour pouvoir le modifier.

## Fonctionnement des scripts

Les Scripts permettent d’exécuter du code que vous maîtrisez dans XSOAR.
Il y en a un certain nombre livrés avec la solution (`Set`, etc.) ou qu’on peut installer à travers un pack (par exemple [Common Scripts](https://cortex.marketplace.pan.dev/marketplace/details/CommonScripts/)).
Vous pouvez également en créer vous-même quand les scripts sur étagère ne conviennent pas. Comparer des objets complexes? manipuler des données exotiques? transformer de la donnée sur mesure? Exécuter un algorithme original? Exploiter des fonctions de librairies existantes? Faites un script!

Par exemple, XSOAR propose un système de _templating_ rudimentaire. Pour bénéficier de la puissance de [Jinja](https://jinja.palletsprojects.com/), on peut facilement créer un Script qui va prendre les données et le template passé en argument et sortir le rendu pour être utilisé plus loin dans le playbook.

Dans XSOAR, les scripts sont composés de code et de métadonnées. Le code contiendra l’algorithme que vous souhaitez exécuter. Les métadonnées rendent plusieurs services:

- Documenter l’usage
- Définir l’interface avec le reste de XSOAR, par exemple les arguments et données renvoyées
- Préciser le contexte d’exécution (quel conteneur docker, est-ce qu’il doit s’exécuter sur un _Engine_?)

Le code du script a accès à certains objets et fonctions spécifique à XSOAR. Il est par exemple possible d’accéder aux données passées en argument du script à travers l’[objet `demisto`](https://xsoar.pan.dev/docs/reference/api/demisto-class):

```python
args = demisto.args()
```

On peut aussi accéder à l’incident sur lequel s’exécute le Script avec le même objet:

```python
incident = demisto.incident()
```

On peut aussi utiliser des classes utilitaires comme [`CommandResults`](https://xsoar.pan.dev/docs/reference/api/common-server-python#commandresults) pour indiquer comment renvoyer des données à la fin de l’exécution du script, [`return_results`](https://xsoar.pan.dev/docs/reference/api/common-server-python#return_results) qui renvoie à XSOAR les résultats d’exécution, ou encore [`tableToMarkdown`](https://xsoar.pan.dev/docs/reference/api/common-server-python#tabletomarkdown) qui transforme une liste en tableau markdown :

```python
mydata = [...]
results = CommandResults(
    outputs_prefix="myscript",
    outputs_key_field="id",
    outputs=mydata,
    readable_output=tableToMarkdown("These are the results", mydata, ...),
)
return_results(results)
```

Dans cet exemple, CommandResults est un objet, et return_results va le lire pour afficher au bon endroit les données: dans le _context data_, dans une _note_ de la _war room_, etc.

Une fois le script créé, il est accessible dans la bibliothèque de l’instance XSOAR, et peut être utilisé dans les playbooks comme tâche autonome.

## Les limites de l’IDE intégré

L’IDE intégré ([Ace](https://ace.c9.io/)) a quelques fonctionnalités basiques de coloration syntaxique ou recherche.

Cependant, certains aspects cruciaux du développement logiciel ne sont pas possibles.

Si vous êtes adepte du « Tester c’est douter », bonne chance!
Pour un SOAR, l’exécution sert à porter des processus de sécurité des systèmes d’information de l’entreprise.
Quand vous voulez supprimer un e-mail, déconnecter du réseau une machine ou remonter une alerte du SOC, la fiabilité a son importance!
Avec l’IDE intégré, impossible de tester son code autrement que l’exécuter manuellement.
À chaque changement, il faut donc valider manuellement le bon fonctionnement dans les cas attendus, potentiellement sur de gros volumes de données difficile à inspecter à l’œil.
Bienvenue à l’âge de pierre du développement informatique.

À l’age de pierre il y a autre chose qu’on ne peut pas faire: débogguer son code dynamiquement. Le déboggueur python permet d’exécuter le code ligne par ligne, mettre des points d’arrêt, observer les variables et l’état du système à chaque instant. Oubliez tout-ceci dans XSOAR, ça n’est pas possible nativement.

Sur un éditeur moderne, le code python est analysé (par exemple avec [pylance](https://learn.microsoft.com/fr-fr/shows/vs-code-livestreams/pylance-new-and-improved-python-experience) sur vscode), et vous avez un retour avec la coloration syntaxique et les « problèmes » remontés par l’analyseur de code.
Cela permet à la personne développeuse d’identifier des défauts dans son code avant de l’exécuter, comme par exemple:

- des objets non importés
- des fautes de frappes
- des erreurs de types
- des arguments incohérents avec la fonction appelée

L’IDE intégré contient une coloration syntaxique basique, qui laisse passer la plupart de ces erreurs.

Un IDE, c’est aussi pour une personne développeuse un espace de développement personnalisé, avec une solution de formattage automatique, des raccourcis clavier, une coloration habituelle, etc.
Plein d’élélments lui permettant d’être plus efficace grâce aux habitudes et automatismes.

## Une solution facile à mettre en place: le développement en local

L’IDE de XSOAR nous limite dans notre capacité à créer rapidement du code fiable.
Il est possible à moindre coût de bénéficier des fonctionnalités et du confort de votre IDE. Je détaille ici comment faire avec [vscode](https://code.visualstudio.com/), le fonctionnement est probablement similaire pour d’autres IDEs comme [PyCharm](https://www.jetbrains.com/pycharm/) ou [Zed](https://zed.dev/).

### Récupération des fichiers

La première étape, c’est d’avoir le code sur lequel on travaille!
Il suffit de copier/coller naïvement le code de notre Script dans un fichier dans vscode.

Il nous manque alors plusieurs éléments qui sont importés automatiquement dans XSOAR.
On doit les expliciter dans notre cas.

`CommandResults`, comme tout un ensemble de fonctions et classes utilitaires, sont publiques.
Elles sont importées à partir de `CommonServerPython.py`, vous pouvez le télécharger [depuis github](https://github.com/demisto/content/blob/master/Packs/Base/Scripts/CommonServerPython/CommonServerPython.py) et le placer à côté de votre Script.
Si vous avez modifié `CommonServerUserPython` dans les Scripts de votre instance XSOAR, récupérez son contenu sur XSOAR et copiez-le également dans un fichier à côté du code de votre Script.

Il y a une petite surprise dans CommonServerPython.py (ligne 12086 au moment où cet article est rédigé):

```python
from DemistoClassApiModule import *     # type:ignore [no-redef]  # noqa:E402
```

Vous l’aurez compris, si on veut que notre IDE résolve bien tous les imports, on doit également récupérer le fichier [`DemistoClassApiModule.py`](https://github.com/demisto/content/blob/master/Packs/ApiModules/Scripts/DemistoClassApiModule/DemistoClassApiModule.py).


Pour l’objet `demisto`, son code source n’est pas accessible.
Cependant, Palo Alto fournit `demistomock` pour permettre de simuler son fonctionnement hors de XSOAR.
Téléchargez-le [depuis github](https://github.com/demisto/content/blob/master/Tests/demistomock/demistomock.py) et ajoutez-le à côté également.

Nous avons maintenant tout nos fichiers en place, il nous reste à les connecter proprement.

### Import du code externe

Il faut modifier le code du script, et ajouter les imports nécessaires en haut du fichier:

```python
import demistomock as demisto
from CommonServerPython import *
```

La première ligne permet d’avoir accès à l’objet `demisto` simulé et la seconde à importer tous les utilitaires natifs.

Si nécessaire, vous pouvez rajouter `from CommonServerUserPython import *` comme troisième  _import_ si vous avez du code dans ce fichier également.

Vous avez également besoin de Pylance dans vscode s’il n’est pas déjà installé.

Vous devez alors vous retrouver avec les fichiers comme ci-dessous, et avez dorénavant un environemment de développement où vous pourrez profiter de la coloration syntaxique, le _linter_ (notamment la très puissante vérification des types) et le débugger dynamique.

![](/img/demisto-local-dev-setup-vscode.png)

### Les entrées et sorties

Les entrées du script sont passées dans XSOAR à travers `demisto.args()`.
Comment passer les arguments de notre choix pendant le développement?

La réponse est à trouver dans `demistomock.py`, on peut y voir

```python
def args():
    """Retrieves a command / script arguments object

    Returns:
      dict: Arguments object

    """
    if os.path.exists(ARGS_COMMAND_PATH):
        with open(ARGS_COMMAND_PATH) as f:
            try:
                args = json.load(f)
            except json.JSONDecodeError:
                return {}
            args.pop("cmd", None)
            return args
    return {}
```

Et un peu plus haut dans le fichier:

```python
ARGS_COMMAND_PATH = os.path.join(os.path.dirname(__file__), ".args_command.json")
```

On peut alors placer nos arguments dans un fichier `.args_command.json` à côté de `demistomock.py` avec le format suivant:

```json
{
    "arg_str": "valeur de l’argument `arg_str`",
    "arg_list": ["a", "b", "c"]
}
```

On aura ici deux arguments: `arg_str` qui est une chaîne de caractères et `arg_list` qui est une liste.

Pour la sortie, on observe par la même méthode, en observant `CommonServerPython`, que `return_results` fait appel à `demisto.results()`.
Dans `demistomock.py` on voit que la fonction `results()` fait appel a la fonction `log()` pour afficher les résultats.
La fonction `log()` quand à elle, exécute le code suivant

```python
logging.getLogger().info(msg)
```

Malheureusement, le logging n’est pas configuré dans le code de `demistomock.py`.
Le code de la fonction `log()` ne fait donc rien!
Il faut ajouter la ligne suivante (en haut du fichier après les imports par exemple) pour qu’il soit affiché.

```python
logging.basicConfig(level=logging.DEBUG)
```

Vous aurez alors la sortie affichée dans votre terminal à l’exécution du code!

```text
INFO:root:demisto results: {
    "Contents": ...,
    "ContentsFormat": "json",
    "EntryContext": {...},
    "HumanReadable": "...",
    "IgnoreAutoExtract": false,
    "IndicatorTimeline": [],
    "Note": false,
    "Relationships": [],
    "Type": 1
}
```

### L’appel à d’autres scripts

Il est possible de faire appel à d’autres Scripts dans XSOAR à partir d’un Script.
Pour cela, la méthode recommandée est d’utiliser la fonction [`execute_command()`](https://xsoar.pan.dev/docs/reference/api/common-server-python#execute_command).

De la même façon que précédemment, on observe dans `CommonServerPython.py` que `execute_command` fait appel à la classe `demisto`: `demisto.executeCommand()`.

Le code dans `demistomock.py` est assez simple.

```python
def executeCommand(command, args):
    """..."""
    commands = {
        "getIncidents": exampleIncidents,
        "getContext": exampleContext,
        "getUsers": exampleUsers,
    }
    if commands.get(command):
        return commands.get(command)

    return ""
```

On peut modifier cette fonction pour renvoyer ce que l’on souhaite pour différents appels de fonction.
Soit en ne se basant que sur le nom de la commande comme c’est déjà le cas, soit en utilisant l’argument `args` et, selon les arguments, renvoyer des résultats pré-enregistrés différents voire faire appel à d’autres fonctions.

### Le développement d’intégrations

Les intégrations fonctionnent globalement comme des scripts avec deux spécificités:

- une commande est spécifiée
- des paramètres d’intégration (identifiants, adresses, etc.) sont accessibles dans le code

La commande est récupérée à travers `demisto.command()`. Son code dans `demistomock.py` est le suivant:

```python
def command():
    """..."""
    if os.path.exists(ARGS_COMMAND_PATH):
        with open(ARGS_COMMAND_PATH) as f:
            try:
                return json.load(f)["cmd"]
            except json.JSONDecodeError:
                return ""
            except KeyError:
                return ""
    return ""
```

On retrouve le `ARGS_COMMAND_PATH` précédent.
Pour tester le code, il suffit de mettre dans le fichier `.args_command.json` le nom de la commande sous la clé `cmd`:

```json
{
    "cmd": "ma-commande",
    "arg_1": "un argument",
    "arg_2": "un autre argument"
}
```

Au sein de `demistomock.py`, les paramètres sont récupérés dans la variable d’environnement `DEMISTO_PARAMS`.

```python
def params():
    """..."""
    demisto_params = os.getenv("DEMISTO_PARAMS")
    if demisto_params:
        try:
            return json.loads(demisto_params)
        except json.JSONDecodeError:
            return {}
    return {}
```

## Retour dans XSOAR

Une fois le code terminé, il est possible de le copier/coller dans XSOAR pour l’enregistrer dans l’outil.

N’oubliez pas de retirer les imports ajoutés, utiles seulement pendant le développement!