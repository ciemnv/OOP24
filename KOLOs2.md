# Java

## Kolos2

### Obsługa bazy danych

```java
import java.sql.*;
try (
    Connection connection = DriverManager.getConnection("jdbc:sqlite:sample.db");
    Statement statement = connection.createStatement();
) {
    statement.executeUpdate("DROP TABLE IF EXISTS person");
    statement.executeUpdate("CREATE TABLE PERSON (id INTEGER, name STRING)");
    statement.executeUpdate("INSERT INTO PERSON VALUES(1, 'leo')");
    statement.executeUpdate("INSERT INTO PERSON VALUES(2, 'yui')");
    ResultSet rs = statement.executeQuery("SELECT * FROM person");
    while (rs.next()) {
        System.out.print("name = " + rs.getString("name"));
        System.out.print("id = " + rs.getInt("id"));
    }
} catch (SQLException e) {
e.printStackTrace(System.err); }
```
### Wstawianie informacji z parametru w bazie danych

```java
statement = connection.getConnection().prepareStatement("INSERT INTO auth_account(name, password) VALUES (?, ?);");
statement.setString(1, name);
```
