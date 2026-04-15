Окей, давай сделаем строго и красиво: все команды — в bash, конфиги — с типом. Ниже финальная версия README.md, уже вылизанная под GitHub.

`markdown

🛡️ Tor Setup on Fedora (with obfs4 Bridges)

Полноценный инженерный гайд по установке, настройке и отладке Tor + obfs4 на Fedora.  
Подходит для обхода DPI, жёстких блокировок и нестабильных сетей.

---

📚 Содержание

- Overview
- Установка
- Запуск Tor
- Настройка bridges
- Минимальный torrc
- Синхронизация времени
- Перезапуск
- Диагностика
- Типичные ошибки
- Использование
- Архитектура
- Полезные команды
- Deep Debug
- Custom systemd unit
- License

---

📌 Overview

Tor с obfs4 позволяет:

- обходить DPI и блокировки  
- скрывать факт использования Tor  
- работать в нестабильных сетях  
- использовать SOCKS5‑прокси для приложений  

Архитектура:

`mermaid
flowchart LR
    A[Application] --> B[SOCKS5 9050]
    B --> C[Tor Client]
    C --> D[Tor Network]
    D --> E[Internet]
`

---

⚙️ 1. Установка

`bash
sudo dnf install tor torsocks obfs4proxy
`

Проверка:

`bash
tor --version
torsocks --version
`

---

🚀 2. Запуск Tor

`bash
sudo systemctl enable --now tor
`

Проверка статуса:

`bash
systemctl status tor
`

---

🔐 3. Настройка bridges (obfs4)

Редактируем конфигурацию:

`bash
sudo nano /etc/tor/torrc
`

---

📄 Минимальный torrc

`conf
UseBridges 1
ClientTransportPlugin obfs4 exec /usr/bin/obfs4proxy

Bridge obfs4 <BRIDGE_1>
Bridge obfs4 <BRIDGE_2>
Bridge obfs4 <BRIDGE_3>
Bridge obfs4 <BRIDGE_4>
Bridge obfs4 <BRIDGE_5>

SocksPort 9050
`

Важно:

- Используйте 5–8 мостов
- Копируйте строки без изменений
- Удаляйте нестабильные / мёртвые мосты

---

⏱️ 4. Синхронизация времени (критично)

Tor зависит от корректного времени.

Проверка:

`bash
timedatectl status
`

Ожидаемый результат:

`text
System clock synchronized: yes
NTP service: active
`

Если нет:

`bash
sudo timedatectl set-ntp true
`

---

🔄 5. Перезапуск Tor

`bash
sudo systemctl restart tor
`

---

🔍 6. Диагностика

Онлайн‑лог:

`bash
journalctl -u tor -f
`

Нормальный запуск:

`text
Bootstrapped 10%
Bootstrapped 25%
Bootstrapped 75%
Bootstrapped 100% (done)
`

---

❌ Частые ошибки

🟥 general SOCKS server failure

Причина: проблемный / мёртвый bridge  
Решение: удалить или заменить соответствующий Bridge в torrc

---

🟥 Consensus is too old

Причина: устаревший bridge / нет доступа к директори‑серверам  
Решение: обновить список мостов, проверить сеть

---

🟥 No circuits are opened

Причина: мало мостов или они нерабочие  
Решение: добавить 5–8 свежих obfs4‑мостов

---

🟥 system clock jumped

Причина: проблема с временем  
Решение: включить NTP, проверить timedatectl

---

🌐 7. Использование

Через torsocks

`bash
torsocks curl https://check.torproject.org
`

Через SOCKS5 напрямую

`bash
curl --proxy socks5h://127.0.0.1:9050 https://api.ipify.org
`

---

🧠 8. Как это работает

`mermaid
flowchart LR
    App[Application] --> S[SOCKS5 9050]
    S --> T[Tor Client]
    T --> N[Tor Network]
    N --> I[Internet]
`

---

🧩 9. Архитектура (рекомендуемая)

`mermaid
flowchart TD
    C[clean profile] -->|direct| Internet
    D[dirty profile] -->|proxy / VPN| Internet
    A[anonymous profile] -->|Tor| TorNetwork
`

---

🧹 10. Полезные команды

`bash
journalctl -u tor -f
systemctl restart tor
torsocks curl https://google.com
`

---

🛠 Deep Debug

Проверка транспорта obfs4

`bash
sudo -u tor /usr/bin/obfs4proxy -enableLogging -logLevel DEBUG
`

Проверка доступности bridge по IP/порту

`bash
torsocks telnet <bridgeip> <bridgeport>
`

или:

`bash
torsocks nc -v <bridgeip> <bridgeport>
`

Проверка DNS‑утечек

`bash
torsocks dig example.com
`

Анализ bootstrap‑этапов

`bash
grep -i "Bootstrapped" /var/log/tor/log
`

(путь к логу может отличаться — см. Log в torrc)

Проверка SELinux (если включён)

`bash
sudo ausearch -m avc -ts recent
`

---

⚙ Custom systemd unit для отдельного профиля Tor

Например, отдельный анонимный профиль tor-anon с собственным torrc.

Создаём юнит:

`bash
sudo nano /etc/systemd/system/tor@anon.service
`

Содержимое:

`ini
[Unit]
Description=Tor Anon Instance
After=network.target

[Service]
User=tor
Group=tor
Type=simple
ExecStart=/usr/bin/tor -f /etc/tor/torrc-anon
Restart=on-failure

[Install]
WantedBy=multi-user.target
`

Пример отдельного конфига:

`bash
sudo nano /etc/tor/torrc-anon
`

`conf
UseBridges 1
ClientTransportPlugin obfs4 exec /usr/bin/obfs4proxy

Bridge obfs4 <BRIDGE_1>
Bridge obfs4 <BRIDGE_2>
Bridge obfs4 <BRIDGE_3>
Bridge obfs4 <BRIDGE_4>
Bridge obfs4 <BRIDGE_5>

SocksPort 0.0.0.0:9150
Log notice file /var/log/tor/tor-anon.log
`

Активируем профиль:

`bash
sudo systemctl daemon-reload
sudo systemctl enable --now tor@anon
systemctl status tor@anon
`

---

📌 Итог

- ✔ Tor установлен  
- ✔ obfs4 bridges настроены  
- ✔ SOCKS5 доступен на 9050 (и опционально 9150 для профиля anon)  
- ✔ обход DPI работает  

---

🚀 Дополнительно

Можно улучшить:

- 🔗 Tor через VPN/Xray  
- 🔄 автообновление bridges  
- 🔐 интеграция с Firejail  
- ⚙ split routing  

---

📜 License

MIT
`

Если хочешь, можем вынести deep debug в отдельный DEBUG.md и сделать из README.md более лаконичную «фронт‑страницу».
