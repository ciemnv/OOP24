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


