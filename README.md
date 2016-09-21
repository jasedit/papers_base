[![Latest Version](https://img.shields.io/pypi/v/scriptorium.svg)](https://pypi.python.org/pypi/scriptorium) [![Downloads](https://img.shields.io/pypi/dm/scriptorium.svg)](https://pypi.python.org/pypi/scriptorium) [![License](https://img.shields.io/pypi/l/scriptorium.svg)](https://pypi.python.org/pypi/scriptorium) [![Code Health](https://landscape.io/github/jasedit/scriptorium/master/landscape.svg?style=flat)](https://landscape.io/github/jasedit/scriptorium/master)

# Scriptorium

Framework for easily using MultiMarkdown and LaTeX based system to write academic papers, especially those with shared templates for organization. This system is designed with several important design guidelines and observations:

1. Many academic publications are collaborative, continuously evolving works.
2. Many academic publications provide LaTeX templates to meet the formatting guidelines for submission.
3. Research groups often use common templates for documents, such as submitting to a particular conference repeatedly over the years, or thesis/dissertation requirements at a particular university.
4. LaTeX can have a high learning curve for people unfamiliar with it's syntax and style. Knowing LaTeX can provide a powerful array of tools and commands for finely tuning the resulting document.
5. Academic publications can have a mix of visibility/availability between groups. Some papers may be private to an individual, a group, some mix of groups, or completely publicly available. The tools for building papers do not share this variability of privacy.

In order to respond to these observations, this framework aims to provide:

1. Version control for documents, allowing concurrent collaboration, full history tracking, branching for edits, etc.
2. User friendly in syntax and operation, minimizing the requirement to learn gory internal details.
3. Optionally exposing internal details, providing the ability to directly access lower level tools for power users to fine tune results.
4. A base for groups to use to distribute/share a common working base, with flexibility for individuals to choose documents which are present on their system.
5. Fine grained privacy controls for controlling access to documents and intellectual property.
6. Plain text as the basis for documents, ensuring that a wide variety of workflows and tools can be used to edit documents.

# Installation

## General Instructions

In general, the following instructions will install the Scriptorium system. For Windows, please be sure to follow the Windows directions to handle the peculiarities of Windows and the command line.

1. Install external dependencies:
    1. [git](https://git-scm.com/)
    1. [MultiMarkdown 5](https://github.com/fletcher/MultiMarkdown-5)
    2. [LaTeX](http://www.latex-project.org/)
    3. [Python](http://python.org/)
2. Execute `pip install scriptorium`

## Windows Setup

These instructions provide a method to configure Scriptorium to work on Windows with a minimum of fuss. There are many other ways to configure this system, and cleaner instructions would be appreciated in a pull request

1. Install the [GitHub Desktop Client](https://desktop.github.com/), and follow the directions to configure it with your GitHub account.
2. Install [MultiMarkdown 5](https://github.com/fletcher/MultiMarkdown-5/releases) using the graphical installer
3. Install [MikTex](http://miktex.org/)
4. Install [Python](https://www.python.org/downloads/), either 2.7 or 3.5.
5. Modify the Environment Variables to add Python to the `PATH` variable. Based on the helpful instructions [here](http://stackoverflow.com/questions/23400030/windows-7-add-path):
    1. Right click on "My Computer" and select Properties
    2. In the System Properties window, click on Advanced
    3. Click on the "Environment Variables" button
    4. Select the `PATH` variable, and add Python. The default values would be `C:\Python27` and `C:\Python27\Scripts` for Python 2.7, or `C:\Python35` and `C:\Python35\Scripts` for Python 3.5, although this would changed if the installation directories were changed in previous steps.
5. Open the "Git Shell" installed by GitHub, and verify that `python.exe` and `pip` are recognized commands.
7. Follow the general installation instructions listed above.

# Tutorial

Scriptorium can be invoked directly from the command line using the name `scriptorium`.

Check that all external dependencies are installed and detected correctly, by veryifying the following command returns nothing:
```
scriptorium doctor
```

Install some example [templates](https://github.com/jasedit/simple_templates):
```
scriptorium template -i https://github.com/jasedit/simple_templates
```

To list which templates are currently available in scriptorium:
```
scriptorium template -l
```

To create a new paper using the report template previously installed:
```
scriptorium new example_report -t report -c author "John Doe" -c title "My Example Report"
```

Adding example content using the command:
```
echo "# Introduction

This is an introductory section." >> example_report/paper.mmd
```

The PDF of the report can be built using:
```
scriptorium build example_report
```
or, if inside `example_report`:
```
scriptorium build
```

# Implementation

The implementation here provides relativey little in the way of actual code or new tools. Instead, the focus is on pulling together existing tools and organizing them in such a way to provide the desired functionality. [MultiMarkdown](http://fletcherpenney.net/multimarkdown/) is used for the user friendly markup language. MultiMarkdown extends the [Markdown](http://daringfireball.net/projects/markdown/) to provide several additional commands for citations, tables, and footnotes. Usefully, MultiMarkdown includes an extension for [LaTeX support](https://github.com/fletcher/peg-multimarkdown-latex-support). This is used to provide a method for pulling in LaTeX templates for various academic publications. Templates are stored in the templates directory of this repository - simply download/clone/store templates there. The default system is configured to recursively search this directory for any templates, so paths are not required (although unique names for templates likely are.) Similarly, papers are simply directories stored in the papers subdirectory. These are highly encourage to be repositories with version control, but the system doesn't enforce this on folders.

## Papers Organization

Since papers in development are generally not open-source, this framework pushes papers into other folders inside the papers directory. This way, any lab can keep their actual academic text in a private repository, while the templates and framework can be left open-source. Submodules are not used so that individual users can share a papers_base repository, while not necessarily sharing the same papers. Generally, version control repositories don't handle binary files (e.g. images) particularly well, so it is recommended to break up papers into more repositories to require less overhead storing history, as well as providing higher granularity in sharing papers.

### Paper Metadata

In order to integrate the template system, the MultiMarkdown metadata header requires a few important statements. Consider an example header, as shown below.

```
Base Header Level: 3
latex author: Author
Title: Paper Title
myemail: author@place.com
latex input: template/setup.tex
latex footer: template/footer.ex
```

The Base Header Level is important for configuring MultiMarkdown to avoid section levels which may not be supported by the template being used. Level 1 is the `\chapter` command in LaTeX, which is often unused in conference papers. The `latex author` key bypasses input sanitization, allowing LaTeX specific commands in the authors title. `myemail` is the author's e-mail address. The input and footer are used to read the template preamble and footer. Some templates will also read a `metadata.tex` file, which provides a direct LaTeX file for specifying metadata when LaTeX specific commands are necessary.

## Template Organization

A template defines the latex setup defining how a paper is going to be laid out, which packages it will use, etc. For reference, consider templates in the [simple templates](https://github.com/jasedit/simple_templates) repository. A template is made in a few steps:

1. A folder inside the templates directory. The name of this folder is what is used to reference the template in a MultiMarkdown paper, by LaTeX's recursive subdirectory search.
2. A LaTeX file named `setup.tex` inside this folder, which contains the template preamble. The preamble should include everything at the start of the document before the content, through the `\begin{document}` statement. More may be included in this preamble, such as seen in the IEEEtran example in the simple templates.
3. A LaTeX file named `footer.tex` inside this folder, which contains any LaTeX which should be appended to the end of the file. This often includes the bibliography commands. The IEEEtran `footer.tex` file is a good example of such a footer.
4. An optional `frontmatter.mmd` and/or `metadata.tex` file, which contains a default values, minus the input and footer values. Any field can have a value starting with a dollar sign, and capital alphanumeric and `_`, `.`, or `-`, which are replaceable during the `new` command.
