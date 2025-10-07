## Task 5

- Acessed redis via Docker (Installation was Pretty easy)
- connected using the following
```bash
docker run -d --name redis-container -p 6379:6379 redis
```
- Afterwards started redis-shell with
```bash
docker exec -it redis-container redis-cli
```
- First Test with PING -->Output PONG
- Did the Tests with user 
- Exprire made the User nill after 5 seconds
- Using SADD I kept track of the logged in users
```bash
SADD logged_in_users alice
SADD logged_in_users bob
SREM logged_in_users alice 
ADD logged_in_users eve 
```
- Using SMEMBERS logged_in_users I was able to display all the logged in users 

- Created HashSet for storing the Poll and added the options; afterwards increased option1 by 1 
```bash
HSET poll:03ebcb7b-bd69-440b-924e-f5b7d664af7b title "Pineapple on Pizza?"
HSET poll:03ebcb7b-bd69-440b-924e-f5b7d664af7b:option:1 caption "Yes, yammy!" voteCount 269
HSET poll:03ebcb7b-bd69-440b-924e-f5b7d664af7b:option:2 caption "Mamma mia, nooooo!" voteCount 268
HSET poll:03ebcb7b-bd69-440b-924e-f5b7d664af7b:option:3 caption "I do not really care ..." voteCount 42
HINCRBY poll:03ebcb7b-bd69-440b-924e-f5b7d664af7b:option:1 voteCount 1
```

- Created a Java Code that did the same as the experiments above; Used the examples from redis to connect
```java
package no.hvl.dat250.jpa.polls.redis;
import redis.clients.jedis.UnifiedJedis;
import java.util.Set;
import java.util.List;

public class RedisQuickTest {
    public static void main(String[] args) {
        try (UnifiedJedis jedis = new UnifiedJedis("redis://localhost:6379")) {
            // logged-in users example
            jedis.sadd("logged_in_users", "alice");
            jedis.sadd("logged_in_users", "bob");
            jedis.srem("logged_in_users", "alice");
            Set<String> members = jedis.smembers("logged_in_users");
            System.out.println("Members: " + members);

            // Poll-Daten
            String pollId = "03ebcb7b-bd69-440b-924e-f5b7d664af7b";
            String pollKey = "poll:" + pollId;
            jedis.hset(pollKey, "title", "Pineapple on Pizza?");

            // Optionen als Hashes speichern
            String opt1Key = pollKey + ":option:1";
            jedis.hset(opt1Key, "caption", "Yes, yammy!");
            jedis.hset(opt1Key, "voteCount", "269");

            String opt2Key = pollKey + ":option:2";
            jedis.hset(opt2Key, "caption", "Mamma mia, nooooo!");
            jedis.hset(opt2Key, "voteCount", "268");

            String opt3Key = pollKey + ":option:3";
            jedis.hset(opt3Key, "caption", "I do not really care ...");
            jedis.hset(opt3Key, "voteCount", "42");

            // Option IDs in einer Liste speichern
            jedis.del(pollKey + ":options"); // alte Werte löschen
            jedis.rpush(pollKey + ":options", "1", "2", "3");


            System.out.println("Poll title: " + jedis.hget(pollKey, "title"));
            List<String> optionIds = jedis.lrange(pollKey + ":options", 0, -1);
            for (String id : optionIds) {
                String optKey = pollKey + ":option:" + id;
                System.out.println("Option " + id + ": " + jedis.hgetAll(optKey));
        }
    }
}}

```
- Created a cache following the logic that was given in the document

```java
package no.hvl.dat250.jpa.polls.redis;
import redis.clients.jedis.UnifiedJedis;
import java.util.Set;
import java.util.List;
import java.util.Map;

public class RedisCache {
    public static void main(String[] args) {
        String pollId = "03ebcb7b-bd69-440b-924e-f5b7d664af7b";
        String pollKey = "poll:" + pollId;
        try (UnifiedJedis jedis = new UnifiedJedis("redis://localhost:6379")) {
            if (jedis.exists(pollKey + ":options")) {
                System.out.println("Cache HIT! Poll from Redis:");
                printPoll(jedis, pollKey);
            } else {
                System.out.println("Cache MISS! Loading from DB and saving in Redis...");
                // Simulate DB load and save to Redis
                addPollToRedis(jedis, pollId, "Pineapple on Pizza?",
                        new String[]{"Yes, yammy!", "Mamma mia, nooooo!", "I do not really care ..."},
                        new int[]{269, 268, 42});
                
                printPoll(jedis, pollKey);
            }

            // Increase vote count for option 1
            System.out.println("\nRaise Vote for Option 1 ...");
            jedis.hincrBy(pollKey + ":option:1", "voteCount", 1);

            // Show current values
            System.out.println("\nCurrent Poll Votes:");
            printPoll(jedis, pollKey);
        }
    }

    // Save a poll with options to Redis
    private static void addPollToRedis(UnifiedJedis jedis, String pollId, String title,
                                       String[] optionCaptions, int[] voteCounts) {
        String pollKey = "poll:" + pollId;
        jedis.hset(pollKey, "title", title);

        jedis.del(pollKey + ":options"); // alte IDs löschen

        for (int i = 0; i < optionCaptions.length; i++) {
            String optKey = pollKey + ":option:" + (i + 1);
            jedis.hset(optKey, "caption", optionCaptions[i]);
            jedis.hset(optKey, "voteCount", String.valueOf(voteCounts[i]));
            jedis.rpush(pollKey + ":options", String.valueOf(i + 1));
        }
    }

    //Read and print poll from Redis
    private static void printPoll(UnifiedJedis jedis, String pollKey) {
        System.out.println("Poll title: " + jedis.hget(pollKey, "title"));
        List<String> optionIds = jedis.lrange(pollKey + ":options", 0, -1);

        for (String id : optionIds) {
            String optKey = pollKey + ":option:" + id;
            Map<String, String> option = jedis.hgetAll(optKey);
            System.out.println("Option " + id + ": " + option.get("caption") + " | Votes: " + option.get("voteCount"));
        }
    }
}

```

- There are no pending technical issues


