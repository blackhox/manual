# 1.Подготовка ОС Linux к установке ноды
### 1.1 Обновновляем пакеты и систему

***sudo apt update && sudo apt upgrade -y***

# 1.2 Устанавливаем все необходимые пакеты и компоненты:

***sudo apt install wget curl jq make git build-essential cmake gcc pkg-config libssl-dev -y***

Для правильной и более безопасной работы с ОС Linux, рекомендуется создавать отдельного пользователя и не работатать от имени root. Но в случае тестнетов, которые, как правило, длятся не более 1 месяца, я часто пользуюсь встроенной в систему администраторской учетной записью. Если вы решите создать отдельного пользователя и дать ему права на установку в системе, то это можно сделать с помощью следующих команд:

***adduser <имя пользователя>***

***usermod -aG sudo <имя пользователя>***

***su -l <имя пользователя>***

Я не буду создавать нового пользователя и продолжу установку от имени root

2. Устанавливаем GO
Если ОС на сервере не "чистая" ранее ставились какие-то проекты, то рекомендуется сначала удалить все предыдущие версии:

***sudo rm -rf /usr/local/go***

а затем установить необходимую версии GO.

Для данного проекта необходим Go 1.17.0+

Запустим скачивание, распаковку и установку GO:

***curl https://dl.google.com/go/go1.17.2.linux-amd64.tar.gz | sudo tar -C /usr/local -zxvf -***

3. Экспортируем переменные:

***cat <<'EOF' >>$HOME/.profile***

***export GOPATH=$HOME/go***

***export GO111MODULE=on***

***export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin***

***EOF***

***source $HOME/.profile***

4. Проверяем коректность установки, запустим команду:

***go version***

В выводе должно быть:

go version go1.17.2 linux/amd64

## 4 Установка ноды Evmos
***git clone https://github.com/tharsis/evmos.git && cd evmos && make install***

Проверим версию:

***evmosd version***

## 4.1 Проинициализируем ноду:

***evmosd init <имя моникера> --chain-id=evmos_9000-1***

## 4.2 Скачаем файл генезиса

***wget -O $HOME/.evmosd/config/genesis.json "https://raw.githubusercontent.com/tharsis/testnets/main/arsia_mons/genesis.json"***
Проверяем валидный ли файл генезиса:

***evmosd validate-genesis***

## 4.3 Отредактируем конфигурационный файл и добавил официальный сид и пиры других валидаторов:

***nano $HOME/.evmosd/config/config.toml***

seeds="c36cec90ded95d162b85f8ecd00ecd7c8849ca75@arsiamons.seed.evmos.org:26656

persistent_peers = "3bd90caf48ddd2d6b290550ecccd63348fc51da0@95.217.107.96:26658,f8da50943569f160854ac21c9ffb46fb4ff7bc0d@144.217.252.197:26626,1c4c38243893889a17fd3e677999f896b2b18586@95.217.35.111:26666,0e4dec8dd2cb74277bae3a9e7f1816603e97ce60@161.97.178.48:26656"

Актуальный списик пиров можно посмотреть тут: https://github.com/qf3l3k/testnets-2/blob/main/arsia_mons/peers.md

Создадим кошелек:

***evmosd keys add <имя кошелька>***

## 4.4 Импорт ключа в Метамаск и запрос токенов

для начала нам необходимо получить приватный ключ, для этого выполним команду:

***evmosd keys unsafe-export-eth-key <имя ключа>***

Копируем полученный ключ и переходим по этой ссылке:

https://evmos.dev/guides/keys-wallets/metamask.html

тут подробно описано,как импортировать приватный ключ в Метамаск и указаны реквизиты тестовой сети.
После импорта приватного ключа в Метамаск, вы получаете адрес кошелька в формате 0х.....
Копируем этот адрес и возвращаемся к консоли ноды.
Переводим данный hex-адрес в bech32 формат, выполним команду:

***evmosd debug addr 0x8D..........***

На выходе данной команды, мы получим преобразованные адреса оператора и валидатора, в формате:

***evmos1......***

***evmosvaloper1.......***

Далее запрашиваем токены на полученный адрес у крана:
https://faucet.evmos.org/

***Примечание.*** Запрашиваем на адрес evmos1, который получили на предыдущем шаге

После успешного запроса токенов, дожидаемся синхронизации ноды и создаем валиадатора.

## 5. Запуск синхронизации ноды.
Созданим сервис, для автоматического запуска ноды:

sudo tee /etc/systemd/system/evmosdd.service > /dev/null <<EOF
                                                               
[Unit]
                                                               
Description=Evmos Node
                                                               
After=network-online.target
                                                               
[Service]
                                                               
User=$USER
                                                               
ExecStart=$(which evmosd) start
                                                               
Restart=always
                                                               
RestartSec=3
                                                               
LimitNOFILE=65535
                                                               

[Install]
                                                               
WantedBy=multi-user.target
                                                               
EOF
                                                               
                                                 
***systemctl daemon-reload***
***systemctl enable evmosdd***
***systemctl restart evmosdd***

Смотрим журнал ноды:
***sudo journalctl -u evmosdd -f***

***Примечание.*** Обращаем вниманире на то, от какого имени будет стартовать данный сервис (User=$USER)
 Если ошибок нет, то ждем выполнения синхронизации
                                                               
 evmosd status 2>&1 | jq
 
  Если статус ##FALSE, то приступаем к созданию вадидатора:
  
                                                               
                                                               
