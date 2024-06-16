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

### Ustawianie hasla z pomoca biblioteki BCrypt
```java
statement.setString(2, BCrypt.withDefaults().hashToString(12, password.toCharArray()));
```


### Tworzenie nowego wątku - extends Thread

### Tworzenie nowego wątku - interfejs Runnable
```java
public class Counter implements Runnable {
    @Override
    public void run() {
        for(int i = 0; i < 10; ++i)
            System.out.print(i);
    }
}

Counter counter1 = new Counter();
Counter counter2 = new Counter();

Thread counterThread1 = new Thread(counter1);
Thread counterThread2 = new Thread(counter2);

counterThread1.start();
counterThread2.start();
```

### Aktualny stan wątku
```java
myThread.getState();
```

### Runnable jako interfejs funkcyjny + executorService (lepszy od executor)
```java
int[] ints = new int[100000];
Arrays.fill(ints, 1);

Thread thread = new Thread(() -> {
    int sum = 0;
    for (int num : ints)
        sum += num;
    System.out.println(sum);
});
thread.start();

ExecutorService executor = Executors.newSingleThreadExecutor();
executor.submit(() -> {
    int sum = 0;
    for (int num : ints)
        sum += num;
    System.out.println(sum);
});
executor.shutdown();
```

### Klasa Graphics
#### Służy do rysowania obiektów w javie - kształtów, tekstu, obrazów, kolorów i czcionek
```java
Graphics graphics = new Graphics();
int width = 256; //szerokosc
int height = 200; //wysokosc

BufferedImage histogramImage = new BufferedImage(width, height, BufferedImage.TYPE_INT_RGB);
Graphics graphics = histogramImage.getGraphics();

 //wypełnianie tła na biało - najpierw ustawiamy kolor, potem coś robimy
graphics.setColor(Color.WHITE);
graphics.fillRect(0, 0, width, height);

//Rysowanie słupka od punktu (x1, y2) do punktu (x2, y2)
graphics.setColor(Color.BLACK);
graphics.drawLine(x1, y1, x2, y1);

//zapisywanie obrazu do pliku
File outputFile = new File(outputPath);
ImageIO.write(histogramImage, "png", outputFile);
```

### Wygenerowana aplikacja Spring Boot
```java
@SpringBootApplication
public class WebappApplication {
    public static void main(String[] args) {
        SpringApplication.run(WebappApplication.class, args);
    }
}
```
####Aplikacja uruchamia się na lokalnym serwerze pod adresem http://localhost:8080.

###Przykład prostego kontrolera w Spring Boot:

```java
@RestController
public class ProductController {
    @GetMapping
    String hello() {
        return "Hello World!";
    }
}
```

###Mapowanie URL
```java
@RestController
@RequestMapping("product")
public class ProductController {
    @GetMapping("hello")
    String hello() {
        return "Hello World!";
    }
}
```
#### Żądanie GET pod adresem http://localhost:8080/product/hello zwróci "Hello World!".


### Dodawanie parametrów do żądania:
```java
@RestController
@RequestMapping("product")
public class ProductController {
    @GetMapping("hello/{who}")
    String hello(@PathVariable String who) {
        return String.format("Hello %s!", who);
    }
}
```
####Żądanie GET z parametrem http://localhost:8080/product/hello?who=friend zwróci "Hello friend!".

### Dodawanie parametrów w ścieżce URL:
```java
@RestController
@RequestMapping("product")
public class ProductController {
    @GetMapping("hello/{who}")
    String hello(@PathVariable String who) {
        return String.format("Hello %s!", who);
    }
}
```
#### Żądanie GET pod adresem http://localhost:8080/product/hello/friend zwróci "Hello friend!".

###Przeciążanie mapowań
```java
@GetMapping("")
List<Product> getItem() {
    return products;
}

@GetMapping("{index}")
Product getItem(@PathVariable int index) {
    return products.get(index);
}
```

###Konfiguracja ekranu błędu
```java
@RestController
public class CustomErrorController implements ErrorController {
    @RequestMapping("/error")
    public ResponseEntity<String> handleError(HttpServletResponse response) {
        Integer status = response.getStatus();
        HttpStatus httpStatus = HttpStatus.valueOf(status);
        if (httpStatus == HttpStatus.NOT_FOUND) {
            return new ResponseEntity<>("Error 404 - not found", HttpStatus.NOT_FOUND);
        } else {
            return new ResponseEntity<>(String.format("Error %d", status), httpStatus);
        }
    }
}
```

