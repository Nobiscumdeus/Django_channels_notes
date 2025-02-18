                                    ------------------> Index <----------------------
        
    Intro

    Consumers

    Routing

    gs1- Basic App with consumers

    gs2- Sync Consumers with testing

    gs3- Async Consumer with testing

    gs4- How to send message from server to client

    gs5- Real time message sending, + difference between async and sync

    gs6- Websocket handling Front End

    gs7-python json lib and json in django channels.


--------------------------------------------------------------------------------------------------------------------

ques.1 Why to use AsyncConsumer?
Ans.  AsyncConsumer allows handling multiple events simultaneously without blocking, improving performance,         scalability, and responsiveness.


Ques.2   How to send messages from server to client?
Ans.2 We can do this by using 

For SyncConsumer:

    def websocket_receive(self,event):
        print('Message received from client',event['text'])
        self.send({
            'type':'websocket.send',
            'text':'Message Sent to Client',
        })

For AsyncConsumer:

    async def websocket_receive(self,event):
        print('Message received from client',event['text'])
        await self.send({
            'type':'websocket.send',
            'text':'Message Sent to Client',
        })

--------------------------------------------------------------------------------------------------------------------
                               
                                ----------> Steps <-------------


Install Django Channels - python -m pip install -U 'channels[daphne]'

        ----------------> Steps to follow <--------------------

Step-1: Create Normal Django project then add channels in its settings.py-> installed app section

        INSTALLED_APPS = [
            "django.contrib.admin",
            "django.contrib.auth",
            "django.contrib.contenttypes",
            "django.contrib.sessions",
            "django.contrib.messages",
            "django.contrib.staticfiles",
            'daphne',
            'app',
        ]

                    +

        ASGI_APPLICATION = "gs1.asgi.application"

Step-2: Create Consumer.py

        from channels.consumer import SyncConsumer,AsyncConsumer
        from channels.exceptions import StopConsumer

        class MySyncConsumer(SyncConsumer):
            def websocket_connect(self,event):
                print('websocket_conneect')
            
            def websocket_receive(self,event):
                print('websocket data receive')

            def websocket_disconnect(self,event):
                print('websocket disconnect')
                raise StopConsumer()        #otherwise will generate error.

step-3: Create Routing.py

        from django.urls import path
        from . import consumers

        websocket_urlpatterns = [
            path('ws/sc/',consumers.MySyncConsumer.as_asgi())
        ]

step-4: Add this routing.py in asgi.py

        from channels.routing import ProtocolTypeRouter,URLRouter
        import app.routing

        application = ProtocolTypeRouter(
            {
                'http': get_asgi_application(),
                'websocket':URLRouter(
                    app.routing.websocket_urlpatterns,
                )
            }
        )


step-5: Now just runserver and use the link to acess 

    ws://127.0.0.1:8000/ws/sc/
    wss://127.0.0.1:8000/ws/sc      #for secure 

step-6: And when you runserver after adding this you will get

April 30, 2024 - 22:58:19
Django version 4.2.6, using settings 'gs2.settings'
Starting ASGI/Daphne version 4.1.2 development server at http://127.0.0.1:8000/
Quit the server with CTRL-BREAK.

--------------------------------------------------------------------------------------------------------------------

                                ----------------> Consumers in channels <----------------

    Consumers are basic unit of Django Code. Consumers are like Django Views.

    Sure! Think of consumers as workers in a factory. In a WebSocket-based application, clients (like web browsers) send messages to the server over WebSocket connections. Consumers are like workers who receive these messages and do something with them.

Here's a breakdown:

1. **WebSocket Connection**: When a client connects to your server over WebSocket, it's like a customer entering a store.

2. **Consumer**: A consumer is like a worker assigned to handle a specific type of task. In this case, it's handling messages from WebSocket connections.

3. **Connect Method**: When a client connects, the `connect` method of the consumer is like the worker greeting the customer. It prepares to handle messages from that customer.

4. **Receive Method**: When the client sends a message, it's like the customer giving a request or asking for something. The `receive` method of the consumer is like the worker receiving this request. The worker then processes the request and decides what to do next.

5. **Disconnect Method**: When the client disconnects, it's like the customer leaving the store. The `disconnect` method of the consumer is like the worker tidying up after the customer and getting ready for the next one.

So, consumers are essentially the workers in your WebSocket-based application who handle incoming messages from clients, do something with them, and possibly send responses back.


                            -------------------> Consumers Type <------------------

    SyncConsumer    And     AsyncConsumer


SyncConsumer

Step-1: Create Project then on your app add file name Consumer.py 

            from channels.consumer import SyncConsumer

            class MySyncConsumer(SyncConsumer):         #inherited SyncConsumer

            #some predifined handlers
                def websocket_connect(self,event):      #calls when cliet initially opens a connection
                    print('Websocket connect')

                def websocket_receive(self,event):      #calls when data received fro client
                    print('websocket Received')

                def websocket_disconnect(self,event):   #calls when connection to the client is lost
                    print('websocket disconnect')   

----------------------------------------------------------------------------------------------------


AsyncConsumer - just add async before function.


Step-1: Create Project then on your app add file name Consumer.py 

            from channels.consumer import SyncConsumer

            class MyAsyncConsumer(AsyncConsumer):         #inherited SyncConsumer

            #some predifined handlers
                async def websocket_connect(self,event):      #calls when cliet initially opens a connection
                    print('Websocket connect')

                async def websocket_receive(self,event):      #calls when data received from client
                    print('websocket Received')

                async def websocket_disconnect(self,event):   #calls when connection to the client is lost
                    print('websocket disconnect')   


-------------------------------------------------------------------------------------------------------------------

                                --------------> Routing <-------------------

Imagine you're working at a postal office, and your job is to sort incoming mail and direct it to the correct departments or recipients. This process is similar to routing in Django Channels, where incoming messages (or events) are directed to the appropriate consumers for processing.

Here's how it works:

Incoming Mail (Messages):
        Incoming mail arrives at the postal office from various sources, such as different neighborhoods or cities. Similarly, in Django Channels, messages (or events) arrive at the server from different clients or channels, such as WebSocket connections.

Sorting (Routing):
        Your task at the postal office is to sort the mail based on the destination address or department. Similarly, in Django Channels, routing determines which consumer should handle each incoming message based on specific criteria, such as the message type or URL path.

Routing Table (URLRouter):
        Just like you have a sorting system or routing table at the postal office to determine where each piece of mail should go, Django Channels uses a routing table (implemented as a URLRouter) to map incoming messages to the appropriate consumers.

Sending Mail (Processing):
        Once the mail is sorted, it's sent to the respective departments or recipients for further processing. Similarly, in Django Channels, once a message is routed to the appropriate consumer, it's processed accordingly, whether it's sending a response, broadcasting to other clients, or performing some other action.



we call as_asgi() classmethod when routing our consumers.
THis will return Asgi wrapper application that will instantiate a new consumer instance for each connection or scope

we can write routing urls in asgi.py but that's not a good practise according to me.

-------> We will create routing.py file

    from django.urls import path
    from . import consumers

    websocket_urlpatterns = [
        path('ws/sc/',consumers.MySyncCOnsumers.as_asgi())
    ]


    -----------> Now we have to mention that to our asgi.py


--------------------------------------------------------------------------------------------------------------------

                        -------------> Handling Websocket in Front End <----------------

    Handling WebSockets in the front end typically involves using JavaScript, as it's the primary language for client-side web development. Here's a basic guide on how to handle WebSockets in the front end:

1. **Create a WebSocket Connection**: First, you need to establish a WebSocket connection from the client-side JavaScript code. You can do this using the `WebSocket` object provided by most modern browsers.

    const socket = new WebSocket('ws://localhost:3000');        // Replace the URL with your WebSocket server URL


2. **Handle WebSocket Events**: Once the connection is established, you'll need to handle various WebSocket events such as `onopen`, `onmessage`, `onclose`, and `onerror`.

    socket.onopen = function(event) {
        console.log('WebSocket connected',event);
    };

    // Listen for messages
    socket.onmessage = function(event) {
        console.log('Message from server:', event.data);
    };

    // Connection closed
    socket.onclose = function(event) {
        console.log('WebSocket closed:', event.code, event.reason);
    };

    // Connection error
    socket.onerror = function(error) {
        console.error('WebSocket error:', error);
    };

3. **Send Data to Server**: You can send data to the server using the `send()` method of the WebSocket object.

    // Sending data to server
    socket.send('Hello Server!');

4. **Close the WebSocket Connection**: When you're done with the WebSocket connection, you should close it properly.

    // Close the WebSocket connection
    socket.close();

5. **Handling Reconnections**: Depending on your application requirements, you might need to handle reconnections in case the WebSocket connection drops.

    // Attempt to reconnect if the connection is closed
    socket.onclose = function(event) {
        console.log('WebSocket closed:', event.code, event.reason);
        // Reconnect after a delay
        setTimeout(() => {
            socket = new WebSocket('ws://localhost:3000');
        }, 1000);
    };

6. **Security Considerations**: Ensure that your WebSocket server supports secure connections (wss://) if your application is served over HTTPS to prevent mixed content issues and ensure data security.

Remember, handling WebSockets involves both client-side and server-side code. Ensure your server is configured to handle WebSocket connections as well.


        -----------------------------------------------------------------------------------------------------

        {isTrusted: true, data: '0', origin: 'ws://127.0.0.1:8000', lastEventId: '', source: null, …}

    we can grab data from event['data']

--------------------------------------------------------------------------------------------------------------------

                ----------------> gs7 json to string and string to json <---------------

    When sending data from a server to a client, especially in web development, JSON (JavaScript Object Notation) is commonly used for its simplicity and compatibility with many programming languages. In Python, you can create JSON data using dictionaries, lists, and other basic data types, and then serialize it into a JSON string using the `json.dumps()` function from the `json` module.

Here's a basic example in Python:
import json

# Create a dictionary representing the data
data = {
    'name': 'John',
    'age': 30,
    'city': 'New York'
}

# Serialize the data to a JSON string
json_data = json.dumps(data)

# Now json_data contains the JSON representation of the data
print(json_data)  # Output: {"name": "John", "age": 30, "city": "New York"}


On the client-side, in JavaScript, you can then parse this JSON string into a JavaScript object using `JSON.parse()`:


// Assuming jsonData is the JSON string received from the server
var jsonObject = JSON.parse(jsonData);

// Now you can access the data as a JavaScript object
console.log(jsonObject.name);  // Output: John
console.log(jsonObject.age);   // Output: 30
console.log(jsonObject.city);  // Output: New York

This enables seamless communication between a Python server and a JavaScript client, as JSON is a language-independent data interchange format.

If you're sending plain text data from the server to the client, you won't need to serialize and deserialize it. You can send it directly, and the client can receive it as a string. In JavaScript, you can simply use the received string data as needed. There's no need for parsing as with JSON data.

            ----------------------------------------------------------------------------------------------

        Data travel between client to server in json string format.


Server Side:
    When sending data to client:
        Python To String
        json.dumps(): Converts python dictionary to json string
    
    When Receiving Data from client
        String to python
        json.loads(): Converts json string to python dictionary


Client Side:
    When sending data to server
        Javascript object to string
        json.stringify(): Converts javascript object to json string

    when Receiving data from server
        String to javascript object
        json.parse(): Converts json string to javascript object


--------------------------------------------------------------------------------------------------------------------

                            ---------> gs7 Real Time Data with FrontEnd <----------

    Converted String data to Js object to grab the values from server.

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>
<body>
    <h1>Home Page</h1>
    <h2 id="ct"></h2>
    <script>
        var ws = new WebSocket('ws://127.0.0.1:8000/ws/sc/')

        ws.onopen = function(){
            console.log('websocket connected <br>')   //agar yha pe document.write kr rhe to received data show nhi krega because document.write page ko refresh kr deta h.
            ws.send('hello')
        }

        {% comment %} ws.onmessage = function(event){
            //document.write(event.data + '<br>')
            console.log(event)
            document.getElementById('ct').innerText = event.data   // Here we can also use innerHtml both will work same, difference between both is that innerHtml can interpret Html tags but innerText can't.
        } {% endcomment %}

        ws.onmessage = function(event){
            //document.write(event.data + '<br>')
            console.log(event)
            object = JSON.parse(event.data)
            document.getElementById('ct').innerText = object.count
        }

        ws.onclose = function(){
            console.log('websocket disconnected')
        }
    </script>
</body>
</html>

        --------------> For this we need to convert our data to string in consumers.py <------------


    from channels.consumer import SyncConsumer
    from time import sleep
    from channels.exceptions import StopConsumer
    import json

    class Myconsumer(SyncConsumer):
        def websocket_connect(self,events):
            print('websocket connected')
            self.send({
                'type':'websocket.accept',
            })

        def websocket_receive(self,event):
            print('Message is '+ event['text'])
            for i in range(30):
                self.send({
                    'type':'websocket.send',
                    'text': json.dumps(i),
                })
                sleep(1)

        def websocket_disconnect(self,event):
            print('websocket connection over',event)
            raise StopConsumer

--------------------------------------------------------------------------------------------------------------------

        ------------------> gs8 Full Messages send/receive, Connect, disconnect, button <-------------

    <form method="post">
        <input type="text" name="message" id="message">
        <input type="button" value="Send" onclick="sendMessage()">
    </form>
    <button type="button" onclick="reopenWebSocket()">Reconnect</button>
    <button type="button" onclick="closeWebSocket()">Stop</button>
    <h1 id="count"></h1>
    <script>
        var socket;

        function openWebSocket() {
            socket = new WebSocket('ws://127.0.0.1:8000/ws/sc/');

            socket.onopen = function() {
                console.log('Connection Established');
                socket.send('hello');
            };

            socket.onmessage = function(event) {
                var object = JSON.parse(event.data);
                document.getElementById('count').innerText = object;
            };

            socket.onclose = function(event) {
                console.log('Connection Closed');
            };
        }

        function closeWebSocket() {
            if (socket) {
                socket.close();
            }
        }

        function sendMessage() {
            var messageInfo = document.getElementById('message');
            var message = messageInfo.value;
            socket.send(message);
            messageInfo.value = '';
        }

        function reopenWebSocket() {
            closeWebSocket(); // Close the existing WebSocket connection if it's open
            openWebSocket(); // Open a new WebSocket connection
        }

        // Call openWebSocket() to open the WebSocket connection initially
        openWebSocket();
    </script>



--------------------------------------------------------------------------------------------------------------------

                        -------------------> gs9 Channel Layers <------------------
Used Both Sync and Asyncconsumer

1. **What are Channel Layers?**
   
   Channel layers provide a communication mechanism between different parts of your application, especially in a distributed environment. They allow different parts of your application to communicate with each other asynchronously. 

2. **How do they work?**
   
   Channel layers work by providing a channel-based communication system. Each channel layer has a name, and you can send messages between different parts of your application by addressing them to specific channels within the layer.

3. **Usage in Django Channels:**
   
   In Django Channels, you typically define a channel layer in your settings file. This layer can use different backends like Redis, in-memory, or others. Then, in your consumers (the parts of your application that handle WebSocket connections or other asynchronous tasks), you can send and receive messages through the channel layer.

4. **Scalability:**
   
   Channel layers are crucial for scalability in Django Channels applications. By using a separate channel layer backend like Redis, you can distribute your application across multiple servers while still allowing communication between different parts of your application.

5. **Example Use Cases:**
   
   - Real-time updates: You might use channel layers to send real-time updates to clients over WebSockets.
   - Background tasks: You could use channel layers to trigger background tasks asynchronously.
   - Coordinating between different parts of your application: For example, you could use channel layers to coordinate between different microservices in a distributed system.

6. **Security Considerations:**
   
   When using channel layers, ensure that you're properly authenticating and authorizing messages. You don't want unauthorized users to be able to send messages to sensitive parts of your application.

Overall, channel layers are a powerful tool for building real-time and asynchronous applications in Django Channels, providing a flexible and scalable way for different parts of your application to communicate with each other.


------> install redis
            pip install channels-redis

    ------> We have 2 options as Channels layers

            1. Redis layer:
                Now from official redis document copy paste below code.
                    CHANNEL_LAYERS = {
                        "default": {
                            "BACKEND": "channels_redis.core.RedisChannelLayer",
                            "CONFIG": {
                                "hosts": [("localhost", 6379)],
                            },
                        },
                    }

            2. In memory layer:
                    Only for Development.
                        CHANNEL_LAYERS = {
                        "default": {
                            "BACKEND": "channels.layers.InMemoryChannelLayer"
                        }
                    }

    I used In-memory channel in my gs9 because i got error in redis channel layer. My connection get closed automatically after 2-3 seconds.


    --------------------> Some Built in functions to efficiently handle channel layers <-----------------------

    Note that all the methods are Async in nature so to use them wwe need to wrap them in async_to_sync

        from asgiref.sync import  async_to_sync

    1. group_add(group_name, channel_name): 
    This function adds a channel to a group. Groups are used to broadcast messages to multiple channels simultaneously. group_name is the name of the group, and channel_name is the name of the channel to be added to the group.

    2. group_discard(group_name, channel_name): 
    This function removes a channel from a group. Similar to group_add(), but it removes the channel instead of adding it.

    3. group_send(group_name, message): 
    This function sends a message to all channels in a group. group_name specifies the group to send the message to, and message is the content to send.

    4.send(channel_name, message): 
    Sends a message to a specific channel. channel_name is the name of the channel, and message is the content to send.

    5. receive(channel_name, callback): This function sets up a callback to be called when a message is received on the specified channel. channel_name is the name of the channel, and callback is the function to call when a message is received.

    6. channel_layer: This is a reference to the current channel layer. You can use it to access other built-in functions and properties of the channel layer.

Complete Real time messaging----------------------------------

    consumers.py ---------------->

        from channels.consumer import AsyncConsumer,SyncConsumer
        from channels.exceptions import StopConsumer
        from asgiref.sync import async_to_sync
        import json

        class Myconsumer(SyncConsumer):
            def websocket_connect(self,event):
                print('websocket connected')
                print('channel layer....',self.channel_layer)
                print('channel name....',self.channel_name)
                async_to_sync(self.channel_layer.group_add)('programmers',self.channel_name) #we have to convert it async_to_sync
                self.send({
                    'type':'websocket.accept',
                })

            def websocket_receive(self,event):
                print('Message is '+ event['text'])
                print(type(event['text']))              #<class 'str'>
                async_to_sync(self.channel_layer.group_send)(
                'programmers',
                {
                    'type':'chat.message',          #handler
                    'message':event['text']
                }
                )

            def chat_message(self,event):
                print('Event',event)
                print('Event',event['message'])
                print('Event type',type(event['message']))                  #Event type <class 'str'>

                self.send({
                    'type':"websocket.send",
                    'text':event['message'],           #added this to get acknowledgment that message send successfully.
                })
                # for i in range(30):
                #     self.send({
                #         'type':'websocket.send',
                #         'text': json.dumps(i),
                #     })
                #     sleep(1)

            def websocket_disconnect(self,event):
                print('websocket connection over',event)
                print('channel layer....',self.channel_layer)
                print('channel name....',self.channel_name)
                async_to_sync(self.channel_layer.group_discard)('programmers',self.channel_name)
                raise StopConsumer


    Home.html ------------------->

                <h1>Home Page</h1>
                    <textarea name="" id="chat_log" cols="100" rows="20"></textarea><br>
                    <input type="text" name="" id="chat_message_input"><br>
                    <input type="button" value="submit" id='chat_message_submit' onclick = 'client_message()'>
                <script>
                    var socket = new WebSocket('ws://127.0.0.1:8000/ws/sc/')

                    socket.onopen = function(){
                        console.log('connection established')
                        //socket.send('hi from client')
                    }

                    socket.onmessage = function(event){
                        console.log('message received ' + typeof(event.data))       //string data type now we need to convert it to js object
                        object1 = JSON.parse(event.data)        // converted
                        console.log(typeof(object1))
                        console.log(object1.msg)

                        document.querySelector('#chat_log').value += (object1.msg + '\n')
                    }

                    socket.onsend = function(event){
                        console.log('message send')
                    }

                    socket.onclose = function(){
                        console.log('connection closed')
                    }

                    /*
                    function client_message(){
                        var rawmessage = document.getElementById('chat_message_input');
                        var message = rawmessage.value;
                        socket.send(message);                                   // without stringify my system is working...
                    }
                    */

                    document.getElementById('chat_message_submit').onclick = function (event){
                        const messageInputDom = document.getElementById('chat_message_input');
                        const message = messageInputDom.value;
                        socket.send(JSON.stringify({                // object to string
                            'msg':message
                        }))
                        messageInputDom.value = '';
                    }
                </script>


--------------------------------------------------------------------------------------------------------------------

                        --------------> gs10 Make Group name dynamic <--------------------

Without Database...
    
    How to make our group name Dynamic?

we can achieve that by first giving room name to our path.
then retrive that in our views
then pass that to our html page.        ---------> {{groupname|json_script:'group_name'}}
then use it create our socket link --> var Groupname = JSON.parse(document.getElementById('group_name').textContent)
then pass that to our consumer and there we will make our group name dynamic


--------------------------------------------------------------------------------------------------------------------

                            -----------------> gs11 With database <-------------------
        
        Simple create models according to need
        now check group is already existed or not in your views through filter queryset.
        if group found then save chat, else create group.
        then pass chat to front end to show, via for loop

        then make changes in consumer.py to save all chats of particular group..

    All code for that is provided at gs11 
        Both Async and Sync Consumer

    In Async Consumer we must need to convert our all ORM code to sync by using Database_async_to_sync

    import it from ---> channels.db import database_async_to_sync
    use await before it wrap it like :-

                    await database_sync_to_async(chat.save)()
                    group = await database_sync_to_async(Group.objects.get)(group = self.group_name) 


--------------------------------------------------------------------------------------------------------------------

                ------------------> gs12 Django channels Authentication <-----------------

    
    AuthMiddleware integrates Django's authentication system with Channels, allowing you to access the user's information in your consumers.

    Step1.    Just add AuthMiddleware in your routing.py file

                from channels.routing import ProtocolTypeRouter, URLRouter
                from channels.auth import AuthMiddlewareStack
                import myapp.routing

                application = ProtocolTypeRouter({
                    "websocket": AuthMiddlewareStack(
                        URLRouter(
                            myapp.routing.websocket_urlpatterns
                        )
                    ),
                })

    Step2. Add self.scope['user'] to check authentication in your Consumers.py file


            Define WebSocket Consumers:
Create your WebSocket consumers as usual. You can access the authenticated user via self.scope['user'] inside the consumer methods.

            # consumers.py
            from channels.generic.websocket import AsyncWebsocketConsumer

            class ChatConsumer(AsyncWebsocketConsumer):
                async def connect(self):
                    user = self.scope['user']
                    if user.is_authenticated:
                        # Accept the connection
                        await self.accept()
                    else:
                        # Reject the connection
                        await self.close()

                async def receive(self, text_data):
                    ...

                async def disconnect(self, close_code):
                    ...

    --------------------------------------------------------------------------------------------------------------

Steps to follow----->

    Step.1 : First Add Authmiddlewarestack in asgi.py file as shown above

    Step.2 : Then i added Authentication check on message sending in consumers.py --->websocket.receive function

                user = self.scope['user']           --- this will provide the identity of the user
    --------condition check

                if user.is_authenticated:

                    group = Group.objects.filter(group = self.group_name)
                    chat = chat(group = group,content = data['msg'])
                    chat.save()

                    ................
                
                else:
                    self.send({
                        'type':'websocket.send',
                        'text': json.dumps({'msg':'login required to send message'})
                    })

--------------------------------------------------------------------------------------------------------------------

                    ------------------> gs13 Generic websocket consumers <-------------------

    There are mainly 4 types of Generic Consumers

    1. WebsocketConsumer                                                    3.JsonWebsocketConsumer
    2. AsyncWebsocketConsumer                                               4.AsyncJsonWebsocketConsumer



    In Django Channels, WebSocket consumers handle WebSocket connections and define how to respond to various WebSocket events such as receiving messages, connecting, and disconnecting. Here's a basic example of a generic WebSocket consumer in Django Channels:

```python
# consumers.py
import json
from channels.generic.websocket import WebsocketConsumer

class MyConsumer(WebsocketConsumer):
    def connect(self):
        # Accept the WebSocket connection
        self.accept()

    def disconnect(self, close_code):
        # Called when the WebSocket closes
        pass

    def receive(self, text_data):
        # Called when a message is received from the WebSocket
        # You can process the message here
        pass
```

In this example:

- `connect()` method is called when a WebSocket connection is established. You usually call `self.accept()` to accept the connection.
- `disconnect()` method is called when the WebSocket connection is closed. You can perform cleanup operations here if needed.
- `receive()` method is called when the consumer receives a message from the WebSocket connection. You can process the received message here.

You can also handle JSON data by overriding `receive_json()` method instead of `receive()` method:


class MyConsumer(WebsocketConsumer):
    def receive_json(self, content):
        # Called when a JSON message is received from the WebSocket
        # Process JSON data here
        pass


        -----------------------------------------------------------------------------------------------------
`In this chapter we will learn about WebsocketConsumer and AsyncWebsocket`

    This wraps the verbose plain-ASGI message sending and receiving into handling that just deal with `TEXT` and    `BINARY FRAMES.`

Here's a more detailed explanation of the `WebsocketConsumer` class and its methods:

1. **`connect(self)`**: Instead of def websocket_connect(self): 

This method is called when a WebSocket connection is established. Inside this method, you typically perform any initial setup or validation needed for the connection.

After that, you call`self.accept()` to accept the WebSocket connection.
                    `self.close()` to close the connection.
                    `self.send()` to send the data

    Instead of      self.send({
                        'type':'websocket.accept',
                    })

2. **`disconnect(self, close_code)`**: Instead of def websocket_disconnect(self):

This method is called when the WebSocket connection is closed. You can perform any cleanup tasks here if necessary. The `close_code` parameter contains the status code indicating the reason for the closure.

3. **`receive(self, text_data=None, bytes_data=None)`**: Instead of def websocket_receive(self,event):

This method is called when the consumer receives a message from the WebSocket connection. You can override this method to define how your application processes incoming messages. You can choose to handle either text data (`text_data`) or binary data (`bytes_data`), depending on the nature of your WebSocket communication.

4. **`receive_text(self, text_data)`**: This method is a convenience method for handling incoming text data. It's an alternative to overriding the `receive()` method and handling text data there.

5. **`receive_bytes(self, bytes_data)`**: Similar to `receive_text()`, this method is a convenience method for handling incoming binary data.

6. **`send(self, text_data=None, bytes_data=None)`**: This method is used to send messages to the WebSocket client. You can send either text data or binary data by passing the appropriate parameters. If you're sending text data, pass it as `text_data`, and if you're sending binary data, pass it as `bytes_data`.

7. **`send_text(self, text_data)`**: A convenience method for sending text data to the WebSocket client. It's equivalent to calling `self.send(text_data=text_data)`.

8. **`send_bytes(self, bytes_data)`**: A convenience method for sending binary data to the WebSocket client. It's equivalent to calling `self.send(bytes_data=bytes_data)`.

These are the main methods provided by the `WebsocketConsumer` class. By subclassing `WebsocketConsumer` and implementing these methods, you can define the behavior of your WebSocket-based Django application.


Example---
            # consumers.py
            import json
            from channels.generic.websocket import WebsocketConsumer

            class EchoConsumer(WebsocketConsumer):
                def connect(self):
                    self.accept()

                def disconnect(self, close_code):
                    print('connection closed',close_code)

                def receive(self, text_data=None, bytes_data=None):
                    if text_data:
                        self.send(text_data='Hello world')
                    elif bytes_data:
                        # Echo back the received binary data
                        self.send(bytes_data=bytes_data)

    ------------------------------------------------------------------------------------------------------------

                            ----------------->AsyncWebsocketConsumer<-------------------
        
    Only handle like async function else eveything same as WebsocketConsumer

        from channels.generic.websocket import AsyncWebsocketConsumer
        class MyAsncConsumer(AsyncWebsocketConsumer):

            async def connect(self):
                print('websocket connected')
                await self.accept()

            async def receive(self,text_data=None,byte_data=None):
                print('Message received from client...',text_data)
                await self.send(text_data = 'hello world!')             #data sending to client

            async def disconnect(self,close_code):
                print('connection ended',close_code)
            
--------------------------------------------------------------------------------------------------------------------

        ----------------> gs14 More about WebsocketConsumer and AsynWebscoketCosumer <--------------

    