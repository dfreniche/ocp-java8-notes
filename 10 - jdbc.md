theme: Simple
autoscale: true


# JDBC: Java Database Connectivity

---

## Using Apache Derby

- RazorSQL to inspect Derby DBs: http://razorsql.com/download_mac.html
- add Apache Derby JAR to classpath
    - open `module settings`
    - copy JDK path
    - add to Libraries `$JAVA_HOME/db/lib/derby.jar`

---

## BASIC SQL

-  DML (Data Manipulation Language)
    - SELECT
    - INSERT
    - DELETE
    - UPDATE
    - COUNT
- DDL (Data Definition Language)
    - DROP
    - ALTER
- ETC.

---

## After running, creates a Derby DB

- it's just a directory
- don't mess with it

---

## JDBC: Interfaces inside java.sql (for the exam)

- Driver
- Connection
- Statement
- ResultSet

Others have been added in javax.sql

---

## JDBC URL

```
protocol:vendor:database specific connection details

jdbc:postgres://localhost:5432/mydb

jdbc:oracle:thin:@localhost:1521:orcl

jdbc:oracle:thin:@[HOST][:PORT]:SID
```

- used to connect to the DB
- Several samples: https://www.ibm.com/support/knowledgecenter/en/SSEP7J_10.2.0/com.ibm.swg.ba.cognos.vvm_ag_guide.10.2.0.doc/c_ag_samjdcurlform.html

---

## Connect to a DB

- Interfaces: `java.sql.DriverManager` vs `javax.sql.DataSource`
- `DataSource`: factory, pooled, distributed
    - not covered in the exam!
- `DriverManager` covered in the exam
    - uses Factory pattern

---

## Creating a connection using DriverManager

```java
public static void main(String[] args) throws SQLException {
    String url = "jdbc:derby:peopledb;create=true";

    try (Connection connection = DriverManager.getConnection(url)) {
        System.out.println(connection);
    }
}
```

- also exists: `getConnection(url, user, password)`

---

## For reference only: using DataSource

```java
EmbeddedDataSource ds = new EmbeddedDataSource();
ds.setDatabaseName("test/mydb");

ds.setCreateDatabase("create");

try (Connection connection = ds.getConnection();
        Statement statement = connection.createStatement()) {

}

```

---

## Manually loading database Driver

- DriverManager search JARs that contain a `Driver` class
- looks for `META-INF/services/java.sql.Driver`
- what if our (old) driver does not have `META-INF/services/java.sql.Driver`?
    - Since JDBC 4.0 (Java 6) should be there

```java
try {
    Class.forName("org.apache.derby.jdbc.EmbeddedDriver");
} catch (ClassNotFoundException e) {
    e.printStackTrace();
}
```


---

## Creating a test DB from Java

```java
String url = "jdbc:derby:peopledb;create=true";

try (Connection connection = DriverManager.getConnection(url);  // this Throws Exception
    Statement statement = connection.createStatement()) {
    statement.executeUpdate("Drop table person");

    statement.executeUpdate("create table PERSON " +
            "(" +
            "id integer primary key," +
            "name varchar(255)" +
            ")");

    statement.executeUpdate("insert into PERSON VALUES (1, 'Diego')");
}
```

---

## Statement: sending SQL to the DB

```java
Statement statement = connection.createStatement()

```

---



## executeUpdate

- should be called _executedSomethingThatsGonnaChangeTheDB_
- returns the number of inserted, deleted or updated rows (or 0 if it's a DDL SQL sentence)

```java
i = statement.executeUpdate("insert into PERSON VALUES (1, 'Diego')");  // i == 1

i = statement.executeUpdate("Drop table person");   // i == 0

System.out.println(statement.getUpdateCount()); // getUpdateCount also returns this
```

---

## Inserting more than one record

```sql
INSERT INTO PERSON VALUES (0, 'Person 0'), (1, 'Person 1'), ...

```

```java
statement.executeUpdate("insert into PERSON VALUES (0, 'Person 0'), (1, 'Person 1')");
```

---

## Inserting more than one record

```java
StringBuffer insertSQL = new StringBuffer("insert into PERSON VALUES ");
for (int j = 0; j < 10; j++) {
    insertSQL.append(String.format("(%d, '%s')", j, "Person " + j));
    if (j < 9) {
        insertSQL.append(", ");
    }
}

System.out.println(insertSQL);

i = statement.executeUpdate(insertSQL.toString());
System.out.println(i);
```

---

## Deleting

---

## Updating

---

## Query

- a `executeQuery` call returns a `ResultSet` (a cursor)

```java
ResultSet rs = statement.executeQuery("SELECT * FROM PERSON");
while (rs.next()) {
    System.out.println(rs.getString(2));    // columns start in 1!
}
```

---

## Query

- we can also use `execute` and check if it returns a `ResultSet`

```java
boolean isResultSet = statement.execute("SELECT * FROM PERSON");
if (isResultSet) {
    ResultSet resultSet = statement.getResultSet();
    System.out.println(resultSet);
} else {
    int result = statement.getUpdateCount();
    System.out.println(result);
}

```

---

## Types of ResultSets

```java
Statement statement = connection.createStatement(ResultSet.TYPE_FORWARD_ONLY, ResultSet.CONCUR_READ_ONLY)
```

`ResultSet.TYPE_FORWARD_ONLY`: can move --> only, can't see last data (snapshop), supported
`ResultSet.TYPE_SCROLL_INSENSITIVE`: can move <-->, can't see last data (snapshop), supported
`ResultSet.TYPE_SCROLL_SENSITIVE`: can move <-->, can see last data (snapshop), not well supported

- createStatement does graceful degradation and returns the available ResultSet

---

## Concurrency modes

```
ResultSet.CONCUR_READ_ONLY: read only cursor
ResultSet.CONCUR_UPDATABLE: read/write cursor. Not required to be supported
```

- to change things in the DB we use Statements, not ResultSets

---


```java
 ResultSet resultSet = statement.getResultSet();
System.out.println(resultSet.getConcurrency() == ResultSet.CONCUR_UPDATABLE);
System.out.println(resultSet.getConcurrency() == ResultSet.TYPE_SCROLL_SENSITIVE);

```

---

## Forward-only means that!

```java
Statement statement = connection.createStatement(ResultSet.TYPE_FORWARD_ONLY, ResultSet.CONCUR_UPDATABLE)) {

ResultSet rs = statement.executeQuery("SELECT * FROM PERSON");
while (rs.next()) {
    System.out.println(rs.getString(2));
}

rs.first(); // Exception in thread "main" java.sql.SQLException: The 'first()' method is only allowed on scroll cursors.
System.out.println(rs.getString(2));

```

---

## Statement vs PreparedStatement

- Only `Statement` in the exam
- use `PreparedStatement` always
    - no SQL injection
    - best performance

---

## Using the wrong method

- a QUERY inside `executeUpdate`

```java
statement.executeUpdate("SELECT * FROM PERSON");
// Exception in thread "main" java.sql.SQLException: 
// Statement.executeUpdate() cannot be called with a statement that returns a ResultSet.
```

- a INSERT, UPDATE, DELETE inside `executeQuery`

```java
ResultSet rs = statement.executeQuery("DELETE FROM PERSON"); 
// Exception in thread "main" java.sql.SQLException:
// Statement.executeQuery() cannot be called with a statement that returns a row count.

while (rs.next()) {
    System.out.println(rs.getString(2));
}
```

---

|Statement Method|Ret|DELETE|INSERT|UPDATE|SELECT|
|----:|:-:|:-:|:-:|:-:|:-:|
|`execute()`|`boolean`|👍|👍|👍|👍|
|`execute Query()`|`ResultSet`|👎|👎|👎|👍|
|`execute Update()`|`int`|👍|👍|👍|👎|

---

## Looping through a ResultSet

- check ResultSet type: remember `ResultSet.TYPE_FORWARD_ONLY`, etc.
- initial position "beforeFirst"
    - `rs.beforeFirst()`, `rs.isBeforeFirst()`
- last position "after last"
    - `rs.afterLast()`, `rs.isAfterLast()`
- rs.previous(), 
- `rs.next()` -> returns true if not at the end

---

## Absolute

- moves to the absolute index provided
- returns true if there's a row there
- absolute(0) == before first
- absolute(1) == first row (if exists)
- absolute(-1) == last element
- absolute(-2) == element before last one

---

## Relative

- skips n rows from current position

---

## Accessing fields

- using name: getInt("id"), getString("name")
- using field position: getInt(1), getString(2) // starts with 1

---

## Counting

```java
ResultSet rsCount = statement.executeQuery("SELECT COUNT(*) FROM PERSON");
if (rsCount.next()) {
    String count = rsCount.getString(1);
    System.out.println(count);
}
```

---

## Times & Dates

|JDBC|Java 8|Contains|
|---|---|--:|
java.sql.Date | java.time.LocalDate | Date
java.sql.Time | java.time.LocalTime | Time
java.sql.TimeStamp | java.time. LocalDateTime | Date & Time

---

## Inserting TimeStamps

- convert into `java.sql.TimeStamp`

```java
// modify table

statement.executeUpdate("create table PERSON " +
                    "(" +
                    "id integer primary key," +
                    "name varchar(255)," +
                    "birthDate timestamp" + // added
                    ")");
// ...

StringBuffer insertSQL = new StringBuffer("insert into PERSON VALUES ");
for (int j = 0; j < 10; j++) {
    java.sql.Timestamp timestamp = new Timestamp(LocalDateTime.now().toEpochSecond(ZoneOffset.UTC));
    insertSQL.append(String.format("(%d, '%s', '%s')", j, "Person " + j, timestamp.toString()));
    if (j < 9) {
        insertSQL.append(", ");
    }
}
```

---

## Closing database

```java
try (Connection connection = DriverManager.getConnection(url);
    Statement statement = connection.createStatement(ResultSet.TYPE_SCROLL_SENSITIVE, ResultSet.CONCUR_UPDATABLE)) {
        // try-with-resources: closes resources in the opposite order as they were created
}

```

- closing Connection closes statement (and resultsets)
- closing Statement closes resultsets

---

## Closing resultsets

```java
ResultSet rs = statement.executeQuery("SELECT * FROM PERSON");

while (rs.next()) {
    System.out.println(rs.getString("name"));
}

boolean isResultSet = statement.execute("SELECT * FROM PERSON");    // closes here
```