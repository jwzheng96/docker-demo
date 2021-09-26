# 用docker部署一个python 后台应用程序
## 下载并安装docker
[教程](https://www.runoob.com/docker/ubuntu-docker-install.html)

## 主要步骤简述
要写一个dockerfile在项目的根目录，然后用docker的程序去在有这个dockerfile的目录下去生成一个docker image，这个docker image内部是一个linux的操作系统，类似于一个VM

## 创建后台python应用
```bash
mkdir backend-project
cd backend-project
vim \src\app.py
```

下面编写具体的应用：
```python
from flask import Flask
server = Flask(__name__)

@server.route("/")
def hello():
    return "hello World!"

if __name__ == "__main__":
    server.run(host="0.0.0.0")
```

## 编写Dockerfile
```bash
vim Dockerfile
```
具体Dockerfile内容：
```bash
# set base image (host OS)
FROM python:3.8

# set the working directory in the container
WORKDIR /code

# copy the dependencies file to the working directory
COPY requirements.txt .

# install dependencies
RUN pip install -r requirements.txt

# copy the content of the local src directory to the working directory
COPY . .

# command to run on container start
CMD [ "python", "./src/app.py" ]
```

## build docker

```bash
docker build -t zjw/backend-python .
```
查看此时的镜像
```bash
REPOSITORY                   TAG              IMAGE ID       CREATED         SIZE
zjw/backend-python           latest           d01f8858bffc   3 minutes ago   920MB
zjw/v-swap                   latest           2e8dfcca94df   10 days ago     3.38GB
zjw/docker-test              latest           9cab0c9d24fe   2 weeks ago     921MB
```

## run docker
此时还没run起来，可以通过以下方式查看
```bash
$ docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```
使用 -d 模式运行镜像将以分离模式运行 Docker 容器，使得容器在后台自助运行。开关符 -p 在容器中把一个公共端口导向到私有的端口，请用以下命令运行你之前构建的镜像：
```bash
$ docker run -d -p 49180:5000 zjw/backend-python

4fc9f83c0b0e3d612067c1d3adb8af64e16ee5819123f371c968f95d83ada39b
```
此时：
```bash
$ docker ps
CONTAINER ID   IMAGE                COMMAND                 CREATED         STATUS         PORTS                     NAMES
4fc9f83c0b0e   zjw/backend-python   "python ./src/app.py"   4 seconds ago   Up 3 seconds   0.0.0.0:49180->5000/tcp   focused_sinoussi
```
此时在浏览器打开http://0.0.0.0:49180/将看到Hello world！