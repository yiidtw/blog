
# yiidtw's blog
- [link](https://yiidtw.github.io/blog)

## Overview
- this repo contains the source (.md) and hexo generated documents (.css .html .js)
- the following command are for myself to deploy blog writing environment on different machines
- it's for laptop or vm right now, and will consider to use docker or CI in the future

## Prerequisite
- npm or yarn, for example
```
$ brew install yarn
```

## Commands
```
$ cd ~
$ git clone git@github.com:yiidtw/blog.git
$ cd blog
$ git clone https://github.com/theme-next/hexo-theme-next themes/next
$ yarn install
# wait for a few second

$ ./node_modules/hexo/bin/hexo s
INFO  Start processing
INFO  Hexo is running at http://localhost:4000/blog/. Press Ctrl+C to stop.
# should be able to see the blog on the above-mentioned in browser

$ cp _config.yml.next.example ./themes/next/_config.yml
```

## Optional
- build alias in ~/.bashrc
```
alias hexo="$HOME/blog/node_modules/hexo/bin/hexo"	
```
