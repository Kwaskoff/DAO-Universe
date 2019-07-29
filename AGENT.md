Запуск Frame
============
В **обычной командной строке** выполнить команды:
```
cd frame
npm run alpha
```
Активировать нужный Ethereum-аккаунт.


Запуск IPFS
===========
В отдельном окне командной строки **Linux**:
```
aragon ipfs
```

Работа с Agent через Aragon CLI
===============================
Работаем в отдельном окне командной строки **Linux**.
Рассмотрены примеры с Compound.finance.

При первом вызове Aragon CLI может выдать ошибку - тогда в окне Frame может потребоваться разрешить Aragon CLI работать с Frame (кнопка Approve)

В папке, в которой мы находимся при работе с командной строкой Linux, желательно разместить файл с набором переменных, чтобы не приходилось копипастить нужные адреса.
Кроме того, это снижает вероятность ошибки.

Файл может иметь вид (например, `env-marsdao-mainnet.sh`):
```
#!/bin/sh

dao=mars-dao.eth
agent=0x0a74D136fAfed0F8d58ce4b7307283695EC7A0B6
flags="--env aragon:mainnet --use-frame"
eth=0x0000000000000000000000000000000000000000
zrx=0xe41d2489571d322189246dafa5ebde1f4699f498
dai=0x89d24a6b4ccb1b6faa2625fe562bdd9a23260359
c_zrx=0xb3319f5d18bc0d84dd1b4825dcde5d5f7266d407
c_dai=0xf5dce57282a584d2746faf1593d3121fcac444dc
```

Следите за актуальностью адресов, дополняйте по необходимости.
Файл можно быстро подредактировать "на месте" через `nano env-marsdao-mainnet.sh`
Ctrl-X - выход из редактора с вопросом о сохранении.

Если файл менялся, перезагружали комп или командную строку - переменные из файла втягиваются через
`source env-marsdao-mainnet.sh`
или
`. ./env-marsdao-mainnet.sh` (точка пробел точка слеш ... - это сокращенная форма)

После этого можно работать с Aragon CLI.

Compound v2 (Presidio)
======================


Все суммы указываются в неделимых единицах.
Для Ether - в wei, т.е. для 1 ether = 1000000000000000000.
Для ERC-20 Token XYZ с 18 digits, 1 xyz = 1000000000000000000.

Approve (в app.compound.finance называется Enable)
----------------------------------------------

Разрешение контракту Compound на снятие токенов с баланса Agent не более определеннай суммы. Для эфира Approve не нужен. Вызовы рассмотрены на примере cDAI.
```
dao $flags act $agent $dai 'approve(address,uint256)' $c_dai <сумма>
```
Функция Enable в app.compound.finance делает сумму максимальной, т.е. достаточно 1 раз вызвать Approve и больше об этом не вспоминать.
**С другой стороны, мы расписываемся в том, что доверяем коду смарт-контракта Compound, и верим в отсуствие в нем уязвимостей, позволяющих злоумышленнику снять с нашего баланса токены от имени Compound на свое усмотрение.**

Неограниченный Approve можно сделать так:
```
dao $flags act $agent $dai 'approve(address,uint256)' $c_dai $ffff
```

Отмена Approve. Кнопки "Disable" в app.compound.finance нет, но запретить контрактам Compound доступ к токенам агента все же можно. Так:

```
dao $flags act $agent $dai 'approve(address,uint256)' $c_dai 0
```

Mint (в app и v1 протокола - Supply)
------------------------------------
Сумма указываемтся в неделимых оригинальных токенах (ETH, DAI etc).
У cEther и сErc20 вызываются по-разному.
Для cEther (нужна патченная версия Aragon CLI).

Отдать 1.234 DAI в обмен на cDAI:
```
dao $flags act $agent $c_dai 'mint(uint256)' 1234000000000000000
```
Отдать 1.234 ETH в обмен на cETH:
```
dao $flags act $agent $c_eth 'mint()' --value 1234000000000000000
```

Redeem (api/v1 Withdraw)
-----------------
Сумма указываемтся в неделимых Compound-токенах (cETH, cDAI etc).
Выкупить DAI обратно за 1.234 cDAI:
```
dao $flags act $agent $c_dai 'redeem(uint256)' 1234000000000000000
```

Redeem Underlying
-----------------
Сумма указываемтся в неделимых оригинальных токенах (ETH, DAI etc).
Выкупить 1.234 DAI обратно за cDAI:
```
dao $flags act $agent $c_dai 'redeemUnderlying(uint256)' 1234000000000000000
```