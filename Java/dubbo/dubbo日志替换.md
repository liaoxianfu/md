dubbo整合springboot后没有日志

添加上pom

```xml
  <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>log4j-over-slf4j</artifactId>
            <version>1.7.30</version>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
            <version>1.7.30</version>
        </dependency>

        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-log4j12</artifactId>
            <version>1.7.30</version>
            <scope>test</scope>
        </dependency>
        
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-nop</artifactId>
            <version>1.7.30</version>
            <scope>test</scope>
        </dependency>
```

在resource文件夹下添加

log4j.properties

```properties

log4j.rootLogger = WARN,C1
log4j.addivity.org.apache=true
 
#category
log4j.category.org.hibernate.tool.hbm2ddl =DEBUG,F1
log4j.category.org.hibernate.SQL =DEBUG,A1 
 
#应用于控制台  
log4j.appender.C1=org.apache.log4j.ConsoleAppender
#log4j.appender.C1.Threshold=WARNING
log4j.appender.C1.Target=System.out
#log4j.appender.C1.Encoding=UTF-8
log4j.appender.C1.layout=org.apache.log4j.PatternLayout
log4j.appender.C1.layout.ConversionPattern=[CONSOLE] %d - %c -%-4r [%t] %-5p %c %x - %m%n   
```

