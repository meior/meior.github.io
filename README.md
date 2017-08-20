# meior.github.io
My personal website, recording some program notes and experiences. Default branch `hexo` includes the source of my blog, a project that build with the Node.js module Hexo. The other branch `master` includes the static pages of my website in accordance with [GitHub Pages Instructions](https://help.github.com/articles/configuring-a-publishing-source-for-github-pages/).

[![node.js](https://img.shields.io/badge/node-4.6.2-brightgreen.svg)](https://github.com/nodejs/node)
[![hexo](https://img.shields.io/badge/hexo-3.2.0-brightgreen.svg)](https://github.com/hexojs/hexo)
[![hexo--theme--next](https://img.shields.io/badge/hexo--theme--next-0.5.0-brightgreen.svg)](https://github.com/iissnan/hexo-theme-next)

## Installation
Download the hexo branch(default):
```sh
$ git clone git@github.com:meior/meior.github.io.git
```

Install dependencies:
```sh
$ cd meior.github.io
$ npm install
```

Start the server at localhost for preview in time(unnecessary):
```sh
$ hexo s
```

## Post & Deploy
Create a new article:
```sh
$ hexo n articleName
```

After editing, generate the static pages of website:
```sh
$ hexo g
```

Deploy them on Github or other remote repositories:
```sh
$ hexo d
```

## Push updates
```sh
$ git add .
$ git commit -m "new post"
$ git push origin hexo
```

## Components
Powered by:
- [node.js](https://nodejs.org)
- [hexo](https://hexo.io)
- [hexo-theme-next](http://theme-next.iissnan.com)

Third-party services:
- [disqus](https://disqus.com)
- [leancloud](https://leancloud.cn)
