# Prototype
# Оглавление
1. [Вступление](#introduction)
2. [Установка необходимых программ](#install)
3. [Запуск](#building)
   
# Вступление <a name="introduction"></a>
Прототип успешно функционирует на машине с процессором архитектуры x86_64 и с ОС Linux (допукается виртуальная машина). 

# Установка необходимых программ <a name="install"></a>

1. Для платформ Windows/Linux установить [Google chrome unstable](https://flathub.org/apps/com.google.ChromeDev).

В случае использования MacOS необходимо загрузить [Chrome Canary](https://www.google.com/intl/ru/chrome/canary/)

2. Установить [python](https://www.python.org/downloads/) для запуска локального веб-сервера 

# Запуск прототипа <a name="building"></a>

1. Перейти в директорию __webusbAuth__ и запустить python сервер.
```
python3 HttpServer.py
```
2. Открыть новое окно терминала и через него запустить Chrome unstable со следующими параметрами:

Для Linux:

```
google-chrome-unstable --enable-features=IsolatedWebApps,IsolatedWebAppDevMode --install-isolated-web-app-from-url=http://localhost:8000
```

Для MacOS:
```
"/Applications/Google Chrome Canary.app/Contents/MacOS/Google Chrome Canary" --enable-features=IsolatedWebApps,IsolatedWebAppDevMode \
                       --install-isolated-web-app-from-url=http://localhost:8000
```

3. В адресной строке браузера указать __chrome://apps__ и перейти по указанному адресу 

4. В списке приложений отобразится __WebUSBAuth__ -- запустить приложение, нажав на его иконку.

5. Подключить токен или ридер со смарт-картой к ПК

6. Нажать "connect reader"

7. В появившемся окне выбрать ранее подключенный токен.

8. Нажать на кнопку `get serial number` -- в консоли отобразятся переданные и полученные от токена данные.