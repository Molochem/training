---
myst:
  html_meta:
    "description": ""
    "property=og:description": ""
    "property=og:title": ""
    "keywords": ""
---

(eggs1-label)=

# Write Your Own Python Add-On to Customize Plone

````{sidebar} Plone Backend Chapter
```{figure} _static/plone-training-logo-for-backend.svg
:alt: Plone backend
:class: logo
```

Get the code! ({doc}`More info <code>`)

Code for the beginning of this chapter:

```
nothing here...
```

Code for the end of this chapter:

```shell
git checkout initial
```
````

(eggs1-create-label)=

In this part you will:

- Create a custom Python package {py:mod}`ploneconf.site`
- Modify the buildout to install {py:mod}`ploneconf.site`

Topics covered:

- {py:mod}`plonecli`, a Plone CLI for creating Plone packages
- the structure of python packages

## Creating the package

```{warning}
You can skip this part since you already added {py:mod}`ploneconf.site` as a source-checkout when running buildout.
And in {ref}`features-create-plonesite-label` you installed that package when creating your site.

You can still follow this chapter to learn hwo to create your own packages.
```

Your own code has to be organized as a [Python package](https://docs.python.org/3/tutorial/modules.html#packages). A python package is a directory that follows certain conventions to hold python modules.

We are going to use [plonecli](https://pypi.org/project/plonecli) to create a skeleton package. You only need to fill in some blanks.

```{note}
{py:mod}`plonecli` uses {py:mod}`bobtemplates.plone` that offers several Plone-specific templates for {py:mod}`mr.bob`, a project template builder similar to {py:mod}`cookiecutter`.
```

Install plonecli:

```shell
$ pip install plonecli
$ pip install bobtemplates.plone==6.0b15
```

Then create the addon:

```shell
plonecli create addon sources/ploneconf.site
```

The new add-on will be created in the {file}`sources` directory.

You have to answer some questions about the add-on. Press {kbd}`Enter` (i.e. choosing the default value) for most questions except where indicated (enter your GitHub username if you have one and do not initialize a GIT repository):

```
--> Author's name [Philip Bauer]:

--> Author's email [bauer@starzel.de]:

--> Author's GitHub username: pbauer

--> Package description [An add-on for Plone]:

--> Do you want me to initialize a GIT repository in your new package? (y/n) [y]: n

--> Plone version [6.0.0]:

--> Python version for virtualenv [python3]:

--> Do you want me to activate VS Code support? (y/n) [y]: n

git init is disabled!
Generated file structure at /Users/pbauer/workspace/training/backend/sources/ploneconf.site
```

```{only} not presentation
If this is your first python package, this is a very special moment.

You generated a package with a lot files. It might look like too much boilerplate but all files in this package serve a clear purpose and it will take some time to learn about the meaning of each of them.
```

## Volto Add-ons

The package that will hold your own code for volto was already created when you installed the frontend with `create-volto-app`.
The folder {file}`frontend/` that you created in the chapter {ref}`instructions-install-frontend-label` not only holds the default volto frontend but also gives you the option to extend and customize the frontend.

## Eggs

When a python package is production-ready you can choose to distribute it as an egg over the python package index, [pypi](https://pypi.org). This allows everyone to install and use your package without having to download the code from github. The over 250 python packages that are used by your current Plone instance are also distributed as eggs.

(eggs1-inspect-label)=

## Inspecting the new package

In {file}`src` there is now a new folder {file}`ploneconf.site` and in there is the new package. Let's have a look at some of the files:

{file}`buildout.cfg`, {file}`.travis.yml`, {file}`.coveragerc`, {file}`requirements.txt`, {file}`MANIFEST.in`, {file}`.gitignores`, {file}`.gitattributes`,

: You can ignore these files for now. They are here to create a buildout only for this package to make distributing and testing it easier.

{file}`README.rst`, {file}`CHANGES.rst`, {file}`CONTRIBUTORS.rst`, {file}`DEVELOP.rst`, {file}`docs/`

: The documentation of your package goes in here.

{file}`setup.py`

: This file configures the package, its name, dependencies and some metadata like the author's name and email address. The dependencies listed here are automatically downloaded when installing that package.

{file}`src/ploneconf/site/`

: The python code of your package itself lives inside a special folder structure.
That seems confusing but is necessary for good testability.
Our package contains a [namespace package](https://peps.python.org/pep-0420/) called _ploneconf.site_ and because of this there is a folder {file}`ploneconf` with a {file}`__init__.py` and in there another folder {file}`site` and in there finally is our code.
Your code is in {file}`backend/sources/ploneconf.site/src/ploneconf/site/{real code}`

```{note}
Unless discussing the installation or the frontend we will from now on silently omit these folders when describing files and assume that {file}`backend/src/ploneconf.site/src/ploneconf/site/` is the root of our package!
```

{file}`configure.zcml` ({file}`src/ploneconf/site/configure.zcml`)

: The phone book of the distribution. By reading it you can find out which functionality is registered using the component architecture. There are more registrations in other zcml-files in this add-on (e.g. {file}`browser/configure.zcml`, {file}`upgrades.zcml` and {file}`permissions.zcml`) that are included in your main {file}`configure.zcml`

{file}`setuphandlers.py` ({file}`src/ploneconf/site/setuphandlers.py`)

: This holds code that is automatically run when installing and uninstalling our add-on.

{file}`interfaces.py` ({file}`src/ploneconf/site/interfaces.py`)

: Here a browserlayer is defined in a straightforward python class. We will need it later.

{file}`testing.py`

: This holds the setup for running tests.

{file}`tests/`

: This holds the tests.

{file}`browser/`

: This directory is a python package (because it has a {file}`__init__.py`) and will by convention hold most things that are visible in the browser.

{file}`browser/configure.zcml`

: The phonebook of the browser package. Here views, resources and overrides are registered.

{file}`browser/overrides/`

: This folder is here to allow overriding existing default Plone templates.

{file}`browser/static/`

: A directory that holds static resources (images/css/js). The files in here will be accessible through URLs like `++resource++ploneconf.site/myawesome.css`

{file}`locales/`

: This directory can hold translations of text used in the package to allow for multiple languages of your user-interface.

{file}`profiles/default/`

: This folder contains the GenericSetup profile. During the training we will put some XML files here that hold configuration for the site.

{file}`profiles/default/metadata.xml`

: Version number and dependencies that are auto-installed when installing our add-on.

% profiles/uninstall/
% This folder holds another GenericSetup profile. The steps in here are executed on uninstalling.

```{note}
This seems like a lot of complicated boilerplate. In fact a Plone-package can be much smaller and simpler. See <https://github.com/starzel/minimal> for a minimal example.
But as stated above the stucture of the package and every part of it serves a well-defined purpose.

When you are working on large projects you will appreciate the best-practices laid down in this package.
```

(eggs1-include-label)=

## Including the package in Plone

Before we can use our new package we have to tell Plone about it. Look at {file}`backend/requirements.txt` and see how `ploneconf.site` can be included in `auto-checkout`:

TODO:
Update instructions

After restarting Plone with {command}`make start` the new add-on {py:mod}`ploneconf.site` is available for install.

We will not install it now since we did not add any of our own code or configuration yet. Let's do that next.

## Exercises

1. Create a new package called {py:mod}`collective.behavior.myfeature`. Inspect the directory structure of this package. Delete it after you are done. Many packages that are part of Plone and some add-ons use a _nested namespace_ such as {py:mod}`plone.app.contenttypes`.
2. Open <https://github.com/plone/bobtemplates.plone> and read about the templates and subtemplates it provides.

## Summary

- You created the package {py:mod}`ploneconf.site` to hold your code.
- You added the new package to your setup so that Plone can use it.
- In one of the next chapters we will also create a add-on for Volto, the React frontend.
