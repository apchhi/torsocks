README.md для GitHub — с заголовками, блоками, подсветкой, структурой и инженерной подачей.


---

# 🛡️ Tor Setup on Fedora (with obfs4 Bridges)

## 📌 Overview

Данный гайд описывает установку и настройку **Tor** на Fedora с поддержкой **obfs4 bridges** для обхода DPI и блокировок.

Архитектура:

Application → SOCKS5 (9050) → Tor → Tor Network → Internet

---

# ⚙️ 1. Установка

Устанавливаем Tor и необходимые компоненты:

```bash
sudo dnf install tor torsocks obfs4proxy

Проверка:

tor --version
torsocks --version


---

🚀 2. Запуск Tor

sudo systemctl enable --now tor

Проверка статуса:

systemctl status tor


---

🔐 3. Настройка bridges (обход DPI)

Редактируем конфигурацию:

sudo nano /etc/tor/torrc


---

📄 Минимальный конфиг

UseBridges 1
ClientTransportPlugin obfs4 exec /usr/bin/obfs4proxy

Bridge obfs4 <BRIDGE_1>
Bridge obfs4 <BRIDGE_2>
Bridge obfs4 <BRIDGE_3>
Bridge obfs4 <BRIDGE_4>
Bridge obfs4 <BRIDGE_5>

SocksPort 9050


---

⚠️ Важно

Используйте 5–8 мостов

Копируйте строки без изменений

Удаляйте нестабильные мосты



---

⏱️ 4. Синхронизация времени (критично)

Tor зависит от корректного времени.

Проверка:

timedatectl status

Ожидаемый результат:

System clock synchronized: yes
NTP service: active

Если нет:

sudo timedatectl set-ntp true


---

🔄 5. Перезапуск Tor

sudo systemctl restart tor


---

🔍 6. Диагностика

journalctl -u tor -f


---

✅ Нормальный запуск

Bootstrapped 10%
Bootstrapped 25%
Bootstrapped 75%
Bootstrapped 100% (done)


---

❌ Частые ошибки

general SOCKS server failure

проблемный bridge

удалить



---

Consensus is too old

устаревший bridge

заменить



---

No circuits are opened

мало мостов

мосты нерабочие



---

system clock jumped

проблема времени

включить NTP



---

🌐 7. Использование

Через torsocks

torsocks curl https://check.torproject.org


---

Через SOCKS5

curl --proxy socks5h://127.0.0.1:9050 https://api.ipify.org


---

🧠 8. Как это работает

Приложение → SOCKS5 → Tor → сеть Tor → интернет


---

🧩 9. Архитектура (рекомендуемая)

clean      → direct
dirty      → proxy / VPN
anonymous  → Tor


---

🧹 10. Полезные команды

journalctl -u tor -f

systemctl restart tor

torsocks curl https://google.com


---

📌 Итог

✔ Tor установлен

✔ obfs4 bridges настроены

✔ SOCKS5 доступен на 9050

✔ работает обход DPI



---

🚀 Дополнительно

Можно улучшить:

🔗 Tor через VPN/Xray

🔄 автообновление bridges

🔐 интеграция с Firejail

⚙️ split routing



---

📜 License

MIT

---

Если хочешь:

- с **mermaid diagram (графика прямо в README)**
- с **разделом troubleshooting уровня “deep debug”**
- с **systemd unit для кастомного tor профиля**
