# Запрос баланса контрагента {#get-balance}

###### Последнее обновление: 2017-11-14 | [Редактировать на GitHub](https://github.com/QIWI-API/topup-wallet-doc/blob/master/_get-balance_ru.html.md)

Запрос `ping` возвращает текущий баланс по агентскому договору в сервисе QIWI Кошелек.

## Формат запроса

### Параметры запроса

~~~xml
<request>
<request-type>ping</request-type>
<terminal-id>44</terminal-id>
<extra name="password">password</extra>
</request>
~~~

Параметр|Описание
-|-
*request*| Группирующий тег
*request-type* | Тип запроса (идентификатор запроса баланса: `ping`)
*terminal-id* | Идентификатор агента в системе QIWI Wallet
*extra name="password"* | Экстра-поле, содержащее пароль для аутентификации в системе QIWI Wallet

## Формат ответа

### Ответ без ошибок обработки запроса

~~~xml
<response>
<result-code fatal="false">0</result-code>
<balances>
  <balance code="428">100.00</balance>
  <balance code="643">200.26</balance>
  <balance code="840">300.00</balance>
</balances>
</response>
~~~

Параметры ответа:

Параметр|Описание|Атрибуты
-|-|-
*response*| Группирующий тег ответа.|Отсутствуют.
*result-code* | [Код результата](#tech_error) выполнения запроса.| `fatal` - логический признак фатальности ошибки обработки платежа.
*balances*|Группирующий тег, содержит информацию о балансе всех активных счетов Контрагента в системе QIWI Wallet.|Отсутствуют.
*balance* | Текущий баланс единичного счета Контрагента в системе QIWI Wallet. | `code` - цифровой код валюты счета (в формате ISO 4217).

### Ответ с ошибками обработки запроса

Если сервер не смог обработать запрос, он возвращает xml-ответ с описанием произошедшей ошибки.

~~~xml
<response>
  <result-code fatal="false">300</result-code>
</response>
~~~

Параметры ответа:

 Тег|Описание|Атрибуты
--------|------|---------
*result-code* | [Код](#tech_error) ошибки обработки запроса.| `fatal` – логический признак фатальности ошибки обработки запроса.
