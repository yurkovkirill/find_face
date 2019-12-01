# Team 912


## installation

dlib у меня получилось только с conda установить.
```
conda install -c conda-forge dlib
```
из-за этого и venv надо использовать из конда и устанавливать в него все зависимости
```
pip install -r web-api/requirements.txt
pip install -r queues/requirements.txt
```

Запуск фронта
Важно! В anaconda может быть протухшая nodejs. Используете системную
```
cd web-front
npm install
npm start
```

Запуск разгребальщиков и апишки
```
PYTHONPATH=. python -m aiotasks -d -vvv worker -A queues.worker.photoloader
PYTHONPATH=. python web-api/api.py
```

Класс VKUser
* MAX_FRIENDS_COUNT = 5 -- максимальное кол-во друзей, которые обкачиваются среди друзей. Слишком большое нельзя поставить, потому что можно напороться на рейтлимиты вк. Чем значение больше, тем дольше будет обрабатываться запрос


# переменные окружения
```
export VKAPPS_TOKEN=        # сервисный токен прлиожения для запросов в апишку вк
export VKAPPS_SECRET_KEY=   # секретный ключ приложения для проверки авторизации пользователя
export VK_COMMUNITY_TOKEN=  # токен сообщества, от которого будут отправляться сообщения
```

# todo
* ассинхрон на походы в вк
Ассинхронная только загрузка фоток. Походы в апишку вк не асиинзронные, но зато они обрабатываюстя батчами (это позволяет интерфейс апиши)

---

Проект по поиску друзей по фотке по курсу [Applied Python](https://github.com/Kinetikm/AppliedPythonF2019/).

---

На первом этапе это будет приложение вк.
Плюсы такого подхода -- то, что не придется пилить собственную авторизацию.

После того, как отработаем схему на вк, можно будет запилить интеграцию и с другими соц сетями. Поэтому нужно писать код так, чтобы его было легко поддержать еще один источник фоток/пользователей/авторизации.

Задачи:

* Сделать веб-морду для проекта.
    Нужно уметь:
    * На каждый запрос в апишку прокидывать авторизационные данные пользвоателя.
    * Запрашивать доступ к сообщениям пользователя
    * Загружать фотку, на которой есть собутыльник в апишку.
    * После загрузки изображения, нужно сказать полльзователю, что его запрос в обработке

    Опционально (если усеет это поддержать бэк):
    * показывать историю поиска собутыльников

    Полезные ссылки:
    * https://vk.com/vkappsdev (см раздел информация -- там полно других полезных ссылок)
* Апишка для веб-морды.
    Нужно уметь:
    * проверять авторизацию пользователя
    * принимать загруженную фотку (как получать фотку, нужно будет подумать -- возможно, мы сможем переииспользовать апишку вк так, чтобы не думать о безопасности и предобработке фоток. Но тогда для интеграции с другими соцсетями нам нужно будет иметь такой же механизм, а это может быть неудобно, поэтому, наверное, лучше иметь собственнный механизм для этого (мб уже есть готовые решения?))
    * подумать о безопасности загруженной фотки (действительно ли это фотка? там точно ничего кроме фотки?)
    * ставить задачу на обработку пользовательского запроса в очередь

    Опционально: (если уже все у всех работает)
    * проверять рейтлимиты пользвоателей. Чтобы не было такого, что из-за того, что одному пользователю нужно найти кучу собутыльников, другие не могут рбаотать. Проверять это нужно еще до того, как пользователь загрузит фотку. И во время того, как он загружает фотку -- короче, всегда))

* Очередь обработки запросов пользователей
    * Получать список друзей пользвоателя и друзей друзей пользователя и выбирать из них только уникальных.
    * После получения списка друзей, мы сохраняем их в базу.
    * Загружаем (подумать об оптимальном разрешении загруженных фоток вместе с ребятами, которые отвечают за распознование лиц) (нужно ассинхронно + многопоточно) несколько фоток из профилей друзей
    * Нужно продумать структуру хранения фоток в системе.
    * После загрузки всех фоток пользователя, нужно будет поставить задачу в другую очередь, которая будет искать лица и пересечения между ними.

* Очередь для распознования
    * должна переиспользовать модуль, который будет наипсан в след задаче.
    После того, как обработка завершится, нужно будет сообщить пользователю через сообщения, о результате.

* Научиться распозновать лица.
    Этот микросервис будет самым ML'шным. Тут можно посравнивать готовые решения. Сделать резерч о том, как вообще это все работает. Резуальтатом ресерча должен быть ноутбук, в котором можно будет показать другим, как оно работает и на каком решении мы остановились и почему.

    После ресерча, нужно будет запилить питонячий модуль, который будет уметь получать кучу фоток на вход и искать пересечение похожих лиц на группе фоток и на целефой фотке.

    В ресерче нужно понять минимально возможное разреешние фоток -- хочется как можно меньше места занимать на диске, но не потерять в точности. Мб будет эффективнее перекодировать фотки в другой формат?


Каждая задача у нас -- это, по сути, отдельный микросервис.
Так как на этапе, когда будем все интегировать вместе, каждой команде нужно будет отлаживать работу с другими сервисами, чтобы было проще разобратся, как запустить ваш сервис, нужно сделать документацию к нему. В идеале это должен быть Dockerfile + docker-compose. Возможно, это будет просто README.md + requirements.txt. То, на каких портах слушает сервис и интерфейс работы с ним (какие параметры принимает и т д) должно быть задокументировано.

По всем вопросам (в том числе "что-то не получается") можно и нужно к d.tarasov. Если я не смогу подсказать, будем эскалировать в преподавателей))