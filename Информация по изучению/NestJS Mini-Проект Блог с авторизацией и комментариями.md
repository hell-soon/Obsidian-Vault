
---

## Этап 1 — Базовая структура
- [ ] Установить NestJS CLI: `npm i -g @nestjs/cli`
- [ ] Создать проект: `nest new nest-blog`
- [ ] Изучить структуру проекта (main.ts, modules, controllers, services)

### Ресурсы
- [Официальная документация NestJS](https://docs.nestjs.com/)
- [NestJS CLI руководство](https://docs.nestjs.com/cli/overview)
- [Видео введение в NestJS (YouTube)](https://www.youtube.com/watch?v=GHTA143_b-s)

---

## Этап 2 — Аутентификация (JWT)
- [ ] Установить зависимости: `@nestjs/jwt`, `@nestjs/passport`, `passport-jwt`, `bcrypt`
- [ ] Создать `auth` модуль, контроллер, сервис
- [ ] Реализовать регистрацию пользователя с хешированием пароля
- [ ] Реализовать вход и генерацию JWT
- [ ] Защитить маршруты с помощью Guard

### Ресурсы
- [NestJS Authentication Guide](https://docs.nestjs.com/security/authentication)
- [Пример проекта с JWT](https://github.com/jmcdo29/nestjs-realworld-example-app)
- [bcrypt npm](https://www.npmjs.com/package/bcrypt)
- [Passport.js](http://www.passportjs.org/)

---

## Этап 3 — Пользователи и роли
- [ ] Создать `user` модуль и модель пользователя
- [ ] Реализовать получение и обновление профиля
- [ ] Добавить поле ролей (например, `"user" | "admin"`)
- [ ] (Опционально) Ограничить доступ к маршрутам по ролям

### Ресурсы
- [Документация по ролям и Guard](https://docs.nestjs.com/security/authorization)
- [Пример ролей в NestJS](https://wanago.io/2020/08/10/api-nestjs-role-based-access-control/)

---

## Этап 4 — Посты (CRUD)
- [ ] Создать `post` модуль
- [ ] Реализовать:
  - [ ] Создание поста (автор из JWT)
  - [ ] Получение всех постов
  - [ ] Получение одного поста
  - [ ] Обновление и удаление (только автор)

### Ресурсы
- [CRUD в NestJS с TypeORM](https://docs.nestjs.com/techniques/database)
- [Пример CRUD приложения](https://github.com/nestjs/typescript-starter)

---

## Этап 5 — Комментарии
- [ ] Создать `comment` модуль
- [ ] Связать комментарии с постами и пользователями
- [ ] Реализовать CRUD для комментариев
- [ ] Включить получение поста с комментариями

### Ресурсы
- [TypeORM отношения (Relations)](https://typeorm.io/#/relations)
- [Обзор работы с отношениями в NestJS](https://wanago.io/2021/06/21/api-nestjs-postgresql-typeorm/)

---

## Этап 6 — Подключение базы данных
- [ ] Выбрать БД и ORM:
  - [ ] PostgreSQL + Prisma (современно и быстро)
  - [ ] PostgreSQL + TypeORM (больше примеров с NestJS)
- [ ] Настроить подключение к БД
- [ ] Создать таблицы и миграции (пользователи, посты, комментарии)

### Ресурсы
- [Prisma Docs](https://www.prisma.io/docs/)
- [TypeORM Docs](https://typeorm.io/)
- [Настройка Prisma + NestJS](https://www.prisma.io/blog/build-a-nestjs-app-with-prisma-3ee1a16b66ff)
- [TypeORM + NestJS Tutorial](https://wanago.io/2020/04/06/api-nestjs-typeorm/)

---

## Этап 7 — Валидация и обработка ошибок
- [ ] Использовать DTO и `class-validator` для валидации данных
- [ ] Обработать ошибки (404, 401, 403) с помощью Exception Filters
- [ ] Вернуть единый формат ошибок

### Ресурсы
- [Валидация в NestJS](https://docs.nestjs.com/techniques/validation)
- [Обработка ошибок в NestJS](https://docs.nestjs.com/exception-filters)
- [class-validator Docs](https://github.com/typestack/class-validator)

---

## Этап 8 — Документация и финал
- [ ] Добавить Swagger документацию (`@nestjs/swagger`)
- [ ] Использовать `.env` и конфиг-модули
- [ ] (Опционально) Подключить WebSocket для реального времени (например, для комментариев)

### Ресурсы
- [NestJS Swagger](https://docs.nestjs.com/openapi/introduction)
- [Конфигурация в NestJS](https://docs.nestjs.com/techniques/configuration)
- [NestJS WebSocket](https://docs.nestjs.com/websockets/gateways)

---

# Дополнительно

- [Официальный Discord NestJS](https://discord.gg/nestjs)
- [Awesome NestJS (список полезных ресурсов)](https://github.com/juliandavidmr/awesome-nestjs)

---

