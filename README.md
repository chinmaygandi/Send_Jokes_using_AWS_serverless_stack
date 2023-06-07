# AWS Serverless Stack [DynamoDB, Lambda, SNS, API Gateway]

In this project, we will learn to implement end to end AWS Stack [Dynamo DB, API gateway, Lambda, SNS]. This is to demonstrate how we can leverage AWS services to make a working application. Our project's goal is simple - There is a public joke API (https://v2.jokeapi.dev/) that sends a joke in JSON format after hitting the API as you can see below. We need to get this joke in our mailbox and store it in our dynamo database as and when our own AWS API gateway is hit. 

![image](https://github.com/chinmaygandi/AWS---Examples/assets/131703516/c1509bd4-d258-419b-8632-b4dfb2d7f574)


Reference has been taken from the site https://github.com/awsdocs/aws-doc-sdk-examples/tree/main/javav2

Our project requires - 
1) IDE (Intellij)
2) Java
3) Maven
4) AWS account

**Step 1 [Installing Maven]**

Maven can be installed with the help of the steps mentioned on the site https://phoenixnap.com/kb/install-maven-windows

**Step 2 [Creating a new Maven project in Intellij]**

Click on New project in Intellij and select language as java and maven as built system. You will find Main class in the project and a pom.xml file. This file will contain all the dependencies that you will require for this AWS Project. You can modify the pom.xml file to the one that you see below 

```<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>LambdaCronFunctions</groupId>
    <artifactId>LambdaCronFunctions</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>jar</packaging>
    <name>java-basic-function</name>
    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
    </properties>
    <dependencies>
        <dependency>
            <groupId>com.amazonaws</groupId>
            <artifactId>aws-lambda-java-core</artifactId>
            <version>1.2.1</version>
        </dependency>
        <dependency>
            <groupId>com.amazonaws</groupId>
            <artifactId>aws-java-sdk-dynamodb</artifactId>
            <version>1.12.181</version>
        </dependency>
        <dependency>
            <groupId>com.amazonaws</groupId>
            <artifactId>aws-lambda-java-events</artifactId>
            <version>3.11.0</version>
        </dependency>
        <dependency>
            <groupId>com.amazonaws</groupId>
            <artifactId>aws-java-sdk-sns</artifactId>
            <version>1.12.39</version>
        </dependency>
        <dependency>
            <groupId>com.google.code.gson</groupId>
            <artifactId>gson</artifactId>
            <version>2.8.9</version>
        </dependency>
        <dependency>
            <groupId>org.apache.logging.log4j</groupId>
            <artifactId>log4j-api</artifactId>
            <version>2.17.0</version>
        </dependency>
        <dependency>
            <groupId>org.apache.logging.log4j</groupId>
            <artifactId>log4j-core</artifactId>
            <version>2.17.0</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.apache.logging.log4j</groupId>
            <artifactId>log4j-slf4j18-impl</artifactId>
            <version>2.17.0</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter-api</artifactId>
            <version>5.8.2</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter-engine</artifactId>
            <version>5.8.2</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>com.googlecode.json-simple</groupId>
            <artifactId>json-simple</artifactId>
            <version>1.1.1</version>
        </dependency>
        <dependency>
            <groupId>software.amazon.awssdk</groupId>
            <artifactId>dynamodb-enhanced</artifactId>
            <version>2.17.110</version>
        </dependency>
        <dependency>
            <groupId>software.amazon.awssdk</groupId>
            <artifactId>dynamodb</artifactId>
            <version>2.17.110</version>
        </dependency>
        <dependency>
            <groupId>software.amazon.awssdk</groupId>
            <artifactId>sns</artifactId>
            <version>2.17.110</version>
        </dependency>
        <dependency>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-surefire-plugin</artifactId>
            <version>3.0.0-M5</version>
            <type>maven-plugin</type>
        </dependency>
    </dependencies>
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-shade-plugin</artifactId>
                <version>3.2.2</version>
                <configuration>
                    <createDependencyReducedPom>false</createDependencyReducedPom>
                </configuration>
                <executions>
                    <execution>
                        <phase>package</phase>
                        <goals>
                            <goal>shade</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.8.1</version>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

After adding this pom.xml file, it will show the below highlighted icon to actually add the dependencies in the project from com.amazonaws. Click on the below highlighted button and it will start adding dependencies in the project. 

![image](https://github.com/chinmaygandi/AWS---Examples/assets/131703516/c0a90d3b-9026-402a-b08d-fb66a2962e41)

This completes adding all the required dependencies for the project. 

**Step 3 [Creating AWS services in AWS console]**

1) We will first require a AWS SNS service. The email ids registered with this topic will recieve any email that we send. You can follow the steps mentioned in the link (https://docs.aws.amazon.com/sns/latest/dg/sns-create-topic.html) for creating an SNS topic and associating emailids with the same. It will look something like this.

![image](https://github.com/chinmaygandi/AWS---Examples/assets/131703516/bb715127-819c-4160-8e6f-4a72a65fa2ab)

Note the ARN of this topic as we will be using them in our code. 

2) The other services that we require is creating an API Gateway and Lambda function. We will require these services after creating the maven project. So we will go through them once our project is ready in intellij. 

**Step 4 [Coding the main logic for our project]**

Our project will contain below various different java classes. 

1) There will be a joke class [Joke.java] that will contain all the different attributes that our public joke API has such as error, category, nsfw etc. We can see in them here (https://v2.jokeapi.dev/). **We will creating the dynamo db table via code and not in the AWS console.**

2) There will be a handler class [joke_handler.java] that will be triggered when the AWS lambda function is called. This will contain all the main logic 

         a) To create dynamo db table
         b) Check if the table already exists (with the help of class file Checktable.java)
         c) Convert the jokes record which is in json format that we get from public API to String 
         d) Insert in dynamo db table as a record 
         e) Send the joke to the SNS topic (with the help of class file SendJokesMail.java)
         


**Joke.java**

```
package org.example;

import com.amazonaws.services.dynamodbv2.datamodeling.DynamoDBHashKey;
import com.amazonaws.services.dynamodbv2.datamodeling.DynamoDBMapperFieldModel;
import com.amazonaws.services.dynamodbv2.datamodeling.DynamoDBTable;
import com.amazonaws.services.dynamodbv2.datamodeling.DynamoDBTyped;
import software.amazon.awssdk.enhanced.dynamodb.mapper.annotations.DynamoDbAttribute;
import software.amazon.awssdk.enhanced.dynamodb.mapper.annotations.DynamoDbBean;
import software.amazon.awssdk.enhanced.dynamodb.mapper.annotations.DynamoDbConvertedBy;
import software.amazon.awssdk.enhanced.dynamodb.mapper.annotations.DynamoDbAttribute;
//import software.amazon.awssdk.enhanced.dynamodb.mapper.annotations.DynamoDbConverted;


@DynamoDBTable(tableName = "JokesTable")
@DynamoDbBean
public class Joke {
    private boolean error;
    private String category;
    private int id;
    private String setup;
    private String delivery;

    private String joke;

    @DynamoDBTyped(DynamoDBMapperFieldModel.DynamoDBAttributeType.M)
    private Flags flags;
    private boolean safe;
    private String lang;
    private String type;

    // Constructor
    public Joke() {
        // Empty constructor
    }

    // Getter and Setter methods
    @DynamoDbAttribute("id")
    @DynamoDBHashKey(attributeName = "id")
    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    @DynamoDbAttribute("error")
    public boolean isError() {
        return error;
    }

    public void setError(boolean error) {
        this.error = error;
    }

    @DynamoDbAttribute("category")
    public String getCategory() {
        return category;
    }

    public void setCategory(String category) {
        this.category = category;
    }

    @DynamoDbAttribute("type")
    public String getType() {
        return type;
    }

    public void setType(String type) {
        this.type = type;
    }
    @DynamoDbAttribute("setup")
    public String getSetup() {
        return setup;
    }

    public void setSetup(String setup) {
        this.setup = setup;
    }

    // Additional getter and setter methods for the new fields (delivery, flags, safe, lang)

    @DynamoDbAttribute("delivery")
    public String getDelivery() {
        return delivery;
    }

    public void setDelivery(String delivery) {
        this.delivery = delivery;
    }
    @DynamoDbAttribute("flags")
    @DynamoDbConvertedBy(JokeFlagsConverter.class)
    public Flags getFlags() {
        return flags;
    }

    public void setFlags(Flags flags) {
        this.flags = flags;
    }

    @DynamoDbAttribute("safe")
    public boolean isSafe() {
        return safe;
    }

    public void setSafe(boolean safe) {
        this.safe = safe;
    }
    @DynamoDbAttribute("lang")
    public String getLang() {
        return lang;
    }

    public void setLang(String lang) {
        this.lang = lang;
    }

    public String getjoke(String joke)
    {
        return joke;
    }

    public void setjoke()
    {
        this.joke = joke;
    }


    // Inner class for flags
    public static class Flags {
        private boolean nsfw;
        private boolean religious;
        private boolean political;
        private boolean racist;
        private boolean sexist;
        private boolean explicit;

        // Constructor
        public Flags() {
            // Empty constructor
        }

        // Getter and Setter methods
        public boolean isNsfw() {
            return nsfw;
        }

        public void setNsfw(boolean nsfw) {
            this.nsfw = nsfw;
        }

        public boolean isReligious() {
            return religious;
        }

        public void setReligious(boolean religious) {
            this.religious = religious;
        }

        public boolean isPolitical() {
            return political;
        }

        public void setPolitical(boolean political) {
            this.political = political;
        }

        public boolean isRacist() {
            return racist;
        }

        public void setRacist(boolean racist) {
            this.racist = racist;
        }

        public boolean isSexist() {
            return sexist;
        }

        public void setSexist(boolean sexist) {
            this.sexist = sexist;
        }

        public boolean isExplicit() {
            return explicit;
        }

        public void setExplicit(boolean explicit) {
            this.explicit = explicit;
        }
    }
}

```

***jokehandler.java***

```
package org.example;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.net.URL;
import com.amazonaws.services.dynamodbv2.AmazonDynamoDB;
import com.amazonaws.services.dynamodbv2.AmazonDynamoDBClientBuilder;
import com.amazonaws.services.dynamodbv2.datamodeling.DynamoDBMapper;
import com.amazonaws.services.dynamodbv2.model.CreateTableRequest;
import com.amazonaws.services.dynamodbv2.model.ProvisionedThroughput;
import com.amazonaws.services.lambda.runtime.Context;
import com.amazonaws.services.lambda.runtime.RequestHandler;
import com.fasterxml.jackson.databind.JsonNode;
import com.fasterxml.jackson.databind.ObjectMapper;
import software.amazon.awssdk.regions.Region;

public class joke_handler implements RequestHandler<Void ,Boolean> {


    public Boolean handleRequest(Void unused, Context context) {

        // You will mention the name of the region where your AWS Services are setup
        Region region = Region.US_WEST_2;

        AmazonDynamoDB client = AmazonDynamoDBClientBuilder.standard()
                .withRegion(String.valueOf(region))
                .build();

        DynamoDBMapper mapper = new DynamoDBMapper(client);
        
        // The name of the dynamo db table that you want
        String b = "JokesTable";
        CheckTable a = new CheckTable();
        boolean d = a.handleRequest(b);

        if (d == false) {
           
            CreateTableRequest createTableRequest = mapper.generateCreateTableRequest(Joke.class);
            TableNameOverride().getProvisionedThroughput());
            ProvisionedThroughput provisionedThroughput = new ProvisionedThroughput()
                    .withReadCapacityUnits(5L)
                    .withWriteCapacityUnits(5L);
            createTableRequest.setProvisionedThroughput(provisionedThroughput);
            client.createTable(createTableRequest);
        }
		
        // Get data from the API
        String apiUrl = "https://v2.jokeapi.dev/joke/Any";
        String jokeData = "";
        try {
            jokeData = readUrl(apiUrl);
        } catch (IOException e) {
            e.printStackTrace();
        }

        // Map the JSON data to the Joke class
        ObjectMapper objectMapper = new ObjectMapper();
        JsonNode rootNode = null;
        try {
            rootNode = objectMapper.readTree(jokeData);
        } catch (IOException e) {
            e.printStackTrace();
        }


        Joke joke = new Joke();
        joke.setError(rootNode.get("error").asBoolean());
        joke.setCategory(rootNode.get("category").asText());
        joke.setType(rootNode.get("type").asText());
        joke.setId(rootNode.get("id").asInt());

        System.out.println(joke.getType());
        if (joke.getType().toString().contains("twopart"))
        {
            joke.setSetup(rootNode.get("setup").asText());
            joke.setDelivery(rootNode.get("delivery").asText());
        }
        else
        {
            joke.setSetup(rootNode.get("joke").asText());
        }

        JsonNode flagsNode = rootNode.get("flags");
        Joke.Flags flags = new Joke.Flags();
        flags.setNsfw(flagsNode.get("nsfw").asBoolean());
        flags.setReligious(flagsNode.get("religious").asBoolean());
        flags.setPolitical(flagsNode.get("political").asBoolean());
        flags.setRacist(flagsNode.get("racist").asBoolean());
        flags.setSexist(flagsNode.get("sexist").asBoolean());
        flags.setExplicit(flagsNode.get("explicit").asBoolean());
        joke.setFlags(flags);

        joke.setSafe(rootNode.get("safe").asBoolean());
        joke.setLang(rootNode.get("lang").asText());

        // Add the joke to the table
        mapper.save(joke);

        // Retrieve the joke from the table to verify that it was added
        Joke retrievedJoke = mapper.load(Joke.class, joke.getId());

        int jokeid = retrievedJoke.getId();
        
        SendJokesMail c = new SendJokesMail();
        Boolean ans = c.sendEmployeMessage(jokeid);

        return ans;
    }

    // Helper method to read the API URL and return the response data as a string
    private static String readUrl(String urlString) throws IOException {
        BufferedReader reader = null;
        try {
            URL url = new URL(urlString);
            reader = new BufferedReader(new InputStreamReader(url.openStream()));
            StringBuffer buffer = new StringBuffer();
            int read;
            char[] chars = new char[1024];
            while ((read = reader.read(chars)) != -1)
                buffer.append(chars, 0, read);

            return buffer.toString();
        } finally {
            if (reader != null)
                reader.close();
        }
    }
}

```

If you see in our JSON format there are input tags inside another input tags. For example - 'nsfw', 'religious' tags are present inside the 'flags' tag. To handle this we will require an additional java class (JokeFlagsConverter.java)

```
{
    "error": false,
    "category": "Programming",
    "type": "single",
    "joke": "The generation of random numbers is too important to be left to chance.",
    "flags": {
        "nsfw": false,
        "religious": false,
        "political": false,
        "racist": false,
        "sexist": false,
        "explicit": false
    },
    "id": 39,
    "safe": true,
    "lang": "en"
}
```


**JokesFlagConverter.java**

```
package org.example;

import software.amazon.awssdk.enhanced.dynamodb.AttributeValueType;
import software.amazon.awssdk.enhanced.dynamodb.EnhancedType;
import software.amazon.awssdk.services.dynamodb.model.AttributeValue;
import software.amazon.awssdk.enhanced.dynamodb.AttributeConverter;

import java.util.HashMap;
import java.util.Map;

public class JokeFlagsConverter implements AttributeConverter<Joke.Flags> {

    @Override
    public AttributeValue transformFrom(Joke.Flags flags) {
        // Implement the logic to convert Joke.Flags object to AttributeValue

        Map<String, AttributeValue> flagsMap = new HashMap<>();
        flagsMap.put("nsfw", AttributeValue.builder().bool(flags.isNsfw()).build());
        flagsMap.put("religious", AttributeValue.builder().bool(flags.isReligious()).build());
        flagsMap.put("political", AttributeValue.builder().bool(flags.isPolitical()).build());
        flagsMap.put("racist", AttributeValue.builder().bool(flags.isRacist()).build());
        flagsMap.put("sexist", AttributeValue.builder().bool(flags.isSexist()).build());
        flagsMap.put("explicit", AttributeValue.builder().bool(flags.isExplicit()).build());

        return AttributeValue.builder().m(flagsMap).build();
    }

    @Override
    public Joke.Flags transformTo(AttributeValue attributeValue) {
        // Implement the logic to convert AttributeValue to Joke.Flags object

        Joke.Flags flags = new Joke.Flags();
        if (attributeValue != null && attributeValue.m() != null) {
            flags.setNsfw(attributeValue.m().containsKey("nsfw"));
            flags.setReligious(attributeValue.m().containsKey("religious"));
            flags.setPolitical(attributeValue.m().containsKey("political"));
            flags.setRacist(attributeValue.m().containsKey("racist"));
            flags.setSexist(attributeValue.m().containsKey("sexist"));
            flags.setExplicit(attributeValue.m().containsKey("explicit"));
        }
        return flags;
    }

    @Override
    public EnhancedType<Joke.Flags> type() {
        return null;
    }

    @Override
    public AttributeValueType attributeValueType() {
        return null;
    }
}

```


**CheckTable.java** 

```
package org.example;

import com.amazonaws.services.dynamodbv2.AmazonDynamoDB;
import com.amazonaws.services.dynamodbv2.AmazonDynamoDBClientBuilder;
import com.amazonaws.services.lambda.runtime.Context;
import com.amazonaws.services.lambda.runtime.RequestHandler;
import com.amazonaws.services.lambda.runtime.events.ScheduledEvent;
import com.amazonaws.services.dynamodbv2.model.ListTablesRequest;
import com.amazonaws.services.dynamodbv2.model.ListTablesResult;
import software.amazon.awssdk.regions.Region;

public class CheckTable implements RequestHandler<ScheduledEvent, String> {

    

    public boolean handleRequest(String TABLE_NAME) {

       //The name of your region where the AWS services are available.
        Region region = Region.US_WEST_2;

        AmazonDynamoDB client = AmazonDynamoDBClientBuilder.standard()
                .withRegion(String.valueOf(region))
                .build();
        ListTablesRequest request = new ListTablesRequest();

        ListTablesResult result = client.listTables(request);

        if (result.getTableNames().contains(TABLE_NAME)) {
            return true ;
        } else {
            return false;
        }
    }


    public String handleRequest(ScheduledEvent scheduledEvent, Context context) {
        return null;
    }
}


```

**SendJokesMail.java**

```
package org.example;

import com.amazonaws.regions.Regions;
import com.amazonaws.services.lambda.runtime.LambdaLogger;
import com.amazonaws.services.sns.AmazonSNS;
import com.amazonaws.services.sns.AmazonSNSClient;
import software.amazon.awssdk.enhanced.dynamodb.*;
import software.amazon.awssdk.enhanced.dynamodb.model.ScanEnhancedRequest;
import software.amazon.awssdk.regions.Region;
import software.amazon.awssdk.services.dynamodb.DynamoDbClient;
import software.amazon.awssdk.services.dynamodb.model.AttributeValue;
import software.amazon.awssdk.services.dynamodb.model.DynamoDbException;
//import software.amazon.awssdk.services.sns.model.PublishRequest;
import java.util.*;
//import com.amazonaws.services.lambda.runtime.events.SNSEvent;
import com.amazonaws.services.sns.AmazonSNSClientBuilder;
import com.amazonaws.services.sns.model.PublishRequest;
import com.amazonaws.services.sns.model.PublishResult;

public class SendJokesMail {
    public Boolean sendEmployeMessage(int jokeid) {

        Boolean send = false;
        
        //The name of your region where the AWS services are available.
        Region region = Region.US_WEST_2;
        DynamoDbClient ddb = DynamoDbClient.builder()
                .region(region)
                .build();

        // Create a DynamoDbEnhancedClient and use the DynamoDbClient object
        DynamoDbEnhancedClient enhancedClient = DynamoDbEnhancedClient.builder()
                .dynamoDbClient(ddb)
                .build();

        // Create a DynamoDbTable object based on jokes_tb_item
        DynamoDbTable<Joke> table = enhancedClient.table("JokesTable", TableSchema.fromBean(Joke.class));


        try {
            AttributeValue attVal = AttributeValue.builder()
                    .n(String.valueOf(jokeid))
                    .build();

            // Get only items in the jokes_tb_item table that match the date
            Map<String, AttributeValue> myMap = new HashMap<>();
            myMap.put(":val1", attVal);

            Map<String, String> myExMap = new HashMap<>();
            myExMap.put("#id", "id");

            Expression expression = Expression.builder()
                    .expressionValues(myMap)
                    .expressionNames(myExMap)
                    .expression("#id = :val1")
                    .build();

            System.out.println("Expression: " + expression.expression());
            System.out.println("Expression Values: " + expression.expressionValues());

            ScanEnhancedRequest enhancedRequest = ScanEnhancedRequest.builder()
                    .filterExpression(expression)
                    .limit(15) // you can increase this value
                    .build();

            // Get items in the dynamodb table
            Iterator<Joke> jokes_tb = table.scan(enhancedRequest).items().iterator();

            while (jokes_tb.hasNext()) {
                Joke jokes_tb_item = jokes_tb.next();
                if(jokes_tb_item != null) {
                    String Setup = jokes_tb_item.getSetup();


                    if (jokes_tb_item.getType().toString().contains("twopart"))
                    {
                        String delivery = jokes_tb_item.getDelivery();
                        String film = Setup + "\n" + delivery;
                        sentTextMessage(film);
                        send = true;
                    }
                    else {
                        sentTextMessage(Setup);
                        send = true;
                    }
                    

                }
                else {
                    sentTextMessage("No Message available");
                }
            }
        } catch (DynamoDbException e) {
            System.err.println(e.getMessage());
            System.exit(1);
        }
        

        return send;
    }


    // Use the Amazon SNS Service to send a text message
    private void sentTextMessage(String joke) {
        String message = joke;
        String subject = **Your subject name**;
        String topicArn = **Your ARN Topic name from the SNS that you created in AWS Console**;

        AmazonSNS snsClient = AmazonSNSClientBuilder.standard().withRegion(Regions.US_WEST_2).build();

        PublishRequest publishRequest = new PublishRequest(topicArn, message, subject);
        PublishResult publishResult = snsClient.publish(publishRequest);

    }

}

```


