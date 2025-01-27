# Bishop CI/CD Python script
*Автоматизируйте анализ безопасности мобильных Android приложений при помощи платформы [Bishop](https://bishop.appsec.global/).*

Данный скрипт предназначен для встраивания анализа безопасности мобильных приложений в непрерывный процесс разработки (CI/CD).
В процессе выполнения скрипта приложение отправляется в платформу Bishop для анализа. На выходе формируется json файл с подробными результатами.

## Варианты запуска
На данный момент поддерживается несколько вариантов запуска:
 * анализ приложения, apk-файл которого расположенного локально
 * анализ приложения из системы [HockeyApp](https://hockeyapp.net/)
 * анализ приложений из системы [AppCenter](https://appcenter.ms)

## Параметры запуска
Параметры запуска зависят от расположения файла apk, отправляемого на анализ. Так же, существуют обязательные параметры, которые необходимо указывать при любом виде запуска:
 * `bishop_url` - сетевой адрес Bishop (путь до корня без последнего `/`), при использовании cloud версии - `https://saas.mobile.appsec.world`
 * `profile` - id профиля для которого проводится анализ
 * `testcase` - id testcase, который будет воспроизведен во время анализа; возможен запуск нескольких тесткейсов, для этого их id перечисляются через пробел
 * `token` - CI/CD токен для доступа (как его получить можно посмотреть в документации)
 * `distribution_system` - способ загрузки приложения, возможные опции: `file` и `hockeyapp`. Более подробно про них описано ниже в соответствующих разделах

Опциональные параметры:
 * `report` - тип отчета, который будет сформирован по результатам проведенного анализа. Возможные опции:
    * `separate` - одиночный отчет по проведенному сканированию, является отчетом по умолчанию при указании одного testcase id, при несколький testcase id создаст отдельный отчет для каждого запуска
    * `standard` - отчет по умолчанию при указании нескольких testcase id, совмещает дубликаты дефектов из нескольких сканирований в один (id сохраняются)
    * `grouping` - отчет, группирующий дефекты одного типа с указанием всех деталей.

 Можно указывать сразу несколько опций для создания отчетов нужных типов, например: `-- report separate grouping standard`. Для сканирований с одним тесткейсом возможно создание отдельных и группирующих отчетов.

### Локальный запуск
Данный вид запуска подразумевает, что apk файл приложения для анализа располагается локально, рядом (на одной системе) со скриптом.
Для выбора этого способа при запуске необходимо указать параметр `distribution_system file`. В этом случае обязательным параметром необходимо указать путь к файлу `file_path`

### HockeyApp
Для загрузки приложения из системы дистрибуции HockeyApp при запуске необходимо указать параметр `distribution_system hockeyapp`. Так же необходимо указать обязательные параметры:
 * `hockey_token` (обязательный параметр) - API токен для доступа. Как его получить можно узнать [здесь](https://rink.hockeyapp.net/manage/auth_tokens)
 * `hockey_version` (необязательный параметр) - при указании данного параметра будет скачана конкретная версия приложения по коду его версии (поле `version` в [API](https://support.hockeyapp.net/kb/api/api-versions)). При отсутствии данного параметра будет загружена последняя доступная версия приложения (latest).
 * `hockey_bundle_id` или `hockey_public_id` (обязательный параметр)
    * `hockey_bundle_id` - идентификатор Android приложения или, по другому, имя пакета (`com.swordfishsecurity.app.example`). При указании данной опции будет осуществлен поиск по всем приложениям внутри HockeyApp и выбрано приложение с соответствующим идентификатором. Поле в API - [bundle_identifier](https://support.hockeyapp.net/kb/api/api-apps)
    * `hockey_public_id` - идентификатор приложения внутри системы HockeyApp. При указании данного параметра будет загружено приложение с соответствующим идентификатором. Поле в API - [public_identifier](https://support.hockeyapp.net/kb/api/api-apps)

### AppCenter
Для загрузки приложения из системы дистрибуции AppCenter при запуске необходимо указать параметр `distribution_system appcenter`. Так же необходимо указать обязательные параметры:
 * `appcenter_token` - API токен для доступа. Как его получить можно узнать [здесь](https://docs.microsoft.com/en-us/appcenter/api-docs/)
 * `appcenter_owner_name` - владелец приложения, как узнать имя владельца можно [здесь](https://intercom.help/appcenter/en/articles/1764707-how-to-find-the-app-name-and-owner-name-from-your-app-url) или в [официальной документации](https://docs.microsoft.com/en-us/appcenter/api-docs/#find-your-app-center-app-name-and-owner-name)
 * `appcenter_app_name` - имя приложения в системе AppCenter. Как его узнать можно по [ссылке](https://docs.microsoft.com/en-us/appcenter/api-docs/#find-your-app-center-app-name-and-owner-name)
 * `appcenter_release_id` или `appcenter_app_version`
    * `appcenter_release_id` - идентификатор загружаемого релиза в системе AppCenter для конкретного приложения. Возможно выставить значение `latest` - тогда будет загружен последний доступный релиз приложения. [Официальная документация](https://openapi.appcenter.ms/#/distribute/releases_getLatestByUser)
    * `appcenter_app_version` - при указании данного параметра будет найдена и скачана конкретная версия приложения по коду его версии (указанной в Android Manifest) (поле `version` в [документации](https://openapi.appcenter.ms/#/distribute/releases_list)).

## Примеры запуска

#### Локальный файл
Для запуска анализа локального файла:

```
python3.6 run-bishop-scan.py --distribution_system file --file_path "/bishop/demo/apk/swordfish-demo.apk" --bishop_url "https://saas.mobile.appsec.world" --profile 1 --testcase 4 --token "eyJ0eXA4OiJKA1QiLbJhcGciO5JIU4I1NiJ1.eyJzdaJqZWNcX2lkIj53LCJle5AiOjf1OTM5OTU3MjB1.hfI6c4VN_U2mo5VfRoENPvJCvpxhLzjHqI0gxqgr2Bs"
```

В результате будет запущен автоматизированный анализ приложения `swordfish-demo.apk` с профилем с `id` 1 и будет запущен тест кейс с `id` 4.

#### HockeyApp по bundle_identifier и указанием версии
Для запуска анализа приложения из системы HockeyApp:

```
python3.6 run-bishop-scan.py --distribution_system hockeyapp --hockey_token 18bc81146d374ba4b1182ed65e0b3aaa --bundle_id com.swordfishsecurity.demo --hockey_version 31337 --bishop_url "https://saas.mobile.appsec.world" --profile 2 --testcase 3 --token "eyJ0eXA4OiJKA1QiLbJhcGciO5JIU4I1NiJ1.eyJzdaJqZWNcX2lkIj53LCJle5AiOjf1OTM5OTU3MjB1.hfI6c4VN_U2mo5VfRoENPvJCvpxhLzjHqI0gxqgr2Bs"
```

В результате в системе HockeyApp будет найдено приложение с идентификатором пакета `com.swordfishsecurity.demo` и версией `31337`. Он будет скачен и для него будет проведен автоматизированный анализ с профилем с `id` 2 и будет запущен тест-кейс с `id` 3.

#### HockeyApp по public_identifier и с последней доступной версией
Для запуска анализа последней версии приложения из системы HockeyApp по его публичному идентификатору:

```
python3.6 run-bishop-scan.py --distribution_system hockeyapp --hockey_token 18bc81146d374ba4b1182ed65e0b3aaa --public_id "1234567890abcdef1234567890abcdef" --bishop_url "https://saas.mobile.appsec.world" --profile 2 --testcase 3 --token "eyJ0eXA4OiJKA1QiLbJhcGciO5JIU4I1NiJ1.eyJzdaJqZWNcX2lkIj53LCJle5AiOjf1OTM5OTU3MjB1.hfI6c4VN_U2mo5VfRoENPvJCvpxhLzjHqI0gxqgr2Bs"
```

В результате в системе HockeyApp будет найдено приложение с уникальным публичным идентификатором `1234567890abcdef1234567890abcdef` и последней доступной версией. Файл приложения будет скачен и для него будет проведен автоматизированный анализ с профилем с `id` 2 и будет запущен тест-кейс с `id` 3.

#### AppCenter по id релиза
Для запуска анализа приложения по известному имени, владельцу и ID релиза необходимо выполнить следующую команду:

```
python3.6 run-bishop-scan.py --distribution_system appcenter --appcenter_token 18bc81146d374ba4b1182ed65e0b3aaa --appcenter_owner_name yshabalin_test_org_or_user --appcenter_app_name Swordfish_debug_version_of_test --appcenter_release_id 710 --bishop_url "https://saas.mobile.appsec.world" --profile 2 --testcase 3 --token "eyJ0eXA4OiJKA1QiLbJhcGciO5JIU4I1NiJ1.eyJzdaJqZWNcX2lkIj53LCJle5AiOjf1OTM5OTU3MjB1.hfI6c4VN_U2mo5VfRoENPvJCvpxhLzjHqI0gxqgr2Bs"
```

В результате у владельца (пользователя или организации `yshabalin_test_org_or_user`) будет найдено приложение `Swordfish_debug_version_of_test` с ID релиза `710`. Данная версия релиза будет загружена и передана на анализ безопасности в Bishop

Для загрузки релиза с последней версией необходимо параметр `appcenter_release_id latest`. Тогда команда будет выглядеть следующим образом:

```
python3.6 run-bishop-scan.py --distribution_system appcenter --appcenter_token 18bc81146d374ba4b1182ed65e0b3aaa --appcenter_owner_name "yshabalin_test_org_or_user" --appcenter_app_name "Swordfish_debug_version_of_test" --appcenter_release_id latest --bishop_url "https://saas.mobile.appsec.world" --profile 2 --testcase 3 --token "eyJ0eXA4OiJKA1QiLbJhcGciO5JIU4I1NiJ1.eyJzdaJqZWNcX2lkIj53LCJle5AiOjf1OTM5OTU3MjB1.hfI6c4VN_U2mo5VfRoENPvJCvpxhLzjHqI0gxqgr2Bs"
```

и загружен последний доступный релиз для данного приложения.

#### AppCenter по версии приложения
Для запуска анализа приложения по известному имени, владельцу и версии приложения (`version_code` в `Android Manifest`) необходимо выполнить следующую команду:

```
python3.6 run-bishop-scan.py --distribution_system appcenter --appcenter_token 18bc81146d374ba4b1182ed65e0b3aaa --appcenter_owner_name "yshabalin_test_org_or_user" --appcenter_app_name "Swordfish_debug_version_of_test" --appcenter_app_version 31337 --bishop_url "https://saas.mobile.appsec.world" --profile 2 --testcase 3 --token "eyJ0eXA4OiJKA1QiLbJhcGciO5JIU4I1NiJ1.eyJzdaJqZWNcX2lkIj53LCJle5AiOjf1OTM5OTU3MjB1.hfI6c4VN_U2mo5VfRoENPvJCvpxhLzjHqI0gxqgr2Bs"
```

В результате у владельца (пользователя или организации `yshabalin_test_org_or_user`) будет найдено приложение `Swordfish_debug_version_of_test` и найден релиз, в котором была указана версия приложения `31337`. Данная версия релиза будет загружена и передана на анализ безопасности в Bishop.
