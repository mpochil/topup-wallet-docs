# Пополнение баланса {#payment}

###### Последнее обновление: 2017-11-14 | [Редактировать на GitHub](https://github.com/QIWI-API/topup-wallet-doc/blob/master/_topup_ru.html.md)

Операция используется для перевода средств с агентского счета на счет Клиента в системе QIWI Wallet.

Если Клиент с указанным идентификатором не существует в системе, то он будет создан в момент регистрации платежа.

Для проверки успешного прохождения платежа, вы должны периодически (но не чаще одного раза в 10 минут) выполнять [запрос проверки статуса платежа](#status) до получения успешного или неуспешного финального статуса платежа.

При возникновении сетевых ошибок (например, таймауты при соединении или чтении ответа), HTTP-ошибок (HTTP-статус не равен 200, пустой ответ), некорректных XML-документов (например, c отсутствующими обязательными тегами и/или атрибутами) вы должны перейти к опросу статуса платежа до получения успешного или неуспешного финального статуса платежа. Поскольку при возникновении данных ошибок информация о статусе транзакции не доступна, вы не должны отклонять платеж на своей стороне.

Фатальность ошибки означает, что повторный запрос с теми же реквизитами приведет к повторению той же ошибки. Фатальные ошибки, как правило, вызваны ошибками конфигурирования и требуют ручного вмешательства (обращения в службу поддержки сервиса QIWI Кошелек).

При появлении фатальных ошибок обработки запроса вы можете не приостанавливать выполнение повторных запросов, либо приостановить до устранения ошибок конфигурирования. Поскольку при ошибках обработки запроса статус транзакции неизвестен, вы не должны отклонять платеж.

## Формат запроса {#payment-req}

~~~xml
<?xml version="1.0" encoding="utf-8"?>
<request>
	<request-type>pay</request-type>
	<terminal-id>123</terminal-id>
	<extra name="password">***</extra>
	<auth>
		<payment>
			<transaction-number>12345678</transaction-number>
			<from>
				<ccy>RUB</ccy>
			</from>
			<to>
				<amount>15.00</amount>
				<ccy>RUB</ccy>
				<service-id>99</service-id>
				<account-number>79181234567</account-number>
			</to>
		</payment>
	</auth>
</request>
~~~

### Параметры запроса

Тег|Описание
--------|------
*request*|Группирующий тег. Дочерние теги содержат параметры платежа.
*request-type* | Тип запроса (идентификатор запроса пополнения QIWI Кошелька: `pay`)
*terminal-id* | Идентификатор агента в системе QIWI Wallet
*extra name="password"* | Экстра-поле, содержащее пароль для аутентификации в системе QIWI Wallet
*auth*|Группирующий тег, описывает платежные данные
*payment*|Группирующий тег, описывает единичный платеж. **В запросе может присутствовать только один тег `payment`.**
*transaction-number* | Номер транзакции в информационной системе агента. Его необходимо использовать при [проверке статуса платежа](#status). Номер транзакции и идентификатор `terminal-id` однозначно определяют транзакцию в системе QIWI Wallet.
*from*|Группирующий тег, содержит информацию о сумме, полученной агентом от клиента
*ccy* | Валюта счета агента, с которого будет произведено списание денежных средств (в качестве значения используется цифровой или буквенный код валюты по ISO 4217)
*service-id* | Идентификатор источника (канала) пополнения (необязательный параметр). Параметр используется в случае необходимости разделения потоков пополнения QIWI Кошельков, сохраняя единый баланс агента.
*to*|Группирующий тег, содержит информацию о платеже
*amount* | Сумма к зачислению на баланс учетной записи Клиента в системе QIWI Wallet. Система QIWI Wallet накладывает ограничения на сумму платежа, исходя из условий на остаток денежных средств учетной записи Клиента QIWI Wallet, указанных в оферте пользователя по адресу <https://static.qiwi.com/ru/doc/oferta_lk.pdf>. Платежи с суммой, менее разрешенной и более разрешенной, завершатся с [ошибками](#error) `241` и `242` соответственно.
*service-id* | Идентификатор сервиса (константа: 99)
*account-number* | Идентификатор Клиента в системе QIWI Wallet (номер телефона Клиента системы QIWI Wallet в международном формате)
*extra="comment"* | Экстра-поле, содержащее комментарий (необязательный параметр)
*ccy* | Валюта счета получателя, на который будут зачислены денежные средства (в качестве значения используется цифровой или буквенный код валюты по ISO 4217).

## Формат ответа {#payment-res}

### Ответ без ошибок обработки

~~~xml
<response>
<payment status='60' txn_id='6060' transaction-number='12345678' result-code='0' final-status='true' fatal-error='false' txn-date='02.03.2011 14:35:46'  >
  <from>
    <amount>15.00</amount>
    <ccy>643</ccy>
  </from>
  <to>
    <service-id>99</service-id>
    <amount>15.00</amount>
    <ccy>643</ccy>
    <account-number>79181234567</account-number>
  </to>
</payment>
<balances>
<balance code="428">0.00</balance>
<balance code="643">200</balance>
<balance code="840">12.20</balance>
</balances>
</response>
~~~

Параметры ответа:

 Тег|Описание|Атрибуты
--------|------|---------
*response*	| Группирующий тег ответа.|Отсутствуют.
*payment* | Описание принятого платежа.| `status` – [статус платежа](#statuses) в системе QIWI Wallet;
 | |`txn_id` – идентификатор транзакции в системе QIWI Wallet. **Если не заполнен, платеж не был зарегистрирован из-за временной ошибки. Попробуйте повторить запрос позже**;
 | |`transaction-number` – номер транзакции в информационной системе Контрагента;
 | |`result-code` – [код](#error) ошибки обработки платежа;
 | |`final-status` – флаг, определяющий финальность статуса платежа;
 | |`fatal-error` - флаг, определяющий фатальность ошибки обработки платежа;
 | |`txn-date` – дата приема платежа в систему QIWI Wallet;
*from*|Группирующий тег, содержит информацию о списанных средствах.|Отсутствуют.
*amount* | Сумма, списанная со счета Контрагента.|Отсутствуют.
*ccy* | Валюта счета (в качестве значения используется цифровой или буквенный код валюты по ISO 4217).|Отсутствуют.
*to*|Группирующий тег, содержит информацию о платеже.|Отсутствуют.
*amount* | Сумма к зачислению на баланс учетной записи Клиента в системе QIWI Wallet.|Отсутствуют.
*service-id* | Идентификатор сервиса, на который производится зачисление средств (константа: 99).|Отсутствуют.
*account-number* | Идентификатор Клиента в системе QIWI Wallet.|Отсутствуют.
*ccy* | Валюта платежа (цифровой или буквенный код валюты по ISO 4217).|Отсутствуют.
*balances*|Группирующий тег, содержит информацию о балансе всех активных счетов Контрагента в системе QIWI Wallet.|Отсутствуют.
*balance* | Текущий баланс единичного счета Контрагента в системе QIWI Wallet| `code` - цифровой код валюты счета (в формате ISO 4217).

### Ответ с ошибками обработки

Если сервер не смог обработать запрос на пополнение баланса учетной записи Клиента в системе QIWI Wallet, он возвращает XML-ответ с описанием произошедшей ошибки.

~~~xml
<response>
  <result-code fatal="false">300</result-code>
</response>
~~~

Параметры ответа:

 Тег|Описание|Атрибуты
--------|------|---------
*result-code* | [Код](#tech_error) ошибки обработки запроса.| `fatal` – логический признак фатальности ошибки обработки запроса.
