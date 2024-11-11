# alpha2j.github.io
This repository serves as a platform to showcase my blog, experiences and articles. It supports GitHub Pages and serves as an online portfolio for my writing.

## How to Use
1. Install hexo: `npm install -g hexo-cli`
2. Start the server: `hexo server`
3. Visit the website at: `http://localhost:4000/`

## How to Upgrade
1. Create `_config.butterfly.yml` in the project root directory; it will override the `_config.yml` in `themes/butterfly/_config.yml`(but don't delete the latter)
2. Clone the latest Butterfly code: `git clone -b master https://github.com/jerryc127/hexo-theme-butterfly.git themes/butterfly`
3. Alternatively, if you already have the `.git` directory of Butterfly, you can use `git pull` directly in the themes directory

## Best Practice
1. Avoid adding, removing or editing files in `themes/butterfly` because updating the Butterfly version will result in the loss of all modifications.
   For custom configurations, edit `/_config.butterfly.yml`, place your resources in `/source/assets`, and reference them using `/assets/xxxx`
2. Follow these steps to incorporate music media: https://butterfly.js.org/posts/507c070f/#%E6%8F%92%E5%85%A5-Aplayer-html

## Related Docs
1. Tutorial on building GitHub Pages using Hexo and Butterfly: https://juejin.cn/post/7095323643277738014
2. Butterfly official GitHub repository: https://github.com/jerryc127/hexo-theme-butterfly 


## Notes
1. How to pin a post: https://github.com/hexojs/hexo-generator-index#usage