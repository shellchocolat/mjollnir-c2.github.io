# Launch your first chocolate

< [index](install-mjollnir-c2.md)

\> 

The agent name used into _Mjollnir_ is named a **Chocolate**. This is the equivalent of the **Meterpreter** into _Metasploit_, or the **Grunt** into _Covenant_, and so on.

Before using a **Chocolate**  you need to start a listener that will handle the connection of all of your **Chocolates**.

## Create a listener

You need to be authenticated in order to be able to create à listener, that's why there are a **user_uid** and a **user_token** in the following request:
```
$ curl -X POST 'http://127.0.0.1:3030/listener' -H 'Content-Type: application/json' -d '{"listener_type": "tcp", "listener_bind_port": "8080", "listener_bind_address": "127.0.0.1"}' -b 'user_uid=XXXXXXXX ; user_token=XXXXXXXX' -v
```

But you can use the webui as well:

![](demos/create_listener.gif)

When a listener is created, Mjollnir will start the corresponding listener. And so, Mjollnir populates the database with the informations of the freshly started listener. The user can manually see the content of the database with:
```
$ psql -h 127.0.0.1 -d mjollnir -U postgres
mjollnir=# select * from listener;
```

Of course, the user can manually check if the binded port is well opened:
```
$ ss -lntp
```

## List all listeners

There is a endpoint that list all the listeners that are saved into the mjollnir database. You can request it like:
```
$ curl -X GET 'http://127.0.0.1:3030/listener/' -b 'user_uid=XXXXXXXX ; user_token=XXXXXXXX' 
$ curl -X GET 'http://127.0.0.1:3030/listener/?search_port=8080' -b 'user_uid=XXXXXXXX ; user_token=XXXXXXXX' 
```

> This endpoint is also reachable from the webui

You have to give the **user_uid** and **user_token** in a cookie to perform the request.

## Delete a listener

You could delete a listener from the database. Besides, doing that action will also shut down the listener if it is still up. The following request show how to contact the corresponding endpoint:
```
curl -X DELETE 'http://127.0.0.1:3030/listener/XXXXXXXX' -b 'user_uid=XXXXXXXX ; user_token=XXXXXXXX' -v
```

> This endpoint is also reachable from the webui

By deleting a listener, the corresponding entry into the Mjollnir database will be deleted too. The user can verify manually into the **listener table** that the listener is not present anymore, and using the **ss -lntp** command to check out that the binded port is not listening anymore.

## Update a listener name
```
curl -X PUT 'http://127.0.0.1:3030/listener' -H 'Content-Type: application/json' -d '{"listener_uid": "XXXXXXXX", "listener_name": "new name"}' -b 'user_uid=XXXXXXXX ; user_token=XXXXXXXX' -v
```
By default the name of the listener is the listener uid, but sometimes you would like to specify directly an other listener name.


