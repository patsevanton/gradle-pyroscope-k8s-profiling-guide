Профилированию Gradle не создает pyroscope.application.name в Pyroscope

Описание проблемы

Ожидаемое поведение:

Текущее (фактическое) поведение:

Шаги для воспроизведения

Установка и настройка окружения Gradle:

# Загрузка дистрибутива Gradle
```
wget https://services.gradle.org/distributions/gradle-8.14.2-bin.zip
```
# Создание директории и распаковка
```
mkdir /opt/gradle
unzip -d /opt/gradle gradle-8.14.2-bin.zip
```
# Настройка переменной окружения PATH
```
export PATH=$PATH:/opt/gradle/gradle-8.14.2/bin
```
Создание тестового проекта:
```
mkdir gradle-profiling-example
cd gradle-profiling-example
gradle init --type java-application --dsl kotlin --test-framework junit-jupiter --project-name simple-app --package com.example
```
Загрузка Java-агента Pyroscope:
Агент загружается в специфичную для версии Gradle директорию кэша демона.
```
wget -O /home/user/.gradle/daemon/8.14.2/pyroscope.jar https://github.com/grafana/pyroscope-java/releases/download/v2.1.2/pyroscope.jar
```
Запуск сервера Pyroscope:
Создается файл docker-compose.yml в корне проекта:
```yaml
version: '3.8'
services:
  pyroscope:
    image: grafana/pyroscope:main-8c89229
    ports:
      - "4040:4040"
    command:
      - "server"
```
И запускается контейнер:
```
docker-compose up -d
```
Конфигурация Gradle для профилирования:
В корне проекта создается или редактируется файл gradle.properties со следующим содержанием:

# JVM аргументы для Gradle Daemon
```
org.gradle.jvmargs=-javaagent:pyroscope.jar -Dpyroscope.application.name=my-application{env=prod,version=1.0} -Dpyroscope.server.address=http://localhost:4040 -Dpyroscope.profile=cpu,alloc,lock,network,wall
```
Запуск профилируемой сборки:
```
gradle clean build --no-daemon
```
