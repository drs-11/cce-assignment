# DokuWiki2Hugo Migration Plan

---

## Introduction

This plan was made as a part of [take-home-assignment](https://www.ccextractor.org/public:gsoc:takehome) for CCExtractor’s qualification task. It aims to migrate the content from a [DokuWiki](https://www.dokuwiki.org/dokuwiki) server/directory and publish it on Github Pages using Hugo as the static file generator.

## Plan Overview

![Plan](plan.png)

## Prerequisites

Softwares

- [Hugo](https://gohugo.io/getting-started/installing/)

- [Git](https://git-scm.com/downloads)

- [Python](https://www.python.org/downloads/)

Others

- Github account

## Procedure

### Getting Content

The main contents that are of concern to us are text files(containing post text) and images. These are located in ```data/pages``` and ```data/media``` respectively in DokuWiki’s root directory. Former versions of these files are located in ```data/attic``` and ```data/media_attic```.

### Converting Content to md

There are very few DokuWiki to Markdown converters available. Of them [dokuwiki-to-hugo](https://github.com/wgroeneveld/dokuwiki-to-hugo) and [alldocs](https://github.com/ueberdosis/alldocs.app) are the most convenient. A rundown of both the converters:

**dokuwiki-to-hugo**:

- written in python

- can generate tags

- can convert dokuwiki metadata

- can't convert tables

**alldocs**:

- written in php

- also exists as an online service

- can convert tables

- converted output is cleanly formatted

- no API exists

- can't convert dokuwiki metadata

Though dokuwiki-to-hugo can be convenient for writing a quick script to convert the files it can't convert tables and has some faults in conversion. 

Alldocs supports most of the syntax conversion and also outputs a clearly formatted(this will be a huge help while rectifying errors) converted file. The downside is no API exists for it so either someone with php skills has to hack it and make one or a script will be needed to upload and download files from the [online service](https://alldocs.app/) via POST and GET method.

### Rectifying conversion errors

Testing the above mentioned converters still gave some faulty conversion. These faults will have to be fixed manually. These include but are not limited to:

- codeblocks and lists mixup

- poor link conversion

### Initialising Hugo Site

Create a Hugo site

```bash
hugo new site <site-name>
```

Select any one of the [themes](https://themes.gohugo.io/).

```bash
cd <site-name>
git init
git submodule add <theme-github-link> themes/<theme-name>
```

Add theme to site config

```bash
echo 'theme = "<theme-name>"' >> config.toml
```

Transfer converted files retaining original directory structure to ```content/``` in site's root directory. Images need to be trasnferred to ```static/```. Image links may need to be updated accordingly.

Fire up Hugo server

```bash
hugo server
```

Navigate to the site at [http://localhost:1313](http://localhost:1313) to have a look.

### Configurations and Customizations

- Add [metadata](https://gohugo.io/content-management/front-matter/) as per needed to the content files.

- *baseURL* and *title* in ```config.toml``` file can be edited wih the desired values.

- The homepage can be set by transfering the old homepage's content to ```content/_index.md``` in site's directory. [Further](https://timhilliard.com/blog/static-home-page-in-hugo/) steps:

```bash
cd themes/my_theme/layouts
rm index.html
cp _default/single.html index.html
```

- A common theme and layout can be applied to a set of pages by creating [sections](https://gohugo.io/content-management/sections/)

- If a custom domain name is available, it can be used by creating a `static/CNAME` file with the custom domain name as the only content.

### Publishing on Github Pages

Assuming the above steps were followed, the site's directory will already be a git repository. The next step will be to add a [Github Action](https://github.com/peaceiris/actions-gh-pages) workflow to generate the site pages from the source code whenever the repo is pushed or a PR is merged.

Create a ```gh-pages.yml``` file in ```.github/workflows``` with the following text:

```yaml
name: github pages  # name of the workfllow

on:
  push:
    branches:
      - master  # Set a branch name to trigger deployment

jobs:
  deploy:
    runs-on: ubuntu-18.04
    steps:
      - uses: actions/checkout@v2
        with:
          submodules: true  # Fetch Hugo themes (true OR recursive)
          fetch-depth: 0    # Fetch all history for .GitInfo and .Lastmod

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v2
        with:
          hugo-version: 'latest'

      - name: Build
        run: hugo --minify

      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./public
          publish_branch: gh-pages  # gh-pages will contain the generated files
```

A Github account will be needed to use the Github Pages.

Create an empty repository on Github with name ```<username>.github.io``` or ```<organisation-name>.github.io```.

Commit changes in the local repository and push it to the remote repository with the following steps:

```bash
git add .
git commit -m "commit message"
git remote add origin <remote-repo-link>
git push origin master
```

[Publishing source](https://docs.github.com/en/github/working-with-github-pages/configuring-a-publishing-source-for-your-github-pages-site) in the Github repository settings may need to be set to ```gh-pages``` branch, if not set automatically.

Voila! Now Github Actions will generate the static site files and Github Pages will host the files everytime a push is made or a PR is merged.
