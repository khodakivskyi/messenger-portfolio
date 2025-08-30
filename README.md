# Messenger

🛡 **Messenger for local networks**, built in pure C# using TCP/IP, MySQL, and WPF.

## Features
- User authentication and registration
- Personal and group chats
- File and image sharing
- Online/offline user statuses
- Modern dark-themed UI inspired by Telegram

## Technologies Used
- **.NET 8**
- **WPF** (desktop client)
- **TCP/IP**
- **MySQL** with **Entity Framework**
- **Newtonsoft.Json**
- **SHA-256**

## 📸 Preview
![image](https://github.com/user-attachments/assets/09755d1a-aff9-4fc2-93c0-832f7e9f1518)
![image](https://github.com/user-attachments/assets/edc89083-0bde-4517-b4e7-a8f6cab43fb2)
![image](https://github.com/user-attachments/assets/57fbac1c-35ab-4b96-ae94-4b8f55be558e)
![image](https://github.com/user-attachments/assets/4513ac37-8fe8-417f-bc2a-6dd18775d26a)
![image](https://github.com/user-attachments/assets/f5dd928b-c42a-40b2-82b2-43f8554cb274)
![image](https://github.com/user-attachments/assets/fe48cd2b-cdc0-432a-a49e-c0b7213b6b4c)


## 📁 Project Structure
- `MessengerApp/` – WPF client application
- `MessengerServer/` – Console-based server
- `MessengerClientTEST/` – Test client (used for debugging and connection testing)

## 🧩 Example: TCP Server Core Logic
The server is built on top of raw TCP using `TcpListener` and handles clients with a custom `ClientHandler`. Below is a simplified version of the main server loop:
```csharp
public void Start()
{
    _listener.Start();
    ServerIsRunning = true;
    Console.WriteLine("Server is running...");

    while (ServerIsRunning)
    {
        var clientSocket = _listener.AcceptTcpClient();
        Console.WriteLine("Client connected");

        var clientHandler = new ClientHandler(clientSocket, this, _userService, _groupChatService, _personalChatService);
        Task.Run(async () =>
        {
            try
            {
                await clientHandler.HandleClient();
            }
            catch (Exception ex)
            {
                Console.WriteLine($"Client error: {ex.Message}");
            }
        });
    }
}
```

## 👤 Example: ClientHandler User Authentication
Each client is handled by a `ClientHandler` instance that processes commands in a loop.
Here's a simplified excerpt showing how user login and registration are managed:
```csharp
public async Task HandleLoginAsync(string[] parts)
{
    if (parts.Length < 3)
    {
        await SendMessageAsync("LOGIN_FAILED", "Некоректний формат");
        return;
    }

    string username = parts[1], password = parts[2];
    var user = await _userService.AuthenticateUserAsync(username, password);

    if (user != null)
    {
        UserId = user.Id;
        ThisUser = user;
        _server.AddConnectedClient(UserId, this);
        Name = user.Name;
        Surname = user.Surname!;
        Avatar = user.AvatarImage;
        await SendMessageAsync("LOGIN_SUCCESS", "Авторизація успішна", UserId);
    }
    else
    {
        await SendMessageAsync("LOGIN_FAILED", "Невірний логін або пароль", 0);
    }
}
```

## 👩‍💻 Example: Client Connection and Login
This is a simplified example of how the client connects to the server and logs in using TCP streams with BinaryReader/Writer:
```csharp
public async Task<bool> ConnectAsync()
{
    var config = ReadConfig();
    _client = new TcpClient();
    await _client.ConnectAsync(config.IpAddress, config.Port);

    if (_client.Connected)
    {
        _stream = _client.GetStream();
        _reader = new BinaryReader(_stream, Encoding.UTF8);
        _writer = new BinaryWriter(_stream, Encoding.UTF8);
        return true;
    }
    return false;
}

public async Task<(bool success, string message)> LoginAsync(string username, string password)
{
    if (!Connected)
        return (false, "Not connected to server.");

    lock (_streamLock)
    {
        _writer!.Write($"LOGIN|{username}|{password}");
        _writer.Flush();

        string responseType = _reader!.ReadString();
        string responseMessage = _reader.ReadString();
        UserId = _reader.ReadInt32();

        if (responseType == "LOGIN_SUCCESS")
            Username = username;

        return responseType == "LOGIN_SUCCESS"
            ? (true, responseMessage)
            : (false, responseMessage);
    }
}
```
