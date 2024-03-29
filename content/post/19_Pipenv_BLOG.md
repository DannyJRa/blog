---
title: Pipenv 
author: DannyJRa
date: '2019-01-25'
slug: 19_pipenv3
categories:
  - Python
tags:
  - Engineering
hidden: false
image: "img/19_pipenv_BLOG.gif"
share: false
output:
  html_document:
    keep_md: yes
    includes:
      #before_body: header.html
      #after_body: footer.html
    theme: cerulean
    highlight: tango
    code_folding: show
    toc: yes
    toc_float: yes
  pdf_document:
    number_sections: yes
geometry: margin = 1.2in
fontsize: 10pt
always_allow_html: yes

---










<a href="https://github.com/DannyJRa/DannyJRa.github.io/tree/master/19_Pipenv/" target="_blank"><img src="/img/forkme_right_orange_ff7600.svg" style="position:absolute;top:1;right:0;" alt="Fork me on GitHub"></a>






# Pipenv tool for Python package management

## Why using pipenv

Instead of needing the pip library for package installation, plus a library for creating a virtual environment,plus a library for managing virtual environments, plus all the commands associated with those libraries, one can just just Pipenv.

Pipenv ships with package management and virtual environment support, so you can use one tool to install, uninstall, track, and document your dependencies and to create, use, and organize your virtual environments. When you start a project with it, Pipenv will automatically create a virtual environment for that project if you aren't already using one.
Pipenv accomplishes this dependency management by abandoning the requirements.txt norm and trading it for a new document called a Pipfile. When you install a library with Pipenv, a Pipfile for your project is automatically updated with the details of that installation, including version information and possibly the Git repository location, file path, and other information.

Furthermore, Pipenv wants to make it easier to manage complex interdependencies. Your app might depend on a specific version of a library, and that library might depend on a specific version of another library, and it's just dependencies and turtles all the way down. When two libraries your app uses have conflicting dependencies, your life can become hard. Pipenv wants to ease that pain by keeping track of a tree of your app's interdependencies in a file called Pipfile.lock. Pipfile.lock also verifies that the right versions of dependencies are used in production.

## Install pipenv

Using Pipenv, which gives you Pipfile, lets you avoid these problems by managing dependencies for different environments for you. This command will install the main project dependencies (install with user option, otherwise access writes issues)

```bash
pip3 install --user pipenv
```
Installing a package (e.g. django) is as easy as:

```bash
pipenv install django
```
Have a look at all installed packages

```bash
pipenv graph
```

### Further usefull commands

Where is the local folder of the packages

```
pipenv --venv
```

Project home


```
pipenv --whre
```

Install from requirements.txt

```
pipenv install -r requirements.txt
```
Or create requirements.txt

```
pipenv lock -r > requirements.txt
```


Set environment variables

```
export PIPENV_DEFAULT_PYTHON_VERSION="--three"
```


Execute shell command within environment

```
pipenv run test.py
```

***
Original post on Github Pages: <a href="https://dannyjra.github.io/19_Pipenv/19_Pipenv_BLOG.html" target="_blank">https://dannyjra.github.io/19_Pipenv/19_Pipenv_BLOG.html</a>
