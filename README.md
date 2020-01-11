# RedisConnection Setup Instructions

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
    
    // Execute this in the main onDisable method 
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

### JedisPublisher Class
```java
import redis.clients.jedis.Jedis;

public class JedisPublisher {

    private Jedis jedis;

    public JedisPublisher(){
        jedis = new Jedis("zany.rip");
    }

    public void write(String msg) {
        try {
            jedis.publish("StaffUtils", msg);
        } catch (Exception ex) {
            System.out.println("StaffUtils: A critical redis error has occured, if you encounter this message please contact a developer.");
        }
    }

    public void onDisable(){
        jedis.close();
    }

}

```

### JedisMessage Class
## Extra helpful class
```java
import com.google.gson.JsonObject;

import java.util.HashMap;
import java.util.Map;
// Import RedisConnection and your main class. 

public class JedisMessage {
    private RedisConnection.Action action;
    private Map<String, Object> data = new HashMap<>();
    public JedisMessage(RedisConnection.Action action){
        this.action = action;
    }
    public void put(String string, Object obj1){
        data.put(string, obj1);
    }
    public void write(){
        JsonObject object = new JsonObject();
        object.addProperty("action", action.name());
        JsonObject payLoad = new JsonObject();
        for(String string : data.keySet()){
            payLoad.addProperty(string, data.get(string).toString());
        }
        object.addProperty("payload", payLoad.toString());
        // In this example Core.getInstance() is refering to the main class. 
        Core.getInstance().getRedisConnection().getPublisher().write(object.toString());
    }
}
```
