---
date: '2024-11-11T18:39:08+01:00'
draft: false
title: 'Local development for XSOAR'
slug: 'local-development-on-xsoar'
categories:
  - HowTo
tags:
  - XSOAR
description: 'Learn how to develop content for XSOAR from your IDE.'
---

XSOAR has a built-in IDE that allows you to develop Scripts - an essential component for customizing the orchestrator.
However, it has a number of limitations.
In this article, you will learn how to use your favorite IDE to develop Scripts.

<!--more-->

## Customizing XSOAR

There are mainly two levels of development in XSOAR: Playbooks and Scripts (Integrations are just a kind of Script).

Playbooks are created through the visual editor integrated into the solution. There is much to say about this editor and its specifics, but for this article, we will focus on editing Scripts.

Scripts are an inevitable aspect of XSOAR, as soon as you want to perform custom operations on your SOAR. They are primarily written in Python (or JavaScript, but the cybersecurity world is not very fond of JS); and when you want to modify one, XSOAR integrates an IDE to allow you to do so.

## How Scripts Work

Scripts allow you to execute code that you control within XSOAR. There are a number of them delivered with the solution (`Set`, etc.) or that can be installed through a pack (for example, [Common Scripts](https://cortex.marketplace.pan.dev/marketplace/details/CommonScripts/)). You can also create your own when off-the-shelf Scripts do not fit. Comparing complex objects? Manipulating exotic data? Transforming custom data? Executing an original algorithm? Leveraging functions from existing libraries? Create a Script!

For example, XSOAR offers a rudimentary templating system. To benefit from the power of [Jinja](https://jinja.palletsprojects.com/), you can easily create a Script that takes the data and the template passed as arguments and outputs the rendering to be used later in the playbook.

In XSOAR, Scripts consist of code and metadata. The code will contain the algorithm you want to execute. The metadata serves several purposes:

- Documenting usage
- Defining the interface with the rest of XSOAR, such as the arguments and returned data
- Specifying the execution context (which Docker container, should it run on an Engine?)

The Script code has access to certain objects and functions specific to XSOAR. For example, it is possible to access the data passed as arguments to the Script through the [object `demisto`](https://xsoar.pan.dev/docs/reference/api/demisto-class):

```python
args = demisto.args()
```

You can also access the incident on which the Script is running with the same object:

```python
incident = demisto.incident()
```

You can also use utility classes like [`CommandResults`](https://xsoar.pan.dev/docs/reference/api/common-server-python#commandresults) to indicate how to return data at the end of the Script execution, [`return_results`](https://xsoar.pan.dev/docs/reference/api/common-server-python#return_results) which returns the execution results to XSOAR, or [`tableToMarkdown`](https://xsoar.pan.dev/docs/reference/api/common-server-python#tabletomarkdown) which transforms a list into a markdown table:

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

In this example, CommandResults is an object, and return_results will read it to display the data in the right place: in the context data, in a note in the war room, etc.

Once the Script is created, it is accessible in the library of the XSOAR instance and can be used in playbooks as a standalone task.

## Limitations of the Built-in IDE

The built-in IDE ([Ace](https://ace.c9.io/)) has some basic features like syntax highlighting or search.

However, some crucial aspects of software development are not possible.

Testing is crucial when working with security orchestrators, as they execute critical processes across the organization's information systems. Whether you're deleting emails, disconnecting machines from the network, or escalating SOC alerts, code reliability is paramount. However, the built-in IDE makes systematic testing nearly impossible. With each change, you must manually validate that it works correctly in the expected cases, potentially on large volumes of data that are difficult to inspect visually. Welcome to the stone age of software development.

In the stone age, there is another thing you cannot do: dynamically debug your code. The Python debugger allows you to execute code line by line, set breakpoints, and observe variables and the system state at every moment. Forget all this in XSOAR; it is not possible natively.

In a modern editor, Python code is analyzed (for example, with [pylance](https://learn.microsoft.com/fr-fr/shows/vs-code-livestreams/pylance-new-and-improved-python-experience) on VSCode), and you get feedback with syntax highlighting and "problems" raised by the code analyzer. This allows the developer to identify defects in their code before executing it, such as:

- Unimported objects
- Typos
- Type errors
- Arguments inconsistent with the called function

The built-in IDE contains basic syntax highlighting, which lets most of these errors slip through.

An IDE is also a personalized development space for a developer, with automatic formatting solutions, keyboard shortcuts, familiar coloring, etc. Many elements allow them to be more efficient thanks to habits and automation.

## An Easy Solution to Implement: Local Development

The XSOAR IDE limits our ability to quickly create reliable code. It is possible at a low cost to benefit from the features and comfort of your IDE. Here's how to do it with [VSCode](https://code.visualstudio.com/), the process is probably similar for other IDEs like [PyCharm](https://www.jetbrains.com/pycharm/) or [Zed](https://zed.dev/).

### Retrieving Files

The first step is to have the code you are working on! You just need to naively copy/paste the code of your Script into a file in VSCode.

We are then missing several elements that are automatically imported into XSOAR. We need to specify them in our case.

`CommandResults`, like a whole set of utility functions and classes, are public. They are imported from `CommonServerPython.py`, you can download it [from github](https://github.com/demisto/content/blob/master/Packs/Base/Scripts/CommonServerPython/CommonServerPython.py) and place it next to your Script. If you have modified `CommonServerUserPython` in the Scripts of your XSOAR instance, retrieve its content from XSOAR and copy it into a file next to your Script code as well.

There is a little surprise in CommonServerPython.py (line 12086 at the time this article is written):

```python
from DemistoClassApiModule import *     # type:ignore [no-redef]  # noqa:E402
```

You will understand, if we want our IDE to resolve all imports correctly, we also need to retrieve the file [`DemistoClassApiModule.py`](https://github.com/demisto/content/blob/master/Packs/ApiModules/Scripts/DemistoClassApiModule/DemistoClassApiModule.py).

For the `demisto` object, its source code is not accessible. However, Palo Alto provides `demistomock` to simulate its functionality outside of XSOAR. Download it [from github](https://github.com/demisto/content/blob/master/Tests/demistomock/demistomock.py) and add it next to it as well.

We now have all our files in place; we just need to connect them properly.

### Importing External Code

You need to modify the Script code and add the necessary imports at the top of the file:

```python
import demistomock as demisto
from CommonServerPython import *
```

The first line allows access to the simulated `demisto` object, and the second imports all the native utilities.

If necessary, you can add `from CommonServerUserPython import *` as a third import if you have code in that file as well.

You also need Pylance in VSCode if it is not already installed.

You should then have the files as shown below, and now have a development environment where you can enjoy syntax highlighting, linting (notably the very powerful type checking), and dynamic debugging.

![](/img/demisto-local-dev-setup-vscode.png)

### Inputs and Outputs

The inputs of the Script are passed in XSOAR through `demisto.args()`. How to pass the arguments of our choice during development?

The answer can be found in `demistomock.py`, where we can see

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

And a little higher up in the file:

```python
ARGS_COMMAND_PATH = os.path.join(os.path.dirname(__file__), ".args_command.json")
```

We can then place our arguments in a `.args_command.json` file next to `demistomock.py` with the following format:

```json
{
    "arg_str": "value of the argument `arg_str`",
    "arg_list": ["a", "b", "c"]
}
```

Here we have two arguments: `arg_str` which is a string and `arg_list` which is a list.

For the output, we observe in the same way, by looking at `CommonServerPython`, that `return_results` calls `demisto.results()`. In `demistomock.py`, we see that the `results()` function calls the `log()` function to display the results. The `log()` function executes the following code

```python
logging.getLogger().info(msg)
```

Unfortunately, logging is not configured in the code of `demistomock.py`. Therefore, the code of the `log()` function does nothing! You need to add the following line (at the top of the file after the imports, for example) for it to be displayed.

```python
logging.basicConfig(level=logging.DEBUG)
```

You will then have the output displayed in your terminal when executing the code!

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

### Calling Other Scripts

It is possible to call other Scripts in XSOAR from a Script. To do this, the recommended method is to use the [`execute_command()`](https://xsoar.pan.dev/docs/reference/api/common-server-python#execute_command) function.

In the same way as before, we observe in `CommonServerPython.py` that `execute_command` calls the `demisto` class: `demisto.executeCommand()`.

The code in `demistomock.py` is quite simple.

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

We can modify this function to return what we want for different function calls. Either by relying solely on the command name as is already the case, or by using the `args` argument and, depending on the arguments, returning different pre-recorded results or even calling other functions.

### Developing Integrations

Integrations generally work like Scripts with two specificities:

- A command is specified
- Integration parameters (identifiers, addresses, etc.) are accessible in the code

The command is retrieved through `demisto.command()`. Its code in `demistomock.py` is as follows:

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

We find the previous `ARGS_COMMAND_PATH`. To test the code, simply put the command name in the `.args_command.json` file under the key `cmd`:

```json
{
    "cmd": "my-command",
    "arg_1": "an argument",
    "arg_2": "another argument"
}
```

Within `demistomock.py`, the parameters are retrieved from the environment variable `DEMISTO_PARAMS`.

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

## Returning to XSOAR

Once the code is finished, it is possible to copy/paste it into XSOAR to save it in the tool.

Don't forget to remove the added imports, which are only useful during development!