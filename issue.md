[Баг] Инструкция по профилированию Gradle Daemon с помощью Java-агента не приводит к появлению данных в Pyroscope

Описание проблемы
При выполнении шагов из официальной инструкции по интеграции Pyroscope с Gradle для профилирования процесса Gradle Daemon, данные о профилировании не появляются в веб-интерфейсе Pyroscope. Сборка проекта проходит успешно, но агент, судя по всему, не подключается к демону или не может отправить данные на сервер.

Ожидаемое поведение:
После выполнения команды gradle clean build --daemon в веб-интерфейсе Pyroscope по адресу http://localhost:4040 должно появиться приложение с именем, указанным в gradle.properties (например, my-application или gradle-build-profiling), содержащее данные о профилировании (CPU, alloc и т.д.).

Текущее (фактическое) поведение:
Веб-интерфейс Pyroscope остается пустым, новое приложение не появляется. При этом в консоли нет явных ошибок, связанных с Pyroscope, и сборка Gradle успешно завершается.

Шаги для воспроизведения
Проблема воспроизводится при точном следовании инструкции.

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
mkdir gradle-profiling-example
cd gradle-profiling-example
gradle init --type java-application --dsl kotlin --test-framework junit-jupiter --project-name simple-app --package com.example
Загрузка Java-агента Pyroscope:
Агент загружается в специфичную для версии Gradle директорию кэша демона.
```
wget -O /home/user/.gradle/daemon/8.14.2/pyroscope.jar https://github.com/pyroscope-io/pyroscope-java/releases/latest/download/pyroscope.jar
```
Запуск сервера Pyroscope:
Создается файл docker-compose.yml в корне проекта:
```yaml
version: '3.8'
services:
  pyroscope:
    image: grafana/pyroscope:latest
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
gradle clean build --daemon
```
