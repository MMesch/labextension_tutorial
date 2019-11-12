# Jupyterlab Extensions Walkthrough #

**Development is now happening in https://github.com/jtpio/jupyterlab-extension-examples with the following roadmap**:

- Update to JupyterLab 1.0
- Update existing examples
- Add more examples
- Move the repo to https://github.com/jupyterlab when ready

## Other tutorials ##
* http://JupyterLab.readthedocs.io/en/stable/developer/xkcd_extension_tutorial.html

## Table of Contents ##

* [Introduction](#introduction)
* [Prerequesites](#prerequesites)
* [1 Hello World: Setting up the development environment](#1-hello-world-setting-up-the-development-environment)
  * [The template folder structure](#the-template-folder-structure)
  * [A minimal extension that prints to the browser console](#a-minimal-extension-that-prints-to-the-browser-console)
  * [Building and Installing an Extension](#building-and-installing-an-extension)
* [2 Commands and Menus: Extending the main app](#2-commands-and-menu-extension-extending-the-main-app)
  * [Jupyterlab Commands](#jupyterlab-commands)
  * [Adding new Menu tabs and items](#adding-new-menu-tabs-and-items)
* [3 Widgets: Adding new Elements to the Main Window](#3-widgets-adding-new-elements-to-the-main-window)
  * [A basic tab](#a-basic-tab)
  * [Datagrid: a Fancy Phosphor Widget](#datagrid-a-fancy-phosphor-widget)
* [4 Kernel Outputs: Simple Notebook-style Rendering](#4-kernel-outputs-simple-notebook-style-rendering)
  * [Reorganizing the extension code](#reorganizing-the-extension-code)
  * [Initializing a Kernel Session](#initializing-a-kernel-session)
  * [OutputArea and Model](#outputarea-and-model)
  * [asynchronous extension initialization](#asynchronous-extension-initialization)
  * [Make it Run](#make-it-run)
  * [Jupyter-Widgets: Adding Interactive Elements](#jupyter-widgets-adding-interactive-elements)
* [5 Buttons and Signals: Interactions Between Different Widgets](#5-buttons-and-signals-interactions-between-different-widgets)
  * [Phosphor Signaling 101](#phosphor-signaling-101)
  * [A simple react button](#a-simple-react-button)
  * [subscribing to a signal](#subscribing-to-a-signal)
* [6 Custom Kernel Interactions: Kernel Managment and Messaging](#6-custom-kernel-interactions-kernel-managment-and-messaging)
  * [Component Overview](#component-overview)
  * [Initializing and managing a kernel session (panel.ts)](#initializing-and-managing-a-kernel-session-panelts)
  * [Executing code and retrieving messages from a kernel (model.ts)](#executing-code-and-retrieving-messages-from-a-kernel-modelts)
  * [Connecting a View to the Kernel](#connecting-a-view-to-the-kernel)
  * [How does it look like](#how-does-it-look-like)

## Introduction

This is a short tutorial series about JupyterLab extensions. Jupyterlab can be
used as a platform to combine existing data-science components into a new
powerful application that can be deployed remotely for many users. Some of the
higher level components that can be used are text editors, terminals,
notebooks, interactive widgets, filebrowser, renderers for different file
formats that provide access to an enormous ecosystem of libraries from
different languages.

## Prerequesites ##

Writing an extension is not particularly difficult but requires very basic
knowledge of javascript and typescript and potentially Python.

_Don't be scared of typescript, I never coded in typescript before I touched 
JupyterLab but found it easier to understand than pure javascript if you have a 
basic understanding of object oriented programming and types._

## 1 Hello World: Setting up the development environment ##

#### The template folder structure ####

Writing a JupyterLab extension usually starts from a configurable template. It
can be downloded with the `cookiecutter` tool and the following command:

```bash
cookiecutter https://github.com/JupyterLab/extension-cookiecutter-ts
```

`cookiecutter` asks for some basic information that could for example be setup
like this:
```bash
author_name []: tuto
extension_name [JupyterLab_myextension]: 1_hello_world
project_short_description [A JupyterLab extension.]: minimal lab example
repository [https://github.com/my_name/JupyterLab_myextension]: 
```

The cookiecutter creates the directory `1_hello_world` [or your extension name]
that looks like this:

```bash
1_hello_world/

├── README.md
├── package.json
├── tsconfig.json
├── src
│   └── index.ts
└── style
    └── index.css
```

* `README.md` contains some instructions
* `package.json` contains information about the extension such as dependencies
* `tsconfig.json` contains information for the typescript compilation
* `src/index.ts` _this contains the actual code of our extension_
* `style/index.css` contains style elements that we can use

What does this extension do? You don't need a PhD in Computer Science to take a
guess from the title of this section, but let's have a closer look:

#### A minimal extension that prints to the browser console ####

We start with the file `src/index.ts`. This typescript file contains the main
logic of the extension. It begins with the following import section:

```typescript
import {
  JupyterLab, JupyterLabPlugin
} from '@JupyterLab/application';

import '../style/index.css';
```

`JupyterLab` is class of the main Jupyterlab application. It allows us to
access and modify some of its main components. `JupyterLabPlugin` is the class
of the extension that we are building. Both classes are imported from a package
called `@JupyterLab/application`. The dependency of our extension on this
package is declared in the file `package.json`:

```json
  "dependencies": {
    "@JupyterLab/application": "^0.16.0"
  },
```

With this basic import setup, we can move on to construct a new instance
of the `JupyterLabPlugin` class:

```typescript
const extension: JupyterLabPlugin<void> = {
  id: '1_hello_world',
  autoStart: true,
  activate: (app: JupyterLab) => {
    console.log('JupyterLab extension 1_hello_world is activated!');
  }
};

export default extension;
```

a JupyterLabPlugin contains a few attributes that are fairly self-explanatory
in the case of `id` and `autoStart`. The `activate` attribute links to a
function (`() => {}` notation) that takes one argument `app` of type
`JupyterLab` and then calls the `console.log` function that is used to output
something into the browser console in javascript. `app` is simply the main
JupyterLab application. The activate function acts as an entry point into the
extension and we will gradually extend it to access and modify functionality
through the `app` object.

Our new `JupyterLabPlugin` instance has to be finally exported to be visible to
JupyterLab, which is done with the line `export default extension`. This brings
us to the next point. How can we plug this extension into JupyterLab?

#### Building and Installing an Extension ####

Let's look at the `README.md` file. It contains instructions how our
labextension can be installed for development:

> For a development install (requires npm version 4 or later), do the following
> in the repository directory:

```bash
npm install
npm run build
jupyter labextension link .
```

Roughly the first command installs the dependencies that are specified in
`package.json`. Among the dependencies are also all of the `JupyterLab`
components that we want to use in our project, but more about this later. The
second step runs the build script. In this step, the typescript code gets
converted to javascript using the compiler `tsc` and stored in a `lib`
directory. Finally, we link the module to JupyterLab.

After all of these steps are done, running `jupyter labextension list` should
now show something like:
```bash
   local extensions:
        1_hello_world: [...]/labextension_tutorial/1_hello_world
```

Now let's check inside of JupyterLab if it works. Run [can take a while]:

```bash
jupyter lab --watch
```

Our extension doesn't do much so far, it just writes something to the browser
console. So let's check if it worked. In firefox you can open the console
pressing the `f12` key. You should see something like:

```
JupyterLab extension 1_hello_world is activated
```

Our extension works but it is incredibly boring. Let's  modify the source code
a bit. Simply replace the `activate` function with the following lines:

```typescript
    activate: (app: JupyterLab) => {
        console.log('the main JupyterLab application:');
        console.log(app);
    }
```

to update the module, we simply need to go into the extension directory and run
`npm run build` again. Since we used the `--watch` option when starting
JupyterLab, we now only have to refresh the JupyterLab website in the browser
and should see in the browser console:

```
Object { _started: true, _pluginMap: {…}, _serviceMap: Map(28), _delegate: {…}, commands: {…}, contextMenu: {…}, shell: {…}, registerPluginErrors: [], _dirtyCount: 0, _info: {…}, … } index.js:12
```

This is the main application JupyterLab object and we will see how to interact
with it in the next sections.


_checkout how the core packages of JupyterLab are defined at
https://github.com/JupyterLab/JupyterLab/tree/master/packages . Each package is
structured similarly to the extension that we are writing. This modular
structure makes JupyterLab very adapatable_

An overview of the classes and their attributes and methods can be found in the
JupyterLab documentation. The `@JupyterLab/application` module documentation is
[here](https://JupyterLab.github.io/JupyterLab/modules/_application_src_index_.html)
and which links to the [JupyterLab class](https://JupyterLab.github.io/JupyterLab/classes/_application_src_index_.JupyterLab.html).
The `JupyterLabPlugin` is a type alias [a new name] for the type `IPlugin`.
The definition of `IPlugin` is more difficult to find because it is defined by
the `phosphor.js` library that runs JupyterLab under the hood (more about this
later). Its documentation is therefore located on the [phosphor.js
website](http://phosphorjs.github.io/phosphor/api/application/interfaces/iplugin.html).

[Click here for the final extension: 1_hello_world](1_hello_world)

## 2 Commands and Menus: Extending the main app ##

For the next extension you can either copy the last folder to a new one or
simply continue modifying it. In case that you want to have a new extension,
open the file `package.json` and modify the package name, e.g. into
`2_commands_and_menus`. The same name changes needs to be done in
`src/index.ts`.

#### Jupyterlab Commands ####

Start it with `jupyter lab --watch`. In this extension, we are going to add a
command to the application command registry and expose it to the user in the
command palette.  The command palette can be seen when clicking on _Commands_
on the left hand side of Jupyterlab. The command palette can be seen as a list
of actions that can be executed by JupyterLab. (see screenshot below).

![Jupyter Command Registry](images/command_registry.png)

Extensions can provide a bunch of functions to the JupyterLab command registry
and then expose them to the user through the command palette or through a menu
item.

Two types play a role in this: the `CommandRegistry` type ([documentation](http://phosphorjs.github.io/phosphor/api/commands/classes/commandregistry.html))
and the command palette interface `ICommandPalette` that is imported with:

```typescript
import {
  ICommandPalette
} from '@JupyterLab/apputils';
```

To see how we access the applications command registry and command palette
open the file `src/index.ts`.

```typescript
const extension: JupyterLabPlugin<void> = {
    id: '2_commands_and_menus',
    autoStart: true,
    requires: [ICommandPalette],
    activate: (
        app: JupyterLab,
        palette: ICommandPalette) =>
    {
        const { commands } = app;

        let command = 'labtutorial';
        let category = 'Tutorial';

        commands.addCommand(command, {
            label: 'New Labtutorial',
            caption: 'Open the Labtutorial',
            execute: (args) => {console.log('Hey')}});

        palette.addItem({command, category});
    }
};

export default extension;
```

The CommandRegistry is an attribute of the main JupyterLab application
(variable `app` in the previous section). It has an `addCommand` method that
adds our own function.
The ICommandPalette
([documentation](https://JupyterLab.github.io/JupyterLab/interfaces/_apputils_src_commandpalette_.icommandpalette.html))
is passed to the `activate` function as an argument (variable `palette`) in
addition to the JupyterLab application (variable `app`). We specify with the
property `requires: [ICommandPalette],` which additional arguments we want to
inject into the `activate` function in the JupyterLabPlugin. ICommandPalette
provides the method `addItem` that links a palette entry to a command in the
command registry. Our new plugin code then becomes:

When this extension is build (and linked if necessary), JupyterLab looks like
this:

![New Command](images/new_command.png)

#### Adding new Menu tabs and items ####

Adding new menu items works in a similar way. The IMainMenu interface can be
passed as a new argument two the activate function, but first it has to be
imported. The Menu class is imported from the phosphor library on top of which
JupyterLab is built, and that will be frequently encountered when developing
JupyterLab extensions:

```typescript
import {
  IMainMenu
} from '@JupyterLab/mainmenu';

import {
  Menu
} from '@phosphor/widgets';
```

We add the IMainMenu in the `requires:` property such that it is injected into
the `activate` function. The Extension is then changed to:

```typescript
const extension: JupyterLabPlugin<void> = {
    id: '2_commands_and_menus',
    autoStart: true,
    requires: [ICommandPalette, IMainMenu],
    activate: (
        app: JupyterLab,
        palette: ICommandPalette,
        mainMenu: IMainMenu) =>
    {
        const { commands } = app;
        let command = 'labtutorial';
        let category = 'Tutorial';
        commands.addCommand(command, {
            label: 'New Labtutorial',
            caption: 'Open the Labtutorial',
            execute: (args) => {console.log('Hey')}});
        palette.addItem({command, category});

        let tutorialMenu: Menu = new Menu({commands});

        tutorialMenu.title.label = 'Tutorial';
        mainMenu.addMenu(tutorialMenu, {rank: 80});
        tutorialMenu.addItem({ command });
    }
};

export default extension;
```

In this extension, we have added the dependencies _JupyterLab/mainmenu_ and
_phosphor/widgets_. Before it builds, this dependencies have to be added to the
`package.json` file:

```json
  "dependencies": {
    "@JupyterLab/application": "^0.16.0",
    "@JupyterLab/mainmenu": "*",
    "@phosphor/widgets": "*"
  }
```

we can then do

```
npm install
npm run build
```

to rebuild the application. After a browser refresh, the JupyterLab website
should now show:

![New Menu](images/new_menu.png)

[ps:

for the build to run, the `tsconfig.json` file might have to be updated to:
```
{
  "compilerOptions": {
    "declaration": true,
    "noImplicitAny": true,
    "noEmitOnError": true,
    "noUnusedLocals": true,
    "module": "commonjs",
    "moduleResolution": "node",
    "target": "ES6",
    "outDir": "./lib",
    "lib": ["ES6", "ES2015.Promise", "DOM"],
    "types": []
  },
  "include": ["src/*"]
}
```
]

[Click here for the final extension 2_commands_and_menus](2_commands_and_menus)


## 3 Widgets: Adding new Elements to the Main Window ##

Finally we are going to do some real stuff and add a new tab to JupyterLab.
Visible elements such as a tab are represented by widgets in the phosphor
library that is the basis of the JupyterLab application.

#### A basic tab ####

The base widget class can be imported with:

```typescript
import {
    Widget
} from '@phosphor/widgets';
```

A Widget can be added to the main area through the main JupyterLab
application`app.shell`. Inside of our previous `activate` function, this looks
like this:

```
    activate: (
        app: JupyterLab,
        palette: ICommandPalette,
        mainMenu: IMainMenu) =>
    {
        const { commands, shell } = app;
        let command = 'ex3:labtutorial';
        let category = 'Tutorial';
        commands.addCommand(command, {
            label: 'Ex3 command',
            caption: 'Open the Labtutorial',
            execute: (args) => {
                const widget = new TutorialView();
                shell.addToMainArea(widget);}});
        palette.addItem({command, category});

        let tutorialMenu: Menu = new Menu({commands});

        tutorialMenu.title.label = 'Tutorial';
        mainMenu.addMenu(tutorialMenu, {rank: 80});
        tutorialMenu.addItem({ command });
    }
```

The custom widget `TutorialView` is straight-forward as well:

```typescript
class TutorialView extends Widget {
    constructor() {
        super();
        this.addClass('jp-tutorial-view')
        this.id = 'tutorial'
        this.title.label = 'Tutorial View'
        this.title.closable = true;
    }
}
```

Note that we have used a custom css class that is defined in the file
`style/index.css` as:

```
.jp-tutorial-view {
    background-color: AliceBlue;
}
```

Our custom tab can be started in JupyterLab from the command palette and looks
like this:

![Custom Tab](images/custom_tab.png)

[Click here for the final extension: 3_widgets](3a_widgets)


#### Datagrid: a Fancy Phosphor Widget ####

Now let's do something a little more advanced. Jupyterlab is build on top of
Phosphor.js. Let's see if we can plug [this phosphor example](http://phosphorjs.github.io/examples/datagrid/)
into JupyterLab.

We start by importing the `Panel` widget and the `DataGrid` and `DataModel`
classes from phosphor:

```typescript
import {
    Panel
} from '@phosphor/widgets';

import {
  DataGrid, DataModel
} from '@phosphor/datagrid';
```

The Panel widget can hold several sub-widgets that are added with its
`.addWidget` method. `DataModel` is a class that provides the data that is
displayed by the `DataGrid` widget.

With these three classes, we adapt the `TutorialView` as follows:

```typescript
class TutorialView extends Panel {
    constructor() {
        super();
        this.addClass('jp-tutorial-view')
        this.id = 'tutorial'
        this.title.label = 'Tutorial View'
        this.title.closable = true;

        let model = new LargeDataModel();
        let grid = new DataGrid();
        grid.model = model;

        this.addWidget(grid);
    }
}
```

That's rather easy. Let's now dive into the `DataModel` class that is taken
from the official phosphor.js example. The first few lines look like this:

```typescript
class LargeDataModel extends DataModel {

  rowCount(region: DataModel.RowRegion): number {
    return region === 'body' ? 1000000000000 : 2;
  }

  columnCount(region: DataModel.ColumnRegion): number {
    return region === 'body' ? 1000000000000 : 3;
  }
```

While it is fairly obvious that `rowCount` and `columnCount` are supposed
to return some number of rows and columns, it is a little more cryptic what
the `RowRegion` and the `ColumnRegion` input arguments are. Let's have a
look at their definition in the phosphor.js source code:

```typescript
export
type RowRegion = 'body' | 'column-header';

/**
 * A type alias for the data model column regions.
 */
export
type ColumnRegion = 'body' | 'row-header';

/**
 * A type alias for the data model cell regions.
 */
export
type CellRegion = 'body' | 'row-header' | 'column-header' | 'corner-header';
```

The meaning of these lines might be obvious for experienced users of typescript
or Haskell. The `|` can be read as or. This means that the `RowRegion` type is
either `body` or `column-header`, explaining what the `rowCount` and
`columnCount` functions do: They define a table with `2` header rows, with 3
index columns, with `1000000000000` rows and `1000000000000` columns.

The remaining part of the LargeDataModel class defines the data values of the
datagrid. In this case it simply displays the row and column index in each
cell, and adds a letter prefix in case that we are in any of the header
regions:

```typescript
  data(region: DataModel.CellRegion, row: number, column: number): any {
    if (region === 'row-header') {
      return `R: ${row}, ${column}`;
    }
    if (region === 'column-header') {
      return `C: ${row}, ${column}`;
    }
    if (region === 'corner-header') {
      return `N: ${row}, ${column}`;
    }
    return `(${row}, ${column})`;
  }
}
```

Let's see how this looks like in Jupyterlab:

![Datagrid](images/datagrid.png)

[Click here for the final extension: 3b_datagrid](3b_datagrid)


## 4 Kernel Outputs: Simple Notebook-style Rendering ##

In this extension we will see how initialize a kernel, and how to execute code
and how to display the rendered output. We use the `OutputArea` class for this
purpose that Jupyterlab uses internally to render the output area under a
notebook cell or the output area in the console.

Essentially, `OutputArea` will renders the data that comes as a reply to an
execute message that was sent to an underlying kernel. Under the hood, the
`OutputArea` and the `OutputAreaModel` classes act similar to the `KernelView`
and `KernelModel` classes that we have defined ourselves before.  We therefore
get rid of the `model.ts` and `widget.tsx` files and change the panel class to:

#### Reorganizing the extension code ####

Since our extension is growing bigger, we begin by splitting our code into more
managable units. Roughly we can see three larger components of our application:

1.  the `JupyterLabPlugin` that activates all extension components and connects
    them to the main `Jupyterlab` application via commands, launcher, or menu
    items.
2.  a Panel that combines different widgets into a single application

We split these components in the two files:

```
src/
├── index.ts
└── panel.ts
```

Let's go first through `panel.ts`. This is the full Panel class that displays
starts a kernel, then sends code to it end displays the returned data with the
jupyter renderers:

```
export
class TutorialPanel extends StackedPanel {
    constructor(manager: ServiceManager.IManager, rendermime: RenderMimeRegistry) {
        super();
        this.addClass(PANEL_CLASS);
        this.id = 'TutorialPanel';
        this.title.label = 'Tutorial View'
        this.title.closable = true;

        let path = './console';

        this._session = new ClientSession({
            manager: manager.sessions,
            path,
            name: 'Tutorial',
        });

        this._outputareamodel = new OutputAreaModel();
        this._outputarea = new SimplifiedOutputArea({ model: this._outputareamodel, rendermime: rendermime });

        this.addWidget(this._outputarea);
        this._session.initialize();
    }

    dispose(): void {
        this._session.dispose();
        super.dispose();
    }

    public execute(code: string) {
        SimplifiedOutputArea.execute(code, this._outputarea, this._session)
            .then((msg: KernelMessage.IExecuteReplyMsg) => {console.log(msg); })
    }

    protected onCloseRequest(msg: Message): void {
        super.onCloseRequest(msg);
        this.dispose();
    }

    get session(): IClientSession {
        return this._session;
    }

    private _session: ClientSession;
    private _outputarea: SimplifiedOutputArea;
    private _outputareamodel: OutputAreaModel;
}
```

#### Initializing a Kernel Session ####

The first thing that we want to focus on is the `ClientSession` that is 
stored in the private `_session` variable:

```
private _session: ClientSession;
```

A ClientSession handles a single kernel session. The session itself (not yet
the kernel) is started with these lines:

```
        let path = './console';

        this._session = new ClientSession({
            manager: manager.sessions,
            path,
            name: 'Tutorial',
        });
```

A kernel is initialized with this line:
```
        this._session.initialize();
```
In case that a session has no predefined favourite kernel, a popup will be
started that asks the user which kernel should be used. Conveniently, this can
also be an already existing kernel, as we will see later.

The following three methods add functionality to cleanly dispose of the session
when we close the panel, and to expose the private session variable such that
other users can access it.
```
    dispose(): void {
        this._session.dispose();
        super.dispose();
    }

    protected onCloseRequest(msg: Message): void {
        super.onCloseRequest(msg);
        this.dispose();
    }

    get session(): IClientSession {
        return this._session;
    }
```

#### OutputArea and Model ####
The `SimplifiedOutputArea` class is a Widget, as we have seen them before. We
can instantiate it with a new `OutputAreaModel` (this is class that contains
the data that will be shown):

```
        this._outputareamodel = new OutputAreaModel();
        this._outputarea = new SimplifiedOutputArea({ model: this._outputareamodel, rendermime: rendermime });
```

`SimplifiedOutputArea` provides the classmethod `execute` that basically sends
some code to a kernel through a ClientSession and that then displays the result
in a specific `SimplifiedOutputArea` instance:

```
    public execute(code: string) {
        SimplifiedOutputArea.execute(code, this._outputarea, this._session)
            .then((msg: KernelMessage.IExecuteReplyMsg) => {console.log(msg); })
    }
```

The `SimplifiedOutputArea.execute` function receives at some point a response
message from the kernel which says that the code was executed (this message
does not contain the data that is displayed). When this message is received,
`.then` is executed and prints this message to the console.

We just have to add the `SimplifiedOutputArea` Widget to our Panel with:
```
        this.addWidget(this._outputarea);
```
and we are ready to add the whole Panel to Jupyterlab.

#### asynchronous extension initialization ####

`index.ts` is responsible to initialize the extension. We only go through
the changes with respect to the last sections.

First we reorganize the extension commands into one unified namespace:

```typescript
namespace CommandIDs {
    export
    const create = 'Ex7:create';

    export
    const execute = 'Ex7:execute';
}
```

This allows us to add commands from the command registry to the pallette and
menu tab in a single call:

```typescript
    // add items in command palette and menu
    [
        CommandIDs.create,
        CommandIDs.execute
    ].forEach(command => {
        palette.addItem({ command, category });
        tutorialMenu.addItem({ command });
    });
```

Another change is that we now use the `manager` to add our extension after the
other jupyter services are ready. The serviceManager can be obtained from the
main application as:

```typescript
    const manager = app.serviceManager;
```

to launch our application, we can then use:

```typescript
    let panel: TutorialPanel;

    function createPanel() {
        return manager.ready
            .then(() => {
                panel = new TutorialPanel(manager, rendermime);
                return panel.session.ready})
            .then(() => {
                shell.addToMainArea(panel);
                return panel});
    }
```

#### Make it Run ####

Let's for example display the variable `df` from a python kernel that could
contain a pandas dataframe. To do this, we just need to add a command to the
command registry in `index.ts`

```
    command = CommandIDs.execute
    commands.addCommand(command, {
        label: 'Ex7: show dataframe',
        caption: 'show dataframe',
        execute: (args) => {panel.execute('df')}});
```

and we are ready to see it. The final extension looks like this:

![OutputArea class](images/outputarea.gif)

[Click here for the final extension: 4a_outputareas](4a_outputareas)

#### Jupyter-Widgets: Adding Interactive Elements ####

A lot of advanced functionality in Jupyter notebooks comes in the form of
jupyter widgets (ipython widgets). Jupyter widgets are elements that have one
representation in a kernel and another representation in the JupyterLab
frontend code. Imagine having a large dataset in the kernel that we want to
examine and modify interactively. In this case, the frontend part of the widget
can be changed and updates synchronously the kernel data. Many other widget
types are available and can give an app-like feeling to a Jupyter notebook.
These widgets are therefore ideal component of a JupyterLab extension.

We show in this extension how the ipywidget `qgrid` can be integrated into
JupyterLab. As a first step, install `ipywidgets` and `grid`. It should work
in a similar way with any other ipywidget.

(These are the commands to install the ipywidgets with anaconda:
```
conda install -c conda-forge ipywidgets
conda install -c conda-forge qgrid
jupyter labextension install @jupyter-widgets/JupyterLab-manager
jupyter labextension install qgrid
```
)

Before continuing, test if you can (a) open a notebook, and (b) see a table
when executing these commands in a cell:

```
import pandas as pd
import numpy as np
import qgrid
df = pd.DataFrame(np.arange(25).reshape(5, 5))
qgrid.show_grid(df)
```

If yes, we can check out how we can include this table in our own app. Similar
to the previous Extension, we will use the `OutputArea` class to display the
widget. Only some minor adjustments have to be made to make it work.

The first thing is to understand the nature of the jupyter-widgets JupyterLab
extension (called `jupyterlab-manager`). As this text is written (26/6/2018) it
is a *document* extension and not a general extension to JupyterLab. This means
it provides extra functionality to the notebook document type and not the the
full Jupyterlab app. The relevant lines from the jupyter-widgets source code
that show how it registers its renderer with Jupyterlab are the following:

```typescript
export
class NBWidgetExtension implements INBWidgetExtension {
  /**
   * Create a new extension object.
   */
  createNew(nb: NotebookPanel, context: DocumentRegistry.IContext<INotebookModel>): IDisposable {
    let wManager = new WidgetManager(context, nb.rendermime);
    this._registry.forEach(data => wManager.register(data));
    nb.rendermime.addFactory({
      safe: false,
      mimeTypes: [WIDGET_MIMETYPE],
      createRenderer: (options) => new WidgetRenderer(options, wManager)
    }, 0);
    return new DisposableDelegate(() => {
      if (nb.rendermime) {
        nb.rendermime.removeMimeType(WIDGET_MIMETYPE);
      }
      wManager.dispose();
    });
  }

  /**
   * Register a widget module.
   */
  registerWidget(data: base.IWidgetRegistryData) {
    this._registry.push(data);
  }
  private _registry: base.IWidgetRegistryData[] = [];
}
```

The `createNew` method of `NBWidgetExtension` takes a `NotebookPanel` as input
argument and then adds a custom mime renderer with the command
`nb.rendermime.addFactory` to it. The widget renderer (or rather RenderFactory)
is linked to the `WidgetManager` that stores the underlying data of the
jupyter-widgets. Unfortunately, this means that we have to access
jupyter-widgets through a notebook because its renderer and WidgetManager are
attached to it.

To access the current notebook, we can use an `INotebookTracker` in the
plugin's activate function:

```
const extension: JupyterLabPlugin<void> = {
    id: '6_ipywidgets',
    autoStart: true,
    requires: [ICommandPalette, INotebookTracker, ILauncher, IMainMenu],
    activate: activate
};


function activate(
    app: JupyterLab,
    palette: ICommandPalette,
    tracker: INotebookTracker,
    launcher: ILauncher,
    mainMenu: IMainMenu)
{
    [...]
```

We then pass the `rendermime` registry of the notebook (the one that has the
jupyter-widgets renderer added) to our panel:

```
    function createPanel() {
        let current = tracker.currentWidget;
        console.log(current.rendermime);

        return manager.ready
            .then(() => {
                panel = new TutorialPanel(manager, current.rendermime);
                return panel.session.ready})
            .then(() => {
                shell.addToMainArea(panel);
                return panel});
    }
```

Finally we add a command to the registry that executes the code `widget` that
displays the variable `widget` in which we are going to store the qgrid widget:

```
    let code = 'widget'
    command = CommandIDs.execute
    commands.addCommand(command, {
        label: 'Ex8: show widget',
        caption: 'show ipython widget',
        execute: () => {panel.execute(code)}});
```

To render the Output we have to allow the `OutputAreaModel` to use non-default
mime types, which can be done like this:

```
        this._outputareamodel = new OutputAreaModel({ trusted: true });
```

The final output looks is demonstrated in the gif below. We also show that we
can attach a console to a kernel, that shows all executed commands, including
the one that we send from our Extension.

![Qgrid widget](images/qgrid_widget.gif)

[Click here for the final extension: 4b_jupyterwidgets](4b_jupyterwidgets)

## 5 Buttons and Signals: Interactions Between Different Widgets ##

#### Phosphor Signaling 101 ####

In this extension, we are going to add some simple buttons to the widget that
trigger the panel to print something to the console. Communication between
different components of JupyterLab are a key ingredient in building an
extension. Jupyterlab's phosphor engine uses the `ISignal` interface and the
`Signal` class that implements this interface for communication
([documentation](http://phosphorjs.github.io/phosphor/api/signaling/globals.html)).

The basic concept is the following: A widget, in our case the one that contains
some visual elements such as a button, defines a signal and exposes it to other
widgets, as this `_stateChanged` signal:

```typescript
    get stateChanged(): ISignal<TutorialView, void> {
        return this._stateChanged;
    }

    private _stateChanged = new Signal<TutorialView, void>(this);
```

Another widget, in our case the panel that boxes several different widgets,
subscribes to the `stateChanged` signal and links some function to it:

```typescript
[...].stateChanged.connect(() => { console.log('changed'); });
```

The function is executed when the signal is triggered with

```typescript
_stateChanged.emit(void 0)
```

Let's see how we can implement this ...

#### A simple react button ####

We start with a file called `src/widget.tsx`. The `tsx` extension allows to use
XML-like syntax with the tag notation `<>`to represent some visual elements
(note that you might have to add a line: `"jsx": "react",` to the
`tsconfig.json` file).

`widget.tsx` contains one major class `TutorialView` that extends the
`VDomRendered` class that is provided by Jupyterlab. `VDomRenderer` defines a
`render()` method that defines some html elements (react) such as a button.

`TutorialView` further contains a private variable `stateChanged` of type
`Signal`. A signal object can be triggered and then emits an actual message.
Other Widgets can subscribe to such a signal and react when a message is
emitted. We configure one of the buttons `onClick` event to trigger the
stateChanged` signal with `_stateChanged.emit(void 0)`:

```typescript
export
class TutorialView extends VDomRenderer<any> {
    constructor() {
        super();
        this.id = `TutorialVDOM`
    }

    protected render(): React.ReactElement<any>[] {
        const elements: React.ReactElement<any>[] = [];
        elements.push(
            <button
                key='header-thread'
                className="jp-tutorial-button"
                onClick={() => {this._stateChanged.emit(void 0)}}>
            Clickme
            </button>
            );
        return elements;
    }

    get stateChanged(): ISignal<TutorialView, void> {
        return this._stateChanged;
    }

    private _stateChanged = new Signal<TutorialView, void>(this);
}
```

#### subscribing to a signal ####

The `panel.ts` class defines an extension panel that displays the
`TutorialView` widget and that subscribes to its `stateChanged` signal.
Subscription to a signal is done using the `connect` method of the
`stateChanged` attribute.  It registers a function (in this case
`() => { console.log('changed'); }` that is triggered when the signal is
emitted:


```typescript
export
class TutorialPanel extends StackedPanel {
    constructor() {
        super();
        this.addClass(PANEL_CLASS);
        this.id = 'TutorialPanel';
        this.title.label = 'Tutorial View'
        this.title.closable = true;

        this.tutorial = new TutorialView();
        this.addWidget(this.tutorial);
        this.tutorial.stateChanged.connect(() => { console.log('changed'); });
    }

    private tutorial: TutorialView;
}
```


The final extension writes a little `changed` text to the browser console when
a big red button is clicked. It is not very spectacular but the signaling is
conceptualy important for building extensions. It looks like this:


![Button with Signal](images/button_with_signal.png)


[Click here for the final extension: 5_signals_and_buttons](5_signals_and_buttons)


## 6 Custom Kernel Interactions: Kernel Managment and Messaging ##

One of the main features of JupyterLab is the possibility to manage and
interact underlying compute kernels. In this section, we explore how to
start a kernel and execute a simple command on it.

#### Component Overview ####

In terms of organization of this app, we want to have these components:

* `index.ts`: a JupyterLabPlugin that initializes the plugin and registers commands, menu tabs and a launcher
* `panel.ts`: a panel class that is responsible to initialize and hold the kernel session, widgets and models 
* `model.ts`: a KernelModel class that is responsible to execute code on the kernel and to store the execution result
* `widget.tsx`: a KernelView class that is responsible to provide visual elements that trigger the kernel model and display its results

The `KernelView` displays the `KernelModel` with some react html elements and
needs to get updated when `KernelModel` changes state, i.e. retrieves a new
execution result. Jupyterlab provides a two class model for such classes,
a `VDomRendered` that has a link to a `VDomModel` and a `render` function.
The `VDomRendered` listens to a `stateChanged` signal that is defined by the
`VDomModel`. Whenever the `stateChanged` signal is emitted, the
`VDomRendered` calls its `render` function again and updates the html elements
according to the new state of the model.


#### Initializing and managing a kernel session (`panel.ts`) ####

Jupyterlab provides a class `ClientSession`
([documentation](http://JupyterLab.github.io/JupyterLab/classes/_apputils_src_clientsession_.clientsession.html))
that manages a single kernel session. Here are the lines that we need to start
a kernel with it:

```
        this._session = new ClientSession({
            manager: manager.sessions,
            path,
            name: 'Tutorial',
        });
        this._session.initialize();
```

well, that's short, isn't it? We have already seen the `manager` class that is
provided directly by the main JupyterLab application. `path` is a link to the
path under which the console is opened (?).

With these lines, we can extend the panel widget from 7_signals_and_buttons to intialize a
kernel. In addition, we will initialize a `KernelModel` class in it and
overwrite the `dispose` and `onCloseRequest` methods of the `StackedPanel`
([documentation](phosphorjs.github.io/phosphor/api/widgets/classes/stackedpanel.html))
to free the kernel session resources if the panel is closed. The whole adapted
panel class looks like this:

```
export
class TutorialPanel extends StackedPanel {
    constructor(manager: ServiceManager.IManager) {
        super();
        this.addClass(PANEL_CLASS);
        this.id = 'TutorialPanel';
        this.title.label = 'Tutorial View'
        this.title.closable = true;

        let path = './console';

        this._session = new ClientSession({
            manager: manager.sessions,
            path,
            name: 'Tutorial',
        });

        this._model = new KernelModel(this._session);
        this._tutorial = new TutorialView(this._model);

        this.addWidget(this._tutorial);
        this._session.initialize();
    }

    dispose(): void {
        this._session.dispose();
        super.dispose();
    }

    protected onCloseRequest(msg: Message): void {
        super.onCloseRequest(msg);
        this.dispose();
    }

    get session(): IClientSession {
        return this._session;
    }

    private _model: KernelModel;
    private _session: ClientSession;
    private _tutorial: TutorialView;
}
```

#### Executing code and retrieving messages from a kernel (`model.ts`) ####

Once a kernel is initialized and ready, code can be executed on it through
the `ClientSession` class with the following snippet:

```
        this.future = this._session.kernel.requestExecute({ code });
```

Without getting too much into the details of what this `future` is, let's think
about it as an object that can receive some  messages from the kernel as an
answer on our execution request (see [jupyter messaging](http://jupyter-client.readthedocs.io/en/stable/messaging.html)).
One of these messages contains the data of the execution result. It is
published on a channel called `IOPub` and can be identified by the message
types `execute_result`, `display_data` and `update_display_data`.

Once such a message is received by the `future` object, it can trigger an
action.  In our case, we just store this message in `this._output` and then
emit a `stateChanged` signal. As we have explained above, our `KernelModel` is
a `VDomModel` that provides this  `stateChanged` signal that can be used by a
`VDomRendered`.  It is implemented as follows:

```
export
class KernelModel extends VDomModel {
    constructor(session: IClientSession) {
        super();
        this._session = session;
    }

    public execute(code: string) {
        this.future = this._session.kernel.requestExecute({ code });
    }

    private _onIOPub = (msg: KernelMessage.IIOPubMessage) => {
        let msgType = msg.header.msg_type;
        switch (msgType) {
            case 'execute_result':
            case 'display_data':
            case 'update_display_data':
                this._output = msg.content as nbformat.IOutput;
                console.log(this._output);
                this.stateChanged.emit(undefined);
            default:
                break;
        }
        return true
    }

    get output(): nbformat.IOutput {
        return this._output;
    }

    get future(): Kernel.IFuture {
        return this._future;
    }

    set future(value: Kernel.IFuture) {
        this._future = value;
        value.onIOPub = this._onIOPub;
    }

    private _output: nbformat.IOutput = null;
    private _future: Kernel.IFuture = null;
    private _session: IClientSession;
}
```

#### Connecting a View to the Kernel ####

The only remaining thing left is to connect a View to the Model. We have
already seen the `TutorialView` before. To trigger the `render` function of a
`VDomRendered` on a `stateChanged` signal, we just need to add our `VDomModel`
to `this.model` in the constructor.  We can then connect a button to
`this.model.execute` and a text field to `this.model.output` and our extension
is ready:

```
export
class KernelView extends VDomRenderer<any> {
    constructor(model: KernelModel) {
        super();
        this.id = `TutorialVDOM`
        this.model = model
    }

    protected render(): React.ReactElement<any>[] {
        console.log('render');
        const elements: React.ReactElement<any>[] = [];
        elements.push(
            <button key='header-thread'
            className="jp-tutorial-button"
            onClick={() => {this.model.execute('3+5')}}>
            Compute 3+5</button>,

            <span key='output field'>{JSON.stringify(this.model.output)}</span>
        );
        return elements;
    }
}
```

#### How does it look like ####

![Kernel Execution](images/kernel_extension.gif)

Well that's nice, the basics are clear, but what about this weird output
object? In the next extensions, we will explore how we can reuse some jupyter
components to make things look nicer...

[Click here for the final extension: 6_kernel_messages](6_kernel_messages)
