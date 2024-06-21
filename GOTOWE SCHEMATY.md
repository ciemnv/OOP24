### KOLOS2 - tworzenie aplikacji klient/serwer - klient zleca, a serwer wykonuje
####
Zadanie 1.
Napisz serwer czatu sieciowego. Serwer powinien:
1. pozwolić dołączyć dowolnej liczbie użytkowników
2. oczekiwać na wiadomość od dowolnego użytkownika
3. rozsyłać tę wiadomość wszystkim użytkownikom.

Zadanie 2.
Napisz klienta czatu sieciowego współpracującego z napisanym serwerem: 
Klient powinien:
1. odczytywać standardowe wejście i przesyłać je do serwera
2. odbierać wiadomości z serwera i wyświetlać je na standardowym wyjściu.

#### Client
```java

public class Client implements Runnable {

    private final Socket socket;
    private final BufferedReader reader;
    private final PrintWriter writer;

    public Client() throws IOException {
        this.socket = new Socket("localhost", 2137);
        this.reader = new BufferedReader(new InputStreamReader(socket.getInputStream()));
        this.writer = new PrintWriter(new OutputStreamWriter(socket.getOutputStream()), true);
    }

    @Override
    public void run() {
        System.out.println("Client connected");
        String message;
        try {
            while((message = reader.readLine()) != null)
                //odbiera wiadomości z serwera i wyświetla je na standardowym wyjściu.
                System.out.println(message);

        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }

    //wysyłanie wiadomosci
    public void send(String message) {
        this.writer.println(message);
    }

}
```
#### Serwer
```java
public class Server {
    ServerSocket serverSocket;
    private ArrayList<Client> clients = new ArrayList<>();

    public Server() throws IOException {
        serverSocket = new ServerSocket(3000);
    }


    public void listen() throws IOException {
        System.out.println("Server started");
        while (true) {
            Socket socket = serverSocket.accept();
            //pozwolenie na dołączenie dowolnej liczbie użytkowników
            Client client = new Client(socket);
            Thread thread = new Thread(client);
            thread.start();
            clients.add(client);
        }
    }


    public void broadcast(String message) {
        clients.forEach(client -> client.send(message));
    }

}
```
#### Main
```java
public class Main {
    public static void main(String[] args) throws IOException {
        Client client = new Client();
        //klient odczytuje standardowe wejscie
        BufferedReader reader = new BufferedReader(new InputStreamReader(System.in));

        //aby klient zadziałał, tworzymy nowy wątek
        new Thread(client).start();

        while (true) {
            //klient przesyła standardowe wejscie do serwera
            client.send(reader.readLine());
        }
    }
}
```


### KOLOS 2023 ROZWIĄZANIE
#### Server
```java
import java.io.*;
import java.net.*;
import java.sql.*;
import java.util.*;
import java.text.SimpleDateFormat;
import javax.imageio.ImageIO;
import java.awt.image.BufferedImage;
import java.awt.image.ConvolveOp;
import java.awt.image.Kernel;

public class ImageServer {

    private static final int PORT = 12345;
    private static final String DB_URL = "jdbc:sqlite:images/index.db";
    private static final String IMAGE_FOLDER = "images";

    public static void main(String[] args) {
        new ImageServer().startServer();
    }

    public void startServer() {
        createImageFolder();
        createDatabase();

        try (ServerSocket serverSocket = new ServerSocket(PORT)) {
            System.out.println("Server is listening on port " + PORT);

            while (true) {
                Socket socket = serverSocket.accept();
                System.out.println("New client connected");
                new ClientHandler(socket).start();
            }
        } catch (IOException ex) {
            ex.printStackTrace();
        }
    }

    private void createImageFolder() {
        File directory = new File(IMAGE_FOLDER);
        if (!directory.exists()) {
            directory.mkdirs();
        }
    }

    private void createDatabase() {
        try (Connection conn = DriverManager.getConnection(DB_URL)) {
            if (conn != null) {
                String sql = "CREATE TABLE IF NOT EXISTS images (" +
                        "id INTEGER PRIMARY KEY AUTOINCREMENT," +
                        "path TEXT NOT NULL," +
                        "size INTEGER NOT NULL," +
                        "delay INTEGER NOT NULL" +
                        ");";
                Statement stmt = conn.createStatement();
                stmt.execute(sql);
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
}
```
#### ClientHandler
```java
    private class ClientHandler extends Thread {
        private Socket socket;

        public ClientHandler(Socket socket) {
            this.socket = socket;
        }

        public void run() {
            try (InputStream input = socket.getInputStream();
                 DataInputStream dataInput = new DataInputStream(input);
                 OutputStream output = socket.getOutputStream();
                 DataOutputStream dataOutput = new DataOutputStream(output)) {

                String fileName = new SimpleDateFormat("yyyy-MM-dd_HH-mm-ss").format(new Date()) + ".png";
                File file = new File(IMAGE_FOLDER, fileName);

                // Read file from client
                long fileSize = dataInput.readLong();
                byte[] fileContent = new byte[(int) fileSize];
                dataInput.readFully(fileContent);
                try (FileOutputStream fos = new FileOutputStream(file)) {
                    fos.write(fileContent);
                }

                // Read kernel size
                int kernelSize = dataInput.readInt();

                // Apply box blur
                long startTime = System.currentTimeMillis();
                BufferedImage image = ImageIO.read(file);
                BufferedImage blurredImage = applyBoxBlur(image, kernelSize);
                long endTime = System.currentTimeMillis();
                long delay = endTime - startTime;

                String blurredFileName = "blurred_" + fileName;
                File blurredFile = new File(IMAGE_FOLDER, blurredFileName);
                ImageIO.write(blurredImage, "png", blurredFile);

                // Save to database
                saveToDatabase(blurredFile.getPath(), kernelSize, delay);

                // Send blurred file to client
                byte[] blurredFileContent = Files.readAllBytes(blurredFile.toPath());
                dataOutput.writeLong(blurredFileContent.length);
                dataOutput.write(blurredFileContent);

            } catch (IOException | SQLException ex) {
                ex.printStackTrace();
            }
        }

        private BufferedImage applyBoxBlur(BufferedImage image, int kernelSize) {
            float[] matrix = new float[kernelSize * kernelSize];
            for (int i = 0; i < kernelSize * kernelSize; i++) {
                matrix[i] = 1.0f / (kernelSize * kernelSize);
            }
            Kernel kernel = new Kernel(kernelSize, kernelSize, matrix);
            ConvolveOp op = new ConvolveOp(kernel, ConvolveOp.EDGE_NO_OP, null);
            return op.filter(image, null);
        }

        private void saveToDatabase(String path, int size, long delay) throws SQLException {
            try (Connection conn = DriverManager.getConnection(DB_URL)) {
                String sql = "INSERT INTO images (path, size, delay) VALUES (?, ?, ?)";
                PreparedStatement pstmt = conn.prepareStatement(sql);
                pstmt.setString(1, path);
                pstmt.setInt(2, size);
                pstmt.setLong(3, delay);
                pstmt.executeUpdate();
            }
        }
    }
```

#### Client
```java
import java.io.*;
import java.net.*;
import java.nio.file.*;

public class ImageClient {

    private static final String SERVER_ADDRESS = "localhost";
    private static final int SERVER_PORT = 12345;

    public static void main(String[] args) {
        if (args.length != 2) {
            System.out.println("Usage: java ImageClient <image_path> <kernel_size>");
            return;
        }

        String imagePath = args[0];
        int kernelSize = Integer.parseInt(args[1]);

        try (Socket socket = new Socket(SERVER_ADDRESS, SERVER_PORT);
             InputStream input = socket.getInputStream();
             DataInputStream dataInput = new DataInputStream(input);
             OutputStream output = socket.getOutputStream();
             DataOutputStream dataOutput = new DataOutputStream(output)) {

            // Send file to server
            File file = new File(imagePath);
            byte[] fileContent = Files.readAllBytes(file.toPath());
            dataOutput.writeLong(fileContent.length);
            dataOutput.write(fileContent);

            // Send kernel size to server
            dataOutput.writeInt(kernelSize);

            // Receive blurred file from server
            long fileSize = dataInput.readLong();
            byte[] blurredFileContent = new byte[(int) fileSize];
            dataInput.readFully(blurredFileContent);

            // Save blurred file
            String blurredFileName = "blurred_" + file.getName();
            try (FileOutputStream fos = new FileOutputStream(blurredFileName)) {
                fos.write(blurredFileContent);
            }

            System.out.println("Blurred image received and saved as " + blurredFileName);





        } catch (IOException ex) {
            ex.printStackTrace();
        }
    }
```


## POWTÓRZENIE:
### Client Application
```java
package client;

import java.io.*;
import java.net.Socket;
import java.nio.file.Files;
import java.nio.file.Paths;
import java.util.List;
import java.util.Scanner;

public class ClientApp {

    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        System.out.println("Enter username:");
        String username = scanner.nextLine();
        System.out.println("Enter path to CSV file:");
        String filepath = scanner.nextLine();

        sendData(username, filepath);
    }

    public static void sendData(String name, String filepath) {
        try (Socket socket = new Socket("localhost", 12345);
             PrintWriter out = new PrintWriter(socket.getOutputStream(), true);
             BufferedReader in = new BufferedReader(new InputStreamReader(socket.getInputStream()))) {
            
            out.println(name);
            List<String> lines = Files.readAllLines(Paths.get(filepath));
            for (String line : lines) {
                out.println(line);
                Thread.sleep(2000);
            }
            out.println("bye");

        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

### Server Application
```java
package server;

import databasecreator.Creator;

import java.io.*;
import java.net.ServerSocket;
import java.net.Socket;
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;
import java.sql.SQLException;
import java.util.Base64;
import org.jfree.chart.ChartFactory;
import org.jfree.chart.ChartUtils;
import org.jfree.chart.JFreeChart;
import org.jfree.data.xy.XYSeries;
import org.jfree.data.xy.XYSeriesCollection;

public class ServerApp {

    public static void main(String[] args) {
        Creator.createDatabase();

        try (ServerSocket serverSocket = new ServerSocket(12345)) {
            System.out.println("Server is listening on port 12345");
            while (true) {
                new ClientHandler(serverSocket.accept()).start();
            }
        } catch (IOException ex) {
            ex.printStackTrace();
        }
    }

    private static class ClientHandler extends Thread {
        private Socket socket;

        public ClientHandler(Socket socket) {
            this.socket = socket;
        }

        public void run() {
            try (BufferedReader in = new BufferedReader(new InputStreamReader(socket.getInputStream()))) {
                String username = in.readLine();
                String line;
                int electrodeLine = 0;

                while (!(line = in.readLine()).equals("bye")) {
                    String base64Plot = createBase64Plot(line, electrodeLine);
                    insertIntoDatabase(username, electrodeLine, base64Plot);
                    electrodeLine++;
                }
            } catch (IOException | SQLException ex) {
                ex.printStackTrace();
            }
        }

        private String createBase64Plot(String data, int line) throws IOException {
            String[] values = data.split(",");
            XYSeries series = new XYSeries("EEG Data Line " + line);
            for (int i = 0; i < values.length; i++) {
                series.add(i * 2, Double.parseDouble(values[i]));
            }
            XYSeriesCollection dataset = new XYSeriesCollection(series);
            JFreeChart chart = ChartFactory.createXYLineChart(
                    "EEG Data Line " + line,
                    "Time (ms)",
                    "Amplitude (µV)",
                    dataset
            );

            ByteArrayOutputStream baos = new ByteArrayOutputStream();
            ChartUtils.writeChartAsPNG(baos, chart, 800, 600);
            return Base64.getEncoder().encodeToString(baos.toByteArray());
        }

        private void insertIntoDatabase(String username, int line, String base64Plot) throws SQLException {
            String url = "jdbc:sqlite:eeg_data.db";
            String sql = "INSERT INTO eeg_data(username, electrode_line, plot_base64) VALUES(?,?,?)";

            try (Connection conn = DriverManager.getConnection(url);
                 PreparedStatement pstmt = conn.prepareStatement(sql)) {
                pstmt.setString(1, username);
                pstmt.setInt(2, line);
                pstmt.setString(3, base64Plot);
                pstmt.executeUpdate();
            }
        }
    }
}
```

### Database Creator
```java
package databasecreator;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;
import java.sql.Statement;

public class Creator {
    public static void createDatabase() {
        String url = "jdbc:sqlite:eeg_data.db";
        String sql = "CREATE TABLE IF NOT EXISTS eeg_data (" +
                     "id INTEGER PRIMARY KEY AUTOINCREMENT, " +
                     "username TEXT NOT NULL, " +
                     "electrode_line INTEGER, " +
                     "plot_base64 TEXT" +
                     ");";

        try (Connection conn = DriverManager.getConnection(url);
             Statement stmt = conn.createStatement()) {
            stmt.execute(sql);
            System.out.println("Database created or opened successfully.");
        } catch (SQLException e) {
            System.out.println(e.getMessage());
        }
    }
}
```
