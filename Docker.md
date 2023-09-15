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

### 构建项目

build -> dockerbuild -> 编译docker-compose或者run

build为了生成jar包，dockerbuild根据dockerfile生成镜像，有可能dockerbuild会包含build

