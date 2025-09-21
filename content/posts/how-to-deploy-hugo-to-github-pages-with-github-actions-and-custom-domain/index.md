+++
date = '2025-09-21T23:56:44+08:00'
draft = false
title = 'How to Deploy Hugo to Github Pages With Github Actions and Custom Domain'
+++

## Prerequisites

1. **Hugo:** The latest version installed. You can find it on the [Hugo Releases page](https://github.com/gohugoio/hugo/releases/latest).
2. **Git:** Installed and configured. [Download Git here](https://git-scm.com/downloads).
3. **GitHub Account:** A free account on [GitHub](https://github.com/).
4. **Custom Domain:** Optional but recommended. If you don't have one, you can purchase one from a domain registrar. Popular options include:
    * [Namecheap](https://www.namecheap.com/)
    * [GoDaddy](https://www.godaddy.com/)

## Process

### 1. GitHub Repository Setup

1. **Create a new repository** on GitHub. Name it `<username>.github.io`, replacing `<username>` with your actual GitHub username. For example, if your username is `octocat`, the repository name must be `octocat.github.io`.

2. **Clone the repository** to your local machine. You can rename the local folder for convenience.

    ```bash
    git clone https://github.com/<username>/<username>.github.io.git blog
    cd blog
    ```

### 2. Hugo Site Initialization

1. **Create a new Hugo site** inside the cloned repository. Use the `--force` flag because the directory already contains the `.git` folder.

    ```bash
    hugo new site . --force
    ```

2. **Choose and install a theme using Git Submodules**. There are many excellent themes available. Here are a few highly recommended options:
    * [**PaperMod**](https://github.com/adityatelange/hugo-PaperMod) — A fast, clean, and feature-rich theme designed for blogs, with built-in search, dark mode, and strong SEO support.
    * [**Stack**](https://github.com/CaiJimmy/hugo-theme-stack) — A modern card-style theme that emphasizes visual hierarchy and readability, ideal for personal blogs and writing platforms.
    * [**Blowfish**](https://github.com/nunocoracao/blowfish) — A powerful and highly customizable theme with Tailwind CSS, multilingual support, and built-in features for charts, math, and technical content.

    For this guide, we will install **Blowfish**. Execute the following command in your project's root directory:

    ```bash
    git submodule add -b main https://github.com/nunocoracao/blowfish.git themes/blowfish
    ```

    This command downloads the theme into the `themes/blowfish` directory and creates a `.gitmodules` file in your project to track it.

3. **Configure your site**. Open the `hugo.toml` file. You must set the `theme` parameter to match the theme's folder name. Also, set your site's `baseURL` and `title`.

    ```toml
    baseURL = "https://foobar.blog/"
    languageCode = "en-us"
    title = "My New Hugo Site"
    theme = "blowfish"
    ```

    *Note: Each theme has its own unique configuration options. Refer to the theme's documentation for details on how to customize it.*

4. **Create your first post**.

    ```bash
    hugo new content posts/my-first-post.md
    ```

    Open the newly created file at `content/posts/my-first-post.md` and ensure the front matter has `draft: false` so it will be published.

5. **Test your site locally** to make sure everything looks correct.

    ```bash
    hugo server
    ```

    You can view your site at `http://localhost:7081/`.

### 3. GitHub Actions Workflow for Automation

This workflow will automatically build your Hugo site and deploy it to the `gh-pages` branch whenever you push changes to your `main` branch.

1. **Create the workflow directory and file**.

    ```bash
    mkdir -p .github/workflows
    touch .github/workflows/deploy.yml
    ```

2. **Add the workflow configuration** to the `deploy.yml` file.

    ```yaml
    name: Deploy Hugo Site to GitHub Pages

    on:
      # Runs on pushes targeting the default branch
      push:
        branches:
          - main # Or whatever your default branch is

      # Allows you to run this workflow manually from the Actions tab
      workflow_dispatch:

    # A workflow run is made up of one or more jobs that can run sequentially or in parallel
    jobs:
      build-deploy:
        runs-on: ubuntu-latest
        steps:
          # Checks-out your repository and its submodules
          - name: Checkout
            uses: actions/checkout@v4
            with:
              submodules: true  # This is CRITICAL for fetching the theme
              fetch-depth: 0    # Fetch all history for .GitInfo and .Lastmod

          # Installs the latest version of Hugo
          - name: Setup Hugo
            uses: peaceiris/actions-hugo@v3
            with:
              hugo-version: "latest"
              extend: true

          # Builds the site and places the output in the 'public' directory
          - name: Build
            run: hugo --minify

          # Deploys the built site to the 'gh-pages' branch
          - name: Deploy
            uses: peaceiris/actions-gh-pages@v4
            with:
              github_token: ${{ secrets.GITHUB_TOKEN }}
              publish_branch: gh-pages # Set a branch name to use as GitHub Pages branch. The default is gh-pages.
              publish_dir: ./public
    ```

### 4. Configuring Your Custom Domain

To point your custom domain to your GitHub Pages site, you need to configure DNS records with your domain registrar.

1. **Log in to your domain registrar's website** (e.g., Namecheap, GoDaddy). Navigate to the DNS management or DNS settings panel for your domain.

2. **Choose the correct configuration** based on whether you are using an apex domain or a subdomain.

    **Option A: For an Apex Domain (e.g., `foobar.blog`)**

    You must create `A` (for IPv4) and `AAAA` (for IPv6) records. Create four of each, pointing to the GitHub Pages IP addresses. The host/name for all records should be `@` or left blank, which represents the root domain itself.

    * **A records (IPv4):**

        ```text
        185.199.108.153
        185.199.109.153
        185.199.110.153
        185.199.111.153
        ```

    * **AAAA records (IPv6):**

        ```text
        2606:50c0:8000::153
        2606:50c0:8001::153
        2606:50c0:8002::153
        2606:50c0:8003::153
        ```

    **Option B: For a Subdomain (e.g., `www.foobar.blog`)**

    Create a single `CNAME` record. Point the subdomain (the host/name will be `www`) to your default GitHub Pages URL.

    * **Type:** `CNAME`
    * **Host/Name:** `www`
    * **Value/Target:** `<username>.github.io`

    *Note: DNS changes can take a few minutes to several hours to propagate across the internet.*

### 5. Final Deployment and Repository Settings

1. **Commit and push all your local changes** to the `main` branch on GitHub. This will include your Hugo configuration, content, the theme submodule (`.gitmodules` file), and the workflow file.

    ```bash
    git add .
    git commit -m "Initial commit of Hugo site with theme submodule"
    git push origin main
    ```

2. **Configure GitHub Pages settings** in your repository.
    * Go to your repository on GitHub and click `Settings` > `Pages`.
    * Under "Build and deployment", set the `Source` to **"Deploy from a branch"**.
    * Under "Branch", select `gh-pages` and `/ (root)` for the folder. Click `Save`.

    Your GitHub Actions workflow will automatically create and push to the `gh-pages` branch. Once this branch is present, these settings will activate GitHub Pages.

3. **Verify your custom domain**.
    * In the same `Pages` settings, the "Custom domain" field should automatically populate with your domain after the workflow has run successfully.
    * Once GitHub verifies your domain (which may take a few moments after DNS has propagated), check the box for **"Enforce HTTPS"** to secure your site.

Your site is now live! Any future push to the `main` branch will automatically trigger the workflow, rebuild your site with the theme, and update the live version.
