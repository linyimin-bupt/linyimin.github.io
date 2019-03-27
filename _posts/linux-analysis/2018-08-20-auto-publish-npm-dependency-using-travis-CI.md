---
layout: post
title: 使用travis CI自动发布npm包
categories: [node, CI]
---

> 破山中贼易，破心中贼难

1. 初始化项目时指定程序入口和相关变异命令

   ```json
   {
     "name": "keypress-event",
     "version": "1.1.1",
     "description": "Listen to the key press",
     "main": "dist/index.js",
     "scripts": {
       "build": "tsc",
       "test": "echo \"no test specified\" && exit 0"
     },
     "repository": {
       "type": "git",
       "url": "git+ssh://git@github.com/linyimin-bupt/keypress-event.git"
     },
     "keywords": [
       "typescript",
       "iohook",
       "keypress",
       "keycode"
     ],
     "author": "Yimin Lin",
     "license": "MIT",
     "bugs": {
       "url": "https://github.com/linyimin-bupt/keypress-event/issues"
     },
     "homepage": "https://github.com/linyimin-bupt/keypress-event#readme",
     "dependencies": {
       "iohook": "^0.2.0",
       "typescript": "^3.0.1"
     }
   }
   ```

   上述文件中,name对应npm包名称,version对应npm包版本,main对应npm包程序的入口文件,dist对应编译文件指定的文件夹.

2. 编写.travis.yml文件

   ```yaml
   language : node_js
   node_js:
     - "10"
     - "9"
   install:
    - npm install
   os:
     - linux
   
   stages:
     - test
     - name: deploy
   
   jobs:
     include:
       - stage: test
         script:
           - node --version
           - npm --version
           - echo "Testing Started ..."
           - npm test
           - echo "Testing Finished."
   
       - stage: deploy
         script:
           - echo "NPM Deploying Started ..."
           - npm version
           - npm run build
           - echo "NPM Building Finished."
   
         deploy:
           provider: npm
           email: linyimin520812@gmail.com
           api_key: "$NPM_TOKEN"
           skip_cleanup: true
           on:
             all_branches: true
   ```

3. 使用github登录Travis

   [Travis官网](https://www.travis-ci.org),并使用github登录.

   ![travis-ci](http://linyimin-blog.oss-cn-beijing.aliyuncs.com/cjl0zdwxq0000zvkh64uuv0xh.png)

   选择指定的github仓库

   ![github仓库](http://linyimin-blog.oss-cn-beijing.aliyuncs.com/cjl0zh69e0001zvkhvah4d0lt.png)

   进入相关项目后,选择右上角的`More options`中的`Settings`,填写token变量`NPM_TOKEN`(在.travis.yml文件中使用)

   ![settings](http://linyimin-blog.oss-cn-beijing.aliyuncs.com/cjl106wl100030skh2mt7wy6b.png)

   填写token变量

   ![Token变量](http://linyimin-blog.oss-cn-beijing.aliyuncs.com/cjl107sb600040skhv0b4m4o8.png)

   npm token的变量在~/.npmrc文件中

   ```shell
   $ cat ~/.npmrc 
   ```

4. 更改npm版本

   npm包发布时, 版本必须要不一致,否则会出现以下错误

   ```
   npm ERR! publish Failed PUT 403
   npm ERR! code E403
   npm ERR! You cannot publish over the previously published versions: 1.0.5. : upload-image-to-oss
   ```

   所以,在push代码到仓库之前,需要更改项目的版本号.在commit所有更改代码之后,使用以下命令可以实现版本号的更改,并且自动完成commit

   ```shell
   $ npm version parch
   ```

5. push相关代码,自动完成npm发布

   push相关代码之后,点击commit,可以看到已经自动开始构建发布

   ![构建发布](http://linyimin-blog.oss-cn-beijing.aliyuncs.com/cjl0zxr8c00000skhxcuq8axi.png)

   点击小黄点(正在构建)或者绿勾(已经构建完成)进入travis的构建页面

   ![](http://linyimin-blog.oss-cn-beijing.aliyuncs.com/cjl100z6q00010skhfdfju8rs.png)

   点击相关任务,可以查看具体构建过程中的日志,如果出错,可以根据日志更改相关内容,再次提交进行构建发布

   ![](http://linyimin-blog.oss-cn-beijing.aliyuncs.com/cjl103mc500020skhj6dmool9.png)
