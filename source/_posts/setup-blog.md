---
title: Build up your personal Blog with Hexo and Github-Pages
date: 2022-07-14 23:29:51
tags: [Hexo, Github Pages]
categories: [Technology, Hexo]
---
If you are looking for a simple way to build your personal Blog, [Hexo](hexo.io) would be a good choice. 
Hexo has many advantages and the most attractive part for me is that Hexo can generate static website, which means you can easily host your blog on Github with [Github-Pages](pages.github.com).  

<!--more-->

## Let\`s start  
### Pre-Request  
- [Node.js](nodejs.org): >= 10.13(recommends 12.0 or higher)
- [Git](git-scm.com)

### Install Hexo
You can install Hexo with npm in one line:
```bash
#!/bin/bash
npm install -g hexo-cli
```

### Setup Your Blog
First initial your Blog with Hexo:
```bash
#!/bin/bash
hexo init blog
cd blog
npm install 
```
After that, your blog is ready to go, check it with:
```bash
#!/bin/bash
hexo server --debug
```
Modify site setting in ``` _config.yml ``` to customize your Blog, check [Configuration](https://hexo.io/docs/configuration) for more detail.

### Setup Next Theme(Optional)
Hexo provide hundreds of theme. I will use the Next theme for my Blog. It\`s easy to use Next in Hexo, install in one line:
```bash
#!/bin/bash
npm install hexo-theme-next@latest
```
Then open ```_config.yml```, find theme option and change its value to ```next```. And we are done, to customize your Next theme, check [Theme Setting](https://theme-next.js.org/docs/theme-settings/) for more detail.

### Deployment
We can host Blog on GitHub Pages with [GitHub Actions](https://docs.github.com/en/actions).

1. Create a repo named ***\<uesrname\>.github.io***
2. Create ```.github/workflows/pages.yml``` with the following contents(substituting ```16``` to the major version of Node.js that ```node --version``` display)
```yaml
name: Pages

on:
  push:
    branches:
      - master  # default branch

jobs:
  pages:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Use Node.js
        uses: actions/setup-node@v2
        with:
          node-version: '<your_node_version>'

      - name: Cache NPM dependencies
        uses: actions/cache@v2
        with:
          path: node_modules
          key: ${{ runner.OS }}-npm-cache
          restore-keys: |
              ${{ runner.OS }}-npm-cache
      
      - name: Install Dependencies
          run: npm install
      
      - name: Build
        run: npm run build

      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./public
```
3. push main branch to Github:
```bash
git push -u origin main
```
4. Once the deployment is finished, the generated pages can be found in the gh-pages branch of your repository.
5. In your GitHub repo’s setting, navigate to **Settings > Pages > Source**. Change the branch to gh-pages and save.

Now your Blog should be avaliable at ***\<username\>.github.io***
