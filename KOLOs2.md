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

### Link do generowania Spring Boot:
#### https://start.spring.io - dodajemy dependencies w zaleznosci, jakie potrzebujemy

### Wygenerowana aplikacja Spring Boot
#### Aplikacja uruchamia się na lokalnym serwerze pod adresem http://localhost:8080.
```java
@SpringBootApplication
public class WebappApplication {
    public static void main(String[] args) {
        SpringApplication.run(WebappApplication.class, args);
    }
}
```

### Przykład kontrolera w Spring Boot:
```java
@RestController
public class ProductController {
    @GetMapping
    String hello() {
        return "Hello World!";
    }
}
```

### Mapowanie URL
#### Żądanie GET pod adresem http://localhost:8080/product/hello zwróci "Hello World!".
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


### Dodawanie parametrów do żądania:
#### Żądanie GET z parametrem http://localhost:8080/product/hello?who=friend zwróci "Hello friend!".
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

### Dodawanie parametrów w ścieżce URL:
#### Żądanie GET pod adresem http://localhost:8080/product/hello/friend zwróci "Hello friend!".
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

### Przeciążanie mapowań
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

### Konfiguracja ekranu błędu
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

### Wyświetlenie obiektu Rectangle zmapowanego na JSON - kontroler
```java

    @GetMapping("rect")
    public Rectangle rectangle() {
        return new Rectangle(20, 10, 22, 123, "black");
    }
```

### Dekodowanie i kodowanie obrazu do base64
```java
 @GetMapping("/adjustBrightness")
    public String adjustBrightness(@RequestParam String base64Image, @RequestParam int brightness) throws IOException {
        // Dekodowanie obrazu z base64
        byte[] imageBytes = Base64.getDecoder().decode(base64Image);
        ByteArrayInputStream bis = new ByteArrayInputStream(imageBytes);
        BufferedImage image = ImageIO.read(bis);

        // Zwiększenie jasności obrazu
        BufferedImage brightenedImage = increaseBrightness(image, brightness);

        // Kodowanie obrazu do base64
        ByteArrayOutputStream bos = new ByteArrayOutputStream();
        ImageIO.write(brightenedImage, "png", bos);
        byte[] brightenedImageBytes = bos.toByteArray();
        return Base64.getEncoder().encodeToString(brightenedImageBytes);
    }
```

### Zwiększanie jasności obrazu
```java
 private BufferedImage increaseBrightness(BufferedImage image, int value) {
        int width = image.getWidth();
        int height = image.getHeight();

        for (int y = 0; y < height; y++) {
            for (int x = 0; x < width; x++) {
                Color color = new Color(image.getRGB(x, y));
                int r = clamp(color.getRed() + value);
                int g = clamp(color.getGreen() + value);
                int b = clamp(color.getBlue() + value);
                int newColor = new Color(r, g, b).getRGB();
                image.setRGB(x, y, newColor);
            }
        }
        return image;
    }
//pomocnicza metoda clamp
private int clamp(int i) {
        return Math.max(0, Math.min(i, 255));
    }

```

### Gniazdo serwerowe
```java
public class Server {
    private ServerSocket serverSocket;

    public Server() throws IOException {
        serverSocket = new ServerSocket(3000);
    }
```

### Listen - oczekiwanie na połączenie
```java
public void listen() throws IOException {
        System.out.println("Server started");
        serverSocket.accept();
        System.out.println("Client connected");
    }
```

### Odebranie wiadomości
```java
public void listen() throws IOException {
    System.out.println("Server started");
    Socket socket = serverSocket.accept();
    System.out.println("Client connected");

    InputStream input = socket.getInputStream();
    BufferedReader reader = new BufferedReader(new InputStreamReader(input));

    String message;
    while (true) {
        message = reader.readLine();
        System.out.println(message);
    }
}
```

### Zakończenie połączenia
#### Doklejamy do odbierania wiadomości
    socket.close();
    serverSocket.close();
    System.out.println("Server closed");

### Wysyłanie wiaodomości przez serwer
#### Serwer może wysłać wiadomość do klienta, używając strumienia wyjściowego OutputStream.
```java
public void listen() throws IOException {
    System.out.println("Server started");
    Socket socket = serverSocket.accept();
    System.out.println("Client connected");

    InputStream input = socket.getInputStream();
    BufferedReader reader = new BufferedReader(new InputStreamReader(input));

    OutputStream output = socket.getOutputStream();
    PrintWriter writer = new PrintWriter(output, true);

    String message;
    writer.println("Hello!");
    while ((message = reader.readLine()) != null) {
        writer.println(message);
    }

    socket.close();
    serverSocket.close();
    System.out.println("Server closed");
}
```

### Obsługa wielu klientów
#### Tworzenie wątków dla każdego klienta umożliwia obsługę wielu klientów równocześnie.
#### Tworzenie wątku obsługującego klienta
```java
public class ClientHandler implements Runnable {
    private final Socket socket;
    private final BufferedReader reader;
    private final PrintWriter writer;

    public ClientHandler(Socket socket) throws IOException {
        this.socket = socket;
        InputStream input = socket.getInputStream();
        OutputStream output = socket.getOutputStream();
        reader = new BufferedReader(new InputStreamReader(input));
        writer = new PrintWriter(output, true);
    }

    @Override
    public void run() {
        System.out.println("Client connected");
        String message;
        try {
            while ((message = reader.readLine()) != null) {
                writer.println(message);
            }
            socket.close();
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
        System.out.println("Client disconnected");
    }
}
```
### Uruchomienie wątku
```java
public class Server {
    private ServerSocket serverSocket;
    private ArrayList<ClientHandler> handlers = new ArrayList<>();

    public void listen() throws IOException {
        System.out.println("Server started");
        while (true) {
            Socket socket = serverSocket.accept();
            ClientHandler handler = new ClientHandler(socket);
            Thread thread = new Thread(handler);
            thread.start();
            handlers.add(handler);
        }
    }
}
```
### Komunikacja między klientami
#### Do ClientHandlera dodajemy jeszcze private final Server server, który dodajemy też w konstruktorze + tworzymy nową metodę send:
```java
public void send(String message) {
        writer.println(message);
    }
```
#### Do pętli w override run() wpisujemy server.broadcast(message) zamiast writera
```java
//Metoda broadcast (w klasie serwera)
public void broadcast(String message) {
        handlers.forEach(handler -> handler.send(message));
    }
//trzeba jeszcze uzupełnić konstruktor ClientHandlera o "this"
```

### Usunięcie zakończonego wątku - do ClientHandlera dodaj metode close(), a do Servera removeHandler;
```java
 private void close() throws IOException {
        socket.close();
        server.removeHandler(this);
    }
//użycie - dodaj close() na koniec "try" w metodzie run()
//metoda do Servera:
    public void removeHandler(ClientHandler handler) {
        handlers.remove(handler);
}
```

### Zamknięcie połączenia z serwerem - do klasy Server dodaj:
```java
  public void disconnectHandlers() {
        handlers.forEach(handler -> handler.send("Bye!"));
        handlers.clear();
    }
// a to do maina:
        Runtime.getRuntime().addShutdownHook(new Thread(() -> {
            server.disconnectHandlers();
```

### Klient łączący się z serwerem i wysyłający wiadomość.
```java
public class Main {
    public static void main(String[] args) throws IOException {
        Socket socket = new Socket("localhost", 3000);

        InputStream input = socket.getInputStream();
        OutputStream output = socket.getOutputStream();
        BufferedReader reader = new BufferedReader(new InputStreamReader(input));
        PrintWriter writer = new PrintWriter(output, true);

        //klient wysyla wiadomosc "message"
        writer.println("message");
        String result = reader.readLine();
        System.out.println(result);
    }
}
```

### Wysyłanie pliku z klienta
```java
///do ClientHandlera
public void sendFile(String path) throws IOException {
    File file = new File(path);
    FileInputStream fileIn = new FileInputStream(file);
    DataOutputStream fileOut = new DataOutputStream(socket.getOutputStream());

    byte[] buffer = new byte[64];
    int count;
    while ((count = fileIn.read(buffer)) > 0) {
        fileOut.write(buffer, 0, count);
    }
    fileIn.close();
}
```

### Przesyłanie pliku przez Serwer
```java
//do Server
public void transferFile(ClientHandler sender, ClientHandler recipient) throws IOException {
    DataInputStream fileIn = new DataInputStream(sender.getSocket().getInputStream());
    DataOutputStream fileOut = new DataOutputStream(recipient.getSocket().getOutputStream());

    byte[] buffer = new byte[64];
    int count;
    while ((count = fileIn.read(buffer)) > 0) {
        fileOut.write(buffer, 0, count);
    }
}
```

### Odbiór pliku przez klienta
```java
public void receiveFile() throws IOException {
    File file = new File(System.getProperty("java.io.tmpdir") + "/result.bin");
    DataInputStream fileIn = new DataInputStream(socket.getInputStream());
    FileOutputStream fileOut = new FileOutputStream(file);

    byte[] buffer = new byte[64];
    int count;
    while ((count = fileIn.read(buffer)) > 0) {
        fileOut.write(buffer, 0, count);
    }
    fileOut.close();
}
```

### Logowanie klienta
```java
// w Client / ClientHandler
    private void authenticate() throws IOException {
        login = reader.readLine();
        server.clientLogged(this);
    }

    public String getLogin() {
        return login;
    }
// w Serwerze
// do pól:
    private Map<String, Client> clientMap = new HashMap<>();
//metoda
     public void clientLogged(Client loggedClient) {
        clientMap.put(loggedClient.getLogin(), loggedClient);
        for(Client currentClient : clients) {
            if(currentClient != loggedClient) {
                currentClient.send(String.format("%s zalogował się", loggedClient.getLogin()));
            }
        }
    } 
```

### Opuszczanie klienta
```java
//do ClientHandler
    private void leave() throws IOException {
        socket.close();
        server.clientLeft(this);
    }
 
//do Server
    public void clientLeft(Client leavingClient) {
        clients.remove(leavingClient);
        try {
            clientMap.remove(leavingClient.getLogin());
        } catch (Exception e) {}

        for(Client currentClient : clients) {
            currentClient.send(String.format("%s wylogował się", leavingClient.getLogin()));
        }
    }
```
