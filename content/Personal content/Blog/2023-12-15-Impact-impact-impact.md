## Как минимум идеально
Разработка - это сфера, где слово "перфекционизм" может поделить людей на два лагеря. Но моя история начал ещё до того, как я узнал о коде. Я был из тех людей, которые могут потратить часы или даже дни на то, чтобы разобрать все закладки в браузере и создать иерархию из директорий для них. Или сесть и переименовать все 400 треков добавленных VK (это вам не Spotify, как назвали при загрузке, так трек и добавился), а потом им сортировать в алфавитном порядке (кнопки для сортировки не было, а хотелось). Позже это вылилось в неприязнь к коду, где разные отступы используются или есть неоднородность в подходе.

Свою роль в отношении сыграло воспитание. В детстве я делал много всего хорошо и это завышало ко мне ожидания. В какой-то момент мне начали ставить требования: "сделать как минимум идеально". Это закрепилось и даже превратилось в ожидание от остальных.

Поиск первой работы может превратиться в неожиданную ловушку. В желании впитать все возможные best practice, узнать о самых "правильных" подходах к решению задачи, можно переоценить работодателя. Стремление к идеальном, желание максимально хорошо подготовить в сумме с академическим подходом в моей истории превратились в то, что я в своём познании ушёл сильно дальше того места, где в итоге оказался. И вот с этого момента начинается страдание:

-Почему у нас в конфигурационных файлах прописаны вот эти значения? Они же не актуальны?
-Да, их когда-то добавили и они так есть.
-Почему у нас основная ветка не собирается?
-Нам пришлось срочно внести много изменений, а времени стабилизировать не было. Но ты просто не собирай ту часть, которая не собирается.
## Пользоориентированность
Мы живём в мире, где у всех нет времени и иногда даже приходится оправдывать своё желание сделать что-то "идеально". И в мире разработки часто приходится принимать компромиссы - "мы закроем глаза на это, у нас нет времени это исправлять". Но я до последнего продолжаю отстаивать свои позиции и идти самыми сложными маршрутами. И у этого есть простая мотивация - это попытка максимизировать пользу от каждой задачи.

Смоделируем ситуацию. Идёт процесс разработки. В какой-то момент представитель отдела QA сообщает о найденной проблеме в продукте. В этот момент появляется множество вариантов. Самый простой - ознакомиться с проблемой, исправить, сообщить, что всё ок. Такой способ решения потребовал мало усилий, на выходе стало на одну проблему меньше. Но если посмотреть на дистанции, то ситуация не сильно изменилась. Проблемы появляются и исчезают. То, что сегодня исправили, завтра могут сломать.

Чтобы получить больше пользы от решения конкретной проблемы, на неё можно посмотреть комплексно, увеличить масштаб анализа. А можем ли мы не просто исправить проблему, а гарантировать что она не вернётся? Да, мы можем написать [тесты](../../Knowledge%20base/Testing/%D0%90%D0%B2%D1%82%D0%BE%D0%BC%D0%B0%D1%82%D0%B8%D1%87%D0%B5%D1%81%D0%BA%D0%BE%D0%B5%20%D1%82%D0%B5%D1%81%D1%82%D0%B8%D1%80%D0%BE%D0%B2%D0%B0%D0%BD%D0%B8%D0%B5%20%D0%BA%D0%BE%D0%B4%D0%B0.md). И такой способ решения не просто уберёт дефект, но и улучшить [качество продукта](../../Knowledge%20base/Project%20management/Product%20quality.md). Это уже единицы пользы, которые не исчезнут через день.

Если это проблема в коде, то могли ли мы отловить её заранее? Да, у нас есть инструменты [анализа код](../../Knowledge%20base/Project%20management/Static%20code%20analyze.md), которые мы можем настроить и они будут нам гарантировать, что проблема не излечится.

Если проблема была концептуальной и связана с тем, что другой разработчик не правильно понимает код, процессы, то можем ли мы исправить это? Да, исправлять людей куда проще чем их код. Можно провести QnA, улучшить документацию, провести митап/лекцию. Все эти активности будут не просто проведены, он дадут некоторые артефакты, которые тоже можно считать полезным.

Если постоянно появляются проблемы и мы каждый раз тратим на это время, но не можем ничего сделать с этим, то можно посмотреть в сторону улучшения процесса исправления проблем. Постоянно читаем много [логов](../../Knowledge%20base/Developing/Code/Logging/Logging.md)? Стоит написать инструменты для анализа логов, которые будут автоматически находить проблемы. Каждая новая проблема будет подкидывать некоторое испытание, которое будет требовать улучшить инструменты. А значит каждая задача не просто отнимает время, она пушит нас делать инструмент лучше, создаёт пользу.
## Итог
Описанный подход с получением пользы от каждой задачи - это основное, что помогает оставаться на плаву и идти сквозь проблемы далее. Например, я всё ещё являюсь недотимплидом, есть два человека, за которых я отвечают. Максимализм меня пушит к тому, чтобы непрерывно следить за их успехами, смотреть какие задачи и как они выполняют. Но это звучит как рутинная задача, которая не приносит долгосрочной пользы. Именно поэтому, я переосознал её для себя. Я больше не занимаюсь мониторингом людей, я занимаюсь разработкой инструментов для мониторинга. Задача "Узнать что человека Х сделал за неделю" превращается в "разработать регламент выполнения задач и автоматизировать сбор данных и метрик". Постановка такой обобщённой задачи требует разработки приложений. Разработка в свою очередь также не является "просто задачей". Вместо того, чтобы "набросать скрипт на питоне", я выбираю путь "разработать библиотеку для создания консольного UI, разработать библиотеку для сбора метрик по пользователю из *название системы трекинга задач*". Это поддерживает жизнь проектов в [Kysect (github.com)](https://github.com/kysect).

Но история не только про написание кода. Следующая стадия пользы от решения задач - обобщить их, решить в общем виде, описать и донести до остальных и превратить это в "знания". Это и есть причина существования этих постов. За время преподавания, менторинга и прочего я не просто помогал, но и накапливал знания. Сейчас такой подход даёт возможность во многих ситуациях просто вернуться назад, взять оттуда решение проблемы и отдать человеку. И тут я тоже хочу достичь идеала - описывать все знаний и получаемую информацию, превратить во что-то, чем легко поделиться с другими. И написание подобных постов - это путь достижения такой цели. И именно осознание этого помогло ответить на вопрос "А хочу ли я написать об этом? А нужно ли это мне?". Теперь я чаще отвечаю да. И это, я надеюсь, решит проблемы с тем, что я не пишу о том, чем занимаюсь и чем хочу поделиться.