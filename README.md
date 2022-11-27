# Connect Java App to Database

## Learning Goals

- Set up the persistence context for communicating with a PostgreSQL database.
- Define a JPA entity using `@Entity`, `@Table`, and `@Id` annotations.
- Use `EntityManagerFactory` and `EntityManager` to manage an entity.
- Use the `persist` method of `EntityManager` to write an entity to the database.
- Use the `find` method of `EntityManager` to retrieve an entity from the database.

## CODE ALONG

## Introduction

We will setup the basic project structure for interacting
with a PostgreSQL database
and configure the database settings for Hibernate.

## Create a new PostgreSQL database

Use the **pgAdmin** tool to create a new database named `student_db`:    

![new student database](https://curriculum-content.s3.amazonaws.com/6036/connect-app-to-database/create_db.png)


## Configure Persistence Context

The persistence context defines how JPA should behave when it connects to the
database. These configurations are added to the `persistence.xml` file.

1. Create a directory named `META-INF` in the `~src/main/resources` folder.
2. Create a file named `persistence.xml` in the `META-INF` folder.
3. Your project directory structure should look like the following:  

![project structure persistence.xml](https://curriculum-content.s3.amazonaws.com/6036/connect-app-database/persistence_xml.png)  

4. Add the following to `persistence.xml`.

```xml
<persistence xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xsi:schemaLocation="http://java.sun.com/xml/ns/persistence
                         http://java.sun.com/xml/ns/persistence/persistence_2_0.xsd"
             version="2.0" xmlns="http://java.sun.com/xml/ns/persistence">

  <persistence-unit name="example" transaction-type="RESOURCE_LOCAL">
    <provider>org.hibernate.ejb.HibernatePersistence</provider>
    <properties>
      <!-- connect to database -->
      <property name="javax.persistence.jdbc.driver" value="org.postgresql.Driver" /> <!-- DB Driver -->
      <property name="javax.persistence.jdbc.url" value="jdbc:postgresql://localhost:5432/student_db" /> <!--DB URL-->
      <property name="javax.persistence.jdbc.user" value="postgres" /> <!-- DB User -->
      <property name="javax.persistence.jdbc.password" value="postgres" /> <!-- DB Password -->
      <!-- configure behavior -->
      <property name="hibernate.hbm2ddl.auto" value="update" /> <!-- create / create-drop / update -->
      <property name="hibernate.dialect" value="org.hibernate.dialect.PostgreSQL94Dialect"/>
      <property name="hibernate.show_sql" value="true" /> <!-- Show SQL in console -->
      <property name="hibernate.format_sql" value="true" /> <!-- Show SQL formatted -->
    </properties>
  </persistence-unit>
</persistence>
```

Let’s look at what these properties do:

- `<persistence-unit>`: defines a single database your app will connect to.
- `<provider>`: defines the implementation of JPA that the app will be using.
- The properties under the “connect to database” comment defines the driver,
  url, user, and password for connecting to the database.
- Behavior configurations:
  - `hibernate.hbm2ddl.auto`: defines initial startup behavior for database.
  - `hibernate.dialect`: defines the type of database which ensures database
    compatible SQL queries are generated.
  - `hibernate.show_sql`: shows the SQL queries performed in the terminal.
  - `hibernate.format_sql`: formats SQL queries in the terminal to display them
    in an easier to read format.
  

## Create an Entity

We will create a `Student` class and use it to create an associated table in the
PostgreSQL database. This will be a basic entity for now but later we will learn how to
customize the properties. 

### Create the Class

1. Create a package named `model` in the `org.example` package.
2. Create a `Student` class in the `org.example.model directory.
3. Your project directory structure should look like this:

![model package](https://curriculum-content.s3.amazonaws.com/6036/connect-app-database/model_student.png)

### Define the Class

We have to add certain annotations to a class in order to tell JPA how to map it
to the database. We will be using the `@Entity`, `@Table`, and `@Id` annotations
from the `javax.persistence` package in this lesson.

Edit the `Student` class and add the following code:

```java
package org.example.model;

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

    @Override
    public String toString() {
        return "Student{" +
                "id=" + id +
                ", name='" + name + '\'' +
                '}';
    }
}

```


- The `@Entity` annotation tells program that JPA has to manage this class and
  will be using this to communicate with the database.
- The `@Table` annotation sets the table name in the database.  The class name is used if this is omitted.
- The `@Id` annotation specifies the property that will be used
  as the unique identifier (i.e. primary key) for rows in the database.

A class with the `@Entity` annotation should have a no-args constructor.
Java generates a default no-args constructor for `Student` since a constructor
is not explicitly defined.  The entity class must have a primary key that uniquely identifies it.
It is also recommended to include setter methods.

Note that we are using the default setters and getters generated by IntelliJ.
In a subsequent lesson, we will create a few custom setters and getters when creating
models with relationships.

## Persist a `Student` entity to the database 

We configured a persistence context in the `persistence.xml` file earlier. Now,
we have to use that context in our app to interact with the database. We
use an `EntityManager` to allow the entity to communicate with the database.

Edit the `JpaCreateStudent` class and add the following code:

```java
package org.example;

import org.example.model.Student;
import javax.persistence.EntityManager;
import javax.persistence.EntityManagerFactory;
import javax.persistence.EntityTransaction;
import javax.persistence.Persistence;

public class JpaCreateStudent {
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

        //close entity manager and factory
        entityManager.close();
        entityManagerFactory.close();
    }
}
```

1. Create a `Student` which is a regular Java object.
2. Create an `EntityManagerFactory` because we want a single instance of an
   `EntityManager` in our app. The parameter to `createEntityManagerFactory` should
   match the string set in the persistence unit  `<persistence-unit name="example" transaction-type="RESOURCE_LOCAL">`.
3. Get an `EntityTransaction` object from the `EntityManager` because we have
   to define our own transactions to persist the student in the database.
4. The `persist` method tells the database to create and insert a table row using
   the data provided by the `Student` class instance.
5. Commit the transaction to ensure the row is inserted in the table.
6. Close the entity manager and factory.

This is boilerplate code required when working with JPA, but this code is not needed when
using frameworks that provide auto configurations such as Spring (with SpringBoot).

### Run `JpaCreateStudent.main`

1. Run the `JpaCreateStudent.main` method. This will create a `STUDENT_DATA`
   table in the PostgreSQL `student_db` database and add a new row. 
2. In IntelliJ, check out the “Run” tab to see the exact query that Hibernate used
   to create the table and insert the data. It shows the SQL queries because we enabled this behavior in
   the `persistence.xml` file.

```text
Hibernate: 
    
    create table STUDENT_DATA (
       id int4 not null,
        name varchar(255),
        primary key (id)
    )
Hibernate: 
    insert 
    into
        STUDENT_DATA
        (name, id) 
    values
        (?, ?)

```

3. Use the **pgAdmin** query tool to confirm the table was created and one row inserted: 

![pgadmin student_data query](https://curriculum-content.s3.amazonaws.com/6036/jpa-connect-app-database/query_student_data.png)

## Read a `Student` entity from the database

Edit the `JpaReadStudent` class and add the following code:

```java
package org.example;

import org.example.model.Student;
import javax.persistence.EntityManager;
import javax.persistence.EntityManagerFactory;
import javax.persistence.Persistence;

public class JpaReadStudent {
    public static void main(String[] args) {
        // create EntityManager
        EntityManagerFactory entityManagerFactory = Persistence.createEntityManagerFactory("example");
        EntityManager entityManager = entityManagerFactory.createEntityManager();

        // get student data using primary key id=1
        Student student1 = entityManager.find(Student.class, 1);
        System.out.println(student1);

        //close entity manager and factory
        entityManager.close();
        entityManagerFactory.close();
    }
}


```

1. Use `EntityManagerFactory` to create a single instance of `EntityManager`.  
2. The `find` method returns a `Student` object based on the method parameters:   
   - `Student.class` indicates the entity class, which provides information about the database table and primary key.
   - `1` indicates the primary key value.

The `find` method returns null if the entity is not in the database.

### Run `JpaReadStudent.main`

1. Run the `JpaReadStudent.main` method. This will query the database for the student entity with `id=1`.
2. In IntelliJ, check out the “Run” tab to see the exact query that Hibernate used
   to find the entity. 

```text
Hibernate: 
    select
        student0_.id as id1_0_0_,
        student0_.name as name2_0_0_ 
    from
        STUDENT_DATA student0_ 
    where
        student0_.id=?

```

3. The `JpaReadStudent.main` implicitly calls the `toString()` method to print the `Student` object state: 

```text
Student{id=1, name='Jack'}
```

## Conclusion

We have learned how to configure a persistence context, create an entity and persist it to the database.
We subsequently queried the database to retrieve the entity. Notice we did this without writing
any SQL!  In the following lessons, we will learn more about
entity mapping, object methods, and how to create relationships between models.

## Resources

[javax.persistence package](https://docs.oracle.com/javaee/7/api/javax/persistence/package-summary.html)
