# Java_ChatApplication

pom.xml

<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-websocket</artifactId>
    </dependency>
    <dependency>
        <groupId>org.webjars</groupId>
        <artifactId>sockjs-client</artifactId>
        <version>1.5.1</version>
    </dependency>
    <dependency>
        <groupId>org.webjars</groupId>
        <artifactId>stomp-websocket</artifactId>
        <version>2.3.4</version>
    </dependency>
    <dependency>
        <groupId>org.webjars</groupId>
        <artifactId>bootstrap</artifactId>
        <version>5.1.3</version>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-thymeleaf</artifactId>
    </dependency>
</dependencies>



WebSocketConfig.java

package com.example.chat.config;

import org.springframework.context.annotation.Configuration;
import org.springframework.messaging.simp.config.MessageBrokerRegistry;
import org.springframework.web.socket.config.annotation.*;

@Configuration
@EnableWebSocketMessageBroker
public class WebSocketConfig implements WebSocketMessageBrokerConfigurer {
    
    @Override
    public void configureMessageBroker(MessageBrokerRegistry config) {
        config.enableSimpleBroker("/topic"); // broadcast
        config.setApplicationDestinationPrefixes("/app"); // receive
    }

    @Override
    public void registerStompEndpoints(StompEndpointRegistry registry) {
        registry.addEndpoint("/chat-websocket").withSockJS();
    }
}



ChatController.java

package com.example.chat.controller;

import com.example.chat.model.Message;
import org.springframework.messaging.handler.annotation.MessageMapping;
import org.springframework.messaging.handler.annotation.SendTo;
import org.springframework.stereotype.Controller;

@Controller
public class ChatController {

    @MessageMapping("/send")
    @SendTo("/topic/messages")
    public Message sendMessage(Message message) {
        return message;
    }
}


Message.java

package com.example.chat.model;

public class Message {
    private String content;

    public Message() {}

    public Message(String content) {
        this.content = content;
    }

    public String getContent() { return content; }

    public void setContent(String content) { this.content = content; }
}


ChatApplication.java

package com.example.chat;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class ChatApplication {
    public static void main(String[] args) {
        SpringApplication.run(ChatApplication.class, args);
    }
}


index.html

<!DOCTYPE html>
<html>
<head>
    <title>Java Chat App</title>
    <meta charset="UTF-8"/>
    <script src="/webjars/sockjs-client/sockjs.min.js"></script>
    <script src="/webjars/stomp-websocket/stomp.min.js"></script>
    <script>
        let stompClient = null;

        function connect() {
            const socket = new SockJS('/chat-websocket');
            stompClient = Stomp.over(socket);
            stompClient.connect({}, function () {
                stompClient.subscribe('/topic/messages', function (messageOutput) {
                    showMessage(JSON.parse(messageOutput.body).content);
                });
            });
        }

        function sendMessage() {
            const message = document.getElementById('message').value;
            stompClient.send("/app/send", {}, JSON.stringify({'content': message}));
            document.getElementById('message').value = '';
        }

        function showMessage(message) {
            const response = document.getElementById('messages');
            const p = document.createElement('div');
            p.appendChild(document.createTextNode(message));
            p.className = 'message';
            response.appendChild(p);
        }

        window.onload = connect;
    </script>
    <style>
        .message { background-color: #f4f4f4; padding: 10px; margin-bottom: 5px; border-radius: 4px; }
    </style>
</head>
<body>
<div style="width:400px; margin:auto; text-align:center;">
    <h2>Real-time Chat</h2>
    <div id="messages" style="border:1px solid #ccc; height:300px; overflow-y:auto; padding:10px;"></div>
    <input type="text" id="message" placeholder="Type a message..."/>
    <button onclick="sendMessage()">Send</button>
</div>
</body>
</html>
