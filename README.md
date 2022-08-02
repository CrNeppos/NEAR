# Установка ноды NEAR Protocol 

Рекомендуемые требования к серверу для установки ноды: CPU: 4 CORE RAM: 8 GB HDD/SSD: 500 GB
Устанавливать будем на арендованном сервере с операционной системой Ubuntu 20.04. Для этого можно восспользоваться услугами какого-нибудь поставщика данных услуг, благо таких большое множество.

### 1. Регистрируем кошелек 

Для начала идем на сайт https://wallet.shardnet.near.org/ где регистрируем себе учетную запись. Для чего на главной странице нажимает Создать учетную запись. Далее вводим желаемое имя учетной записи и нажимаем кнопку Reserve my account id, на следующем экране нажимаем Продолжить, сохраняем себе в блокнот мнемоническую фразу, нажимаем Продолжить, вводим запрашиваемое слово, после чего вводим всю мнемоническую фразу. Поздравляю, учетная запись создана!

![img](./images/1.png)

![img](./images/2.png)

### 2. Покупаем сервер

Компаний предоставляющих данный сервис великое множество, в моем случае это будет https://www.hetzner.com/. Сразу скажу, что для оплаты российские карты не подходят, поэтому придется обзавестись иностранной. На пробу взял СPX31 за 14.28 евро c идеей расширить SSD в случае необходимости в последствии за счет присоединения соответствующего Volume.

![img](./images/3.png)

### 3. Подключаемся к серверу 

Подключаемся к приобретенному серверу по ssh. Для этого я использую программу iTerm2 для MacOS. 

![img](./images/4.png)

### 4. Устанавливаем интерфейс командной строки NEAR (NEAR-CLI) 

Для этого вводим все команды по очереди: 

```
sudo apt update && sudo apt upgrade -y

curl -sL https://deb.nodesource.com/setup_18.x | sudo -E bash -  

sudo apt install build-essential nodejs

PATH="$PATH"
```

Проверяем версии Node.js и npm:
```
node -v

npm -v
```
Должны вернуться, что-то похожее на v18.x.x и 8.x.x

Устанавливаем NEAR-CLI
```
sudo npm install -g near-cli
```


### 5. Устанавливаемся ноду NEAR

Для начала проверим, что сервер подходит для установки ноды. Копируем команду ниже, вставляем в окно терминала и нажимает enter.
```
lscpu | grep -P '(?=.*avx )(?=.*sse4.2 )(?=.*cx16 )(?=.*popcnt )' > /dev/null \
  && echo "Supported" \
  || echo "Not supported"
```
Если получаем ответ 
> Supported 

, то можно двигаться дальше.

Теперь устанавливаем требуемые программные пакеты. Для этого вводим все команды по очереди:
```
sudo apt install -y git binutils-dev libcurl4-openssl-dev zlib1g-dev libdw-dev libiberty-dev cmake gcc g++ python docker.io protobuf-compiler libssl-dev pkg-config clang llvm cargo

sudo apt install python3-pip

USER_BASE_BIN=$(python3 -m site --user-base)/bin

export PATH="$USER_BASE_BIN:$PATH"

sudo apt install clang build-essential make

curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```
Дойдя до момента на скрине ниже напечатайте цифру 1 и нажмите enter. 

![img](./images/5.png)

Продолжаем вводить команды по одному:
```
source $HOME/.cargo/env

git clone https://github.com/near/nearcore

cd nearcore

git fetch

git checkout <commit>

cargo build -p neard --release --features shardnet
```
После этого запустится процесс компиляции исполняемого файла ноды. Это займет некоторое время. Готовый файл можно найти по следующему пути: target/release/neard

![img](./images/6.png)

Далее вводим: 
```
rm ~/.near/config.json

wget -O ~/.near/config.json https://s3-us-west-1.amazonaws.com/build.nearprotocol.com/nearcore-deploy/shardnet/config.json

Все готово к запуску ноды. Для этого вводим следующие команды:

cd ~/nearcore

./target/release/neard --home ~/.near run
```
После чего пойдут логи запущенной ноды. Должны подключится пиры и загрузится хидеры до 100%, после чего начнутся грузится блоки. Если все так, то нода успешно установлена.

![img](./images/7.png)

Так же не забудем сделать сервисный файл для ноды:
```
nano /etc/systemd/system/neard.service
```
В пустое окно терминала копируем и вставляем следующее (не забудьте вместо <USER> вписать свое имя в Ubuntu):

[Unit]
Description=NEARd Daemon Service

[Service]
Type=simple
User=<USER>
#Group=near
WorkingDirectory=/home/<USER>/.near
ExecStart=/home/<USER>/nearcore/target/release/neard run
Restart=on-failure
RestartSec=30
KillSignal=SIGINT
TimeoutStopSec=45
KillMode=mixed

[Install]
WantedBy=multi-user.target

Нажимает ctrl+O и ctrl+X. 

Остается ввести пару команд:
```
sudo systemctl enable neard
sudo systemctl start neard
```


### 6. Активируем ноду в качестве валидатора

Вводим в терминале следующую команду:
```
near login
```
На появившейся запрос отвечаем n. 

![img](./images/8.png)

Далее копируете появившуюся ссылку и вставляете ее в том же браузере, где ранее регистрировали учетную запись для кошелька NEAR.  

![img](./images/9.png)

После чего в появившемся окне нажимаем Next и подтверждаем предоставление доступа к учетной записи. Нас должно перекинуть на 127.0.0.1 или что-то похожее. Можем закрыть сайт и вернуться в терминал. 

![img](./images/10.png)

В терминале вводим название нашей учетной записи. 

![img](./images/11.png)

В последствии появятся несколько запросов на русском языке, на которые нужно будет ответить. Тут ничего сложного - просто делайте то, что от вас просят. По итогам в терминале должно появится что-то похожее на это: 

![img](./images/12.png)

### Полезные ссылки и команды
[Список всех валидаторов](https://explorer.shardnet.near.org/nodes/validators)

[Список всех заданий](https://github.com/near/stakewars-iii/blob/main/challenges/challenge-summary.md)

Что бы использовать следующие команды, прописываем:
```
  export NEAR_ENV=shardnet
```
- Проверка наличие на попадание в валидаторы:
```
  near proposals | grep название_пула
```
- Проверить являетесь ли вы валидатором в данной эпохе
```
  near validators current | grep название_пула
```
- Проверить стоимость входа в валидаторы ( на данный момент 530 )
```
  near validators next | grep seat
```
- Проверить, сможете ли вы стать валидатором в следующей эпохе
```
  near validators next | grep название_пула
```
- Перезапуск ноды
```
  sudo systemctl restart neard
```
- Проверка логов ноды
```
  journalctl -n 100 -f -u neard | ccze -A
```
- Проверка логов стейкинг пула
```
  cat /home/logs/all.log
```
- Просмотреть общий баланс стейкинг пула
```
  near view название_пула get_account_total_balance '{"account_id": "имя_пользователя"}'
```
Название пула: yourname.factory.shardnet.near ( пример test.factory.shardnet.near, вы указывали имя в конце установки )

Имя пользователя: yourname.shardnet.near

- Для проверки версии ноды необходимо установить curl и jq
```
  sudo apt install curl jq
```
- Теперь можно проверить версию ноды
```
  curl -s http://127.0.0.1:3030/status | jq .version
```
- Проверить делигаторов
```
  near view название_пула get_accounts '{"from_index": 0, "limit": 10}' --accountId имя_пользователя
```
- Анстейкинг монет NEAR. Кол-во указываем в yoctoNEAR
```
  near call название_пула unstake '{"amount": "кол-во yoctoNEAR"}' --accountId имя_пользователя --gas=300000000000000
```
- Анстейкинг всех NEAR
```
  near call название_пула unstake_all --accountId имя_пользователя --gas=300000000000000
```
- Анстейкинг и вывод. Анстейкинг занимает 2–3 эпохи, после чего вы сможете вывести в YoctoNEAR из пула
```
  near call название_пула withdraw '{"amount": "кол-во yoctoNEAR"}' --accountId имя_пользователя --gas=300000000000000
```
- Вывод всех средств
```
  near call название_пула withdraw_all --accountId имя_пользователя --gas=300000000000000
```
- Приостановить стейкинг
```
  near call название_пула pause_staking '{}' --accountId имя_пользователя
```




