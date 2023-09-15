#### 重启项目坑点：

1.  zookeeper.json配置文件更改：

   ```json
    {
   	"connectString": "localhost:2181", // 这里要改成本机地址
   	"configPath": "/TMD_CONFIG",
   	"servicePath": "/TMD_SERVICE",
   	"sessionTimeout": "30000",
   	"connectionTimeout": "30000"
   }
   ```

   2. redis未设置密码，每次开关机会导致docker关机，每次进入容器设置密码不方便，在创建容器的配置上解决

   ```yaml
     redis:
       image: 192.168.1.138:5008/redis:5.0.5
       container_name: redis
       ports:
         - 6379:6379
       command: redis-server --requirepass 123456  //添加这行设置密码
       volumes:
         - tmd-docker-data-redis:/data
       restart: unless-stopped
   ```

   

