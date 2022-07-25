# Connect Java App to Database

## Learning Goals

- Set up persistence context for communicating with the database.
- Configure database communication settings.

## Introduction

We will be setting up the basic structure for interacting with the H2 database
and configure the database settings. Make sure the H2 database is running on
your machine by logging in on the browser before proceeding.

## Configure Persistence Context

The persistence context defines how JPA should behave when it connects to the
database. These configurations are added to the `persistence.xml` file.

1. Create a directory called `META-INF` in the `~src/main/resources` folder.
2. Create a `persistence.xml` file in the `META-INF` folder.
3. Your directory structure should look like the following:

   ```java
   ~/project-root-directory
   ├── pom.xml
   └── src
       ├── main
       │   ├── java
       │   │   └── org
       │   │       └── example
       │   │           └── JpaMain.java
       │   └── resources
       │       └── META-INF
       │           └── persistence.xml
       └── test
           └── java
   ```

4. Add the following to your `persistence.xml` file.

   ```xml
   <persistence xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
                xsi:schemaLocation="http://java.sun.com/xml/ns/persistence
                         http://java.sun.com/xml/ns/persistence/persistence_2_0.xsd"
                version="2.0" xmlns="http://java.sun.com/xml/ns/persistence">

       <persistence-unit name="example" transaction-type="RESOURCE_LOCAL">
           <provider>org.hibernate.ejb.HibernatePersistence</provider>
           <properties>
               <!-- connect to database -->
               <property name="javax.persistence.jdbc.url" value="jdbc:h2:tcp://localhost/~/test" />
               <property name="javax.persistence.jdbc.driver" value="org.h2.Driver" />
               <property name="javax.persistence.jdbc.user" value="sa" />
               <property name="javax.persistence.jdbc.password" value="" />
               <!-- configure behavior -->
               <property name="hibernate.show_sql" value="true"/>
               <property name="hibernate.format_sql" value="true"/>
               <property name="hibernate.dialect" value="org.hibernate.dialect.H2Dialect"/>
               <property name="hibernate.hbm2ddl.auto" value="create-drop" />
           </properties>
       </persistence-unit>
   </persistence>
   ```

Let’s look at what these properties do:

- `<persistence-unit>`: defines a single database your app will connect to.
- `<provider>`: defines the implementation of JPA that the app will be using.
- The properties under the “connect to database” comment defines the url,
  driver, username, and password for connecting to the database. The URL we are
  using is for connecting to the H2 database in server mode. It would be
  different if we were using the in-memory mode.
- Behavior configurations:
  - `hibernate.show_sql`: shows the SQL queries performed in the terminal.
  - `hibernate.format_sql`: formats SQL queries in the terminal to display them
    in an easier to read formate.
  - `hibernate.dialect`: defines the type of database which ensures database
    compatible SQL queries are generated.
  - `hibernate.hbm2ddl.auto`: defines initial startup behavior for database.

## Create an Entity

We will create a `Student` class and use it to create an associated table in the
H2 database. This will be a basic entity for now but later we will learn how to
customize the properties.

### Create the Class

1. Create a `models` package in the `org.example` package.
2. Create a `Student` class.
3. Your directory structure should look like this:

   ```xml
   ~/project-root-directory
   ├── pom.xml
   └── src
       ├── main
       │   ├── java
       │   │   └── org
       │   │       └── example
       │   │           ├── JpaMain.java
       │   │           └── models
       │   │               └── Student.java
       │   └── resources
       │       └── META-INF
       │           └── persistence.xml
       └── test
           └── java
   ```

### Define the Class

We have to add certain annotations to a class in order to tell JPA how to map it
to the database. We will be using the `@Entity`, `@Table`, and `@Id` annotations
from the `javax.persistence` package in this lesson.

Open your `Student.java` file and add the following code:

```java
package org.example.models;

import javax.persistence.Entity;
import javax.persistence.Id;
import javax.persistence.Table;

@Entity
@Table(name = "STUDENT_DATA")
public class Student {
    @Id
    private int id;
    private String name;

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
```

The `@Entity` annotation tells program that JPA has to manage this class and
will be using this to communicate with the database. The `@Table` annotation is
being used to set the table name in the database. Finally, the `@Id` annotation
specifies the property that will be used as the unique identifier for rows in
the database.

Note that we are using the default setters and getters generated by IntelliJ.
Later, we will have to create a few custom setters and getters when creating
models with relationships.

## Use the Persistence Context and Entity

We configured a persistence context in the `persistence.xml` file earlier. Now,
we have to use that context in our app to interact with the database. We have to
use an `EntityManager` to use our entity to communicate with the database.

Open up your `[JpaMain.java](http://JpaMain.java)` file and add the following
code:

```java
package org.example;

import org.example.models.Student;

import javax.persistence.Persistence;
import javax.persistence.EntityManagerFactory;
import javax.persistence.EntityManager;
import javax.persistence.EntityTransaction;

public class JpaMain {
    public static void main(String[] args) {
				// create a new student instance
        Student student1 = new Student();
        student1.setId(1);
        student1.setName("Jack");

				// create EntityManager
        EntityManagerFactory entityManagerFactory = Persistence.createEntityManagerFactory("example");
        EntityManager entityManager = entityManagerFactory.createEntityManager();

				// access transaction object
        EntityTransaction transaction = entityManager.getTransaction();

				// create and use transactions
        transaction.begin();
        entityManager.persist(student1);
        transaction.commit();
    }
}
```

In the “create a new student instance”, we are creating a new student instance
which is a regular Java object.

We need to use an `EntityManagerFactory` because we want a single instance of an
`EntityManager` in our app. We also have to get the `EntityTransaction` object
from the `EntityManager` because we have to define our own transactions. This is
boilerplate code required when working with JPA but these aren’t required when
using frameworks which provide auto configurations such as Spring (with
SpringBoot).

The `persist` method tells the database to create and insert a student row using
the data provided.

Make sure your H2 database is running in the terminal or command line and then
run the `main` method. This will create a `STUDENT_DATA` table in the H2
database along with a single row of data. In IntelliJ, you can check out the
“Run” tab to see the exact query that Hibernate used to create the table and
insert the data. It shows the SQL queries because we enabled this behavior in
the `persistence.xml` file.

```
Hibernate:

    drop table if exists STUDENT_DATA CASCADE
Hibernate:

    create table STUDENT_DATA (
       id integer not null,
        name varchar(255),

Hibernate:
    insert
    into
        STUDENT_DATA
        (name, id)
    values
        (?, ?)
```

Tip: If you click on the `STUDENT_DATA` table in the H2 browser GUI sidebar, it
will automatically create the SQL query. Click the “Run” button to execute the
SQL statement.

![H2 database SQL query example](https://curriculum-content.s3.amazonaws.com/java-spring-1/h2-sql-query-example.png)

## Conclusion

We have learned how to configure a persistence context, create an entity, and
interact with a database. In the following lessons, we will learn more about
entity mapping, object methods, and how to create relationships between models.
