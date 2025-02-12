# Підготовка Dockerfile та index.html

```
$ mcedit Dockerfile
$ mcedit index.html
$ sudo docker build -t hw5-nginx-image .
$ sudo docker images
```

## Створення нового образу на основі Dockerfile

```
$ sudo docker images | grep hw5-nginx-image
hw5-nginx-image               latest    efcd5749868a   8 minutes ago   197MB
```

# NETWORK

## Bridge network

```
$ sudo docker network create hw5-bridge
$ sudo docker network ls
```

```
NETWORK ID     NAME         DRIVER    SCOPE
ebdeec58d0d4   bridge       bridge    local
308b88814943   host         host      local
3f6c98a228ab   hw5-bridge   bridge    local
eafa12d960de   none         null      local
```

### Створення контейнера в мережі bridge

```
$ sudo docker run -d --name bridge-container -p 8080:80 --network hw5-bridge hw5-nginx-image
```

```
$ sudo docker ps
CONTAINER ID   IMAGE             COMMAND                  CREATED         STATUS         PORTS                                     NAMES
82d8f4ed5a5b   hw5-nginx-image   "/docker-entrypoint.…"   6 seconds ago   Up 5 seconds   0.0.0.0:8080->80/tcp, [::]:8080->80/tcp   bridge-container
```

### Перевірка доступності nginx з MacBook. У VirtualBox NAT попередньо налаштований на форвард портів.

```
andriyan@MacBookPro ~ % curl localhost:8080
Hello from student Andriian!
This is homework №5 for testing difference networks
```

## Host network

### Створюємо контейнер з образом та вказуємо йому network host
```
$ sudo docker run -d --name host-container --network host hw5-nginx-image
```

### У випадку використання host-мережі 80-й порт контейнера стає 80-м портом хоста.

```
andriyan@MacBookPro ~ % curl localhost:80  
Hello from student Andriian!
This is homework №5 for testing difference networks
```

## None network

### Створюємо контейнер з образом та вказуємо йому network none

```
$ sudo docker run -d --name none-container --network none hw5-nginx-image
```

### Тут буде дві перевірки, щоб впевнитись, що nginx недоступний з віртуальної машини, але доступний всередині контейнера.

#### Перевірка зсередини контейнера
```
$ sudo docker exec -it none-container sh
# curl localhost 
Hello from student Andriian!
This is homework №5 for testing difference networks
```

#### Перевірка з віртуальної машини
```
gimalaj@ubuntu:~/docker/HW5-docker-network-volumes$ curl localhost:80
curl: (7) Failed to connect to localhost port 80 after 3 ms: Couldn't connect to server
```

## Macvlan network

### У VirtualBox потрібно додати ще один мережевий інтерфейс Bridge Adapter та увімкнути Promiscuous Mode у режим Allow all.

#### Виконуємо команду щоб отримати IP адресу на другий мережевий інтерфейс віртуальної машини від домашнього роутера
```
$ sudo dhcpcd enp0s9
```

### Створення мережі Macvlan
```
$ sudo docker network create -d macvlan --subnet=192.168.50.0/24 --gateway=192.168.50.1 -o parent=enp0s9 macvlan
```

### Створення контейнера з Macvlan у режимі DHCP
```
$ sudo docker run -d --name macvlan-dhcp-container --network=macvlan hw5-nginx-image
```

### Перевіряємо IP контейнера
```
andriyan@MacBookPro ~ % curl http://192.168.50.2:80
Hello from student Andriian!
This is homework №5 for testing difference networks
```

### Створення контейнера з Macvlan та статичним IP
```
$ sudo docker run -d --name macvlan-static-container --network=macvlan --ip 192.168.50.22 hw5-nginx-image
```

### Перевіряємо доступність контейнера за статичним IP
```
andriyan@MacBookPro ~ % curl http://192.168.50.22:80
Hello from student Andriian!
This is homework №5 for testing difference networks
```

# Volumes

### Створення нового Volume
```
$ sudo docker volume create hw5-shared-volume
```

### Запуск контейнера з підключеним Volume
```
$ sudo docker run -d --name container1 -v hw5-shared-volume:/opt alpine sleep 3600
```

### Запуск другого контейнера з тим самим Volume
```
$ sudo docker run -d --name container2 -v hw5-shared-volume:/opt alpine sleep 3600
```

### Створення файлу у першому контейнері
```
$ sudo docker exec -it container1 sh -c 'echo "Created in container1" > /opt/testfile'
```

### Читання файлу з другого контейнера
```
$ sudo docker exec -it container2 sh -c 'cat /opt/testfile'
Created in container1
```

### Створення файлу у другому контейнері
```
$ sudo docker exec -it container2 sh -c 'echo "Created in container2" > /opt/testfile2'
```

### Читання файлу з першого контейнера
```
$ sudo docker exec -it container1 sh -c 'cat /opt/testfile2'
Created in container2
