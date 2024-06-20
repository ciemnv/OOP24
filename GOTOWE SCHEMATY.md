### KOLOS2 - tworzenie aplikacji klient/serwer
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

    public Client(Socket socket) throws IOException {
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


