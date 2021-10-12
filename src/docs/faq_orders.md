#Алгоритм и особенности исполнения торговых поручений

##Выставление торговых поручений

###Лимитное торговое поручение

Чтобы не тратить время на постоянное отслеживание котировок на бирже, вы можете выставить так называемую 
лимитную заявку — это указание брокеру, какую бумагу или валюту, сколько лотов и по какой цене вы хотите 
купить или продать.

Биржа ограничивает цену, которую можно указать в лимитной заявке, крайними предложениями в биржевом 
стакане (параметры *limit_up* и *limit_down*).

**Обратите внимание, что в некоторых случаях может возникнуть ситуация, когда limitDown > limitUp. Это 
нормально, т.к. limitUp ограничивает на покупку, а limitDown - на продажу, потому ситуация 
limitDown > limitUp не является аномальной.**

Выставить лимитное торговое поручение (заявку) можно при помощи unary-метода [postOrder](/investAPI/orders#postorder)
или при использовании bidirectional-stream [OrdersStream](/investAPI/orders#ordersstream).

###Рыночное торговое поручение
Это заявка на покупку или продажу активов по той цене, что есть на бирже в данный момент. У нее есть важная 
особенность.
При исполнении рыночной заявки может оказаться, что в данный момент на бирже по текущей цене 
торгуется меньше лотов, чем вы указали, при этом другие лоты есть, но их цена отличается в негативную для 
вас сторону.
В этом случае брокер купит имеющееся количество лотов по текущей цене, а оставшуюся часть заявки будет 
покупать по следующей по списку цене. Если на бирже низкая ликвидность — например, торги рано утром или до 
открытия американской биржи, — то оставшаяся часть заявки может быть исполнена по невыгодной для вас цене. 
Проверить текущую ликвидность можно в [биржевом стакане](/investAPI/marketdata#getorderbook).

Выставить рыночное торговое поручение (заявку) можно при помощи метода unary-метода [postOrder](/investAPI/orders#postorder)
или при использовании bidirectional-stream [OrdersStream](/investAPI/orders#ordersstream)

##Особенности исполнения заявок на Санкт-Петербургской бирже
Все операции с иностранными ценными бумагами в [Тинькофф Инвестициях](https://www.tinkoff.ru/invest/) 
производятся на Санкт-Петербургской бирже, которая имеет ряд особенностей исполнения заявок.

У данной биржи есть два пула ликвидности — США и собственный. Пул США подключается в 14:30 (мск) по летнему 
американскому времени. Ранее выставленные ордера остаются на СПБ. В момент выставления торгового поручения 
механизм [best execution](https://nprts.ru/ru/projects/bestexecution/) проверяет, в каком пуле лучше условия 
и выставляет заявку туда. После выставления заявка уже не переносится между пулами, поэтому может быть ситуация, 
что пул, в котором выставлена заявка, не достиг лимитной цены, а в другом пуле нужная цена достигалась. 

Вся же биржевая информация (свечи, стаканы) транслируется, агрегируя информацию со всех пулов ликвидности.

Исходя из этого может случиться ситуация, когда выставленное торговое поручение не исполняется, 
хотя имеются подходящие в стакане цены/предложения. В данных ситуациях команда OpenAPI рекомендует перевыставлять 
(отменить и выставить заново) заявки, чтобы механизм выбора пула ликвидности разместил заявку в более подходящем пуле.

Неисполненные заявки могут сниматься в разное время по описанным выше причинам. Время жизни заявки в каждом пуле ликвидности
соответствует [времени работы](https://spbexchange.ru/ru/stocks/inostrannye/raspisanie/) соответствующей сессии пула.

#Определение доступности выставления торгового поручения 

Для успешного размещения заявки на бирже должно выполняться ряд условий: 

1. **Счёт доступен пользователю для совершения операции**. 
2. **У пользователя достаточно активов для совершения операции или пользователю доступно необходимое количество 
активов для совершения [маржинальной сделки](https://help.tinkoff.ru/margin-trade/)**. Для получения текущего 
состояния портфеля пользователя можно использовать метод [getPositions](/investAPI/operations#getpositions). 
Для получения маржинальных показателей счёта — метод [getMarginAttributes](/investAPI/users#getmarginattributes).
3. **Инструмент доступен для торгов**. Большая часть инструментов доступна для торговли в течении всего времени
работы торговой площадки (для получения расписания торгов используйте метод [getTradingSchedules](/investAPI/instruments#tradingschedules)).
Однако, возможны случаи перерывов и остановок торгов по той или иной бумаге. Для получения актуального статуса
торговле по инструменту можно использовать методы [сервиса инструментов](/investAPI/head-instruments) или 
[стрим информации по инструменту](/investAPI/head-marketdata#stream).
