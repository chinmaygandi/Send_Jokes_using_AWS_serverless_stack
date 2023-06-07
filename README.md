# AWS---Examples

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

This completes all the required dependencies for the project. 

**Step 3 [Creating AWS services in AWS console]**

1) We will first require a AWS SNS service. The email ids registered with this topic will recieve any email that we send. You can follow the steps mentioned in the link (https://docs.aws.amazon.com/sns/latest/dg/sns-create-topic.html) for creating an SNS topic and associating emailids with the same. It will look something like this.

![image](https://github.com/chinmaygandi/AWS---Examples/assets/131703516/bb715127-819c-4160-8e6f-4a72a65fa2ab)

Note the ARN of this topic as we will be using them in our code. 

2) The other service that we will require will be used 

**Step 4 [Coding the main logic for our project]**

Our project will contain below various different java classes. 

1) There will be a joke class [Joke.java] that will contain all the different attributes that our public joke API has such as error, category, nsfw etc. We can see in them here (https://v2.jokeapi.dev/). **We will creating the dynamo db table via code and not in the AWS console.**

2) There will be a handler class [joke_handler.java] that will be triggered when the AWS lambda function is called. This will contain all the main logic 

         a) To create dynamo db table
         b) Check if the table already exists (with the help of class file Checktable.java)
         c) Convert the jokes record which is in json format that we get from public API to String 
         d) Insert in dynamo db table as a record 
         e) Send the joke to the SNS topic


Joke.java
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


 


