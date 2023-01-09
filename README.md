# Additional ER Modeling Examples

## Learning Goals

- Model multiple relationships between a pair of entity types
- Model a recursive/unary/self relationship within a single entity type

## Introduction

This lesson will explore additional examples of entity relationships.
We will see how to model and implement multiple relationships between
a pair of entity types. This often occurs when an entity may have multiple roles
in a relationship. We will also look at a relationship between an entity
and another entity of the same type, which is referred
to as a recursive or unary or self relationship.

## Modeling Multiple Relationships Between an Entity Pair

Sometimes a pair of entity types may have more than one relationship between them.

Consider a sporting event such as a basketball game during the
regular season. Each game is played between two teams: the home team and the away team.
For example, the Cleveland Cavaliers play some games at their home stadium in Cleveland,
where they are considered the home team.  When the Cavaliers play games
at another team's home stadium they are considered the away or visiting team.
We could model the relationship between
a game and a team as many-to-many (each team plays multiple games, each game consists of multiple
teams), but that design does not accurately reflect
the cardinality (a game is between exactly two teams) and it does not capture the **role**
each team plays for a particular game (home team vs away team).

A more accurate design would include two separate one-to-many relationships
between game and team:   
- The first relationship models the "home" association between a team and a game.
  - A game has one home team. A team plays many home games.
- The second relationship models the "away" association between a team and a game. 
  - A game has one away team. A team plays many away games.

![basketball erd](https://curriculum-content.s3.amazonaws.com/6036/java-mod-5-ermodeling/basketball.png)

`BasketballGame` owns the relationships since it is on the many side.
Each `BasketballGame` has one reference to the home team, along with
a second reference to the away team:

```java
    @ManyToOne
    @JoinColumn(name="home_team", nullable=false)
    BasketballTeam homeTeam;

    @ManyToOne
    @JoinColumn(name="away_team", nullable=false)
    BasketballTeam awayTeam;
```

The non-owning side `BasketballTeam` stores a collection
of home game references, along with a collection of away game references:

```java
    @OneToMany(mappedBy = "homeTeam")
    private Set<BasketballGame> homeGames = new HashSet<>();

    @OneToMany(mappedBy = "awayTeam")
    private Set<BasketballGame> awayGames = new HashSet<>();
```

The code for the `BasketballGame` class is as shown:

```java
package org.example.basketball;

import javax.persistence.*;
import java.time.LocalDateTime;

@Entity
@Table(name="Basketball_Game")
public class BasketballGame {
    @Id
    @GeneratedValue
    private int id;

    @Column(nullable = false, name = "scheduled_date")
    private LocalDateTime scheduledDate;

    @ManyToOne
    @JoinColumn(name="home_team", nullable=false)
    BasketballTeam homeTeam;

    @ManyToOne
    @JoinColumn(name="away_team", nullable=false)
    BasketballTeam awayTeam;

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public LocalDateTime getScheduledDate() {
        return scheduledDate;
    }

    public void setScheduledDate(LocalDateTime scheduledDate) {
        this.scheduledDate = scheduledDate;
    }

    public BasketballTeam getHomeTeam() {
        return homeTeam;
    }

    public void setHomeTeam(BasketballTeam homeTeam) {
        this.homeTeam = homeTeam;
    }

    public BasketballTeam getAwayTeam() {
        return awayTeam;
    }

    public void setAwayTeam(BasketballTeam awayTeam) {
        this.awayTeam = awayTeam;
    }

    @Override
    public String toString() {
        return "BasketballGame{" +
                "id=" + id +
                ", scheduled_date=" + scheduledDate +
                '}';
    }
}
```

The code for the `BasketballTeam` class is as shown:

```java
package org.example.basketball;

import javax.persistence.*;
import java.util.HashSet;
import java.util.Set;

@Entity
@Table(name="Basketball_Team")
public class BasketballTeam {
    @Id
    @GeneratedValue
    private int id;

    @Column(nullable = false)
    private String name;

    @Column(nullable = false)
    private String city;

    @Column(nullable = false)
    private String state;

    @OneToMany(mappedBy = "homeTeam")
    private Set<BasketballGame> homeGames = new HashSet<>();

    @OneToMany(mappedBy = "awayTeam")
    private Set<BasketballGame> awayGames = new HashSet<>();

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

    public String getCity() {
        return city;
    }

    public void setCity(String city) {
        this.city = city;
    }

    public String getState() {
        return state;
    }

    public void setState(String state) {
        this.state = state;
    }

    public Set<BasketballGame> getHomeGames() {
        return homeGames;
    }

    public Set<BasketballGame> getAwayGames() {
        return awayGames;
    }

    @Override
    public String toString() {
        return "BasketballTeam{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", city='" + city + '\'' +
                ", state='" + state + '\'' +
                '}';
    }
}
```

Let's create some teams and games.

Each team plays one or more home games, along with one or more away games:

```java
package org.example.basketball;

import javax.persistence.EntityManager;
import javax.persistence.EntityManagerFactory;
import javax.persistence.EntityTransaction;
import javax.persistence.Persistence;
import java.time.LocalDateTime;
import java.time.Month;

public class JpaCreateBasketball {
    public static void main(String[] args) {
        // create Entities
        BasketballTeam team1 = new BasketballTeam();
        team1.setName("Cleveland Cavaliers");
        team1.setCity("Cleveland");
        team1.setState("OH");

        BasketballTeam team2 = new BasketballTeam();
        team2.setName("Golden State Warriors");
        team2.setCity("San Francisco");
        team2.setState("CA");

        BasketballTeam team3 = new BasketballTeam();
        team3.setName("Dallas Mavericks");
        team3.setCity("Dallas");
        team3.setState("TX");

        BasketballGame game1 = new BasketballGame();
        game1.setScheduledDate(LocalDateTime.of(2023, Month.FEBRUARY,1,18,30));

        BasketballGame game2 = new BasketballGame();
        game2.setScheduledDate(LocalDateTime.of(2023,Month.FEBRUARY,10,18,30));

        BasketballGame game3 = new BasketballGame();
        game3.setScheduledDate(LocalDateTime.of(2023,Month.MARCH,8,15,30));

        BasketballGame game4 = new BasketballGame();
        game4.setScheduledDate(LocalDateTime.of(2023,Month.MARCH,5,20,30));


        // create relationships
        game1.setHomeTeam(team1);  //Cleveland Cavaliers
        game1.setAwayTeam(team2);  //Golden State Warriors

        game2.setHomeTeam(team2);  //Golden State Warriors
        game2.setAwayTeam(team3);  //Dallas Mavericks

        game3.setHomeTeam(team2);  //Golden State Warriors
        game3.setAwayTeam(team1);  //Cleveland Cavaliers

        game4.setHomeTeam(team3);  //Dallas Mavericks
        game4.setAwayTeam(team1);  //Cleveland Cavaliers

        // create EntityManager
        EntityManagerFactory entityManagerFactory = Persistence.createEntityManagerFactory("example");
        EntityManager entityManager = entityManagerFactory.createEntityManager();

        // access transaction object
        EntityTransaction transaction = entityManager.getTransaction();

        // create and use transactions
        transaction.begin();

        //persist the objects
        entityManager.persist(team1);
        entityManager.persist(team2);
        entityManager.persist(team3);
        entityManager.persist(game1);
        entityManager.persist(game2);
        entityManager.persist(game3);
        entityManager.persist(game4);

        transaction.commit();

        //close entity manager and factory
        entityManager.close();
        entityManagerFactory.close();
    }
}
```

Finally, let's retrieve some data to confirm a few of the
one-to-many relationships between `Team` and `Game`:

```java
package org.example.basketball;

import javax.persistence.EntityManager;
import javax.persistence.EntityManagerFactory;
import javax.persistence.Persistence;

public class JpaReadBasketball {
    public static void main(String[] args) {
        // create EntityManager
        EntityManagerFactory entityManagerFactory = Persistence.createEntityManagerFactory("example");
        EntityManager entityManager = entityManagerFactory.createEntityManager();

        BasketballGame game = entityManager.find(BasketballGame.class, 4);
        System.out.println(game);
        System.out.println("home team: " + game.getHomeTeam());
        System.out.println("away team: " + game.getAwayTeam());

        BasketballTeam team = entityManager.find(BasketballTeam.class, 1);
        System.out.println(team);
        System.out.println("home games " + team.getHomeGames());
        System.out.println("away games " + team.getAwayGames());

        // close entity manager and factory
        entityManager.close();
        entityManagerFactory.close();
    }
}
```

The output is as shown:

```text
BasketballGame{id=4, scheduled_date=2023-02-01T18:30}
home team: BasketballTeam{id=1, name='Cleveland Cavaliers', city='Cleveland', state='OH'}
away team: BasketballTeam{id=2, name='Golden State Warriors', city='San Francisco', state='CA'}
BasketballTeam{id=1, name='Cleveland Cavaliers', city='Cleveland', state='OH'}
home games [BasketballGame{id=4, scheduled_date=2023-02-01T18:30}]
away games [BasketballGame{id=6, scheduled_date=2023-03-08T15:30}, BasketballGame{id=7, scheduled_date=2023-03-05T20:30}]
```


## Recursive/Unary/Self Relationships

A recursive relationship, also called a unary or self relationship,
exists between occurrences of the same entity set. In this relationship, the primary 
and foreign keys are the same data type, but they represent two objects with different roles.

For example, consider the relationship between an employee and their supervisor.

- Let's assume an employee has at most one direct supervisor.
- A supervisor may supervise many employees. The set of supervised employees
  are referred to as reportees
  (since they report directly to their supervisor).
- A supervisor is also an employee who may have a supervisor.  
- The relationship is optional since an employee may not have a supervisor
  (i.e. the company president or CEO)

There are many ways to design this data model, but one approach is to
use a single entity `Employee` having a supervising relationship to itself.
The relationship is one-to-many between `Employee` and `Employee`:  

![employee erd](https://curriculum-content.s3.amazonaws.com/6036/java-mod-5-ermodeling/employee.png)

The `Employee` entity is on both the owning (many) and non-owning (one) side of the relationship!

- Each employee is supervised by at most one employee.   

  ```java
  @ManyToOne
  private Employee supervisor;
  ```
  
- An employee may supervise many employees (labeled reportees).  
  Note the collection will be empty for employees who are not supervisors.   

  ```java
  @OneToMany(mappedBy = "supervisor")
  private Set<Employee> reportees = new HashSet<>();
  ```

The Java class to implement the `Employee` entity is as shown:

```java
package org.example.employeeV1;

import javax.persistence.*;
import java.util.HashSet;
import java.util.Set;

@Entity
public class Employee {
    @Id
    @GeneratedValue
    private int id;

    @Column(nullable = false)
    private String name;

    private Double salary;

    @ManyToOne
    private Employee supervisor;

    @OneToMany(mappedBy = "supervisor")
    private Set<Employee> reportees = new HashSet<>();

    public int getId() {return id;}

    public void setId(int id) {this.id = id;}

    public String getName() {return name;}

    public void setName(String name) {this.name = name;}

    public Double getSalary() {return salary;}

    public void setSalary(Double salary) {this.salary = salary;}

    public Employee getSupervisor() {return supervisor;}

    public void setSupervisor(Employee supervisor) {this.supervisor = supervisor;}

    public Set<Employee> getReportees() {return reportees;}

    @Override
    public String toString() {
        return "Employee{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", salary=" + salary +
                '}';
    }
}
```


Let's assume we have 4 employees:    
- #1 is supervised by #3.
- #2 is supervised by #3.
- #3 is supervised by #4.
- #4 does not have a supervisor.

We can create this set of employees with the following Java class:

```java
package org.example.employeeV1;

import javax.persistence.EntityManager;
import javax.persistence.EntityManagerFactory;
import javax.persistence.EntityTransaction;
import javax.persistence.Persistence;

public class JpaCreateEmployee {
    public static void main(String[] args) {
        // create Entities
        Employee employee1 = new Employee();
        employee1.setName("Kai");
        employee1.setSalary(80000.0);

        Employee employee2 = new Employee();
        employee2.setName("Kim");
        employee2.setSalary(70000.0);

        Employee employee3 = new Employee();
        employee3.setName("Kalani");
        employee3.setSalary(130000.0);

        Employee employee4 = new Employee();
        employee4.setName("Kiran");
        employee4.setSalary(140000.0);

        // create relationships
        employee1.setSupervisor(employee3);
        employee2.setSupervisor(employee3);
        employee3.setSupervisor(employee4);

        // create EntityManager
        EntityManagerFactory entityManagerFactory = Persistence.createEntityManagerFactory("example");
        EntityManager entityManager = entityManagerFactory.createEntityManager();

        // access transaction object
        EntityTransaction transaction = entityManager.getTransaction();

        // create and use transactions
        transaction.begin();

        //persist the objects
        entityManager.persist(employee1);
        entityManager.persist(employee2);
        entityManager.persist(employee3);
        entityManager.persist(employee4);

        transaction.commit();

        //close entity manager and factory
        entityManager.close();
        entityManagerFactory.close();
    }
}
```

The employee database table stores the relationships in a column labeled`supervisor_id`,
which is a foreign key holding the primary key id of another employee:

| id  | name   | salary | supervisor_id |
|-----|--------|--------|---------------|
| 1   | Kai    | 80000  | 3             |
| 2   | Kim    | 70000  | 3             |
| 3   | Kalani | 130000 | 4             |
| 4   | Kiran  | 140000 | null          |


We can retrieve some employee data to check the supervising relationships:

```java
package org.example.employeeV1;


import javax.persistence.EntityManager;
import javax.persistence.EntityManagerFactory;
import javax.persistence.Persistence;
import java.util.List;

public class JpaReadEmployee {
    public static void main(String[] args) {
        // create EntityManager
        EntityManagerFactory entityManagerFactory = Persistence.createEntityManagerFactory("example");
        EntityManager entityManager = entityManagerFactory.createEntityManager();

        List<Employee> employees = entityManager.createQuery("SELECT e FROM Employee e ORDER BY id", Employee.class).getResultList();
        for (Employee e: employees) {
            System.out.println(e);
            System.out.println("supervisor: " + e.getSupervisor());
            System.out.println("reportees: " + e.getReportees());
            System.out.println();
        }

        // close entity manager and factory
        entityManager.close();
        entityManagerFactory.close();
    }
}
```

The program output is as shown:

```text
Employee{id=1, name='Kai', salary=80000.0}
supervisor: Employee{id=3, name='Kalani', salary=130000.0}
reportees: []

Employee{id=2, name='Kim', salary=70000.0}
supervisor: Employee{id=3, name='Kalani', salary=130000.0}
reportees: []

Employee{id=3, name='Kalani', salary=130000.0}
supervisor: Employee{id=4, name='Kiran', salary=140000.0}
reportees: [Employee{id=2, name='Kim', salary=70000.0}, Employee{id=1, name='Kai', salary=80000.0}]

Employee{id=4, name='Kiran', salary=140000.0}
supervisor: null
reportees: [Employee{id=3, name='Kalani', salary=130000.0}]
```

## Conclusion

An ERD may contain more than one relationship between a pair of entity types.
This occurs when an entity type may have multiple roles in relation
to another entity type. 

It is also possible that an entity may have an association
to another entity of the same type, which is referred
to as a recursive/unary/self relationship.  A single Java class will implement
both the owning and non-owning side of the relationship.


## DBML Solutions

Basketball ERD:

```text
Table Basketball_Game {
  id INTEGER PK
  scheduled_date datetime
  home_team_id INTEGER [NOT NULL, ref: > Basketball_Team.id]
  away_team_id INTEGER [NOT NULL, ref: > Basketball_Team.id]
}

Table Basketball_Team {
  id INTEGER PK
  name TEXT [NOT NULL]
  city TEXT [NOT NULL]
  state TEXT [NOT NULL]
}
```
Employee ERD:

```text
Table Employee {
  id INTEGER PK
  name TEXT [NOT NULL]
  salary numeric
  supervisor_id INTEGER [ref: > Employee.id]
}
```