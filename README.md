
 

# Chat-Mania

Chat-Mania is a Full Stack Chatting App.
Uses Socket.io for real time communication and stores user details in encrypted format in Mongo DB Database.
## Tech Stack

**Client:** React JS

**Server:** Node JS, Express JS

**Database:** Mongo DB
  
## Run Locally

Clone the project

```bash
  git clone https://github.com/ShaunAryan28/Chat-Mania
```

Go to the project directory

```bash
  cd ChatMania
```

Install dependencies

```bash
  npm install
```

```bash
  cd frontend/
  npm install
```

Start the server

```bash
  npm run start
```
Start the Client

```bash
  //open now terminal
  cd frontend
  npm start
```
 
 
# **Real-Time Chat Application Design Document**  

## **1. Technology Stack Selection**  
Choosing the right stack is crucial for ensuring **scalability, low latency, and real-time performance**. Below is the selected stack with justifications:

### **Cloud Provider:**  
✅ **AWS (Amazon Web Services)** – Using **EC2** for backend hosting and **AWS Lambda** for handling background tasks.  
✅ **S3** for media storage (images, videos in chats).  
✅ **CloudFront CDN** for delivering static assets faster.  
✅ **Amazon RDS (optional)** if a relational database is needed in the future.  

*Justification:* AWS provides high scalability, low latency, and seamless integration with WebSocket-based architectures.  

### **Database:**  
✅ **MongoDB Atlas** – A **NoSQL** database best suited for chat applications.  
✅ **Redis** – Used as a caching layer for frequently accessed messages and active users.  

*Justification:* MongoDB’s document-based structure is ideal for storing messages, users, and chats efficiently. Redis caching minimizes database queries for better performance.  

### **Backend:**  
✅ **Node.js + Express.js** – Lightweight, non-blocking event-driven backend.  
✅ **Socket.io** – For real-time messaging using WebSockets.  
✅ **JWT Authentication** – Secure user authentication and session management.  

*Justification:* Node.js is highly efficient for real-time apps due to its **event-driven, non-blocking I/O model**. Socket.io ensures persistent bidirectional communication between users.  

### **Frontend:**  
✅ **React.js + Chakra UI** – Responsive, user-friendly chat interface.  
✅ **React Query / SWR** – For efficient data fetching and caching.  
✅ **Axios** – To handle API calls to the backend.  

*Justification:* React’s component-based structure makes UI updates seamless. Chakra UI provides **fast, accessible, and visually appealing** UI components.  

---

## **2. Message Flow (How Messages Travel Through the System)**
A clear breakdown of the chat application's **data flow** is essential for a strong architectural plan.

### **User Login & Authentication**  
1. The user logs in using **email/password** or **OAuth (Google, GitHub, etc.)**.  
2. The backend verifies credentials and issues a **JWT (JSON Web Token)**.  
3. The frontend stores this JWT in **HTTP-only cookies** or **localStorage**.  
4. All subsequent API/WebSocket requests include this token for authentication.  

### **Real-Time Messaging Flow**  
1. **User A** sends a message through the frontend UI.  
2. The message is transmitted via **Socket.io WebSockets** to the backend.  
3. The backend verifies the user’s **JWT token** to ensure security.  
4. The message is stored in **MongoDB Atlas** and temporarily cached in **Redis** for quick access.  
5. The backend emits the message via WebSocket to **User B** in real time.  
6. **User B’s UI updates immediately** upon receiving the message.  

*Offline Handling:*  
- If the recipient is offline, the message is stored in MongoDB and marked as **unread**.  
- Notifications can be triggered using **Firebase Cloud Messaging (FCM)**.  

---

## **3. API Endpoints and WebSocket Events**
A well-defined API is crucial for maintaining a **scalable and structured** backend.  

### **Authentication APIs**
| Method | Endpoint               | Description |
|--------|------------------------|-------------|
| `POST` | `/api/auth/register`   | Register a new user |
| `POST` | `/api/auth/login`      | Login and return JWT token |
| `POST` | `/api/auth/logout`     | Destroy session |
| `GET`  | `/api/auth/me`         | Get logged-in user data |

### **Chat and Messaging APIs**
| Method | Endpoint               | Description |
|--------|------------------------|-------------|
| `POST` | `/api/chats`           | Create a new chat |
| `GET`  | `/api/chats`           | Fetch user’s chat list |
| `POST` | `/api/messages`        | Send a message (fallback for WebSocket) |
| `GET`  | `/api/messages/:chatId` | Fetch messages of a chat |

### **WebSocket Events**
| Event Name       | Description |
|------------------|-------------|
| `sendMessage`   | Sent by the sender when a new message is sent |
| `receiveMessage`| Received by the recipient when a message arrives |
| `userTyping`    | Indicates when a user is typing |
| `userOnline`    | Updates a user's online status |

---

## **4. Optimizations for Scalability & Low Latency**
To ensure smooth performance even with thousands of concurrent users, we incorporate the following strategies:

✅ **WebSockets for real-time messaging** – Persistent connections eliminate the need for polling.  
✅ **Redis Caching** – Frequently accessed data (recent chats, active users) are stored in **Redis** for quick retrieval.  
✅ **Database Indexing** – Indexing user IDs and timestamps in MongoDB ensures **fast queries**.  
✅ **Horizontal Scaling** – Multiple WebSocket servers using **Redis Pub/Sub** for synchronization.  
✅ **Lazy Loading** – Messages are loaded **incrementally** when scrolling up.  

---

## **5. Sample Code Snippets**
### **WebSocket Server (Backend - `server.js`)**
```javascript
const express = require("express");
const http = require("http");
const socketIo = require("socket.io");

const app = express();
const server = http.createServer(app);
const io = socketIo(server, { cors: { origin: "*" } });

io.on("connection", (socket) => {
  console.log(`User connected: ${socket.id}`);

  socket.on("sendMessage", (data) => {
    io.to(data.chatId).emit("receiveMessage", data);
  });

  socket.on("disconnect", () => {
    console.log("User disconnected");
  });
});

server.listen(5000, () => console.log("Server running on port 5000"));
```

---

### **React WebSocket Client (Frontend - `Chat.js`)**
```javascript
import { useEffect, useState } from "react";
import { io } from "socket.io-client";

const socket = io("http://localhost:5000");

const Chat = () => {
  const [messages, setMessages] = useState([]);
  const [newMessage, setNewMessage] = useState("");

  useEffect(() => {
    socket.on("receiveMessage", (message) => {
      setMessages((prev) => [...prev, message]);
    });

    return () => {
      socket.off("receiveMessage");
    };
  }, []);

  const sendMessage = () => {
    socket.emit("sendMessage", { text: newMessage, chatId: "1234" });
    setNewMessage("");
  };

  return (
    <div>
      <div>
        {messages.map((msg, i) => (
          <p key={i}>{msg.text}</p>
        ))}
      </div>
      <input 
        type="text" 
        value={newMessage} 
        onChange={(e) => setNewMessage(e.target.value)} 
      />
      <button onClick={sendMessage}>Send</button>
    </div>
  );
};

export default Chat;
```

---

## **6. Additional Features (Future Enhancements)**
✅ **End-to-End Encryption** – Encrypt messages using AES-256 before sending.  
✅ **Push Notifications** – Use **Firebase Cloud Messaging (FCM)** for mobile/web notifications.  
✅ **Voice & Video Calls** – Integrate **WebRTC** for real-time calls.  
✅ **AI Chatbot** – Automate responses using **ChatGPT API** for a smart assistant.  

---
 