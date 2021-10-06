Documentation project for the HLA Workshop. In addition it describes how to install pandoc and generate PDF or WORD docs from the Markdown files. 

# Project Documentation

* [HLA Workshop Installation Guide](docs/hla-workshop-installation-guide.md)
* [HLA Workshop Script](docs/hla-workshop-script.md)
* [HLA Workshop User Guide](docs/hla-workshop-user-guide.md)

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
    sudo apt-get install -y pandoc
    ```

# Pandoc Installation for Mac

* Install Pandoc using HOMEBREW

    ```
    brew install pandoc
    ```

# Convert Markdown to WORD

* Generate one DOCX file from a single Markdown file in a named temp folder using a template

```
pandoc "hla-workshop-installation-guide.md" --reference-doc=servicenow.docx --highlight-style tango --toc -o "/mnt/c/Temp/hla-workshop-installation-guide.docx"
```

* Generate multiple DOCX files from a set of Markdown files in a named temp folder

```
`find ./ -iname "*.md" -type f -exec sh -c 'pandoc "${0}" --highlight-style tango --toc -o "/mnt/c/Temp/${0%.md}.docx"' {} \;`
```

* Generate multiple DOCX files from a set of Markdown files in a named temp folder using a template

```
`find ./ -iname "*.md" -type f -exec sh -c 'pandoc "${0}" --reference-doc=servicenow.docx --highlight-style tango --toc -o "/mnt/c/Temp/${0%.md}.docx"' {} \;`
```