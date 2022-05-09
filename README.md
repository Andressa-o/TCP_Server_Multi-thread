# TCP_Server_Multi-thread
<h2>Description</h2>

This project is the implementation of a TCP client that retrieves and requests files. Once the connection is established the client sends the name of the file, and the server searches for it in its memory cache. If the file is found, the server transmits the content back to the client by the same connection, and caches the file content, if it is not inside the memory cache. The cache has a maximum of 64MB.
First the program checks if the file exists, then if it is in the cache. if it is not in the cache it allocates the file. Check if there's enough space for the new file and delete sufficient files if it is full. If the size is bigger than the cache it does not allocate it.

<h2>Servers</h2>
The TCP server initiates with a certain command:

→ python3 Tcp_server.py localhost 3030 .\ ←

the command being the execution program, followed by the host, port and the directory being used.

The client is executed depending on the user’s needs.

→ python3 Tcp_client.py localhost 3030 file_name .\ ←

To retrieve the file from the server, or

→ python3 Tcp_server.py localhost 3030 list ←

To get a list of the files in the cache.

<h2>Sending or Receiving a Package</h2>
The program recovers files basically sending packages from the server to the client.
On the server side a file is accessed in a serialization in packet format. These packages received by the client, are combined to form the file in the directory that was chosen. The codes below show how that works
                
                #serialization of the file for sending
                with FileLock(dir + res2 + '.lock'):
                    with open(dir + res2, OPEN_FILE_SERVER) as f:
                        package = f.read(BUFFER_SIZE)
                        while package:
                            # sending file packages
                            c.send(package)
                            package = f.read(BUFFER_SIZE)
                        sleep(1)
                        f.close()

                    # serialize file
                    packages = []
                    with FileLock(dir + res2 + '.lock'):
                        with open(dir + res2, OPEN_FILE_SERVER) as f:
                            package = f.read(BUFFER_SIZE)
                            while package:
                                packages.append(package)
                                package = f.read(BUFFER_SIZE)
                            sleep(1)
                            f.close()

<h2>Multi-Thread</h2>
The TCP server allows a connection of more than one client per time, different clients connected in the server are allowed to retrieve files or the list or files in the cache memory. That is represented in the way in which the server listens for new connections. The sockets are waiting for a new connection and after it is established, it is allocated as a thread and begins listening for other connections.
As we can see here:

          try:
              #forever loop to server
              while True:
                  # establish connection with client
                  c, addr = s.accept()
                  print('|LOG|: -- [port: %s] new connection with client' % (addr[1]))

                  #new thread client
                  start_new_thread(threaded, (c, addr, dir))

<h2>Cache</h2>
The python dictionary was used to create a hash table structure so the cache memory can be build, so it has a key and associated content for each position. The construction is based in:
The key defined by the file name;
The content of the key, with three pieces of information:


   The file size;
   
   Serialization of the file by sending packets, cording to the size;
   
   A flag that identifies if the file is being solicited for the client or not;

The procedure is based on mutual exclusion promoted by the Python Thread library itself. Basically, the code responsible for performing the operations of altering or accessing the cache, are located between blocking or releasing.

<h2>File Access</h2>
Access to the files in the directory is done by the server as the client makes a request. In this way, when a file is requested, the server performs the serialization process and as there can be a query by multiple clients, the moment a file is opened to be serialized, it is blocked and only released for another serialization after being finalized. This serialization process also occurs when a file is cached.

When a file is not allocated in the memory cache, it is sent only through serialization whenever it is requested. (FileLock)

<h2>Results</h2>
The tests were based on two simultaneous clients trying to access various archive files and can be seen in the results video that was sended for the professor. The results were satisfactory and showed the functioning of the code.











