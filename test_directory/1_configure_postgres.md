#### Firewall configure
0) Для работы с удаленными машинами, необходимо сконфигурировать firewall(он может быть не сконфигурирован, так и сконфигурирован только на 22 порт)
Рассмотрим случай когда мы хотим сами сконфигурировать firewall:
- apt install ufw
- sudo ufw status
- sudo ufw enable
- sudo ufw status
- sudo ufw allow OpenSSH
- sudo ufw status
- sudo ufw allow 5432/tcp
- sudo ufw reload
- sudo ufw status
- sudo ufw allow 80/tcp
Это активирует firewall, и открывает нужные для нас порты. 5432 db, 22 ssh, 80 наше приложение

### Переконфигурация postgresql для возможности удаленного подключения
1) внесем пароль юзера postgres в сам postgres для возможности удаленного подключения по паролю:
```
su - postgres
psql
ALTER ROLE postgres PASSWORD 'OUR_PASS';
exit
exit
```
2) Postgresql по дефолту запрещает подключение не из localhost. Для того чтобы разрешить подключения необходимо изменить конфигурацию нескольких файлов:
```
/etc/postgresql/14/main:
.
└── main
    ├── conf.d
    ├── environment
    ├── pg_ctl.conf
    ├── pg_hba.conf
    ├── pg_ident.conf
    ├── postgresql.conf
    └── start.conf

2 directories, 6 files
```
Здесь нас интересует **pg_hba.conf, postgresql.conf**

**sudo vim /etc/postgresql/15/main/pg_hba.conf:**
Добавим новую строку в раздел IPv4: меняем все methods на sha256, а также с host'ом all:

```
host    all             all             all                     scram-sha-256
```
**будет примерно так:**
```
# Database administrative login by Unix domain socket
local   all             postgres                                peer

# TYPE  DATABASE        USER            ADDRESS                 METHOD

# "local" is for Unix domain socket connections only
local   all             all                                     peer
# IPv4 local connections:
host    all             all             127.0.0.1/32            scram-sha-256
host    all             all             all                     scram-sha-256
# IPv6 local connections:
host    all             all             ::1/128                 scram-sha-256
# Allow replication connections from localhost, by a user with the
# replication privilege.
local   replication     all                                     peer
host    replication     all             127.0.0.1/32            scram-sha-256
host    replication     all             ::1/128                 scram-sha-256
```
далее делаем: ```sudo service postgresql restart```

**sudo vim /etc/postgresql/15/main/postgresql.conf:**
раскомментируем

```
listen_addresses = '*'
```


и зададим * как все возможные хосты

```
# - Connection Settings -

listen_addresses = '*'          # what IP address(es) to listen on;
                                        # comma-separated list of addresses;
                                        # defaults to 'localhost'; use '*' for all
                                        # (change requires restart)
port = 5432                             # (change requires restart)
max_connections = 100                   # (change requires restart)
#superuser_reserved_connections = 3     # (change requires restart)
unix_socket_directories = '/var/run/postgresql' # comma-separated list of directories
                                        # (change requires restart)
#unix_socket_group = ''                 # (change requires restart)
#unix_socket_permissions = 0777         # begin with 0 to use octal notation
                                        # (change requires restart)
#bonjour = off                          # advertise server via Bonjour
                                        # (change requires restart)
#bonjour_name = ''                      # defaults to the computer name
                                        # (change requires restart)
```
Далее делаем ```sudo service postgresql restart```

PS. Говоря иначе (после установки postgresql необходимо разрешить коннект для нашего пользователя а также коннект как с локального соединения так и с удаленного, а также сделать так чтобы этот коннект был по паролю который будет шифроваться. Делается это все следующим образом)

после установки postgresql необходимо создать пользователя внутри postgres, поменять ему пароль, создать базу для него и навесить права
