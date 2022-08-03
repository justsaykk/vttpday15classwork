# Setting up Redis

## Information needed prior to setup
1. Redis Host (a.k.a "public endpoint")
2. Redis Port (the numbers at the end of the endpoint after the colon)
3. Redis Database (Should be 0)
4. Redis username (Default username is "Default")
5. Redis password

## Adding Jedis dependencies
```xml
<!-- https://mvnrepository.com/artifact/redis.clients/jedis -->
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
    <version>3.9.0</version>
</dependency>
```
## Adding application properties
At the root/src/main/resources/application.properties,</br>
Add the following properties:
```
spring.redis.database=0
spring.redis.host= <redis hostname here>
spring.redis.port= <redis port here>
spring.redis.username=default
```

## Setting up Redis template:
At the same folder as the initial class file, create another file named "AppConfiguration.java". </br>
This is the standard boilerplate for Redis template setup:
```java
package your.packagename.here;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.connection.RedisStandaloneConfiguration;
import org.springframework.data.redis.connection.jedis.JedisClientConfiguration;
import org.springframework.data.redis.connection.jedis.JedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.serializer.StringRedisSerializer;

@Configuration
public class AppConfiguration {
    @Value("${spring.redis.host}")
    private String redisHost;

    @Value("${spring.redis.port}")
    private String redisPort;

    @Value("${spring.redis.database}")
    private String redisdb;

    @Value("${spring.redis.username}")
    private String redisUserName;

    /*
     * For Redis password, use this CLI command:
     * export REDIS_PASSWORD="<password here>"
     * at the root folder before performing ./mvnw package and/or ./mvnw spring-boot:run
     */
    @Value("${REDIS_PASSWORD}")
    private String redisPassword;

    @Bean
    public RedisTemplate initRedisTemplate() {
        // 1st Step: Config Redis Database
        RedisStandaloneConfiguration redisConfig = new RedisStandaloneConfiguration();
        redisConfig.setHostName(redisHost);
        redisConfig.setPort(Integer.parseInt(redisPort));
        redisConfig.setDatabase(Integer.parseInt(redisdb));
        redisConfig.setUsername(redisUserName);
        redisConfig.setPassword(redisPassword);

        // 2nd Step: Create instance of Jedis Driver
        JedisClientConfiguration jedisConfig = JedisClientConfiguration.builder().build();

        // 3rd Step: Create factory for jedis connection
        JedisConnectionFactory jedisFac = new JedisConnectionFactory(redisConfig, jedisConfig);
        jedisFac.afterPropertiesSet();

        // Last Step: Create new redis template
        RedisTemplate<String, String> redisTemplate = new RedisTemplate<>();
        redisTemplate.setConnectionFactory(jedisFac);
        redisTemplate.setKeySerializer(new StringRedisSerializer());
        redisTemplate.setValueSerializer(new StringRedisSerializer());

        return redisTemplate;
    }
}
```

# Redis CLI Commands
To connect to redis

`~$ redis-cli -h <hostname> -p <port> --user <username> --pass <password>`

To select database (after connecting)

`~$ select <database number>`
> For free users, you only have 1 database and the no. is 0. 
> Therefore the code is:
> `~$ select 0`


