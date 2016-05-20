# Клиент для работы c QIWI
Класс для работы с интерфейсом QIWI на PHP при помощи Api. Благодаря данному классу, вы можете работать с личным кабинетом QIWI через API, без использования браузера. Написан с использованием стандартов PSR.

##Требования

* PHP 5.6 
* Composer
* Guzzle 6 (если нету, то будет автоматический установлен)

##Установка

Установка с использованием менеджера пакетов [Composer](http://getcomposer.org):

Вносим изменения в composer.json:

```json
{ 
    "require": {
        "firepro/qiwiclient": "*"
    }
}
```

Установка через require:

```bash
$ composer require firepro/qiwiclient
```

##Возможности

* Авторизация по логину и паролю
* Запрос истории платежей
* Перевод средств на любой номер QIWI
* Подтверждение платежа через СМС код
* Проверка баланса
* Проверка подключенного СМС-подтверждения
* Работа со счетами
* Покупка/активация Qiwi Eggs

##Пример работы

###Инициализация

Для начала работы необходимо создать экземпляр класса QiwiClient, в который передать по умолчанию $login и $password.

```php
$client = new QiwiClient($login, $password);
```

###Авторизация

Теперь для осуществления дальнейших действий необходимо вызвать функцию authorize;

```php
$client->authorize();
```

Скрипт выполнит авторизацию в системе и запишет в память все требуемые куки для осуществления дальнейших операций

###Совершение платежей

Для создания платежа, необходимо создать экземпляр класса с назначением платежа.

* QiwiWalletPay - оплата на кошелек QIWI
* CardPay - оплата на банковскую карту
* YandexMoneyPay - Оплата на кошелек Яндекс.Деньги
* PhonePay - Оплата на счет мобильного телефона
* WebmoneyPay - Оплата на Webmoney

Далее, передать данный обьект в метод pay:

```php
$client->pay(QiwiPayRequest $request)
```

После передачи параметра, Вам вернется экземпляр класса QiwiPayResponse, который будет в себе содержать 

* Уникальный id созданного платежа
* Статус платежа
* Ошибки, если таковые произошли при выполнении запроса

Если статус платежа NEED_SMS, то необходимо выполнить следующий метод, необходимый для подтверждения платежа.

###Подтверждение перевода через SMS

Для того, чтобы завершить платеж, вызовите метод confirm(int $id, int $code);

```php 
$client->confirm($id, $code);
```

###Получение истории платежей

История платежей возвращается итератором экземпляром класса QiwiHistory, перебор которого возвращает обьект QiwiInvoice. Доступные свойства которого:

* $id
* $status
* $text
* $date
* $tiem
* $provider
* $datetime
* $comment
* $cash

###Получение платежа по комментарию

Для того, чтобы найти платеж по комментарию, вызовите метод findInvoice и передайте в него комментарий, по которому нужно найти платеж. Производится получение первого доступного платежа с данным комментарием.

```php 
$client->findInvoice($comment);
```

Возвращаемый обьект Invoice, который содержит в себе всю требуемую информацию о платеже.

###Получение баланса 

Для получения баланса, необходимо вызвать метод getBalance()

```
$client->getBalance();
```

Вам вернется обьект QiwiBalance, который содержит в себе свойства:

* rub
* eur

###Покупка ваучеров

С помощью данного клиента можно удобно работать с QIWI EGGS, для того, чтобы купить ваучер, 

$client->buyEgg();

###Активация ваучеров

$client->activeEgg();

###Создать счет на оплату

$client->createInvoice();

###Cписок входящих счетов

$client->listInvoices();

###Оплата входящего счета

$client->payInvoice();

##Экземпляры работы

###Перевод средств на другой кошелек

```PHP
<?php

$client = new QiwiClient("+7903000000", "test");
$client->authorize();

$request = new QiwiWalletPay("+79030172919");
$request->setAmount(1);
$request->setCurrency("RUB");
$request->setComment("test invoice");

//Вы можете подключить EventListener, наследуемый от интерфейса Firepro/Qiwi/PayEventListener 
//$request->setEventListener(new MyEventListener())

$response = $client->pay($request);

switch ($response->getState()) {

QiwiPayStates::NEED_SMS: 

    $smsCode = "Здесь полученный SMS код";
    $confirmResponse = $client->confirmPay($response->getId(), $smsCode);
    
    if ($confirmResponse->getState()==QiwiPayStates::DONE) {
        echo "Платеж успещно завершен";
    } else {
        var_dump($confirmResponse->getErrors()):
    }
    
    break;
    
QiwiPayStates::DONE:
    
    echo 'Платеж '.$response->getId().' успешно завершен'; 
    
    break;

QiwiPayStates::FAILED:
    echo "Произошли ошибки при выполнении платежа!";
    var_dump($response->getErrors()):
}

?>
```

##Поддержка и контрибьюция

Вы можете обратиться на support@funser.ru для получения помощи по данному клиенту. Также можете вносить изменения через Pull Requests.


