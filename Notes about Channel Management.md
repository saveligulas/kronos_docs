# UDP Protocol - Socket & Connection Design

## Connection-Architektur

### Target-basierte Benennung

- **ServerConnection**: Verbindung ZU einem Server (Client-Seite)
- **ClientConnection**: Verbindung ZU einem Client (Server-Seite)

```c
// Client-Side
ServerConnection* connect_to_channel(const char* ip, int port, int channel_id);

// Server-Side  
ClientConnection* accept_client_connection(int server_socket);
void broadcast_to_channel(int channel_id, void* data);
```

## Channel-Design

### Grundprinzip: 1 Channel = 1 Connection

- Client verbindet sich zu **spezifischem Channel**
- Ein Client kann **mehrere ServerConnections** haben (eine pro Channel)
- Server-Endpoint wird **mehrfach verwendet** für verschiedene Channels
- Jede Connection ist **Channel-spezifisch**

### Channel-Range (8-bit = 255 Channels)

```c
#define GLOBAL_CHANNELS     0-9      // Chat, Announcements
#define GAME_CHANNELS      10-199    // Game Rooms (190 concurrent)
#define SERVICE_CHANNELS  200-249    // Microservice Communication
#define ADMIN_CHANNELS    250-255    // Admin/Debug
```

**Warum 255 ausreicht:**

- Microservices: 1 Channel pro Service-Pair
- Multiplayer: 190 concurrent Rooms möglich
- Kleiner Protocol-Overhead
- Verhindert "Channel-Chaos"

## Channel-Modes

### Socket-Mode vs. Datagram-Mode

```c
typedef enum {
    CHANNEL_MODE_DATAGRAM,    // Basic UDP messaging
    CHANNEL_MODE_SOCKET,      // WebSocket-like functionality
    CHANNEL_MODE_BROADCAST    // One-way broadcast only
} channel_mode_t;

// Server konfiguriert Channel-Mode
configure_channel(10, CHANNEL_MODE_SOCKET);  // Game Room
configure_channel(0, CHANNEL_MODE_BROADCAST); // Announcements
```

### Socket-Mode Features

- **Connection-aware**: Server weiß wer connected ist
- **Bidirectional**: Client ↔ Server + Broadcasting
- **Auto-disconnect handling**: Connection-state wird verwaltet
- **WebSocket-like APIs**: `onConnect`, `onDisconnect`, `onMessage`

## Game Room Architektur

### Channel-per-Room Ansatz

```c
#define ROOM_CHANNEL_START  10
int get_room_channel(int room_id) {
    return ROOM_CHANNEL_START + room_id;
}

// Broadcasting automatisch nur an Room-Mitglieder
broadcast_to_channel(room_channel, game_update);
```

**Vorteile:**

- Zero Message-Filtering nötig
- Clean Room Isolation
- Einfache Server Logic
- Automatisches Broadcasting an richtige Spieler

## Single-Port Design

### Ein UDP Port für alles

- Game Rooms, Microservice Communication, Admin
- Shared Infrastructure: ACKs, Fragmentation, Packet-Counting
- Deployment Simplicity: Nur ein Port zu konfigurieren
- Zentrale Connection-Verwaltung

### Alternative (bei Performance-Issues):

```c
Port 8080: Game Channels (0-199)     // High frequency, low latency
Port 8081: Service Channels (200-249) // Reliable, less frequent  
Port 8082: Admin Channels (250-255)   // Monitoring, Debug
```

## Multithreading-Strategie

### C-Library Level (minimal)

```c
// Nur für Core Protocol Operations
- Receive Thread: UDP Socket listening
- Send Thread: Outbound packet queue  
- Timer Thread: ACK timeouts, retries
```

### Spring Boot Level (Application Logic)

```java
@Async
@ChannelSocket(10)
public class GameRoomHandler {
    @OnMessage
    void handleGameInput(GameInput input) {
        // Spring's ThreadPool verarbeitet Messages
    }
}
```

**Aufteilung:**

- **C-Library**: Thread-Safe aber minimal threaded
- **Spring Boot**: Nutzt eigene Threading-Capabilities optimal
- **Clean Separation of Concerns**

## Authorization (Zukunft)

### Channel-basierte Authorization

```c
// Mögliche Ansätze:
allow_channel_range(CLIENT_TYPE_GAME, 1000, 1999);
ServerConnection* connect_to_channel(ip, port, channel_id, auth_token);
bool authorize_channel_access(client_id, channel_id, permissions);
```

## Spring Boot Integration

```java
@ChannelSocket(10)
public class GameRoomHandler {
    @OnConnect
    void playerJoined(ClientConnection conn) {
        send_room_state(conn);
        broadcast_player_joined(conn.getChannel());
    }
    
    @OnMessage  
    void handleGameInput(GameInput input) {
        gameService.processInput(input);
    }
}
```