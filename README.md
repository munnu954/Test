Certainly! Here's an example of how you can achieve this using Spring Boot to fetch data from an H2 database and send it to Kafka at a regular 5-minute interval.

First, let's set up the necessary dependencies in your `pom.xml` file:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.kafka</groupId>
    <artifactId>spring-kafka</artifactId>
</dependency>

<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <scope>runtime</scope>
</dependency>
```

Next, create a Spring Boot application class, `DemoApplication.java`:

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.scheduling.annotation.EnableScheduling;

@SpringBootApplication
@EnableScheduling
public class DemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }
}
```

Next, create a model class, `DataModel.java`, to represent the data you want to fetch from the H2 database:

```java
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;

@Entity
public class DataModel {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String data;

    // getters and setters
}
```

Next, create a repository interface, `DataRepository.java`, to interact with the H2 database using Spring Data JPA:

```java
import org.springframework.data.jpa.repository.JpaRepository;

public interface DataRepository extends JpaRepository<DataModel, Long> {
    // custom query methods if needed
}
```

Now, create a service class, `DataFetcherService.java`, to fetch data from the H2 database and send it to Kafka:

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.kafka.core.KafkaTemplate;
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Service;

@Service
public class DataFetcherService {
    
    private final DataRepository dataRepository;
    private final KafkaTemplate<String, String> kafkaTemplate;
    
    @Autowired
    public DataFetcherService(DataRepository dataRepository, KafkaTemplate<String, String> kafkaTemplate) {
        this.dataRepository = dataRepository;
        this.kafkaTemplate = kafkaTemplate;
    }

    @Scheduled(fixedDelay = 300000) // Run every 5 minutes (300,000 milliseconds)
    public void fetchDataAndSendToKafka() {
        // Fetch data from the H2 database
        List<DataModel> data = dataRepository.findAll();
        
        // Process the data if needed
        
        // Send data to Kafka
        data.forEach(d -> kafkaTemplate.send("your-kafka-topic", d.getData()));
    }
}
```

Finally, configure Kafka in your `application.properties` file:

```properties
spring.kafka.bootstrap-servers=your-kafka-bootstrap-servers
spring.kafka.producer.key-serializer=org.apache.kafka.common.serialization.StringSerializer
spring.kafka.producer.value-serializer=org.apache.kafka.common.serialization.StringSerializer
```

Replace `your-kafka-topic` with the actual Kafka topic you want to use, and `your-kafka-bootstrap-servers` with the proper Kafka bootstrap servers.

That's it! You now have a Spring Boot application that fetches data from an H2 database and sends it to Kafka every 5 minutes.
