# Mjollnir c2 setup

< [index](install-mjollnir-c2.md)

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

