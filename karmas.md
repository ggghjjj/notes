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


#### 当一个接口有不同对象时

使用configuration + bean 的方式更合适

```java
@Configuration
public class SensitiveWordConfig {

    // 而且这里还可以进行一些初始化
    @Bean
    public SensitiveWordEngine sensitiveWordEngine(SensitiveWordLoader sensitiveWordLoader) {
        List<String> sensitiveWords = sensitiveWordLoader.loadWordsFromFile();
        return new ACSensitiveWord(sensitiveWords);
    }
}

```

这个时候的实现类可以不用Component

```java
public class ACSensitiveWord implements SensitiveWordEngine {

    private ACTrie acTrie;

    private Character replaceChar = '*';

    public ACSensitiveWord(List<String> sensitiveWords) {
        acTrie = new ACTrie();
        Log.info("ACSensitiveWord - init");
        // 构建trie
        for (String word: sensitiveWords) {
            acTrie.insert(word);
        }
        // 构建fail
        acTrie.build();
    }
```





#### nats

```java
@Service
public class JetChannelInitializer {

    private ChannelDomainManager channelDomainManager;
    private JetChannelManager  jetChannelManager;
    private JetChannelHandlerManager jetChannelHandlerManager;

    @Resource(name = "chatSystemOServiceImpl")
    private IChatOSystemService chatOSystemService;
    @Resource(name = "chatSystemTServiceImpl")
    private IChatTSystemService chatTSystemService;

    @Resource(name = "jetChannelChatOSystemClientService")
    private IChatOSystemService jetChannelChatOSystemClientService;
    @Resource(name = "jetChannelChatTSystemClientService")
    private IChatTSystemService jetChannelChatTSystemClientService;

    public JetChannelInitializer(ChannelDomainManager channelDomainManager,
                                 JetChannelManager jetChannelManager,
                                 JetChannelHandlerManager jetChannelHandlerManager) {
        this.channelDomainManager = channelDomainManager;
        this.jetChannelManager = jetChannelManager;
        this.jetChannelHandlerManager = jetChannelHandlerManager;
    }
	// 这里是初始化部分
    public void init(int serverId) {
        // 注册handlers   这里是注册一些类和service到map里,以至于可以拿到相应的处理器和反射找相对应的处理器
        registerHandlers();
        // 绑定server channel
        bindServerChannel(serverId);
        // 订阅channel domain
        subscribeChannelDomain(serverId);
    }

    private void registerHandlers() {
        jetChannelHandlerManager.registerHandler(IChatOSystemService.class, chatOSystemService);
        jetChannelHandlerManager.registerHandler(IChatTSystemService.class, chatTSystemService);
        jetChannelHandlerManager.registerClientHandler(IChatOSystemService.class, jetChannelChatOSystemClientService);
        jetChannelHandlerManager.registerClientHandler(IChatTSystemService.class, jetChannelChatTSystemClientService);
    }
	
    private void bindServerChannel(int serverId) {
        jetChannelManager.bindServerChannel(serverId);
    }

    //这里是订阅的入口  如果有绑定的channel发生变化  或者订阅到相应的subject会触发 交给handleRequest去反射处理
    private void subscribeChannelDomain(int serverId) {
        var domainKey = new ChannelDomainKey(ChannelDomainType.Chat.name(), Integer.toString(serverId));
        this.channelDomainManager.subscribeDomain(domainKey, ctx -> {
            var request = (JetRpcRequest)ctx.getPayload();
            this.jetChannelHandlerManager.handleRequest(request);
            return null;
        });
    }

    public JetChannelHandlerManager getJetChannelHandlerManager() {
        return jetChannelHandlerManager;
    }
}

```



```java
// 在别的类中获取这个 类对应的handler,然后去走相应打发送消息的handler   

public <T> T getClientHandler(Class<T> tClass) {
        return jetChannelInitializer.getJetChannelHandlerManager().getClientHandler(tClass);
    }
```

