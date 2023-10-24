### Docker常用命令

https://www.acwing.com/blog/content/10878/

1. 运行docker-compose.yml的文件

   ```js
   docker-compose -f my-custom-compose.yml up -d
   ```

2. 进入运行中的容器

```js
docker exec -it my_container /bin/bash
```



#### 使用docker遇到的一些问题

1. windows和linux换行符不一样，如果代码一模一样，但是容器还是运行不起来，爆错

<span style="color:red;"> no such file or directory</span>

​    这个可以在idea里 ，一般是run.sh的脚本文件，把格式换成LF

![img](https://s2.loli.net/2023/10/24/HhpjkgubdSA4s15.png)





### 构建项目

build -> dockerbuild -> 编译docker-compose或者run

build为了生成jar包，dockerbuild根据dockerfile生成镜像，有可能dockerbuild会包含build





### 手动上传镜像

1. docker tag  原来的   现在的 

例如

docker tag tmd-gateway-pressure-client:latest 

192.168.1.138:5007/tmd/tmd-gateway-pressure-client:pressure-test

注意命名 前面的一般是推的镜像仓库，没有会有默认的dockerhub， 后面是名字  ：跟的是version

2. docker push 刚才弄好的tag

3. 登录到目标服务器 docker pull 那个镜像

4. 找到docker-compose 启动

   docker compose up 启动   -d 后台启动



