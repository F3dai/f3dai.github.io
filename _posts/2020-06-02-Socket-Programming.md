---
published: true
title: Socket Programming with Python (TCP and UDP)
category: Other
author: F3dai
image: 'https://i.imgur.com/PMcj7KY.png'
---
This article will be explaining and demonstrating simple socket programming used to connect 2 application layer programs over a network. A network socket is essentially an internal endpoint for sending or receiving data within a node on a computer network. 

I will be using Python as the programming language in Mint OS with Anaconda (Mint and Anaconda is a great development environment).

## Simple TCP client-server architecture

Let's start by analysing this very simple client-side script:

<pre># This is client.py file
import socket
sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
host = socket.gethostname()
port = 9999
sock.connect((host, port))
msg = sock.recv(1024)
sock.close()
print (msg.decode('ascii'))</pre>

This script establishes the client on the system. The import socket statement imports all the necessary functions needed to create the client. 

The function socket.socket on line 3 creates a socket object to the variable “sock”. The socket has been created using the given address family (AF_INET) and socket type (SOCK_STREAM).socket.gethostname. 

Another variable “host” has been determined by assigning the hostname of the system.

In order to connect to a remote socket at address, the socket.connect(address) has been executed in line 6 of the code above (client.py file). This uses both variables that have just been declared. In this case, the host “mint19-anaconda” with port 9999 have been passed through this function. 

Sock.recv(1024), assigned to “msg” receives data from the socket. The value returned represents the data received. This line only allows 1024 bytes as the maximum data received in one go. 

Finally, sock.close() closes the socket file. This is executed once the socket connection has been made. A print statement is executed as confirmation. 

Below is the server side script:

<pre># This is server.py file 
import socket 
serversocket = socket.socket(socket.AF_INET, socket.SOCK_STREAM) 
host = socket.gethostname() 
port = 9999 
serversocket.bind((host, port)) 
serversocket.listen(5) 
while True: 
	clientsocket,addr = serversocket.accept() 
	print("Got a connection from %s" % str(addr)) 
	msg = 'Thank you for connecting'+ "\r\n" 
	clientsocket.send(msg.encode('ascii')) 
	clientsocket.close()</pre>

server.py creates a socket on line 3 similarly to client.py but assigns it to serversocket with the socket.socket() function. 

This has essentially defined the server endpoint for receiving or sending data within a node in the network. 

The hostname of the server is returned to host by socket.gethostname(). 

The next statement in bold, serversocket.bind() essentially associates the socket to the server address (host + port, defined earlier on in the script). 

The while loop acts as a persistent lookout for connections until it finds one. 

The function serversocket.accept() accepts the connection and returns a pair of values: new socket object to send or receive data, and the address bound to the client side of the connection represented as clientsocket, addr respectively. 

A confirmation is printed as a result of the address.

The connection is closed with .close()at the end of the for loop. 

This screenshot demonstrates exactly how a socket was created in order to connect the two running applications:

![simple scripts connection](https://i.imgur.com/TxIjFY8.png)

### Ports

There are 2 conditions that must be met for functioning communication on ports – both the server and client must be configured with the same port as we have just seen and well-known ports in the range 1-1023 must only be used for the associated services to avoid conflict.

### Improvements

The design of the applications is so far relatively simplistic and inflexible, sometimes the user may encounter an error when the client reconnects multiple times. The architecture assumes the server is running, allowing for errors. There is also no functionality to allow for multi-user management.

## Advanced TCP client-server architecture

This section will aim to focus on more practical scenarios where there will be multiple inbound connections with more data to handle and manage on a single server

**Server script:**

<pre>#MThreadserver.py
import socket
import threading


class ClientThread(threading.Thread):
    def __init__(self, clientAddress, clientsocket):
        threading.Thread.__init__(self)
        self.csocket = clientsocket
        print("New connection added: ", clientAddress)

    def run(self):
        print("Connection from : ", clientAddress)
        #self.csocket.send(bytes("Hi, This is from Server..",'utf-8'))

        msg = ''
        while True:
            data = self.csocket.recv(2048)
            msg = data.decode()
            if msg == 'bye':
                break
            print("from client", msg)
            self.csocket.send(bytes(msg, 'UTF-8'))
        print("Client at ", clientAddress, " disconnected...")
        
LOCALHOST = "127.0.0.1"
PORT = 8080
server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
server.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
server.bind((LOCALHOST, PORT))
print("Server started")
print("Waiting for client request..")
while True:
    server.listen(1)
    clientsock, clientAddress = server.accept()
    newthread = ClientThread(clientAddress, clientsock)
    newthread.start()
</pre>

**Client script:**

<pre>#client1.py
import socket
SERVER = "127.0.0.1"
PORT = 8080
client = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
client.connect((SERVER, PORT))
client.sendall(bytes("This is from Client", 'UTF-8'))
while True:
    in_data = client.recv(1024)
    print("From Server :", in_data.decode())
    out_data = input()
    client.sendall(bytes(out_data, 'UTF-8'))
    if out_data == 'bye':
        break
client.close()</pre>

MThreadserver.py starts with a print statement “Waiting for client request..” indicating that it is listening for an inbound connection. Once the client1.py script was executed, the client gave a confirmation that a connection was made, “From Server : This is from Client” as well as server.py with “New connection added”. This illustrates the inter-connectivity across the client-server architecture. 

Multi-threading is essentially a process that allows multiple executions of threads simultaneously. The imported module Threading helps in synchronisation and will allow the multiple use of threads to ideally allow various connections at the same time. The module socket will also be included which allows socket objects, necessary for socket programming.

The scripts most identifiable difference is the more object orientated approach. A class is made with 2 functions. The server is listening for a connection in the while loop which starts in line 32, ready to accept a connection with server.listen(1) which enables the server to accept connections, specifying the number of unaccepted connections that may be allowed prior to refusing new connections. 

While the script is listening, if a connection is found by server.accept(), the corresponding socket and address are assigned to variables in line 34, clientsock and clientAddress. 

The single client-server architecture would treat each connection as they came, one at a time in the while loop however the MthreadServer.py script accepts each connection as its object, where it executes code in the ClientThread class elsewhere in the program, in parallel. This allows the server to simultaneously accept independent connections as demonstrated below:

![advanced connectivity](https://i.imgur.com/RTocLn6.png)

Two clients (positioned on the bottom) have successfully send a message over a TCP connection to the server (positioned across the top). I have inputted "Hey this is client x" from both clients to show how the message has been transferred to the server-side applicaiton. 

An input is allowed with the client.sendall() function which sends data to the socket. On the other side with the server script, from line 16 to 23, a while loop is utilised to listen to any data being sent. All data is decoded with data.decode(). 

## UDP communication

TCP communication (on the transport layer) is the most common protocol found across the internet for the transport layer, there are also UDP-based Data Transfer Protocol. This high-performance data transfer protocol is designed to provide a high volumetric data transfer over networks. 

**Server Script**

<pre>#UDP_Server.py
import socket

LOCALHOST = "127.0.0.1"
PORT = 8080
sock = socket.socket(family=socket.AF_INET, type=socket.SOCK_DGRAM) # Create datagram socket
sock.bind((LOCALHOST, PORT)) # Bind to address and ip

print("UDP server started")
print("Waiting for data..\n")
# Listen for incoming datagrams
while(True):
    message, address = sock.recvfrom(1024) # Receiving UDP data
    in_data = message.decode() # Decode message from bytes
    print("Client IP Address:{}".format(address)) # Display client IP information
    print("Message from Client:", in_data) # Display UDP data from client
    
    # Sending a reply to client
    sock.sendto(message, address) </pre>

**Client Script**

<pre># Client1.py
import socket
import sys
address = ("127.0.0.1",8080) # Must be tuple
out_data = "Hello server, from UDP client!"
sock = socket.socket(family=socket.AF_INET, type=socket.SOCK_DGRAM)
sock.connect((address))
sock.sendto(bytes(out_data,'UTF-8'),address)
while True:
    try:
        message, address = sock.recvfrom(1024)
    except:
        print("[!] Server not running")
        sys.exit()
    in_data = message.decode()
    print("Message from Server: ", in_data)
    out_data = input()
    sock.sendto(bytes(out_data,'UTF-8'), address)
    if out_data == 'bye':
        break
sock.close()</pre>

The UDP connection allows for communication between the client and the server until the input ‘bye’ is detected. It is also capable of handling and receiving different clients at the same time. The clients have also been designed to only connect if the server is up, essentially avoiding errors that occur when the server-side architecture is not working or running. 

![UDP connection](https://i.imgur.com/dt7UkNP.png)

The 2 clients running simultaneously (positioned at the bottom) have connected to the server-side application (positioned across the top). I have sent a few messages from the both clients to show that they have successfully arrived on the server console. 

