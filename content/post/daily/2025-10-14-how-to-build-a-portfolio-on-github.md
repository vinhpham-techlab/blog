---
title: How to build a Portfolio on Github Pages
subtitle: Using the Beautiful Hugo Theme
date: 2025-10-14
tags: [howto, github-pages]
---

A portfolio website is your digital resume, showcasing your projects, skills, and technical chops. GitHub Pages offers free hosting for static sites, and Hugo, a blazing-fast static site generator written in Go, lets you spin up a professional portfolio with minimal effort. This step-by-step guide will walk you through creating a portfolio using the Beautiful Hugo theme, which is perfect for displaying your coding projects, skills, and experience with a clean, techy vibe.

## Prerequisites

---

Before diving in, make sure you have:

- A GitHub account (github.com).
- Git installed locally.
- A code editor like VS Code or your preferred IDE.

## Installation

---

### Step 1: Install Hugo

Hugo is lightweight and cross-platform. Install it based on your OS:

- macOS:

```sh
brew install hugo
```

- Windows (with Chocolatey):

```sh 
choco install hugo -confirm
```

- Linux (Ubuntu/Debian):

```sh
sudo apt update
sudo apt install hugo
```

Other platforms? Check Hugo’s install guide.

Run hugo version to confirm it’s installed. You should see the version number.

### Step 2: Create a New Hugo Site

In your terminal, navigate to your desired project directory and create a new Hugo site:

```sh
hugo new site my-dev-portfolio
cd my-dev-portfolio
```

This sets up a bare-bones Hugo project structure.

### Step 3: Add the Beautiful Hugo Theme

The Beautiful Hugo theme is clean, responsive, and ideal for developers, with support for project pages, blog posts, and custom styling. Add it as a Git submodule:

```bash
git init
git submodule add https://github.com/halogenica/beautifulhugo.git themes/beautifulhugo
```

Copy the example site to get started:

```bash
cp -r themes/beautifulhugo/exampleSite/* .
```

This copies sample configs, content, and assets to your root directory.

### Step 4: Configure Your Site

Edit `hugo.toml` (Beautiful Hugo uses TOML by default) in the root directory. Key settings for a developer portfolio:

- `baseURL`: Set to `https://yourusername.github.io/` (replace yourusername).
- `title`: Your portfolio’s name, e.g., "Vinh Pham’s Data Engineer Portfolio".
- `theme`: Set to `github.com/halogenica/beautifulhugo`.
- Custom params: Add your social links, profile details, and enable/disable features.

Example `hugo.toml`:

```toml
baseurl = "https://vinhpham-techlab.github.io"
DefaultContentLanguage = "en"
title = "Data Engineer"
theme = "github.com/halogenica/beautifulhugo"
pygmentsStyle = "trac"
pygmentsUseClasses = true
pygmentsCodeFences = true
pygmentsCodefencesGuessSyntax = true
#pygmentsUseClassic = true
#pygmentOptions = "linenos=inline"

[pagination]
  pagerSize = 5

[Services]
  [Services.Disqus]
    # Shortname = "XXX"
  [Services.googleAnalytics]
    # id = "XXX"

[Params]
  homeTitle = "Vinh Pham"
  subtitle = "Data Engineer| Building Scalable Data Platform"
  mainSections = ["post"]
  logo = "img/avatar-icon.png"
  favicon = "img/favicon.ico"
  dateFormat = "January 2, 2006"
  commit = false
  rss = false
  comments = false
  readingTime = true
  wordCount = true
  useHLJS = true
  socialShare = false
  delayDisqus = false
  showRelatedPosts = true

[Params.author]
  name = "Vinh Pham"
  website = "https://vinhpham-techlab.github.io/blog"
  email = "phamtanvinh.me@gmail.com"
  github = "vinhpham-techlab"
  gitlab = "phamtanvinh.me"
  linkedin = "vinh-pham-444314148"

[taxonomies]
  tag = "tags"
  category = "categories"

[[menu.main]]
  name = "Blog"
  url = "/"
  weight = 1

[[menu.main]]
  identifier = "projects"
  name = "Projects"
  weight = 2

[[menu.main]]
  parent = "projects"
  name = "Data Platform"
  url = "post/data-platform"
  weight = 1

[[menu.main]]
  parent = "projects"
  name = "Infrastructure"
  url = "post/infrastructure"
  weight = 2

[[menu.main]]
  parent = "projects"
  name = "Databases"
  url = "post/databases"
  weight = 3

[[menu.main]]
  name = "About"
  url = "page/about/"
  weight = 4

[[menu.main]]
  name = "Tags"
  url = "tags"
  weight = 99
```

Save the file. Add your avatar and favicon to `static/img/`.

### Step 5: Test Locally

Preview your site:

```bash
hugo server
```

Open `http://localhost:1313/` in your browser. Edit files and see changes live.

### Step 6: Set Up GitHub Actions for Deployment

Create a GitHub Actions workflow to automate building and deploying your site.

1. In your project root, create a `.github/workflows/` directory:

```bash
mkdir -p .github/workflows
```

1. Add the `hugo.yml` file (see the artifact above) to `.github/workflows/hugo.yml`. Example `hugo.yml`:

```yaml
name: Build and deploy
on:
  push:
    branches:
      - main
  workflow_dispatch:
permissions:
  contents: read
  pages: write
  id-token: write
concurrency:
  group: pages
  cancel-in-progress: false
defaults:
  run:
    shell: bash
jobs:
  build:
    runs-on: ubuntu-latest
    env:
      DART_SASS_VERSION: 1.93.2
      GO_VERSION: 1.25.1
      HUGO_VERSION: 0.151.0
      NODE_VERSION: 22.18.0
      TZ: Asia/Ho_Chi_Minh
    steps:
      - name: Checkout
        uses: actions/checkout@v5
        with:
          submodules: recursive
          fetch-depth: 0
      - name: Setup Go
        uses: actions/setup-go@v5
        with:
          go-version: ${{ env.GO_VERSION }}
          cache: false
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
      - name: Setup Pages
        id: pages
        uses: actions/configure-pages@v5
      - name: Create directory for user-specific executable files
        run: |
          mkdir -p "${HOME}/.local"
      - name: Install Dart Sass
        run: |
          curl -sLJO "https://github.com/sass/dart-sass/releases/download/${DART_SASS_VERSION}/dart-sass-${DART_SASS_VERSION}-linux-x64.tar.gz"
          tar -C "${HOME}/.local" -xf "dart-sass-${DART_SASS_VERSION}-linux-x64.tar.gz"
          rm "dart-sass-${DART_SASS_VERSION}-linux-x64.tar.gz"
          echo "${HOME}/.local/dart-sass" >> "${GITHUB_PATH}"
      - name: Install Hugo
        run: |
          curl -sLJO "https://github.com/gohugoio/hugo/releases/download/v${HUGO_VERSION}/hugo_extended_${HUGO_VERSION}_linux-amd64.tar.gz"
          mkdir "${HOME}/.local/hugo"
          tar -C "${HOME}/.local/hugo" -xf "hugo_extended_${HUGO_VERSION}_linux-amd64.tar.gz"
          rm "hugo_extended_${HUGO_VERSION}_linux-amd64.tar.gz"
          echo "${HOME}/.local/hugo" >> "${GITHUB_PATH}"
      - name: Verify installations
        run: |
          echo "Dart Sass: $(sass --version)"
          echo "Go: $(go version)"
          echo "Hugo: $(hugo version)"
          echo "Node.js: $(node --version)"
      - name: Install Node.js dependencies
        run: |
          [[ -f package-lock.json || -f npm-shrinkwrap.json ]] && npm ci || true
      - name: Configure Git
        run: |
          git config core.quotepath false
      - name: Cache restore
        id: cache-restore
        uses: actions/cache/restore@v4
        with:
          path: ${{ runner.temp }}/hugo_cache
          key: hugo-${{ github.run_id }}
          restore-keys:
            hugo-
      - name: Build the site
        run: |
          hugo \
            --gc \
            --minify \
            --baseURL "${{ steps.pages.outputs.base_url }}/" \
            --cacheDir "${{ runner.temp }}/hugo_cache"
      - name: Cache save
        id: cache-save
        uses: actions/cache/save@v4
        with:
          path: ${{ runner.temp }}/hugo_cache
          key: ${{ steps.cache-restore.outputs.cache-primary-key }}
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: ./public
  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```


### Step 7: Deploy to GitHub Pages

1. Create a GitHub repository named `blog.github.io`.
2. Commit and push:

```bash
git add .
git commit -m "Initial portfolio commit"
git remote add origin https://github.com/yourusername/blog.github.io.git
git branch -M main
git push -u origin main
```

3. In your GitHub repo, go to `Settings > Pages`. Set the source to the `main` branch and `/(root)` folder. Save.

Your portfolio will be live at `https://yourusername.github.io/blog` shortly.

## Conclusion

You’ve built a clean, professional Data Engineer portfolio with Beautiful Hugo on GitHub Pages! It’s fast, free, and perfect for showcasing your data engineering skills. Update it regularly with new projects, and explore Hugo’s docs or the Beautiful Hugo repo for advanced tweaks like custom shortcodes or i18n support. Go make your mark in the data world!
