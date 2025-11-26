# Система сборки, JDK и настройки проекта

Перед тем как приступить к созданию проекта в IntelliJ, полезно понять, что означают опции в окне создания (система сборки, JDK, DSL и т.п.).

**Что такое «система сборки» (build system)**

Система сборки — это инструмент, который автоматизирует:

- компиляцию исходников в байт-код,
- управление зависимостями (скачивание библиотек из репозиториев),
- запуск тестов,
- упаковку в артефакт (jar/war) и другие задачи (генерация документации, деплой и т.д.).

Примеры: **Gradle**, **Maven**. Также IDE (IntelliJ) может хранить собственные проектные файлы — это удобно для быстрой работы в IDE, но для автоматизации и CI чаще используют Gradle или Maven.

**Почему важно:** система сборки делает проект повторяемым и переносимым: другой разработчик или сервер сборки (CI) сможет собрать проект теми же командами.

**Что такое JDK (Java Development Kit)**

**JDK** — набор инструментов для разработки на Java: компилятор (javac), виртуальная машина (JRE), стандартные библиотеки и утилиты.

- JDK нужен для компиляции и запуска приложения.
- Версия JDK влияет на доступные возможности языка/библиотек и на совместимость (например, код, скомпилированный под Java 17, может не работать на Java 8).
- При создании проекта выбирают JDK, который будет использоваться IDE и системой сборки. В реальном проекте обычно указывают минимальную и целевую версии Java в настройках сборки.

**Maven vs Gradle vs «IntelliJ»**

- **Maven** — декларативный, строгий, имеет устоявшуюся структуру проекта и центральную концепцию lifecycle/plguins. Хорош для проектов с устоявшейся конфигурацией.
- **Gradle** — более гибкий и быстрый, использует сценарии (DSL) для описания сборки. Часто предпочтительнее для современных проектов.
- **IntelliJ (IDE)** — создаёт проектные файлы, удобные для работы в IDE, но это не замена полноценной системе сборки, если нужно CI/CD или сборка вне IDE.

**Gradle DSL: Kotlin vs Groovy**

Gradle позволяет писать скрипты сборки двумя языками:

- **Kotlin DSL (build.gradle.kts)** — статически типизирован, лучше поддерживается IDE (автодополнение, рефакторинг). Современный выбор.
- **Groovy DSL (build.gradle)** — исторически устоявшийся, много примеров в интернете.

Выбор влияет только на синтаксис файла сборки — функционально оба варианта дают те же возможности.

**Приложение Б**

(справочное)

# Модель GitHub Flow

GitHub Flow — простая и гибкая модель, разработанная для проектов, где главенствует принцип Continuous Integration. Модель предполагает создание новой ветки для каждой новой задачи, непрерывное тестирование при слиянии, а также постоянные/частые релизы без определенного релизного цикла.

Весь production-код содержится в ветке master/main. Для каждой новой фичи разработчик создает отдельную ветку от master/main. Одно из негласных правил данной модели — именовать ветки так, чтобы они описывали основную задачу фичи.

По завершению работы создается PR — pull request (Merge Request в Gitlab), запрос на слияние веток. Это ключевая особенность модели, когда перед слиянием (merge) инструментами системы git создается запрос на ревью кода, а после есть возможность поднять автотесты, только при прохождении которых будет выполнен merge в master.

**Основные ветки GitHub Flow:**

- main/master: единственная ветка, которая содержит стабильный код, готовый к развертыванию в продакшен.

**Преимущества GitHub Flow:**

- Простота и понятность процесса.
- Быстрый цикл разработки и релизов.
- Идеально подходит для CI/ CD

**Недостатки GitHub Flow:**

- Не подходит для проектов со сложными релизными циклами.
- Может быть рискованно для очень крупных проектов с множеством зависимостей.

**Правила (в сжатом виде):**

- master/main всегда в рабочем состоянии, готов к деплою.
- На любую задачу — отдельная ветка feature/имя, feat/имя и fix/имя от master/main.
- Часто коммитим и пушим.
- Когда фича готова — открываем PR в master/main, получаем «Approve».
- После merge в master/main — немедленная поставка (у нас это автоматический Release).

# GitHub Actions и папка .github/workflows/

**GitHub Actions** — встроенная CI/CD-платформа GitHub. Позволяет автоматизировать сборку, тесты, релизы, деплой и любые другие задачи, реагируя на события (push, PR, release, schedule и т.д.).

**.github/workflows/** — _стандартное_ место в репозитории для конфигурации workflow в виде YAML-файлов. GitHub автоматически сканирует эту папку и запускает workflows при совпадении триггеров. Каждый файл \*.yml в .github/workflows/ — отдельный workflow. Это даёт гибкость: один workflow — тесты, другой — статический анализ, третий — релиз.

**Анатомия workflow-файла (YAML)**

Каждый .yml в .github/workflows/ содержит примерно такие блоки:

- name — читаемое имя workflow.
- on — триггеры (push, pull_request, schedule, workflow_dispatch и т.д.).
- jobs — набор задач; каждая job выполняется на runner'е.
- в job: runs-on (типы runner'ов), steps (шаги: checkout, setup, build и т.д.), needs (зависимости от других jobs), env, permissions.

**Типовые триггеры и их назначение:**

- push — запускать при пуше в ветку(и) (обычно: main/master, develop).
- pull_request — проверка PR (линт, тесты, сборочные проверки) перед merge.
- workflow_dispatch — запуск вручную из UI (удобно для релиза или ручных задач).
- schedule — cron-расписание (например, nightly build).
- release, create, issue_comment и т.д. — очень гибко.

**Секреты в GitHub Actions и как их хранить**

**Секреты** — это защищённые значения (пароли, токены, ключи доступа, сертификаты и т.п.), которые хранит GitHub и предоставляет workflow'ам безопасным способом. Они нужны, чтобы не вносить чувствительные данные прямо в код или конфигурацию.

Секреты хранятся Settings → Secrets (или Environments) и используются  
как ${{ secrets.MY_SECRET }}.

GitHub **маскирует** значения секретов в логах: если значение встречается в выводе, его заменяют на \*\*\* (это помогает, но не абсолютная гарантия).

Секрет можно передать в шаг как переменную окружения:
```yaml
steps:
  - uses: actions/checkout@v4

  - name: Use secret
    env:
      API_KEY: ${{ secrets.MY_API_KEY }}
    run: |
      echo "Using key length: ${#API_KEY}" # не выводите само значение!
      ./deploy --api-key "$API_KEY"

# Или передать в action через with:
- name: Publish
  uses: some/publish-action@v1
  with:
    token: ${{ secrets.PUBLISH_TOKEN }}
```
**Простой шаблон workflow:**
```yaml
on:
  push:
    branches: [ main ]
  pull_request:

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: Set up JDK
        uses: actions/setup-java@v4
        with:
          distribution: 'temurin'
          java-version: '17'

      - name: Build
        run: ./gradlew build
```
**Полезные официальные Actions:**

- actions/checkout — клонирование репозитория.
- actions/setup-java — установка JDK.
- actions/cache — кеширование.

Всегда фиксируйте версию (например @v4 или конкретный tag), а не @master/@main.

# Git-тег и GitHub Releases

**Git-тег — это метка (имя), указывающая на конкретный коммит**. Теги часто используют для пометки релизов (версий), чтобы в любой момент можно было точно вернуться к состоянию кода, соответствующему той версии.

Теги нужны для:

- Фиксации версии/релиза: v1.2.3 отмечает коммит, который соответствует релизу.
- Исторических точек: легко переключиться на точку во времени (например, для отладки старой версии).
- Интеграции с CI/CD: workflow может реагировать на push по тегам (релизы по тегу).
- Документации/дистрибуции: к тегам прикрепляют binary assets через GitHub Releases.

Часто думают, что теги – это ветки. Нет: тег — неизменяемая ссылка на конкретный коммит. Ветки «передвигаются», теги — нет.

**Release** — это объект над Git-тегом, который служит «публичной точкой доставки» конкретной версии вашего ПО. Release хранит:

- ссылку на тег (например v1.2.3),
- заголовок и подробные заметки (release notes / changelog),
- прикреплённые бинарные артефакты (assets) — JAR/ZIP/EXE и т.п.,
- флаги draft / prerelease.

Releases дают удобный UI для пользователей (скачать бинарник), историю релизов, места для заметок и автоматические ссылки (API). Это — официальная «веха» выпуска.

**Release vs Tag — в чём разница**

- **Tag** — это чисто Git-объект (аннотированный git tag -a v1.0.0 или лёгкий git tag v1.0.0). Тег привязывает версию к коммиту.
- **Release** — надстройка GitHub над тегом: включает заметки и assets. Когда вы создаёте Release, GitHub либо создает тег, либо связывает Release с уже существующим тегом.
- Практика: тег — источник правды для версии; Release — удобный интерфейс/контейнер для публикации артефактов и заметок.

# JAR и fat JAR

**Обычный JAR (thin JAR)**

- Формируется стандартной задачей jar в Gradle.
- Содержит **только ваш код** (классы из src/main/java + ресурсы).
- Все **зависимости (библиотеки)** туда не входят.
- Чтобы запустить такое приложение, нужны:

Такой формат удобен, если приложение будет использоваться **как библиотека** в других проектах, где зависимости подтягивает Maven/Gradle.

Fat JAR (толстый JAR, uber JAR)

Формируется с помощью плагина **Shadow** (shadowJar).

Содержит:

- ваш код,
- **все сторонние зависимости**, упакованные внутрь.

Самодостаточный: можно просто скачать один файл и запустить:

Минус: размер файла больше (вместе с библиотеками).

Такой формат удобен для **приложений**, которые нужно распространять как готовый исполняемый артефакт.

|     |     |     |
| --- | --- | --- |
| **Характеристика** | **Обычный JAR (thin JAR)** | **Fat JAR (uber JAR, shadow JAR)** |
| **Состав** | Только ваш код и ресурсы | Ваш код + все зависимости внутри |
| **Размер файла** | Маленький (десятки–сотни КБ) | Большой (МБ, зависит от числа зависимостей) |
| **Запуск** | Нужны зависимости отдельно: java -cp app.jar:libs/\* org.example.Main | Самодостаточный файл: java -jar app.jar |
| **Когда использовать** | Библиотеки и модули, которые будут подключаться через Maven/Gradle в других проектах | Самостоятельные приложения (CLI, сервисы, утилиты), которые нужно запускать «из коробки» |
| **Плюсы** | Малый размер<br><br>Удобно подключать как зависимость | Самодостаточность<br><br>Удобно распространять и запускать |
| **Минусы** | Нельзя запустить без зависимостей<br><br>Требует правильного classpath | Большой размер файла<br><br>Неудобно использовать как библиотеку |

# CI/CD

**CI/CD** — это совокупность практик и инструментов, которые автоматизируют процесс разработки, тестирования и доставки программного обеспечения.

Расшифровка:

- **CI (Continuous Integration, непрерывная интеграция)** — практика частой интеграции изменений в общий репозиторий с автоматической сборкой и тестированием.
- **CD (Continuous Delivery или Continuous Deployment)** — автоматическая доставка изменений в целевую среду (staging, production) после успешной интеграции.

**Основные цели:**

- Сокращение времени между написанием кода и его доставкой пользователям.
- Минимизация ошибок, возникающих при ручной сборке и деплое.
- Повышение качества за счёт регулярного тестирования.
- Автоматизация рутинных процессов (сборка, тесты, публикация).

**Continuous Integration (CI)**

**Суть:**

Каждый разработчик как можно чаще (несколько раз в день) сливает изменения в основную ветку. При этом автоматически запускается:

- **Сборка проекта** (Gradle, Maven и т. д.),
- **Запуск тестов** (юнит-тесты, интеграционные тесты),
- **Статический анализ** (линтеры, проверка стиля, SonarQube и др.),
- **Проверка артефактов** (создание JAR, Docker-образа и т. п.).

**Преимущества CI:**

- Быстрая обратная связь о проблемах (сломался билд, упали тесты).
- Исключение ситуации «работает только у меня на компьютере».
- Поддержка главной ветки (main/master) в «рабочем» состоянии.

**Continuous Delivery (CD)**

**Суть:**

После успешного прохождения CI продукт автоматически **готов к развертыванию**.  
Обычно:

- Пакет публикуется в артефактное хранилище (например, **GitHub Releases**, **GitHub Packages**, **Nexus**, **Artifactory**),
- Создаётся Docker-образ,
- Формируется релиз-кандидат.

Развёртывание (deployment) может выполняться вручную (по кнопке).

**Delivery** = продукт всегда «готов к доставке».

**Continuous Deployment**

**Суть:**

Расширение практики Delivery.

Изменения, которые прошли CI, автоматически разворачиваются в **production** (рабочей среде) без участия человека.

Пример:

- Изменение прошло тесты,
- Автоматически собрано и задеплоено на сервер/в облако,
- Пользователь уже видит обновление.

**Deployment** = продукт не только готов, но и доставляется автоматически.

**Инструменты для CI/CD**

- **GitHub Actions** (встроено в GitHub, используется в нашем проекте),
- GitLab CI/CD,
- Jenkins,
- Travis CI,
- CircleCI,
- TeamCity, Bamboo.

# Пример выполнения Java + GitHub Flow + GitHub Actions (CI + релизы)

1.Требования**:**

- Установленный JavaJDK 23 (или версия по вашему выбору — тогда поменяйте в build.gradle и в workflow).
- Gradle.
- Git и аккаунт GitHub.
- IDE (IntelliJ IDEA (рекомендовано) / VSCode).

# 2\. Настройка проекта

Создайте проект со следующими параметрами:

**Name / Location** — имя проекта и папка, где создаётся корень.

**Create Git repository** — если включить, IntelliJ выполнит git init и добавит базовую инициализацию репозитория. Полезно сразу вести версионность.

**JDK** — выбор установленного JDK, который будет использоваться по умолчанию.

**Add sample code** — создаст шаблонный Main/тесты, чтобы быстро запустить приложение.

**Advanced Settings** — обычно содержат groupId/artifactId/package (важно для Maven/Gradle), настройки модулей и целей компиляции.

Реализуйте структуру проекта (добавьте .github/workflows и TodoApp.java, TodoList.java)

Следующим шагом необходимо задать фильтры для временный файлов сборки, конфигов IDE и системных файлов, которые не должны попадать в Git. Данный функционал реализуется с помощью .gitignore.

.gitignore — это **текстовый файл конфигурации Git**, в котором перечисляются шаблоны файлов и папок, которые **не должны попадать в систему контроля версий (Git)**.

Git ориентируется на этот файл при добавлении (git add) и коммите (git commit). Всё, что подходит под шаблон — **игнорируется**, т.е. не будет отслеживаться и храниться в истории репозитория.

Это нужно, чтобы в репозитории были только **исходники и важные конфиги**, а всякий «мусор» (собранные бинарники, временные файлы IDE, кеши и логи) туда не попадал.

В файле .gitignore добавьте следующие исключения:

**/build**

- Игнорирует папку build в корне проекта.
- Это стандартная папка Gradle, где хранятся **скомпилированные классы, JAR-файлы, тестовые отчёты** и т.д.
- Всё, что там — результат сборки, его всегда можно пересобрать, поэтому в Git это не нужно.

**/.gradle**

- Игнорирует папку .gradle в корне проекта.
- Там Gradle хранит **свой кеш и служебные данные** (кэшированные зависимости, compiled scripts, lock-файлы).
- Эти данные специфичны для каждой машины и не нужны в репозитории.

**/.idea**

- Игнорирует каталог .idea (служебные файлы IntelliJ IDEA/Android Studio).
- Там хранятся настройки проекта, привязанные к IDE (структура окон, расположение курсора, локальные плагины).
- Эти файлы могут отличаться у каждого разработчика, поэтому в Git не коммитят.

**\*.iml**

- Игнорирует файлы с расширением .iml (модули IntelliJ IDEA).
- Опять же, IDE-специфичные файлы, которые можно сгенерировать автоматически.

**\*.class**

- Игнорирует .class-файлы (скомпилированные Java-классы).
- Они всегда получаются при компиляции (javac или Gradle build), поэтому хранить их в репозитории нет смысла.

**\*.log**

- Игнорирует любые файлы логов (.log).
- Обычно логи генерируются при запуске приложения или тестов, и они нужны только для отладки локально.

**/.vscode**

- Игнорирует папку .vscode (настройки редактора Visual Studio Code).
- Тоже специфично для IDE: например, настройки плагинов или launch-конфиги. Их лучше держать у каждого локально.

**.DS_Store -** Системный файл macOS, который автоматически создаётся Finder для хранения метаданных папки (например, иконки, порядок отображения).Не несёт пользы для проекта.

Создайте локальный и удаленный (GitHub) репозитории. Для этого в верхнем меню выберите VLS → Share Project on GitHub.

Далее введите имя репозитория и нажмите «Share».

Появится окно добавления файлов в коммит:

Создайте feature-ветку от main: feat/todo-app-implementation. Для этого нажмите на main ветвь в верхнем меню и выберите «New Branch…»:

В окне создания ветви введите наименование и нажмите «Create»:

# 3\. Реализация приложения + создание GitHub Release и загрузка JAR в раздел Releases

Реализуйте класс TodoList с минимальной бизнес-логикой: add, remove(index), getAll, size.

```java
package org.example;

import java.util.ArrayList;
import java.util.List;

public class TodoList {
    private final List<String> items = new ArrayList<>();

    public void add(String item) {
        if (item != null) {
            item = item.trim();
            if (!item.isEmpty()) {
                items.add(item);
            }
        }
    }

    public boolean remove(int index) {
        if (index >= 0 && index < items.size()) {
            items.remove(index);
            return true;
        }
        return false;
    }

    public List<String> getAll() {
        return new ArrayList<>(items);
    }

    public int size() {
        return items.size();
    }
}
```


Создайте коммит с внесенными изменениями:

Реализуйте класс TodoApp: оболочка ввода/вывода (команды add, remove, list, exit)

```java
package org.example;

import java.util.List;
import java.util.Scanner;

public class TodoApp {
    public static void main(String[] args) {
        TodoList list = new TodoList();
        Scanner scanner = new Scanner(System.in);

        System.out.println("Simple Todo CLI. Commands: add <task>, remove <index>, list, exit");

        while (true) {
            System.out.print("> ");

            if (!scanner.hasNextLine()) break;

            String line = scanner.nextLine().trim();
            if (line.isEmpty()) continue;

            String[] parts = line.split(" ", 2);
            String cmd = parts[0].toLowerCase();

            switch (cmd) {
                case "add":
                    if (parts.length > 1) {
                        list.add(parts[1]);
                        System.out.println("Added.");
                    } else {
                        System.out.println("Usage: add <task>");
                    }
                    break;

                case "remove":
                    if (parts.length > 1) {
                        try {
                            int idx = Integer.parseInt(parts[1]);
                            if (list.remove(idx)) System.out.println("Removed.");
                            else System.out.println("Index out of range.");
                        } catch (NumberFormatException e) {
                            System.out.println("Invalid index.");
                        }
                    } else {
                        System.out.println("Usage: remove <index>");
                    }
                    break;

                case "list":
                    List<String> all = list.getAll();
                    for (int i = 0; i < all.size(); i++) {
                        System.out.printf("%d: %s%n", i, all.get(i));
                    }
                    if (all.isEmpty()) System.out.println("(empty)");
                    break;

                case "exit":
                    System.out.println("Bye!");
                    scanner.close();
                    return;

                default:
                    System.out.println("Unknown command. Commands: add, remove, list, exit");
            }
        }
    }
}
```


Создайте коммит с внесенными изменениями и отправьте в удаленный репозиторий:

Добавьте тестирование. Для этого создайте TodoListTest.java в src/test/java/:

После чего добавьте реализацию данного класса:

- добавление + нормализация пробелов;
- удаление по индексу и граничные случаи;
- игнорирование пустых строк.

CI должен «ломаться», если поведение нарушено — значит master/main не пропустит нерабочие изменения.

Реализация TodoListTest:

```java
import org.example.TodoList;
import org.junit.jupiter.api.Test;

import static org.junit.jupiter.api.Assertions.assertEquals;
import static org.junit.jupiter.api.Assertions.assertFalse;
import static org.junit.jupiter.api.Assertions.assertTrue;

public class TodoListTest {

    @Test
    void addAndList() {
        TodoList t = new TodoList();
        t.add(" task1 ");
        assertEquals(1, t.size());
        assertEquals("task1", t.getAll().get(0));
    }

    @Test
    void remove() {
        TodoList t = new TodoList();
        t.add("a");
        t.add("b");
        assertTrue(t.remove(0));
        assertEquals(1, t.size());
        assertFalse(t.remove(10));
    }

    @Test
    void addEmptyIgnored() {
        TodoList t = new TodoList();
        t.add(" ");
        assertEquals(0, t.size());
    }
}
```


Создайте и отправьте коммит с внесенными изменениями:

Откройте GitHub репозиторий и перейдите к вкладке PR (Pull requests):

Перейдём к «ручному созданию» пулреквест на странице со списком пулреквестов с помощью кнопки «New pull request»:

Выберите в качестве compare ветви (то, что хотим применить к базовой/основной ветви) ранее созданную ветвь с реализацией приложения и нажмите «Create pull request»:

Вас перекинет на страницу с настройками будущего пулреквеста. На ней можете добавить название и описать изменения. Также можете добавить ревьюеров для проверки изменений и тестирования.

Внизу страницы будут перечислены коммиты с изменениями.

Для создания пулреквеста нажмите «Create pull request»:

После этого появится созданный вами пулреквест, для которого необходимо будет выполнить слияние всех фиксаций в базовую ветвь. Нажмите «Merge pull request»:

Должен появится запрос с вводом содержимого коммита при слиянии. Здесь необходимо оставить значения по умолчанию и нажать «Confirm merge»:

Результат мерджа:

Если вернуться к коду репозитория, то можно увидеть коммит, созданный при PR.

Обновите main ветвь:

После обновления должен отобразиться новый граф коммитов:

Вернитесь в main ветвь:

Создайте feature-ветку от main: feat/actions.

Далее создайте и реализуйте конфигурационный файл Gradle, где можно задавать **свойства проекта** в виде ключ=значение (gradle.properties):

Данное свойство будет читаться в build.gradle через:

version = findProperty('version') ?: '0.1.0'

Оно будет управлять версией артефакта, который собирает Gradle (JAR → todo-app-0.1.0.jar) и участвовать в GitHub Actions workflow как tag_name для GitHub Release.

Это позволяет нам легко обновлять версию: перед релизом меняете строку version=0.2.0, коммитите → написанный нами в дальнейшем CI подхватывает это автоматически.

Реализация build.gradle:

Что делает:

- Подключает плагины для Java, CLI и Shadow.
- Определяет метаданные (group, version, mainClass).
- Использует Java 23 (или LTS по необходимости).
- Настраивает зависимости для тестирования (JUnit 5).
- Включает платформу JUnit Jupiter.
- Собирает **fat-JAR** (todo-app-&lt;version&gt;.jar) и делает его артефактом по умолчанию.
- Настраивает дистрибутивные задачи (distZip, distTar) на работу с fat-JAR.
- Гарантирует, что при build результат — полноценный исполняемый JAR.

Разберём, что здесь происходит:

1.  Подключение плагинов

```gradle
plugins {
id 'java'
id 'application'
id 'com.github.johnrengelman.shadow' version '8.1.1'
}
```
- **java**: Подключает базовый Java-плагин: компиляция .java файлов, задачи compileJava, test, jar и т.д.
- **application**: Добавляет задачи для запуска приложения (run) и генерации дистрибутивов (distZip, distTar). Нужен, если вы хотите запускать gradlew run или собирать дистрибутив с бинарями.
- **shadow** (Shadow plugin): Создаёт **fat-JAR** — один JAR, включающий все зависимости. Удобно для публикации в Release: пользователю достаточно одного файла. Версия 8.1.1 — актуальная для Gradle 8+.

1.  Группа, версия, совместимость
```gradle
group = 'org.example'
version = findProperty('version') ?: '0.1.0'
sourceCompatibility = '23'
targetCompatibility = '23'
mainClassName = 'org.example.TodoApp'
```
- **group** — идентификатор организации/пакета. Обычно совпадает с корневым пакетом (com.company, org.example).
- **version** — версия проекта. Здесь берётся из gradle.properties (через findProperty('version')), иначе подставляется 0.1.0. Это значение используется для имени JAR (todo-app-0.1.0.jar) и для релизов в CI.
- **sourceCompatibility / targetCompatibility** — версия Java, под которую компилируется код. Учтите, Gradle должен быть новой версии (8.4+), иначе Java 23 может не поддерживаться. Для надёжности часто используют LTS (17 или 21).
- **mainClassName** — главный класс для запуска через application и для манифеста в JAR.

1.  Репозитории
```gradle
repositories {

mavenCentral()

}
```
Здесь указываются места, откуда Gradle качает зависимости. mavenCentral() — стандартный глобальный репозиторий Maven.

1.  Зависимости
```gradle
dependencies {

testImplementation platform('org.junit:junit-bom:5.10.0')

testImplementation 'org.junit.jupiter:junit-jupiter'

}
```
- **testImplementation** — зависимости, доступные только в тестах.
- **junit-bom** — BOM (Bill of Materials), чтобы версии артефактов JUnit были согласованы.
- **junit-jupiter** — модуль JUnit 5 для написания и запуска тестов.

1.  Тесты
```gradle
test {

    useJUnitPlatform()

}
```
Говорит Gradle использовать JUnit Platform (JUnit 5) для запуска тестов.

Если этого не указать, тесты JUnit 5 могут не запускаться (по умолчанию Gradle использует JUnit 4).

1.  ShadowJar (сборка fat-JAR)
```gradle
shadowJar {
    archiveBaseName.set("todo-app")
    archiveClassifier.set("")        // чтобы не было "-all" в имени
    archiveVersion.set(project.version.toString())

    mergeServiceFiles()
}
```
- **archiveBaseName** — имя файла: todo-app.
- **archiveClassifier ''** — без дополнительного суффикса (-all и т.п.), чтобы выходной файл назывался просто todo-app-0.1.0.jar.
- **archiveVersion** — берём версию из version.
- **mergeServiceFiles()** — объединяет service-файлы (например, META-INF/services) из зависимостей, чтобы не было конфликтов.

Результат: при сборке в build/libs/ появляется **fat-JAR**, готовый к запуску.

1.  Отключение обычного jar
```java
tasks.jar {
    enabled = false
}
```
Выключает стандартную задачу jar (которая собирает «тонкий» JAR без зависимостей).

Таким образом, остаётся только shadowJar

1.  Зависимости задач дистрибутива
```gradle
tasks.named('distZip') {
    dependsOn tasks.named('shadowJar')
}

tasks.named('distTar') {
    dependsOn tasks.named('shadowJar')
}

tasks.named('startScripts') {
    dependsOn tasks.named('shadowJar')
}

tasks.named('startShadowScripts') {
    dependsOn tasks.named('jar')
}
```

- Эти строки перенастраивают задачи дистрибутива (distZip, distTar) так, чтобы они зависели от shadowJar.
- В результате, в архивы попадает **fat-JAR**, а не пустая или «тонкая» версия.
- startScripts — генерирует скрипты запуска (bin/ для Linux/Mac, bat/ для Windows).
- startShadowScripts → здесь сделана зависимость от jar, но поскольку jar отключён (enabled = false) — можно убрать (показано для примера).

1.  Главная задача build
```gradle
tasks.build {
    dependsOn shadowJar
}
```
Обычно build включает jar, но так как jar отключён, мы явно говорим: при gradlew build всегда собирать shadowJar.

Это важно для CI: шаг ./gradlew build гарантированно сделает fat-JAR.

settings.gradle оставляем без измненений:

settings.gradle — это конфигурационный файл Gradle, который выполняется **первым** при запуске Gradle и определяет структуру проекта: имя корневого проекта и (при необходимости) набор подключённых модулей (subprojects), политики разрешения плагинов/зависимостей и другие «глобальные» настройки сборки.

При любом запуске Gradle (например ./gradlew build, ./gradlew tasks) сначала выполняется settings.gradle из корневой папки проекта. На этом этапе Gradle узнаёт:

- имя проекта (rootProject.name),
- какие модули входят в сборку (include 'moduleA', 'moduleB'),
- настройки управления плагинами / репозиториями на уровне сборки,
- точки встраивания composite builds (includeBuild), и т.д.

После этого Gradle загружает build.gradle для каждого подключённого проекта.

Создайте коммит с внесенными изменениями:

[(Может возникнуть ошибка при коммите)](#commitFailed)

Добавьте файл .github/workflows/ci.yml:

И пропишите следующую реализацию:

```yaml
name: CI / Build & Release

on:
  push:
    branches:
      - main

jobs:
  build-and-release:
    runs-on: ubuntu-latest

    permissions:
      contents: write

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Make Gradle wrapper executable
        run: chmod +x gradlew

      - name: Set up JDK 23
        uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: '23'

      - name: Cache Gradle packages
        uses: actions/cache@v4
        with:
          path: |
            ~/.gradle/caches
            ~/.gradle/wrapper
          key: ${{ runner.os }}-gradle-${{ hashFiles('**/*.gradle*','**/gradle-wrapper.properties') }}
          restore-keys: |
            ${{ runner.os }}-gradle-

      - name: Get project version
        id: get_version
        run: |
          VERSION=$(grep '^version=' gradle.properties | cut -d'=' -f2)
          echo "VERSION=$VERSION" >> $GITHUB_OUTPUT

      - name: Build (run tests and create fat JAR)
        run: ./gradlew --no-daemon clean build
        env:
          JAVA_HOME: ${{ env.JAVA_HOME_23_X64 || '' }}

      - name: Create GitHub Release
        id: create_release
        uses: actions/create-release@v1
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          tag_name: v${{ steps.get_version.outputs.VERSION }}
          release_name: Release v${{ steps.get_version.outputs.VERSION }}
          body: Automated release for version ${{ steps.get_version.outputs.VERSION }}
          draft: false
          prerelease: false

      - name: Upload artifact to Release
        uses: actions/upload-release-asset@v1
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          upload_url: ${{ steps.create_release.outputs.upload_url }}
          asset_path: build/libs/todo-app-${{ steps.get_version.outputs.VERSION }}.jar
          asset_name: todo-app-${{ steps.get_version.outputs.VERSION }}.jar
          asset_content_type: application/java-archive
```

**Пояснения:**

- Workflow запускается при push в main. Это в точности соответствует GitHub Flow (изменения в main — автоматически тестируются и раскатываются / публикуются).
- Сначала извлекаем версию из gradle.properties и используем её как tag_name для релиза (v0.1.0).
- ./gradlew build запускает тесты и собирает JAR; в build.gradle мы сделали tasks.build зависящим от shadowJar, поэтому fat-JAR будет в build/libs/todo-app-&lt;version&gt;.jar.
- При упавших тестах job остановится — релиз не создастся (жёсткая гарантия качества в main).

Разберём данный workflow по блокам:

**1\. name и on**
```yaml
name: CI / Build & Release
on:
push:
branches:
\- main
```
Workflow называется «CI / Build & Release» и запускается на **каждом пуше в main**.

**2\. jobs / build-and-release / runs-on / permissions**
```yaml
jobs:
build-and-release:
runs-on: ubuntu-latest
permissions:
contents: write
```
**runs-on: ubuntu-latest** — раннер на Linux (обычная и недорогая опция для Java/Gradle).

**permissions: contents: write** — важно: даёт GITHUB_TOKEN право создавать релизы и загружать ассеты. Без этого шаг create-release/upload-release-asset вернёт 403.

**3\. Шаг: Checkout**
```yaml
- name: Checkout
uses: actions/checkout@v4
```
Клонирует репозиторий в раннер.

**4\. Шаг: Make Gradle wrapper executable**
```yaml
- name: Make Gradle wrapper executable
run: chmod +x gradlew
```

В Linux раннере файл gradlew должен иметь бит исполнителя. Если данное условие не выполняется, то происходит частая ошибка — ./gradlew: Permission denied.

Поэтому нужно ставить бит в репозиторий: _git update-index --chmod=+x gradlew && git commit -m "Make gradlew executable"_

Чтобы CI проходил и не выбрасывалась ./gradlew: Permission denied даже если файл gradle не имеет бит исполнителя используется run: chmod +x gradlew.

**5\. Шаг: Set up JDK 23**
```yaml
- name: Set up JDK 23
uses: actions/setup-java@v4
with:
distribution: temurin
java-version: '23'
```

Устанавливает Temurin JDK 23 и экспортирует переменные JAVA_HOME, JAVA_HOME_23_X64, т.к проект компилируется под Java 23 (sourceCompatibility = '23').

env JAVA_HOME: ${{ env.JAVA_HOME_23_X64 || '' }} в дальнейших шагах — забирает путь к установленной JDK.

**6\. Шаг: Cache Gradle packages**
```
- name: Cache Gradle packages
uses: actions/cache@v4
with:
path: |
~/.gradle/caches
~/.gradle/wrapper
key: ${{ runner.os }}-gradle-${{ hashFiles('\*\*/\*.gradle\*','\*\*/gradle-wrapper.properties') }}
restore-keys: |
${{ runner.os }}-gradle-
```
Ускоряет CI, повторно используя скачанные зависимости и wrapper.

**7\. Шаг: Get project version**
```yaml
- name: Get project version

id: get_version

run: |

VERSION=$(grep '^version=' gradle.properties | cut -d'=' -f2)

echo "VERSION=$VERSION" >> $GITHUB_OUTPUT
```

Пытается прочитать version из gradle.properties и выставить её в output шага get_version. Версия используется в tag_name, release_name и в имени артефакта.

**8\. Шаг: Build (run tests and create fat JAR)**
```yaml
- name: Build (run tests and create fat JAR)
run: ./gradlew --no-daemon clean build
env:
JAVA_HOME: ${{ env.JAVA_HOME_23_X64 || '' }}
```
Запускает Gradle сборку, которая (в build.gradle) вызывает shadowJar и запускает тесты (test).

Если тесты падают — job завершится с ошибкой и дальнейшие шаги (релиз) не выполнятся. Это хорошо — не публикуем нерабочие сборки.

**9\. Шаг: Create GitHub Release**
```yaml
- name: Create GitHub Release
id: create_release
uses: actions/create-release@v1
env:
GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
with:
tag_name: v${{ steps.get_version.outputs.VERSION }}
release_name: Release v${{ steps.get_version.outputs.VERSION }}
body: Automated release for version ${{ steps.get_version.outputs.VERSION }}
draft: false
prerelease: false
```
Создаёт Release в GitHub с тегом v&lt;version&gt;. Здесь:

- GITHUB_TOKEN автоматом доступен в Actions (не надо добавлять вручную). Он будет использоваться экшеном.
- Если тег с таким именем уже существует в репо — экшен может упасть (или создать release, связанный с существующим тегом; поведение зависит от версии экшена). Частая причина ошибки — повторная попытка создать релиз с тем же тегом.

**10\. Шаг: Upload artifact to Release**
```yaml
- name: Upload artifact to Release
uses: actions/upload-release-asset@v1
env:
GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
with:
upload_url: ${{ steps.create_release.outputs.upload_url }}
asset_path: build/libs/todo-app-${{ steps.get_version.outputs.VERSION }}.jar
asset_name: todo-app-${{ steps.get_version.outputs.VERSION }}.jar
asset_content_type: application/java-archive
```
Прикрепляет JAR к созданному релизу.

Создайте коммит с изменениями и отправьте в удаленный репозиторий:

Создайте PR:

Перейдите к пункту «Actions» вашего Github репозитория:

Должен появиться воркфлоу, запущенный при PR (т.к. изменения при слиянии пушатся в main ветвь):

Если вернуться к коду репозитория, то в блоке Releases должен появится Release v0.1.0:

Если нажать на Releases, то должна открыться страница с подробной информацией о релизах:

# 4\. GitHub Packages

Далее рассмотрим публикацию JAR в GitHub Packages (Maven Repository).

**GitHub Packages** — это встроенный пакетный репозиторий GitHub (аналог Maven Central, Docker Hub, npm registry и т. д.).

Вы можете загружать туда:

- Maven/Gradle пакеты (Java JARs),
- Docker-образы,
- npm-пакеты,
- NuGet и др.

Для Java-проектов GitHub Packages можно использовать как **Maven Repository**. Это особенно удобно, если проект задуман как библиотека или как приложение, которое должно быть доступно коллегам в виде зависимости.

Вернемся в проект и [обновим main ветвь, как это делали ранее, перед созданием ветви feat/actions.](#ОбновлениеMainВетви)

После обновления должен отобразиться новый граф коммитов:

Переключитесь на ветвь main

И от ветви main создайте feature-ветку от main: feat/github-packages

Добавим и реализуем workflow .github/workflows/publish-package.yml:

```yaml
name: Publish to GitHub Packages (Maven)

on:
  workflow_run:
    workflows: ["CI / Build & Release"]
    types:
      - completed

jobs:
  publish:
    if: ${{ github.event.workflow_run.conclusion == 'success' }}
    runs-on: ubuntu-latest
    env:
      GITHUB_ACTOR: ${{ github.actor }}
      GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
    permissions:
      contents: read
      packages: write

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Set up JDK 23
        uses: actions/setup-java@v4
        with:
          java-version: '23'
          distribution: 'temurin'

      - name: Make Gradle wrapper executable
        run: chmod +x gradlew

      - name: Publish package
        run: ./gradlew publish
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

**Разбор ключевых моментов:**

- on.workflow_run → workflow запускается только после завершения CI / Build & Release.
- if: ... == 'success' → публикация выполняется только если CI прошёл успешно.
- permissions.packages: write → даёт право загружать пакеты в GitHub Packages.
- ./gradlew publish → запускает задачу публикации из Gradle.

Настроим gradle для публикации. Для этого в build.gradle внесём изменения:

**Разбор ключевых моментов:**

- id 'maven-publish' → плагин Gradle для публикации артефактов.
- publications.mavenJava → описание, какой артефакт публикуем.
- artifact(tasks.shadowJar) → публикация **fat JAR**, собранного плагином Shadow (вместо обычного JAR).
- classifier = null → чтобы имя артефакта было todo-app-0.5.0.jar, а не todo-app-0.5.0-all.jar.
- repositories.maven → указывает, куда публиковать (maven.pkg.github.com/&lt;OWNER&gt;/&lt;REPO&gt;).
- credentials → берём логин (GITHUB_ACTOR) и токен (GITHUB_TOKEN) из переменных среды, которые Actions передаёт автоматически.

При публикации был выбран **fat JAR**, т.к наш проект – **консольное приложение (ToDo)**, а не библиотека, и мы хотим, чтобы любой пользователь мог просто скачать один файл (todo-app-0.5.0.jar) и сразу запустить.

Т.к. версионирование проекта у нас не работает автоматически изменим версию на 0.2.0 в gradle.properties:

Создайте коммит с изменениями и отправьте в удаленный репозиторий:

<div class="joplin-table-wrapper"><table><tbody><tr><td><p><a id="commitFailed"></a>В случае возникновения ошибки:</p><p><img src=""></p><p>Нажмите «Commit Anyway and Push…» или «Commit Anyway», если не нужно отправлять в удаленный репозиторий.</p><p></p><p>Ошибка «Commit and push checks failed» возникает потому, что система контроля качества или автоматических проверок JetBrains IDE обнаружила проблемы, препятствующие обычной фиксации и пушу изменений.</p><p>В данном случае в сообщении причины конкретные:</p><ul><li>В проекте обнаружены <strong>3 предупреждения</strong>&nbsp;(warnings) — обычно это замечания компилятора, инспектора или статического анализатора. (в данном случае никак не препятствуют корректной работе)</li><li>Найден <strong>1 TODO</strong> — это оставленный в коде комментарий TODO, который IDE идентифицирует как неразрешённую задачу (ошибочно помеченный IDE комметнарий, т.к. наименование нашего jar начинается с todo-app).</li></ul><p>Такие проверки часто настроены для предотвращения попадания неочищенного или неполного кода в репозиторий. Они могут срабатывать на:</p><ul><li>Компиляторные предупреждения (например, устаревшие конструкции, возможные ошибки, нарушения best practices).</li><li>Оставленные комментарии // TODO, // FIXME.</li><li>Нарушения стиля или стандарта в коде.</li></ul></td></tr></tbody></table></div>

После отправки изменений в удаленный репозиторий на GitHub создайте PR и замерджите:

Во вкладке Actions вашего GitHub репозитория запустится workflow «CI / Build & Release» и в случае его успешного прохождения запустится Publish to GitHub Packages (Maven):

После усчпешного выполнения второго воркфлов в репозитории появится package:

Теперь у нас:

1.  В GitHub Actions первый workflow (**CI / Build & Release**) выполняет **сборку, тесты и создание релиза**.
2.  Второй workflow (**Publish to GitHub Packages**) запускается **автоматически** после успешного завершения первого (используется on: workflow_run).

Таким образом реализуется схема **CD**:

- Код попал в main,
- он собран, протестирован,
- опубликован в Releases (как исполняемый JAR),
- и автоматически доставлен в GitHub Packages (как Maven-артефакт).

Это и есть **Continuous Delivery**: при каждом успешном изменении в main артефакт доступен как зависимость в пакетном репозитории.

# 5\. Как использовать пакет из GitHub Packages

После публикации пакет можно подключить в другом Gradle-проекте:

```gradle
repositories {
    maven {
        url = uri("https://maven.pkg.github.com/<ваш_GitHub_ник>/todo-app")
        credentials {
            username = project.findProperty("gpr.user") ?: System.getenv("GITHUB_ACTOR")
            password = project.findProperty("gpr.key") ?: System.getenv("GITHUB_TOKEN")
        }
    }
    mavenCentral()
}

dependencies {
    implementation 'org.example:todo-app:0.2.0'
}
```

**Итог**

В результате выполнения у нас:

- CI создаёт Release с исполняемым JAR.
- publish-package (CD) автоматически публикует тот же JAR в GitHub Packages как Maven-артефакт.
- Это позволяет:
- **скачивать готовый JAR из Releases**,
- **подключать пакет как зависимость через Gradle/Maven**.

# Дополнить функционал приложения:

1\. Новые команды (**в случае реализации приложения по методичке**):

\- clear

\- очистка всех задач;

\- done &lt;index&gt; - отметка задачи выполненной;

\- search &lt;строка&gt; - поиск задач по подстроке.

2\. Написать тесты для новых функций (**в случае реализации приложения по методичке**).

3\. Усложнение CI/CD. Настроить ещё один workflow, который (**для всех**):

\- запускается по расписанию (например, nightly build раз в день);

\- выполняет сборку и тесты, но не делает релиз, а только проверяет стабильность.