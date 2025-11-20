Проект “Планировщик задач”
===============================================================================================

Многопользовательский планировщик задач, реализованный как многомодульное приложение. Пользователи могут использовать его как TODO-лист: регистрироваться, авторизовываться, создавать, редактировать и удалять задачи. При регистрации отправляется приветственное email-сообщение, а ежедневные отчёты о прогрессе задач приходят пользователям по email с помощью микросервисов и Kafka. Источником вдохновения для проекта является Trello.

(Пет-проект, написанный для освоения и закрепления навыков в Java, Spring Boot и других технологиях)

Оглавление
----------

1. [Использованные инструменты / технологии](#использованные-инструменты--технологии)
2. [Архитектура и модули](#архитектура-и-модули)
3. [Интерфейс приложения](#интерфейс-приложения)
4. [Базы данных](#базы-данных)
5. [Требования приложения](#требования-приложения)
6. [Инструкция по запуску приложения](#инструкция-по-запуску-приложения)
7. [CI/CD](#cicd)
8. [Техническое задание](#техническое-задание)

## Использованные инструменты / технологии:

### Backend

![Java](https://img.shields.io/badge/Java-ED8B00?style=for-the-badge&logo=java&logoColor=ED8B00&labelColor=333333) &nbsp;
![Spring Boot](https://img.shields.io/badge/Spring_Boot-6DB33F?style=for-the-badge&logo=spring-boot&logoColor=6DB33F&labelColor=333333) &nbsp;
![Spring Security](https://img.shields.io/badge/Spring_Security-6DB33F?style=for-the-badge&logo=spring-security&logoColor=6DB33F&labelColor=333333) &nbsp;
![Spring Kafka](https://img.shields.io/badge/Spring_Kafka-6DB33F?style=for-the-badge&logo=spring&logoColor=6DB33F&labelColor=333333) &nbsp;
![Spring Mail](https://img.shields.io/badge/Spring_Mail-6DB33F?style=for-the-badge&logo=spring&logoColor=6DB33F&labelColor=333333) &nbsp;
![Postgres](https://img.shields.io/badge/Postgres-4169E1?style=for-the-badge&logo=postgresql&logoColor=4169E1&labelColor=333333) &nbsp;
![Kafka](https://img.shields.io/badge/Kafka-231F20?style=for-the-badge&logo=apachekafka&logoColor=231F20&labelColor=333333) &nbsp;
![Liquibase](https://img.shields.io/badge/Liquibase-2962FF?style=for-the-badge&logo=liquibase&logoColor=2962FF&labelColor=333333) &nbsp;
![Lombok](https://img.shields.io/badge/Lombok-1A80C3?style=for-the-badge&logo=lombok&logoColor=1A80CELL&labelColor=333333) &nbsp;
![Maven](https://img.shields.io/badge/Maven-C71A36?style=for-the-badge&logo=apachemaven&logoColor=C71A36&labelColor=333333) &nbsp;
![SLF4J](https://img.shields.io/badge/SLF4J-2C2255?style=for-the-badge&logo=slf4j&logoColor=2C2255&labelColor=333333)

### Frontend

![HTML](https://img.shields.io/badge/HTML-E34F26?style=for-the-badge&logo=html5&logoColor=E34F26&labelColor=333333) &nbsp;
![CSS](https://img.shields.io/badge/CSS-1572B6?style=for-the-badge&logo=css3&logoColor=1572B6&labelColor=333333) &nbsp;
![JavaScript](https://img.shields.io/badge/JavaScript-F7DF1E?style=for-the-badge&logo=javascript&logoColor=F7DF1E&labelColor=333333) &nbsp;
![jQuery](https://img.shields.io/badge/jQuery-0769AD?style=for-the-badge&logo=jquery&logoColor=0769AD&labelColor=333333) &nbsp;
![Nginx](https://img.shields.io/badge/Nginx-009639?style=for-the-badge&logo=nginx&logoColor=009639&labelColor=333333)

### Инфраструктура

![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=2496ED&labelColor=333333) &nbsp;
![Git](https://img.shields.io/badge/Git-F05032?style=for-the-badge&logo=git&logoColor=F05032&labelColor=333333) &nbsp;
![GitHub Actions](https://img.shields.io/badge/GitHub_Actions-2088FF?style=for-the-badge&logo=github-actions&logoColor=2088FF&labelColor=333333)

## Архитектура и модули

Проект построен на микросервисной архитектуре и состоит из четырёх основных модулей, взаимодействующих через REST API и брокер сообщений Kafka. Каждый модуль выполняет свою задачу, а Kafka используется для асинхронной отправки email-уведомлений.

### task-tracker-backend
- **Функционал**: Основной REST API сервис для работы с пользователями и задачами.
    - Регистрация (`POST /user`) с автоматической отправкой приветственного email через Kafka.
    - Получение информации о текущем пользователе (`GET /user`)
    - Аутентификация (`POST /auth/login`) с использованием JWT-токенов.
    - Управление задачами: 
      - создание (`POST /tasks`), 
      - получение списка (`GET /tasks`), 
      - редактирование (`PUT /tasks/{taskId}`), 
      - удаление (`DELETE /tasks/{taskId}`), 
      - получение по ID (`GET /tasks/{taskId}`).
- **Технологии**: Spring Boot, Spring Security, Spring Data JPA, Liquibase.
- **Зависимости**: Postgres для хранения данных, Kafka для отправки задач на email.

### task-tracker-frontend
- **Функционал**: Одностраничное веб-приложение, раздаваемое через Nginx.
    - Отображает интерфейс для авторизации, регистрации и управления задачами.
    - Взаимодействует с `task-tracker-backend` через Ajax-запросы с префиксом `api/`, добавляемым Nginx при проксировании.
- **Технологии**: HTML, CSS, JavaScript, jQuery, Nginx.

### task-tracker-email-sender
- **Функционал**: Сервис для отправки email-уведомлений.
    - Слушает Kafka-топик `EMAIL_SENDING_TASKS`, десериализует сообщения и отправляет email через SMTP (например, SendGrid).
    - Обрабатывает приветственные письма при регистрации и ежедневные отчёты от `task-tracker-scheduler`.
- **Технологии**: Spring Boot, Spring Mail, Spring Kafka.

### task-tracker-scheduler
- **Функционал**: Сервис для формирования и отправки ежедневных отчётов.
    - Каждую полночь (по Cron) анализирует задачи пользователей за сутки.
    - Отправляет сообщения в Kafka-топик `EMAIL_SENDING_TASKS` с отчётами:
        - Количество выполненных задач за день (если есть).
        - Список невыполненных задач (если остались).
- **Технологии**: Spring Boot, Spring Scheduler, Spring Kafka.

### Роль Kafka
Kafka используется как брокер сообщений для асинхронной передачи задач на отправку email между сервисами:
- `task-tracker-backend` отправляет приветственное письмо при регистрации.
- `task-tracker-scheduler` отправляет ежедневные отчёты.
- `task-tracker-email-sender` читает топик `EMAIL_SENDING_TASKS` и выполняет отправку.

## Интерфейс приложения

Приложение одностраничное, всё взаимодействие с сервером осуществляется через JavaScript/Ajax, а интерфейс обновляется с помощью jQuery.

### Главная страница

URL - '/'

![main-page-non-auth](assets/main-page-non-auth.png)

- Для неавторизованных пользователей: отображает кнопки входа и регистрации.

![main-page](assets/main-page.png)

- Для авторизованных пользователей: отображает список задач (сделанные и несделанные), кнопку для создания новой задачи и кнопку "Logout".

![edit-modal](assets/edit-modal.png)

- Модальное окно редактирования задачи, позволяет изменять заголовок, описание и статус задачи.

### Страница входа

![login](assets/login.png)

URL - '/auth/login' (обрабатывается через модальное окно на главной странице)

### Страница регистрации

![register](assets/register.png)

URL - '/user' (обрабатывается через модальное окно на главной странице)

## Базы данных

![db-diagram](assets/db-diagram.png)

- **Postgres**: Хранит данные пользователей (`users`) и задач (`tasks`).
    - Таблица `username`: `id`, `password` (хранится в зашифрованном виде для Spring Security).
    - Таблица `tasks`: `id`, `title`, `description`, `created_at`, `completed`, `completed_at`, `user_id`.
- **Liquibase**: Используется для управления миграциями базы данных (создание и обновление схемы). Миграции находятся в модуле `task-tracker-backend`.

## Требования приложения

- Java 21+
- Apache Maven
- Docker и Docker Compose
- SMTP-сервер (например, SendGrid) для отправки email-уведомлений

## Инструкция по запуску приложения

### Клонирование репозитория

1. **Клонирование репозитория с помощью Git**:
    - Откройте терминал или командную строку.
    - Выполните команду:
      ```sh
      git clone https://github.com/VladShi/task-tracker.git
      ```
    - Перейдите в директорию проекта:
      ```sh
      cd task-tracker
      ```

### Настройка проекта

1. **Настройка конфигурационных файлов**:

    - В каждом модуле (`task-tracker-backend`, `task-tracker-scheduler`, `task-tracker-email-sender`) в директории `src/main/resources` есть файл `application.properties.example`.
    - Скопируйте его в `application.properties`:
      ```sh
      cp task-tracker-backend/src/main/resources/application.properties.example task-tracker-backend/src/main/resources/application.properties
      cp task-tracker-scheduler/src/main/resources/application.properties.example task-tracker-scheduler/src/main/resources/application.properties
      cp task-tracker-email-sender/src/main/resources/application.properties.example task-tracker-email-sender/src/main/resources/application.properties
      ```
    - Значения по умолчанию в этих файлах подходят для работы "из коробки", но вы можете изменить их при необходимости.

2. **Настройка переменных окружения**:

   - В корне проекта находится файл `.env.example`.
   - Создайте файл `.env` на его основе:
     ```sh
     cp .env.example .env
     ```
   - Откройте `.env` и проверьте значения. Для работы "из коробки" они уже настроены, но для отправки email-уведомлений заполните SMTP-настройки своим данными:
     ```env
     # SMTP server
     MAIL_HOST=smtp.sendgrid.net
     MAIL_PORT=587
     MAIL_USERNAME=your_username
     MAIL_PASSWORD=your_password
     MAIL_VERIFIED_EMAIL=your_email@example.com
     ```
   - Остальные переменные (например, `POSTGRES_HOST`, `KAFKA_BOOTSTRAP_SERVERS`) можно оставить как есть.

### Запуск приложения

1. **Запуск всех модулей приложения через Docker Compose**:

   - Убедитесь, что Docker и Docker Compose установлены.
   - В корне проекта выполните:
     ```sh
     docker-compose up -d --build
     ```
   - Это соберёт образы для всех модулей (`backend`, `frontend`, `scheduler`, `email-sender`) и запустит контейнеры вместе с зависимостями (`postgres`, `kafka`, `zookeeper`).
   - Приложение будет доступно по адресу http://localhost:80.

2. **Запуск модулей и окружения для разработки**:

    Если нужно запустить только окружение и отдельные модули вручную (например, через IDE):
   - Используйте `docker-compose.yml` вместе с `docker-compose.dev.yml`:
     ```sh
     docker-compose -f docker-compose.yml -f docker-compose.dev.yml up -d
     ```
   - Отключите сборку ненужных модулей с помощью профилей в `docker-compose.dev.yml`. Например, чтобы отключить `task-tracker-backend`:
     ```yaml
     task-tracker-backend:
       profiles:
         - disabled
     ```
   - Запустите модуль вручную, например, через IntelliJ IDEA, указав главный класс модуля: 
     - `BackendApplication`
     - `EmailSenderApplication`
     - `SchedulerApplication`
   - Или для запуска модуля без IDE с помощью Maven и терминала:
     - Перейдите в директорию нужного модуля, например:
       ```sh
       cd task-tracker-backend
       ```
     - Соберите проект и запустите приложение:
       ```sh
       mvn clean package
       java -jar target/task-tracker-backend-1.0-SNAPSHOT.jar
       ```
       имя JAR-файла может отличаться, это зависит от настроек сборки и версии указанных в `pom.xml`.

## CI/CD
- Проект использует GitHub Actions для автоматизации сборки и деплоя.
- Workflow-файлы находятся в `.github/workflows/`:
  - `ci.yml`: Сборка Docker-образов для всех модулей (`backend`, `frontend`, `scheduler`, `email-sender`) и их публикация на Docker Hub.
  - `cd.yml`: Деплой приложения на удалённый сервер с использованием `docker-compose.prod.yml`, который загружает образы с Docker Hub и запускает их.
- Для деплоя требуется настроить секреты в GitHub (например, `SSH_PRIVATE_KEY`, `DOCKERHUB_USERNAME`, `MAIL_PASSWORD`).
