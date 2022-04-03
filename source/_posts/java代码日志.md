---
title: java代码日志
date: 2020-02-26 15:38:06
---
### java8计算两个日期之间的工作日
``` java
/**
* 计算两个日期之间的工作日，包含起始日不包含结束日期
* @param start 开始时间
* @param end 截止时间
* @param legalHolidays 法定节假日
* @param legalWorkdays 法定工作日：周末调休
* @return 工作日
*/
public static int workDaysBetweenDate(LocalDate start, LocalDate end, Set<String> legalHolidays, Set<String> legalWorkdays){
   if (Objects.isNull(start) || Objects.isNull(end) 
           || Objects.isNull(legalHolidays) || Objects.isNull(legalWorkdays) ||start.isAfter(end)){
       throw new IllegalArgumentException("输入的参数不合法");
   }
   int betweenDays = Period.between(start, end).getDays();
   DateTimeFormatter dateTimeFormatter = DateTimeFormatter.ofPattern("yyyy-MM-dd");
   int minusDays = 0;
   while (start.isBefore(end)){
       String startString = dateTimeFormatter.format(start);
       if (legalHolidays.contains(startString)){
           minusDays--;
       }
       if ((start.getDayOfWeek() == DayOfWeek.SATURDAY || start.getDayOfWeek() == DayOfWeek.SUNDAY)
               && !legalWorkdays.contains(startString)){
           minusDays--;
       }
       start = start.plusDays(1);
   }
   return betweenDays+minusDays;
}
```


### 分布式锁redis实现

``` java
/**
  * 分布式锁接口类，只有一台机器可以获得锁
  *
*/
public interface RedisLock {
    /**
     * 尝试加锁，不可重入
     * 发生在执行任务之前，只有申请到锁才可以继续执行
     * @param lockName
     * @return
     */
    String tryLock(String lockName, long timeout);

    /**
     * 尝试加锁,使用默认超时时间
     *
     * @param lockName the lock name
     * @return the string
     */
    String tryLock(String lockName);

    /**
     * 释放锁
     * 发生在获得锁但是后续处理失败，否则之后无法再获得锁。
     * @param lockName 锁实例名字，唯一
     * @return
     */
    boolean unLock(String lockName, String identifier);
}
```
``` java
/**
  * 设置过期时间：可以防止锁删除失败，导致线程无线等待；带来的问题，A线程设置的锁过期后，B线程成功设置锁，A线程执行完后删除了B线程的锁；
  * 解决方案，删除的时候获取锁的value，相等则删除；带来的问题，删除锁操作不是原子操作（判断相等的时候，C线程成功设置锁，删除了C线程的锁），
  * 解决方案，rubby脚本。
  * 第三方实现，Redisson
  *
*/
public class RedisLockImpl implements RedisLock {

    private static final String REDIS_LOCK_NAME_PREFIX = "REDIS_LOCK_";
    private static final String LOCK_SUCCESS = "OK";

    //NX是不存在时才set， XX是存在时才set， EX是秒，PX是毫秒
    private static final String SET_IF_NOT_EXIST = "NX";
    private static final String SET_WITH_EXPIRE_TIME = "PX";

    /**
     * 锁的释放时间, 上锁后超过此时间则自动释放锁. 单位为毫秒.
     */
    private static final long DEFAULT_TIMEOUT_MILLISECOND = 1000 * 60 * 10;



    @Override
    public String tryLock(String lockName, long timeout) {
        if (Objects.isNull(lockName))
            return null;
        Jedis jedis = null;
        try {
            jedis = jedisPool.getResource();
            String value = UUID.randomUUID().toString();
            String key = REDIS_LOCK_NAME_PREFIX + lockName;
            String result = jedis.set(key, value, SET_IF_NOT_EXIST, SET_WITH_EXPIRE_TIME, timeout);
            if (LOCK_SUCCESS.equals(result)){
                return value;
            }
        }catch (Exception e){
            System.err.printf("获取锁失败,lockName:%s", lockName);
        }finally {
            if (Objects.nonNull(jedis)){
                jedis.close();    
            }
        }

        return null;
    }

    @Override
    public String tryLock(String lockName) {
        return tryLock(lockName, DEFAULT_TIMEOUT_MILLISECOND);
    }

    @Override
    public boolean unLock(String lockName, String identifier) {
        if (Objects.isNull(lockName) || Objects.isNull(identifier))
            return false;

        Jedis jedis = null;
        String key = REDIS_LOCK_NAME_PREFIX + lockName;
        try {
            jedis = jedisPool.getResource()；
            StringBuilder script = new StringBuilder();
            //if redis.call('get','orderkey')=='1111' then return redis.call('del','orderkey') else return 0 end
            script.append("if redis.call('get','").append(key).append("')").append("=='").append(identifier).append("'").
                    append(" then ").
                    append("    return redis.call('del','").append(key).append("')").
                    append(" else ").
                    append("    return 0").
                    append(" end");
            return Integer.parseInt(jedis.eval(script.toString()).toString()) > 0 ? true : false;
        }catch (Exception e){
            System.err.printf("释放锁失败，lockName:%s,identifier:%s", lockName, identifier);
        }finally {
            if (Objects.nonNull(jedis)){
                jedis.close();
            }
        }
        return false;
    }
}
```

### log4j配置文件

```
<?xml version="1.0" encoding="UTF-8"?>
<configuration status="OFF">

    <Properties>
        <!-- 将系统属性进行一次本地转换 -->
        <Property name="APPLICATION_LOG_LEVEL">${sys:logging.level.com.xxx}</Property>
        <Property name="APPLICATION_LOG_PATH">${sys:logging.path}</Property>
        <Property name="APPLICATION_FILE_ENCODING">${sys:file.encoding}</Property>
    </Properties>

    <!--  log4j2 appenders -->

    <Appenders>

        <RollingFile name="ERROR-APPENDER" fileName="${APPLICATION_LOG_PATH}/application/application-error.log" append="true"
                     filePattern="${APPLICATION_LOG_PATH}/application/common-error.log.%d{yyyy-MM-dd}">
            <!-- only print error log -->
            <ThresholdFilter level="error" onMatch="ACCEPT" onMismatch="DENY"/>
            <PatternLayout charset="${APPLICATION_FILE_ENCODING}">
                <pattern>%d %-5p %-32t - %m%n %throwable</pattern>
            </PatternLayout>
            <Policies>
                <!-- 按天分日志文件:重要的是 filePattern 配置到按照天 -->
                <TimeBasedTriggeringPolicy interval="1" modulate="true"/>
            </Policies>
        </RollingFile>

        <RollingFile name="ROOT-APPENDER" fileName="${APPLICATION_LOG_PATH}/application/application-default.log" append="true"
                     filePattern="${APPLICATION_LOG_PATH}/application/common-default.log.%d{yyyy-MM-dd}">
            <ThresholdFilter level="${APPLICATION_LOG_LEVEL}" onMatch="ACCEPT" onMismatch="DENY"/>
            <PatternLayout charset="${APPLICATION_FILE_ENCODING}">
                <pattern>%d %-5p %-32t - %m%n %throwable</pattern>
            </PatternLayout>
            <Policies>
                <!-- 按天分日志文件:重要的是 filePattern 配置到按照天 -->
                <TimeBasedTriggeringPolicy interval="1" modulate="true"/>
            </Policies>
        </RollingFile>

        <RollingFile name="EXECUTE-APPENDER" fileName="${APPLICATION_LOG_PATH}/application/application-execute.log" append="true"
                     filePattern="${APPLICATION_LOG_PATH}/application/common-default.log.%d{yyyy-MM-dd}">
            <ThresholdFilter level="${APPLICATION_LOG_LEVEL}" onMatch="ACCEPT" onMismatch="DENY"/>
            <PatternLayout charset="${APPLICATION_FILE_ENCODING}">
                <pattern>%d %-5p %-32t - %m%n %throwable</pattern>
            </PatternLayout>
            <Policies>
                <!-- 按天分日志文件:重要的是 filePattern 配置到按照天 -->
                <TimeBasedTriggeringPolicy interval="1" modulate="true"/>
            </Policies>
        </RollingFile>



    </Appenders>

    <Loggers>
        <AsyncLogger name="com.xxx" level="${APPLICATION_LOG_LEVEL}" additivity="false">
            <appender-ref ref="ERROR-APPENDER"/>
            <appender-ref ref="ROOT-APPENDER"/>
        </AsyncLogger>

        <AsyncLogger name="com.xxxx.xxx" level="${APPLICATION_LOG_LEVEL}" additivity="false">
            <appender-ref ref="ERROR-APPENDER"/>
            <appender-ref ref="EXECUTE-APPENDER"/>
        </AsyncLogger>

        <AsyncRoot level="${APPLICATION_LOG_LEVEL}">
            <appender-ref ref="ROOT-APPENDER"/>
            <appender-ref ref="ERROR-APPENDER"/>
        </AsyncRoot>
    </Loggers>
</configuration>
```
