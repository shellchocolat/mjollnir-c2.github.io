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




