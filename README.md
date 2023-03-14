<div id="top"></div>

# Dockerizing a ruby on rails app

<div style="display: inline_block;"><br>
  <img align="center" alt="alvaro-ruby" height="30" width="40" src="https://raw.githubusercontent.com/devicons/devicon/master/icons/ruby/ruby-original.svg">
  <img align="center" alt="alvaro-ruby" height="30" width="40" src="https://raw.githubusercontent.com/devicons/devicon/master/icons/rails/rails-plain.svg">
  <img align="center" alt="alvaro-go" height="30" width="40" src="https://raw.githubusercontent.com/devicons/devicon/master/icons/docker/docker-plain.svg">
  <img align="center" alt="alvaro-go" height="30" width="40" src="https://raw.githubusercontent.com/devicons/devicon/master/icons/postgresql/postgresql-plain.svg">
  <img align="center" alt="alvaro-go" height="30" width="40" src="https://raw.githubusercontent.com/devicons/devicon/master/icons/mongodb/mongodb-plain.svg">
  <img align="center" alt="alvaro-go" height="30" width="40" src="https://raw.githubusercontent.com/devicons/devicon/master/icons/redis/redis-plain.svg">
</div>

##


### 📌 Que es Docker?

Docker le permite empaquetar una aplicación o servicio con todas sus dependencias en una unidad estandarizada. Esta unidad normalmente se etiqueta como una imagen de Docker.

Todo lo que la aplicación necesita para funcionar está incluido. La imagen de Docker contiene el código, el tiempo de ejecución, las bibliotecas del sistema y cualquier otra cosa que instalaría en un servidor para que se ejecute si no estuviera usando Docker.


<details>
  <summary>Table of Contents</summary>
  <ol>
    <li><a href="#hola mundo!">Hola Mundo</a></li>
    <li><a href="#create-railsapp">Creando rails app con docker</a></li>
    <li><a href="#dockerfile">Dockerfile</a></li>
    <li><a href="#dockerfile">Docker-Compose</a></li>
    <li><a href="#dockerfile">Volumenes</a></li>
    <li><a href="#dockerfile">Agregando Redis</a></li>
    <li><a href="#dockerfile">Comunicando Redis-Rails</a></li>
  </ol>
</details>

### 📌 Hola Mundo!
```docker
docker run ruby:3.1 ruby -e "puts 'Hola Mundo'"
```
```bash
Unable to find image 'ruby:3.1' locally
3.1: Pulling from library/ruby
e4d61adff207: Already exists
4ff1945c672b: Already exists
ff5b10aec998: Already exists
12de8c754e45: Already exists
ada1762e7602: Already exists
f8f0dec0b2ef: Already exists
7109f2ab3080: Already exists
fe1e1dda18a5: Already exists
Digest: sha256:ed1b6900bb43260c4cb53aac70448fc3bbec0749c355dbfcee2f6378882b2a14
Status: Downloaded newer image for ruby:3.1
Hola Mundo
```

```bash
CONTAINER ID   IMAGE      COMMAND  ...
f2eb4b680cb6   ruby:3.1   "ruby -e 'puts 'Hola…" ...
eb054dce6812   ruby:3.1   "ruby -e 'puts 'Hola…" ...
a7f97cb8bd6b   ruby:3.1   "ruby -e 'puts 'Hola…" ...
```

<p>borrando los contenedores..</p>

```bash
docker rm <container_id> <container_id> <container_id> ...
```

```docker
docker run --rm ruby:3.1 ruby -e "puts 'Hola Mundo'"
```
### 📌 Creando rails app con docker

```docker
docker run -it --rm -v ${PWD}:/usr/src/app ruby:3.1 bash
```

## 🔥 Consideraciones

* `rm` -> remover el contenedor despues de su ejecucion
* `it` -> ejecutar contenedor de forma interactiva
* `v` -> volumen(vinculo entre contenedor y directorio local)

📝 __reails new app!__
```bash
cd /usr/src/app
gem install rails
rails new railsapp --database=postgresql

docker compose run web rake db:create
```

```bash
echo "source 'https://rubygems.org'" > Gemfile
echo "gem 'rails'" >> Gemfile
bundle exec rails new . --force
```

### 📌 Dockerfile

```docker
FROM ruby:3.1
RUN apt-get update -yqq
RUN apt-get install -yqq --no-install-recommends nodejs
COPY . /usr/src/app/
WORKDIR /usr/src/app
RUN bundle
```

## 🔥 Consideraciones

* `FROM` -> imagen docker a utilizar
* `COPY` -> copia desde el ordenador local hacia algún directorio de la imagen
* `WORKDIR` -> entrypoint
* `RUN` -> lanza ejecuciones

📝 __Construyamos la imagen!__

```docker
docker build .
```

📝 __Probemos nuestra imagen!__

```docker
docker run -p 3000:3000 <imagen_id> rails s -b 0.0.0.0
```
📝 __Agregemos un tag!__

```docker
docker tag e85596429b64 railsapp
```
🚀 __Mejoremos nuestro Dockerfile!__

```docker
FROM ruby:3.1
RUN apt-get update -yqq
RUN apt-get install -yqq --no-install-recommends nodejs
COPY . /usr/src/app/
WORKDIR /usr/src/app
RUN bundle
➤ CMD ["bin/rails", "s", "-b", "0.0.0.0"]
```

📝 __Generamos una nueva imagen!__

```bash
docker build -t railsapp:1.1 .
```

📝 __Levantamos un nuevo container!__

```bash
docker run -p 3000:3000 railsapp:1.1
```

### 📌 Docker-Compose

Docker Compose nos permite gestionar un grupo de contenedores. Al manejar un diseño declarativo, su gestion se realiza desde su archivo de configuración, describiendo cuales servicios vamos a utilizar y de qué manera.

📝 __creamos nuestro docker-compose.yml!__

```docker
version: '3.8'
services:

  web:
    build: .
    ports:
      - "3000:3000"
```

```
version: '3.8'
services:
  db:
    image: postgres
    volumes:
      - ./tmp/db:/var/lib/postgresql/data
    environment:
      POSTGRES_PASSWORD: password
  web:
    build: .
    command: bash -c "rm -f tmp/pids/server.pid && bundle exec rails s -p 3000 -b '0.0.0.0'"
    ports:
      - "3000:3000"
    volumes:
      - .:/usr/src/app
    depends_on:
      - db
```

## 🔥 Consideraciones

* `VERSION` -> version del fichero de configuración
* `SERVICES` -> cada uno de los contenedores a orquestar
* `BUILD` -> ubicacion de la imagen que se va a utilizar
* `ports` -> puerto para el contenedor

📝 __Ejecutamos docker-compose!__

```bash
docker compose run web rake db:create

docker-compose up
```

## 🔥 Consideraciones

`1.` -> se crea una `isolated network`.

`2.` -> se crean los `volumenes`.

`3.` -> si el servicio tiene la directiva `build` genera una imagen.

`4.` -> crea y lanza los contenedores por servicio.


📝 Se crean los nombres de cada servicio concatenando el `project name` y el `service name`

```bash
docker images
```


### 📌 Volumenes

📝 Son sistemas de archivos para los contenedores, en el caso de  `volumenes locales` comparte el sistema de archivos entre la maquina local y el contenedor.

```docker
version: '3.8'
services:

  web:
    build: .
    ports:
      - "3000:3000"
    volumes:
      - .:/usr/src/app
```
docker-compose encontrará una nueva configuración y regenerará la imagen para añadir la nueva configuración.

```bash
docker-compose up -d
docker-compose [up, stop] [nombre de servicio]
```
```bash
docker-compose ps
```


### 📌 Agregando Redis

📝 Redis es una base de datos NoSQL del tipo clave:valor, que almacena en memoria los datos a los que necesitemos acceder de forma rápida o para compartir información con otras aplicaciones.


```docker
version: '3.8'
services:

    web:
      build: .
      ports:
        - "3001:3000"
      volumes:
        - .:/usr/src/app
    redis:
      image: redis
```

📝 Al no indicar ningun puerto a exportar no podremos acceder desde nuestra maquina local.

```bash
docker-compose run redis redis-cli -h redis
```
```bash
SET number xx
GET number
```

### 📌 Comunicando Redis-Rails

📝 Todos los servicios de una aplicacion comparten la misma red y se pueden acceder pro su nombre descriptivo, docker gestina internamente un DNS para que los servicios puedan interactual entre si.

```ruby
# Use Redis adapter to run Action Cable in production
gem “redis”, “~> 4.0”
```
necesitamos detener los servicios y reconstruir la imagen.

```bash
docker-compose stop web
docker-compose build web
docker-compose start web
```

📝 Agragamos un controlador de prueba.

```bash
docker-compose exec web bin/rails g controller home index
```

```ruby
resources :home, only: :index

class HomeController < ApplicationController
  def index
    redis = Redis.new(host: 'redis', port: 6379)
    @number = redis.get :number
  end
end
```

```erb
<h1>Número obtenido desde REDIS: <%= @number %></h1>
```