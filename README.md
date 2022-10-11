# ebotFuture
E-Bot Future.
Бот для торговли на Binance Future с использованием стратегии мартингейла (усреднения) и выбором растущей (падающей) монеты. Может работать и в LONG и в SHORT.

E-Bot выбирает монету, выросшую за указанный в настройках период времени на указанный процент (при работе в  LONG, а при работе в SHORT упавшую), 
- покупает ее маркет-ордером на указанный объем (при работе в  LONG, а при работе в SHORT продает), 
- выставляет купленные монеты на продажу лимитным sell-ордером по курсу на указанный процент прибыли выше курса покупки (в SHORT-е на покупку) ,
- выставляет лимитный buy-ордер на покупку этой же монеты по курсу ниже предыдущей покупки на указанный процент (на случай падения курса монеты и уменьшения средней цены входа в сделку)(в SHORT-е лимитный SELL-ордер на случай повышения курса).

Затем, в зависимости от того, какой ордер исполнился (примеры для LONG, для SHORT- наоборот):
- если исполнился sell-ордер, бот фиксирует прибыль и отменяет buy-ордер для усреднения(если buy-ордер при этом успел исполниться частично или полностью- выставляется sell-ордер), затем снова ищет подходящую пару,
- если исполнился buy-ордер, бот отменяет sell-ордер, и выставляет новый sell-ордер уже с новым количеством монет и по новой цене(средняя цена входа + указанный процент прибыли),
- если buy-ордер исполнился больше, чем наполовину и прошло более 5-ти минут после этого , бот отменяет sell-ордер, и выставляет новый sell-ордер уже с новым количеством монет и по новой цене(средняя цена входа + указанный процент прибыли).

Бот устанавливается на VPS сервер ubuntu 20 и запускается в SCREEN (чтобы бот не отключался при разрыве SSH-соединения с VPS), настраивается telegram-бот и канал, куда приходит информация о работе бота. 
- Запуск бота командой:         ./ebotFuture-6 
- Остановка бота командой:      ctrl+c    (важно!: не останавливайте бот в момент совершения сделок, возможна ошибка записи в базу данных бота).
- В white_list (список пар для работы) можно внести от 1 до нескольких сотен пар, главное, чтобы квотируемая валюта была одна: если торгуете к USDT, то пары ETHUSDT, BTCUSDT, XRPUSDT и т.д., если торгуете к BUSD, то пары ETHBUSD, XRPBUSD, DASHBUSD и т.д.
- При изменении квотируемой валюты не забывайте проверять и min_order в настройках бота (в USDT min_order должен быть больше 11)
- При работе с парами к USDT активы должны находится на фьючерсном балансе USDT, если работаете с парами к BUSD, активы должны находится на фьючерсном балансе  BUSD.
- Для прокрутки экрана терминала вверх есть команда: ctrl+a, esc и далее стрелка вверх. Для выхода из этого режима: esc, esc.

Для работы E-Bota можно использовать BNB для оплаты комиссий биржи (нужно перевести нужное количество BNB на фьючерсный счет в лк binance) и следить за наличием BNB на future аккаунте.

После закрытия каждой сделки E-Bot:
- отправляет сообщение в telegram-канал, 
- каждую минуту в описание канала отправляет информацию об открытой позиции, 
- в полночь в канал отправляет суточный отчёт о работе.

Настройки бота (в основном описано для LONG, для SHORT применяется наоборот):
- fix_perc: процент повышения цены для продажи при LONG или падения для SHORT,
- aver_perc: процент падения цены для докупки при LONG или роста для SHORT,
- step_aver: шаг увеличения aver_perc,
- qty_aver: кратное увеличение объема выставляемого ордера,
- k_step: коэффициент умножения предыдущего aver_perc,
- формула расчета: 
1-й усреднение при падении (росте для SHORT) цены от последней покупки (продажи) на
aver_perc*k_step + step_aver,
2-е усреднение (и последующие) при падении (росте для SHORT) цены от последней покупки (продажи) на 
(1-й усред)*k_step+step_aver,
- min_order: минимальная покупка (продажа) в квотируемой валюте (например, в паре ETH/USDT это USDT, ставить больше 11 и учитывайте, что
в зависимости от выбранного leverage будет использоваться меньше USDT, например: если min_order указан 20, а leverage указан 10, то для ордера будет использовано 20/10=2 USDT),
- delta_start: на сколько % должен подняться (упасть) курс от цены открытия выбранной свечи до текущей цены для старта,
- stop_loss- на сколько % должен упасть курс монеты для закрытия в минус,
- used_stop_loss- включить использование stop_loss для закрытия в минус (да/нет),
- completed- поставить бота на паузу при закрытии очередной сделки (1-вкл/0-выкл),
- kline_interval: интервал свечей для анализа (1m, 3m, 5m, 15m, 30m, 1h, 2h, 4h, 8h, 12h, 1d, 3d),
- interval_limit: какое количество свечей анализируем,
- super_asset: пара для бесконечной торговли независимо от delta_start, при этом пары из white_list не будут работать (вводится командой -super_asset_add в формате ETHUSDT),
-fix_loss: команда для закрытия открытой позиции по рынку, 
- clear: сброс из базы данных сведений об открытых ботом ордерах,
- leverage: размер кратного увеличения USDT от 1 до 20 (иногда до 120)
- marginType: ISOLATED или CROSSED
- direction: LONG или SHORT
- clear_profit: сброс из базы данных сведений о прибыли,
- white_list: список пар, которые бот будет использовать для анализа и выбора подходящей для открытия сделки (вводится по одной паре командой -w_list_add),
- api_key: открытый api-ключ от биржи с разрешением на фьючерсную торговлю,
- api_secret: секретный api-ключ от биржи,
- botID- api телеграм бота полученный от @BotFather (пример: 5656544920:AAHrXhjhujhfdf7RPJlheqJXEulBW),
- channelID- ID канала telegram бота для уведомлений, полученное от @userinfobot (пример: -1001656543985),
- tguserid- ID основного user-a телеграм, полученное от @userinfobot (пример: 346549043)

При изменении marginType, direction, leverage убедитесь, что нет открытых позиций

Здесь E-Bot-Future представлен для ознакомления и использования в течении пробного периода до Mon Oct 24 2022 20:59:59 GMT+0000
Если Вы хотите увеличить время работы до 1/6/12 месяцев: напишите в телеграм, по данным, указанным при запуске E-Bot.

Если хотите испытать E-Bot на фьючерсной тестовой бирже Binance- 
переходите на:
https://testnet.binancefuture.com/ru/futures/
Где получите тестовые api-ключи, прописываете их в настройках бота, в used_testnet записываете: да, и экспериментируйте.

E-Bot поставляется по принципу «как есть». Никаких гарантий не прилагается и  не  предусматривается. Вы берете на себя весь риск относительно использования этого бота и должны понимать, что торговля на криптобиржах сопряжена с повышенным риском, и подходить к управлению рисками со всей ответственностью. 

Пояснения по установке, запуску, настройке бота и телеграм, screen, ошибке на  VPS utf-8.

Иногда бот может получить от биржи неправильные ответы на api-запросы и выдавать ошибку, поэтому рекомендуется периодически заглядывать в лк binance, и, если бот показывает открытые ордера а в лк binance их нет (или наоборот), нужно использовать команду -clear, чтобы сбросить в боте данные о неактуальных ордерах.
Редко, но бывает, что сервера telegram кратковременно недоступны, и в этот момент сообщение от E-Bot может не доходить в канал бота. 
Для управления ботом на VPS сервере с телефона можно использовать приложение JuiceSSH (или другое для SSH-соединения).

Установка и запуск E-Bot:
- на VPS-сервере ubuntu 20 создайте новую папку, например, ebotFuture (mkdir ebotFuture)
- зайдите в эту папку (cd ebotFuture)
- перенесите в эту папку файл бота ebotFuture-6
- откройте screen-сессию (например: screen -S ebotFuture)
- дайте права запуска файла (команда: chmod 755 ebotFuture-6)
- запустите E-Bot (команда: ./ebotFuture-6)
- команда для остановки бота: ctrl+c
- после запуска бота введите свои параметры: api_key и т.д.
- откорректируйте, при необходимости, настройки
- жмите ENTER и наблюдайте
- для выхода из SCREEN перед закрытием SSH-сессии используйте команду ctrl+a, d 
- для входа в screen работающего бота используйте команду: screen -x ebotFuture


Скриншоты

![Screenshot](https://github.com/ebot732/ebotFuture/blob/main/Screenshot_20221010-203431_Telegram.jpg)

====================================================================================================================================

![Screenshot](https://github.com/ebot732/ebotFuture/blob/main/Screenshot_20221010-203426_Telegram.jpg)

====================================================================================================================================

![Screenshot](https://github.com/ebot732/ebotFuture/blob/main/Screenshot_20221010-203406_Telegram.jpg)

====================================================================================================================================

![Screenshot](https://github.com/ebot732/ebotFuture/blob/main/Screenshot_20221010-203252_Telegram.jpg)

====================================================================================================================================

![Screenshot](https://github.com/ebot732/ebotFuture/blob/main/Screenshot_20221010-203218_JuiceSSH.jpg)


 
