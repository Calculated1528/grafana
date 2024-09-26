### Только инсталяция ничего больше

#### Проверено на manjaro/ubuntu:

##### manjaro:
```
sudo pacman -S postgresql 
sudo -iu postgres - перейдем под пользователя
initdb  --locale $LANG -E UTF8 -D ‘/var/lib/postgres/data/’ - инициализируем директорию
ИЛИ вот так
initdb  --locale $LANG -E UTF8 -D /var/lib/postgres/data/ - инициализируем директорию
sudo systemctl start postgresql.service
sudo systemctl enable postgresql.service
sudo passwd postgres - сбросим пароль и установим новый на unix пользователя. Внутри самой db есть также пароль его нужно будет установить позднее

su - postgres   - входим в пользователя postgres
psql  - входим в psql
```

##### ubuntu:

```
sudo apt install postgresql
sudo su
passwd postgres
```