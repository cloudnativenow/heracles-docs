Documentation project for the Predictive AIOps Workshop. In addition it describes how to install [Pandoc](https://pandoc.org/MANUAL.html) and generate PDF or WORD docs from the Markdown files.

# Project Documentation

* [Predictive AIOps Workshop Introduction](docs/workshop-introduction.md)
* [Predictive AIOps Workshop Installation Guide](docs/workshop-installation-guide.md)
* [Predictive AIOps Workshop User Guide](docs/workshop-user-guide.md)

# Prerequisites

* Git 
* Bash (Mac or WSL)

# Clone Project

1. Clone Git Project

    ```
    $ git clone git@github.com:pangealab/heracle-docs.git
    $ cd heracles-docs/docs
    ```

# Pandoc Installation for WSL

* Install Pandoc using APT GET

    ```
    sudo apt-get update
    sudo apt-get install -y pandoc # Core Pandoc
    sudo apt-get install -y texlive-latex-extra # For PDF features
    ```

# Pandoc Installation for Mac

* Install Pandoc using HOMEBREW

    ```
    sudo apt-get update
    brew install pandoc
    brew install texlive-latex-extra
    ```

# Convert Markdown to WORD

* Generate one DOCX file from a single Markdown file in a named temp folder using a template

    ```
    pandoc "workshop-installation-guide.md" --reference-doc=servicenow.docx --highlight-style tango --toc -o "/mnt/c/Temp/workshop-installation-guide.docx"
    ```

* Generate multiple DOCX files from a set of Markdown files in a named temp folder

    ```
    `find ./ -iname "*.md" -type f -exec sh -c 'pandoc "${0}" --highlight-style tango --toc -o "/mnt/c/Temp/${0%.md}.docx"' {} \;`
    ```

* Generate multiple DOCX files from a set of Markdown files in a named temp folder using a template

    ```
    `find ./ -iname "*.md" -type f -exec sh -c 'pandoc "${0}" --reference-doc=servicenow.docx --highlight-style tango --toc -o "/mnt/c/Temp/${0%.md}.docx"' {} \;`
    ```
