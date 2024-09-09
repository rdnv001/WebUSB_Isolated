# Оглавление
- [Оглавление](#оглавление)
- [Текущее состояние возможности доступа к смарт-картам из браузера через WebUSB API ](#текущее-состояние-возможности-доступа-к-смарт-картам-из-браузера-через-webusb-api-)
- [Isolated Web App ](#isolated-web-app-)
- [Прототип использования WebUSB API для доступа к смарт-картам](#прототип-использования-webusb-api-для-доступа-к-смарт-картам)
  - [MacOS ARM ](#macos-arm-)
  - [Windows x86\_64 ](#windows-x86_64-)
  - [Linux x86\_64 ](#linux-x86_64-)

# Текущее состояние возможности доступа к смарт-картам из браузера через WebUSB API <a name="introduction"></a>

__Web USB__ – это JavaScript API, который даёт возможность веб-приложениям взаимодействовать с локальными USB-устройствами на компьютере.
[Спецификация WebUSB](https://wicg.github.io/webusb) определяет блок-лист уязвимых устройств и таблицу защищённых классов интерфейсов, доступ к которым заблокирован через WebUSB, в число которых входят смарт-карты и HID: https://wicg.github.io/webusb/#has-a-protected-interface-class

Начиная с версии 129 Chrome Desktop, в браузер добавляется возможность обхода описанных выше ограничений в Isolated Web Apps при задании разрешения __«usb-unrestricted»__ в поле «permissions_policy» манифеста.

    "permissions_policy": {
        "usb": [ "self" ],
        "usb-unrestricted": [ "self" ]
    }

Стоит отметить, что существует заявка на добавление в Chrome Web Smart Card API: https://chromestatus.com/feature/6411735804674048 , однако последнее сообщение по этой заявке датируется 2022 годом.

Спецификация: https://wicg.github.io/web-smart-card/

# Isolated Web App <a name="isolated"></a>               
__Isolated Web App (IWA)__ – это приложения, упакованные в Web Bundles (веб-пакеты), подписанные их разработчиком и распространённые среди конечных пользователей.

IWA функционируют в изолированной среде с использованием собственной схемы адресации (isolated-app://). Они должны быть установлены на устройство пользователя и распространяются в виде подписанных веб-пакетов (Signed Web Bundles). Это отличает их от традиционных веб-приложений, которые загружаются и выполняются динамически через HTTP протокол. 

Браузер может проверить целостность IWA, проверив подпись и сравнив соответствующий открытый ключ с списком известных доверенных открытых ключей. 

При загрузке URL с префиксом isolated-app://, пользовательский агент проверяет подписи веб-пакета и доверяет ли он открытому ключу, который использовался для его подписи. Агент может доверять, например, открытым ключам, настроенным через корпоративную политику, открытым ключам известных механизмов/магазинов распространения или отдельным разрешенным открытым ключам индивидуальных изолированных веб-приложений.

URL-адреса isolated-app: выглядят следующим образом:

    isolated-app://signed-web-bundle-id/path/inside/app.js?some-query#foo
    ^           ://^                   /^                 ?^         #^
    scheme         opaque host          path               query      fragment

__Content-Security-Policy__

Content-Security-Policy (CSP) не допускает загрузку ресурсов извне веб-пакета приложения. 

CSP строго ограничивает источники, из которых  приложение  может  загружать  контент,  чтобы  убедиться,  что  все  ресурсы  проходят  проверку и не являются  вредоносными: 

    Content-Security-Policy: base-uri 'none';
                            default-src 'self';
                            object-src 'none';
                            frame-src 'self' https: blob: data:;
                            connect-src 'self' https: wss: blob: data:;
                            script-src 'self' 'wasm-unsafe-eval';
                            img-src 'self' https: blob: data:;
                            media-src 'self' https: blob: data:;
                            font-src 'self' blob: data:;
                            style-src 'self' 'unsafe-inline';
                            require-trusted-types-for 'script';

__Web app manifest__

Манифест веб-приложения — это файл JSON, который сообщает браузеру, как ваше прогрессивное веб-приложение (PWA) должно вести себя при установке на компьютер или мобильное устройство пользователя.

Минимальный манифест для изолированного веб-приложения (IWA) может выглядеть примерно так: 

    {
    "name": "IWA Kitchen Sink",
    "version": "0.1.0",
    "update_manifest_url": "https://example.com/updates.json",
    "start_url": "/",
    "icons": [
        {
        "src": "/images/icon.png",
        "type": "image/png",
        "sizes": "512x512",
        "purpose": "any"
        },
        {
        "src": "/images/icon-mask.png",
        "type": "image/png",
        "sizes": "512x512",
        "purpose": "maskable"
        }
    ]
    }

"version" - требуется для изолированных веб-приложений. Строка, состоящая из одного или нескольких целых чисел, разделенных точкой (.).

"update_manifest_url" - необязательное, но рекомендуемое поле, указывающее на URL-адрес HTTPS (или localhost для тестирования), где можно извлечь манифест обновления веб-приложения.

Разберем два режима разработки изолированных веб-приложений, описанных в проекте [telnet-client](https://github.com/GoogleChromeLabs/telnet-client/tree/main). 

Chrome поддерживает два режима разработки изолированных веб-приложений.

__"Proxy" Mode__:Вы запускаете локальный сервер разработки, как это делается для обычных веб-приложений, используя URL типа http://localhost:8000.  При установке приложения создается случайное isolated-app:// происхождение, и браузер перенаправляет запросы к этому происхождению на ваш локальный сервер. Это позволяет вам быстро редактировать и обновлять приложение, чтобы увидеть изменения.

В нашем прототипе мы используем именно этот метод, когда запускаем Chrome для установки приложения в режиме "Proxy" Mode.

__Developer Mode__: Когда режим разработчика включен, Chrome также позволяет самостоятельно подписывать веб-пакет и загружать его так же, как это делалось бы для приложения в производственной среде.

# Прототип использования WebUSB API для доступа к смарт-картам

Создан [прототип](../prototype/), выполняющий запрос серийного номера смарт-карты из браузера. 

В нашем случае, для корректной работы прототипа, необходимо указать в __HttpServer.py__: 

    self.send_header("Content-Security-Policy", "script-src 'self'; object-src 'self'; script-src-elem 'self' 'unsafe-inline'")  

Таким же образом, нужно указать в __index.html__: 

      <meta http-equiv="Content-Security-Policy" content= "script-src 'self'; object-src 'self'; script-src-elem 'self' 'unsafe-inline'">

## MacOS ARM <a name="macosarm"></a>
При попытке соединения с токеном, возникает ошибка: 

    NetworkError: Failed to execute 'claimInterface' on 'USBDevice': Unable to claim interface.

## Windows x86_64 <a name="windows"></a>
На ОС Windows изолированные веб-приложения успешно функционируют, но теперь проблема возникает на стороне WebUSB API. 

При попытке соединения с токеном, возникает ошибка: "Failed to execute 'open' on 'USBDevice': Access denied". -- Windows ограничивает доступ к смарт-картам как к USB-устройству, что подтверждено в [summary](https://groups.google.com/a/chromium.org/g/blink-dev/c/LZXocaeCwDw/m/GLfAffGLAAAJ) описания ограничений доступа к классам устройств из WebUSB.
 
    "These interface classes are already mostly blocked by an operating system’s built-in class drivers."

Снять ограничение можно путем установки WinUSB-драйвера, как, например, указано в документации [webusbAuth](https://github.com/jbirkholz/webusbAuth):

    "For Windows Zadig is recommended to load the generic WinUSB driver for your CCID."

## Linux x86_64 <a name="linux"></a>
Для тестирования прототипа использовалась виртуальная машина __Oracle Virtualbox__ на которую предварительно установили образ __Ubuntu 24.04__. 

В точности выполняя [инструкцию](../prototype/README.md), последует успешная сборка и запуск прототипа. 