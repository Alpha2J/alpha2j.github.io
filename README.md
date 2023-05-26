# alpha2j.github.io
This repository serves as a platform to showcase my blog, experiences and articles. It supports GitHub Pages and serves as an online portfolio for my writing.

## how to use
1. install hexo `npm install -g hexo-cli`
2. start the server: `hexo server`
3. visit the website at: `http://localhost:4000/`

## how to upgrade
1. create _config.butterfly.yml in the project root dir, it will cover the `_config.yml` in `themes/butterfly/_config.yml`(but don't delete this one)
2. clone the newest butterfly code: `git clone -b master https://github.com/jerryc127/hexo-theme-butterfly.git themes/butterfly`
3. or once you have the `.git` dir of butterfly, then you can use `git pull` directly in themes dir

## best practice
1. don't add, remove or edit files in themes/butterfly, cause when you update the butterfly version, 
   all updates will be lost: https://butterfly.js.org/posts/4073eda/. if you want to make the custom config,
   what you need to do is to edit `/_config.butterfly.yml`, put your resources into `/source/assets` and then 
   reference to it with `/assets/xxxx`
2. steps to open music media: https://butterfly.js.org/posts/507c070f/#%E6%8F%92%E5%85%A5-Aplayer-html

## related docs
1. tutorial on building GitHub Pages using Hexo and Butterfly: https://juejin.cn/post/7095323643277738014
2. butterfly official github: https://github.com/jerryc127/hexo-theme-butterfly 
