# 用docker部署一个Node.js web应用程序

本示例的目标是给你演示如何将一个Node.js 的应用装入到Docker容器中。本教程旨在针对于开发人员，而非产品发布人员。此教程同样假定你有一个可以正常工作的Docker安装，并且对于Node.js的应用程序是如何组织的有一个大致的基本了解。

在本教程的第一部分我们在 Node.js 中创建一个Web的应用程序，然后我们为那个应用构建一个Docker镜像；最后我们将把那个镜像作为容器运行之。

Docker 允许你以应用程序所有的依赖全部打包成一个标准化的单元，这被称为一个容器。对于应用开发而言，一个容器就是一个蜕化到最基础的Linux操作系统。一个镜像是你加载到容器中的软件。

## 创建Node.js web应用
首先，创建一个新文件夹以便于容纳需要的所有文件，并且在此其中创建一个package.json文件，描述你应用程序以及需要的依赖：

```json
{
    "name": "docker_web_app",
    "version": "1.0.0",
    "description": "Node.js on Docker",
    "author": "Jianwei Zheng <zjwofpku@gmail.com>",
    "main": "server.js",
    "scripts": {
      "start": "node server.js"
    },
    "dependencies": {
      "express": "^4.16.1"
    }
}
```

## 编写Dockerfile
我们要做的第一件事是定义我们需要从哪个镜像进行构建。这里我们将使用最新的 LTS（长期服务器支持版），Node 的版本号为 12。你可以从 Docker 站点 获取相关镜像：
```bash 
FROM node:12
```
下一步在镜像中创建一个文件夹存放应用程序代码，这将是你的应用程序工作目录：
```bash 
# Create app directory
WORKDIR /usr/src/app
```
此镜像中 Node.js 和 NPM 都已经安装，所以下一件事对于我们而言是使用npm安装你的应用程序的所有依赖。请注意，如果你的 npm 的版本是4或者更早的版本,package-lock.json文件将不会自动生成。
```bash
# Install app dependencies
# A wildcard is used to ensure both package.json AND package-lock.json are copied
# where available (npm@5+)
COPY package*.json ./

RUN npm install
# If you are building your code for production
# RUN npm ci --only=production
```
请注意，我们只是拷贝了 package.json 文件而非整个工作目录。这允许我们利用缓存 Docker 层的优势。进一步说，对于生产环境而言，注释中提及的 npm ci 命令协助提供了一个更快、可靠、可再生的构建环境。

在 Docker 镜像中使用 COPY 命令绑定你的应用程序：
```bash
# Bundle app source
COPY . .
```
你的应用程序绑定的端口为 8080，所以你可以使用 EXPOSE 命令使它与 docker 的镜像做映射：
```bash
EXPOSE 8080
```
最后但同样重要的事是，使用定义运行时的 CMD 定义命令来运行应用程序。这里我们使用 node server.js 来启动你的服务器：
```bash
CMD [ "node", "server.js" ]
```

最终的 Dockerfile 现在看上去是这个样子：
```bash
FROM node:12

# Create app directory
WORKDIR /usr/src/app

# Install app dependencies
# A wildcard is used to ensure both package.json AND package-lock.json are copied
# where available (npm@5+)
COPY package*.json ./

RUN npm install
# If you are building your code for production
# RUN npm ci --only=production

# Bundle app source
COPY . .

EXPOSE 8080
CMD [ "node", "server.js" ]
```
## .dockerignore 文件

在 Dockerfile 的同一个文件夹中创建一个 .dockerignore 文件，带有以下内容：
```bash
node_modules
npm-debug.log
```
这将避免你的本地模块以及调试日志被拷贝进入到你的 Docker 镜像中，以至于把你镜像原有安装的模块给覆盖了。

## 构建镜像
这将避免你的本地模块以及调试日志被拷贝进入到你的 Docker 镜像中，以至于把你镜像原有安装的模块给覆盖了。
```bash
docker build . -t <your username>/node-web-app
```
Docker 现在将给出你的镜像列表：
```bash
$ docker images

# Example
REPOSITORY                      TAG        ID              CREATED
node                            12         1934b0b038d1    5 days ago
<your username>/node-web-app    latest     d64d3505b0d2    1 minute ago
```

## 运行镜像
使用 -d 模式运行镜像将以分离模式运行 Docker 容器，使得容器在后台自助运行。开关符 -p 在容器中把一个公共端口导向到私有的端口，请用以下命令运行你之前构建的镜像：
```bash
docker run -p 49160:8080 -d <your username>/node-web-app
```
把应用程序的输出打印出来：
```bash
# Get container ID
$ docker ps

# Print app output
$ docker logs <container id>

# Example
Running on http://localhost:8080
```
如果需要进入容器中，请运行 exec 命令：
```bash
# Enter the container
$ docker exec -it <container id> /bin/bash
```
## 测试
为测试你的应用程序，给出与 Docker 映射过的端口号：
```bash
$ docker ps

# Example
ID            IMAGE                                COMMAND    ...   PORTS
ecce33b30ebf  <your username>/node-web-app:latest  npm start  ...   49160->8080
```
在上面的例子中，在容器中 Docker 把端口号 8080 映射到你机器上的 49160 。
现在你可以使用 curl（如果需要的话请通过 sudo apt-get install curl 安装）调用你的程序了：
```bash
$ curl -i localhost:49160

HTTP/1.1 200 OK
X-Powered-By: Express
Content-Type: text/html; charset=utf-
Content-Length: 12
ETag: W/"c-M6tWOb/Y57lesdjQuHeB1P/qTV0"
Connection: keep-alive

Hello world
```

你也可以在以下一些地方寻觅到更多有关于 Docker 和基于 Docker 的 Node.js 相关内容：
•	[官方 Node.js 的 Docker 镜像](https://hub.docker.com/_/node/)
•	[Node.js 基于 Docker 使用的最佳经验](https://github.com/nodejs/docker-node/blob/main/docs/BestPractices.md)
•	[官方 Docker 文档](https://docs.docker.com/)
