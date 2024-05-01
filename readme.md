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


--------------------------------------------------------------------------------------------------------------------

        {isTrusted: true, data: '0', origin: 'ws://127.0.0.1:8000', lastEventId: '', source: null, …}

    we can grab data from ['data']