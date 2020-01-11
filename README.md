# RedisConnection
Redis connection 

### RedisConnection Class
```java
import com.google.gson.JsonObject;
import com.google.gson.JsonParser;
import lombok.Getter;
import org.bson.Document;
import org.bukkit.Bukkit;
import org.bukkit.entity.Player;
import redis.clients.jedis.Jedis;
import redis.clients.jedis.JedisPubSub;

import java.util.UUID;

public class RedisConnection {

    private Core core = Core.getInstance();

    @Getter
    private JedisPubSub pubSub;
    @Getter
    private Jedis jedis;

    @Getter private JedisPublisher publisher;

    public RedisConnection() {
        subscribe();
        publisher = new JedisPublisher();
    }

    public void onDisable(){
        jedis.close();
        publisher.onDisable();
    }

    private void subscribe() {
        pubSub = get();
        jedis = new Jedis("zany.rip");
        new Thread() {
            public void run() {
                jedis.subscribe(pubSub, "StaffUtils");
            }
        }.start();
    }

    private JedisPubSub get() {
        return new JedisPubSub() {
            public void onMessage(String channel, String message) {
                if (channel.equals("StaffUtils")) {
                    try {
                        JsonObject object = new JsonParser().parse(message).getAsJsonObject();
                        Action action = Action.valueOf(object.get("action").getAsString());
                        JsonObject payLoad = new JsonParser().parse(object.get("payload").getAsString()).getAsJsonObject();
                        try {
                            checkMessage(action, payLoad);
                        } catch (Exception ex) {

                        }
                    } catch (Exception ex) {
                    }
                }
            }
        };
    }

    private void checkMessage(Action action, JsonObject payLoad) {
        
    }

    public enum Action {
        EXAMPLE_ACTION
    }

```
