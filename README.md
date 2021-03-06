# crud notes #
## install and run ##
### create .env from .dist ###
replace `/YOUR/HOME/` with **static** path to you home dir
### run ###
```bash
export _UID="$(id -u)" \
    && export _GID="$(id -g)" \
    && time docker-compose run --rm --no-deps --user="${_UID}:${_GID}" composer \
    && time docker-compose run --rm --user="${_UID}:${_GID}" migration_and_fixtures \
    && docker-compose up --remove-orphans nginx
```
[click me](http://crudnotes.localhost) or use api helpers
## api helpers ##
### users ###
#### create user #### 
```bash
curl -s http://crudnotes.localhost/users \
    --user admin:admin \
    --data '{"username":"username","fullname":"fullname"}' \
    --request POST
```
#### read user #### 
```bash
curl -s http://crudnotes.localhost/users/21 \
    --user admin:admin \
    --request GET
```
#### update user #### 
```bash
curl -s http://crudnotes.localhost/users/21 \
    --user admin:admin \
    --data '{"fullname":"James Bond"}' \
    --request PUT
```
#### delete user #### 
```bash
curl -s http://crudnotes.localhost/users/21 \
    --user admin:admin \
    --request DELETE
```
#### list users ####
```bash
curl -s http://crudnotes.localhost/users \
    --user admin:admin \
    --request GET
```
### notes ###
#### create note ####
```bash
curl -s http://crudnotes.localhost/notes \
    --user note:note \
    --data '{"i_am":"username_1","title":"title","body":"body"}' \
    --request POST
```
#### read note #### 
```bash
curl -s http://crudnotes.localhost/notes/21 \
    --user note:note \
    --data '{"i_am":"username_1"}' \
    --request GET
```
#### update note #### 
```bash
curl -s http://crudnotes.localhost/notes/1 \
    --user note:note \
    --data '{"i_am":"username_1","title":"Foo Bar","body":"Eu non diam phasellus vestibulum lorem sed risus ultricies tristiqu"}' \
    --request PUT
# or by share write access
curl -s http://crudnotes.localhost/notes/1 \
    --user note:note \
    --data '{"i_am":"username_12","title":"Foobar","body":"The etymology of foobar is generally traced to the World War II military slang FUBAR"}' \
    --request PUT
```
#### delete note #### 
```bash
curl -s http://crudnotes.localhost/notes/21 \
    --user note:note \
    --data '{"i_am":"username"}' \
    --request DELETE
```
#### list notes ####
```bash
curl -s http://crudnotes.localhost/notes \
    --user note:note \
    --data '{"i_am":"username_1"}' \
    --request GET
```
#### available notes ####
notes shared for this user by read or write access
```bash
curl -s http://crudnotes.localhost/notes/available \
    --user note:note \
    --data '{"i_am":"username_1","access":"read"}' \
    --request GET
```
#### share note ####
```bash
curl -s http://crudnotes.localhost/notes/1/share \
    --user note:note \
    --data '{"i_am":"username_1","access":"read","usernames":["username_3","username_4"]}' \
    --request PUT
```
#### deshare note ####
```bash
curl -s http://crudnotes.localhost/notes/1/share \
    --user note:note \
    --data '{"i_am":"username_1","access":"read","usernames":["username_3","username_4"]}' \
    --request DELETE
```
## mysql/docker helpers ##
### create ###
```bash
( \
export NAME=crudnotes && docker run \
    -e MYSQL_ROOT_PASSWORD=root \
    -e MYSQL_DATABASE=${NAME} \
    -e MYSQL_USER=${NAME} \
    -e MYSQL_PASSWORD=${NAME} \
    --rm -d --name ${NAME} mysql:5.7.31 \
)
```
### find ip ###
```bash
( \
export NAME=crudnotes && \
    echo $(docker inspect --format '{{ .NetworkSettings.IPAddress }}' ${NAME}) \
)
```
### connect ###
```bash
( \
export NAME=crudnotes && mysql \
    -h $(docker inspect --format '{{ .NetworkSettings.IPAddress }}' ${NAME}) \
    -u ${NAME} \
    -n ${NAME} \
    --password=${NAME} \
)
```
### recreate database and load fixtures ###
```bash
bin/console doctrine:database:drop --if-exists -f \
&& bin/console doctrine:database:create --if-not-exists \
&& bin/console doctrine:migrations:migrate -n \
&& bin/console doctrine:fixtures:load -n
```
### create migration ###
```bash
bin/console doctrine:migrations:diff --allow-empty-diff --line-length=120 --formatted -n
```