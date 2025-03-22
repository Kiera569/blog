---
author: Kiera
pubDatetime: 2024-04-02T13:05:56.066Z
modDatetime: 2024-04-02
title: docker 入门
slug: docker
featured: false
draft: false
tags:
  - docker
description: docker入门之部署前端项目，以create-react-app为例
---

1.github上先下载create-react-app的源码

```html
git clone https://github.com/facebook/create-react-app.git
```

执行ls查看一下并cd 进入该项目.

2.在项目根目录下创建Dockerfile文件,并编写以下内容

```html
vi Dockerfile // 创建文件 编写内容： # node版本号 FROM node:15-alpine # 工作目录
WORKDIR /create-react-app # 添加所有文件到create-react-app目录 ADD .
/create-react-app # 执行命令 RUN npm install && npm run build && npm install -g
http-server # 暴露端口号 EXPOSE 3000 # 容器启动之后, 执行http-server
注：./build是指打包后的文件 CMD http-server ./build -p 3000
```

3.创建image文件

```html
docker image build --progress plain -t create-react-app-demo .
```

执行成功后, 查看一下image文件
![在这里插入图片描述](https://img-blog.csdnimg.cn/97cd623f287a427c9ae352c961448179.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA6LGG6Iq95LiN5ZCD6LGG,size_20,color_FFFFFF,t_70,g_se,x_16) 4.从image文件生成容器：

```html
// 将Dockerfile中暴露出来的3000端口映射到本机的1234端口 docker container run -p
1234:3000 create-react-app-demo
```

然后在本地执行http://localhost:1234/， 成功～
![在这里插入图片描述](https://img-blog.csdnimg.cn/42b0355ad3c54557a11826f6f0001ff5.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA6LGG6Iq95LiN5ZCD6LGG,size_20,color_FFFFFF,t_70,g_se,x_16)

是不是小有成就感？ 那你就错了，正常工作里不会这么用哈哈哈哈。

看看运行的create-react-app的镜像多大：docker images
![在这里插入图片描述](https://img-blog.csdnimg.cn/36c9ca21ba8e48f887190e5deccf9f54.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA6LGG6Iq95LiN5ZCD6LGG,size_20,color_FFFFFF,t_70,g_se,x_16)
吓人！一个g.....一般的前端项目打包过后的代码也就几十兆，然后你一个docker就要一个G？加载起来时间也很漫长......运维哭晕在厕所，还是看看怎么优化一下吧

利用镜像缓存
1.package.json算是相对稳定的吧，在没有新的安装包需要下载的时候，这个文件是无需重新构建的吧？！ 好像有点道理，盘他！
vim Dockerfile重新修改一下配置信息

```html
# node版本号 FROM node:15-alpine # 工作目录 WORKDIR /create-react-app #
添加所有文件到create-react-app目录 ADD package.json package-lock.json
/create-react-app ADD . /create-react-app # 执行命令 RUN npm install && npm run
build && npm install -g http-server # 暴露端口号 EXPOSE 3000 # 容器启动之后,
执行http-server 注：./public是指打包后的文件 CMD http-server ./build -p 3000
```

每次修改完配置文件之后都需要重新构建镜像：

```html
docker image build --progress plain -t create-react-app-demo .
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/8450e2d1b3e343b2b4202a8b3cf58b67.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA6LGG6Iq95LiN5ZCD6LGG,size_20,color_FFFFFF,t_70,g_se,x_16)
接下来利用ci代替npm install.ci会比npm install的执行速度快，而且当package.json跟package-lock.json不匹配时,ci会报异常

```html
# node版本号 FROM node:15-alpine # 工作目录 WORKDIR /create-react-app #
添加所有文件到create-react-app目录 ADD package.json package-lock.json
/create-react-app ADD . /create-react-app # 执行命令 RUN npm ci RUN npm run
build && npm install -g http-server # 暴露端口号 EXPOSE 3000 # 容器启动之后,
执行http-server 注：./public是指打包后的文件 CMD http-server ./build -p 3000
```

再重新构建一次镜像
![在这里插入图片描述](https://img-blog.csdnimg.cn/2d8acd89f3704b599fb14db2d3a56669.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA6LGG6Iq95LiN5ZCD6LGG,size_20,color_FFFFFF,t_70,g_se,x_16)
嗯。。。构建时间是优化了那么一点，接下来看看1G的体积吧 吓人！！！！

体积那么大，基本都归功于我们npm install的时候生成的node_module了吧。那把node_module搞掉，应该会小很多，试一波呗
我们只需要静态生成的静态资源，那就只提取编译后的文件,而且部署项目推荐用nginx,所以这里也需要把http-server的镜像干掉 ，改成用ngnix的。

那 再改改dockerfile配置文件。vim dockerfile

```html
FROM node:15-alpine as builder WORKDIR /create-react-app ADD . /create-react-app
ADD package.json package-lock.json /create-react-app RUN npm ci RUN npm run
build FROM nginx:alpine COPY --from=builder /create-react-app/build
/usr/share/nginx/html
```

ngnix/html是部署项目时需要添加编译文件的路径，这里需要先抓取一下nginx的image：
`docker pull nginx:alpine`

拉下来过后我们进入该容器看一下执行成功了没`docker run -it --rm nginx:alpine sh`
![在这里插入图片描述](https://img-blog.csdnimg.cn/d00876ab13ec4a119bd89faa12e3a6d8.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA6LGG6Iq95LiN5ZCD6LGG,size_20,color_FFFFFF,t_70,g_se,x_16)
成功之后我们再重新构建一下image  
`docker image build -t create-react-app-demo .`

然后再执行docker images ，哈哈哈哈小成就感又来了～～～
![在这里插入图片描述](https://img-blog.csdnimg.cn/ccb1e82ad6084c50a5971b0d8f77c551.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA6LGG6Iq95LiN5ZCD6LGG,size_20,color_FFFFFF,t_70,g_se,x_16)
总结：
镜像中使用基于 alpine 的镜像，减小镜像体积。
镜像中需要锁定 node 的版本号，尽可能也锁定 alpine 的版本号，如 node:15-alpine。
npm ci 替代 npm i，避免版本问题及提高依赖安装速度
package.json 单独添加，充分利用镜像缓存
只提取编译文件（多阶段构建），减小镜像体积
