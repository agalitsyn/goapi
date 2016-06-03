[![Build Status](https://travis-ci.org/agalitsyn/goapi.svg?branch=master)](https://travis-ci.org/agalitsyn/goapi)

# goapi

Репозиторий служит примером 12-факторного приложения, с минимальной струтурой и
3rd-party пакетам, а так же содержит тулинг для разработчика, такой как makefiles, настройки CI, небольшие советы.

По мотивам [статьи](https://medium.com/@kelseyhightower/12-fractured-apps-1080c73d481c#.ihna7diaw)

Не рекомендуется для production.

Если вы уже готовы писать настоящее приложение для production, то попробуйте решения 
- https://github.com/labstack/echo
- https://github.com/ant0ine/go-json-rest

Cтруктура проекта имеет схему:
```
github.com/agalitsyn/foo
    foo.go      // package foo
    foo_test.go // package foo
    cmd/
        foo/
            main.go      // package main
            main_test.go // package main
```

Получается инвертированная структура, когда код библиотеки кладётся в корень,
а во вложенной папке `cmd/foo/` хранится код исполняемых программ.

Промежуточный уровень `cmd/` удобен по двум причинам:
* Инструментарий Go автоматически именует двоичные файлы по названию папки,
в которой находится package `main`, так что мы получаем наилучшие имена файлов без
возможных конфликтов с другими пакетами в репозитории.
* Если ваши пользователи применяют `go get` на путь, в котором содержится `/cmd/`,
они сразу понимают, что получили. Подобным образом устроен репозиторий сборочной утилиты [gb](https://github.com/constabulary/gb).

## Зависимости

### Управление зависимостиями

Для управления зависимостиями используется утилита [govendor](https://github.com/kardianos/govendor).
Ее выбор вместо популярной [godep](https://github.com/tools/godep) обусловлен более богатым функционалом,
например выводом статистики по пакетам, и отсутсвием проблем при работе с go 1.6+.

Обязательно нужно ознакомиться с инструкцией по работе с `govendor`.

### Библиотеки

Все библиотеки и их версии описаны в файле-манифесте `vendor/vendor.json`.

#### Почему именно эти библиотеки?

В go есть большое многообразие разных пакетов и часто возникает вопрос, а что взять?

Лично я руководствовался следующими принципами:
* Зависимости
    * Их мало или их нет. Вспоминаем left-pad problem и постулат go, что небольшое копирование лучше зависимости.
    * Зависимости библиотеки не завендорены. Иначе, если у вас будет подобное дерево, у вас будет проблема сборки.

        ```
        - foo.go
          - vendor/
            - a/
            - b/
                - vendor/a/
        ```

    *Tip:* проверить грамотность использования зависимостей можно используя тулзу https://github.com/divan/depscheck

* Тесты
    * У пакета есть автотесты и код имеет coverage как минимум 60%. Естественно, чем больше тем лучше, зависит от наличия конкурентов, может есть только один пакет.
    * Настроен CI и пакет постоянно тестируется.
    * Можно найти какие-то benchmarks.

    *Tip:* всегда можно проверить пакет самому, например запустить gometalinter, тесты + race detector или написав benchmark, например вот так http://lk4d4.darth.io/posts/bench/.

* Пакет реализует минимальный функционал, т.е. нет побочных "дополнительных" фичей, которые раздувают кодовую базу и не являются прямой обязанностью пакета.
* Отзывы коллег и сообщества.
* Активность на github, количество открытых issue, pull requests и pulse проекта.

## Тесты

```sh
$ make test
```

## CI

### Gitlab CI

Все настройки в файле `.gitlab-ci.yml`.

Документация по синтаксису [.gitlab-ci.yml](http://doc.gitlab.com/ce/ci/yaml/README.html)

Используется docker runner.

Так же есть [документация](https://gitlab.com/gitlab-org/gitlab-ce/blob/76109d754e167e05db7897f6b89a36b2fadffc65/doc/ci/examples/test-golang-application.md),
в которой есть пример настройки окружения не в docker контейнере.

Если когда-нибудь для тестов понадобится база данныхб смотрите раздел services [postgres](http://docs.gitlab.com/ce/ci/services/postgres.html).

Сам образ для CI собран на основе apline linux, найти его можно тут https://github.com/agalitsyn/goenv/tree/master/docker/ci

### Travis CI

Все настройки в файле `.travis.yml`
