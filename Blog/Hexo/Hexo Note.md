# How to setup Hexo



在通过命令`hexo d`推送到服务器时，如果出现下方提示

```
E:\Hexo-Blog>hexo g -d
INFO  Validating config
INFO  Start processing
INFO  Files loaded in 119 ms
INFO  Generated: archives/2022/index.html
INFO  Generated: archives/index.html
INFO  Generated: archives/2022/11/index.html
INFO  Generated: index.html
INFO  Generated: fancybox/jquery.fancybox.min.css
INFO  Generated: js/script.js
INFO  Generated: css/fonts/FontAwesome.otf
INFO  Generated: fancybox/jquery.fancybox.min.js
INFO  Generated: js/jquery-3.4.1.min.js
INFO  Generated: css/style.css
INFO  Generated: css/fonts/fontawesome-webfont.eot
INFO  Generated: 2022/11/29/hello-world/index.html
INFO  Generated: css/fonts/fontawesome-webfont.ttf
INFO  Generated: css/fonts/fontawesome-webfont.woff
INFO  Generated: css/images/banner.jpg
INFO  Generated: css/fonts/fontawesome-webfont.woff2
INFO  Generated: css/fonts/fontawesome-webfont.svg
INFO  17 files generated in 300 ms
INFO  Deploying: git
INFO  Clearing .deploy_git folder...
INFO  Copying files from public folder...
INFO  Copying files from extend dirs...
[master 5cda14a] Site updated: 2022-11-29 23:03:19
 2 files changed, 2 insertions(+), 2 deletions(-)
Enumerating objects: 15, done.
Counting objects: 100% (15/15), done.
Delta compression using up to 8 threads
Compressing objects: 100% (4/4), done.
Writing objects: 100% (8/8), 692 bytes | 692.00 KiB/s, done.
Total 8 (delta 2), reused 0 (delta 0), pack-reused 0
remote: fatal: cannot create directory at '2022': Permission denied
To 43.135.74.44:/home/git/blog.git
   3c67782..5cda14a  HEAD -> master
branch 'master' set up to track 'git@43.135.74.44:/home/git/blog.git/master'.
INFO  Deploy done: git
```



倒数第五行`remote: fatal: cannot create directory at '2022': Permission denied`，表示目标文件夹权限不够，那么可以对目标文件夹修改权限，例如`chmode 777 目标文件夹`