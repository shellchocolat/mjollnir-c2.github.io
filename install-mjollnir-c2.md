# Mjollnir c2 setup

< [index](install-mjollnir-c2.md)

\> [Launch your first chocolate](first-chocolate.md)

## Full setup 

First construct the postgresql database:
```
$ sudo apt-get install postgresql postgresql-contrib
$ sudo systemctl enable postgresql
$ sudo systemctl start postgresql
$ sudo systemctl status postgresql
$ psql -> error need to create user/password
$ sudo -u postgres psql
postgres=# \password postgres -> set the password
postgres=# \q
```

To create or delete the Mjollnir database, the user has to do:
```
postgres=# CREATE DATABASE mjollnir;
postgres=# DROP DATABASE mjollnir;
```

The user can verify that the Mjollnir database is well created:
```
$ sudo -u postgres psql
postgres=# \l -> list all the databases
```

Install rust and cargo to use the mjollnir backend:
> https://www.rust-lang.org/tools/install

```
$ curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
$ source $HOME/.cargo/env
$ source ~/.profile (if using zsh, write into ~/.zshrc: export PATH="$HOME/.cargo/bin:$PATH")
$ rustup toolchain install nightly
$ cargo -V
$ rustc -V
```

Install nodejs to use the mjollnir frontend:
```
$ sudo apt-get install nodejs npm
$ sudo npm install -g @quasar/cli
```

Then the user can test the Mjollnir C2 backend:
```
$ cd mjollnir-backend
$ cargo build
$ cargo run --bin mjollnir
```

Once the previous commands are executed, the tables are created into the **mjollnir database** and the binaries are built. The user can verify that tables are well created:
```
$ psql -h 127.0.0.1 -d mjollnir -U postgres
mjollnir=# \dt -> list all tables
mjollnir=# \d agent -> describe the fields of the agent table
```

To launch the Mjollnir C2 frontend in dev mode:
```
$ cd mjollnir-frontend
$ npx quasar dev
```

If the user doesn't want to install all thoses dependencies, he could use the docker instead.

## Docker setup

### database connection
The postgresql database is not dockerized, that means that the database is located on the host machine.

You have to modify 2 files in order to work properly.

If you want that the postgresql database be located on the same machine that the mjollnir-backend/frontend:
```
$ sudo vim /etc/postgresql/10/main/pg_hba.conf
```

Find the line:
```
host all all 127.0.0.1/32 md5
```

and replace it with 
```
host all all MACHINE_IP/32 mdr
```

Then you need to modify the file:
```
$ sudo vim /etc/postgresql/10/main/postgresql.conf
```

Then find the line:
```
listen_addresses = 'localhost'
```

And replace it with:
```
listen_addresses = '*'
```

Then you can verify that postgresql is well listening on port 5432:
```
$ sudo systemctl restart postgresql
$ ss -lntp
$ psql -h DATABASE_IP -U postgres -p 5432
```

If you want the database be located on an other machine, you have to specify the ip that will be connected to it, which is the ip where the mjollnir-backend is located into the file __pg_hba.conf__

If you have already a .sql file containing all the commands for your agents. Please do:
```
$ psql -h 127.0.0.1 -d mjollnir -U postgres -f command_init.sql
```

### mjollnir backend
generate certificate (https://www.linode.com/docs/guides/create-a-self-signed-tls-certificate/):
```
$ cd mjollnir-backend\ssl
$ openssl req -new -newkey rsa:4096 -x509 -sha256 -days 365 -nodes -out backend.crt -keyout backend.key
```

```
$ cd mjollnir-backend
$ cargo build --release --bin mjollnir
```

Then copy .env and db.sql where you want to execute mjollnir:
```
$ cd mjollnir-backend
$ cp -r .env db.sql ssl/ target/release/mjollnir /home/
$ cd /home
$ ./mjollnir
```

You need to modify the __DATABASE_URL__ inside the __.env__ file in order to match what you specify inside __pg_hba.conf__ !!

```
$ sudo docker build -f Dockerfile_backend -t mjollnir-backend .
$ sudo docker run -it --network host -v ~/mjollnir-c2/mjollnir-chocolates:/mjollnir-chocolates -v ~/mjollnir-c2/mjollnir-listeners:/mjollnir-listeners -v ~/mjollnir-c2/mjollnir-launchers:/mjollnir-launchers -v ~/mjollnir-c2/mjollnir-phishing:/mjollnir-phishing --rm mjollnir-backend
```

### mjollnir frontend
```
$ cd mjollnir-c2
$ sudo docker build -f Dockerfile_frontend -t mjollnir-frontend .
$ docker run -it -p 8000:80 --rm mjollnir-frontend
```

## firewall rule

You can restrict the access to the mjollnir-backend to a specific machine. Remember that mjollnir-backend is listening on port 3030. I use __ufw__ as a firewall which is a wrapper for __iptables__:

```
$ ufw allow from MY_IP_HERE proto tcp to any port 3030
$ ufw status numbered
$ ufw delete 3 (for example)
```

Of course you will need to allow the listeners' port as well, but without specify the IP, because you don't know from where the connection will be established.

## Login for the first time

> The default user is  **thor**.
> The user is encouraged to take a strong password.

```
$ curl -X POST 'http://127.0.0.1:3030/login' -H 'Content-Type: application/json' -d '{"username": "thor", "password": "mjollnir", "role": "admin"}' -v
```

When connected, a **user_token** and a **user_uid** are given to the user that will be used to authenticate to other endpoints of the mjollnir api. You can verify that checking your cookies. 

Currently, only one session is allowed. That means that if someone got access to your instance, you will be disconnected, letting you know that someone successfully logins. You could then go to your backend and modify directly the mjollnir database in order to kick him out.

## Check authorization

You can check that you are well logged using this request:

```
$ curl -X GET 'http://127.0.0.1:3030/auth' -b 'user_uid=XXXXXXXX; user_token=XXXXXXXX'  -v
```

* If the response is **200** the user has a valid token.
* If the response is **401** the user has not a valid token.

## Logout

In order to logout, join the **logout** endpoint with a correct cookie containing **user_uid** and **user_token**.

```
curl -X GET 'http://127.0.0.1:3030/logout' -b 'user_uid=XXXXXXXX;user_token=XXXXXXXX' -v
```

In order to logout, the api just replace the token with a random one. Good luck bruteforcing that.

