## Часть 1. Логирование

1. Создали файлы `docker-compose.yml` и `promtail_config.yml`
2. Запустили compose файл командой `docker-compose up -d`
	![[image-1.png]]
	![[image-2.png]]
1. Создали учётную запись администратора на nextcloud
2. Убедились, что логи Nextcloud записываются в файл `/var/www/html/data/nextcloud.log` после инициализации сервиса с помощью `docker exec -it ea34e3f77832 bash` и `cat data/nextcloud.log` 
	![[image-3.png]]
1. Через команду `docker logs promtail` проверили, что Promtail собирает логи Nextcloud
	![[image-4.png]]
## Часть 2. Мониторинг

1. Подключились к Zabbix и импортировали кастомный шаблон для мониторинга nextcloud.
2. Командами `docker exec -u www-data -it nextcloud bash` и `php occ config:system:set trusted_domains 1 --value="nextcloud"` разрешаем zabbix подключаться к nexcloud
3. В _Data collection → Hosts_ создали хост для Nextcloud и подключили шаблон мониторинга.
4. Проверили в _Monitoring → Latest data_ — данные успешно поступают (значение `healthy`).
	![[image-5.png]]
5. Проверили работу триггеров, включив в nextcloud maintenance mode командой `php occ maintenance:mode --on`. После чего зафиксировали проблему в разделе Monitoring → Problems, связанную с режимом обслуживания Nextcloud. Отключаем maintenance mode командой `php occ maintenance:mode --off` и проблема становится решенной
## Часть 3. Визуализация

1. Установили плагин Zabbix для Grafana командой `docker exec -it grafana bash -c "grafana cli plugins install alexanderzobnin-zabbix-app"`, после чего перезапустили контейнер Grafana с помощью команды `docker restart grafana`.
2. После перезапуска вошли в Grafana и в разделе `Administration → Plugins` активировали плагин Zabbix.
3. Подключили Loki и Zabbix к Grafana. 
4. В Explore, выбрали в качестве селектора индекс job, как результат мы увидели логи, подтверждая корректную настройку.
	![[image-6.png]]
1. То же самое было проделано с Zabbix.
	![[image-7.png]]
## Создание дашбордов
![[image-8.png]]

## Ответы на вопросы

1. **Чем SLO отличается от SLA?**
   - **SLO** (Service Level Objective) — это целевые значения метрик качества сервиса.
   - **SLA** (Service Level Agreement) — это соглашение, описывающее уровень качества сервиса и что делать, если SLO не достигаются.

2. **Чем отличается инкрементальный бэкап от дифференциального?**
   - **Инкрементальный бэкап** сохраняет изменения с момента последнего любого бэкапа (полного или инкрементального).
   - **Дифференциальный бэкап** сохраняет изменения с момента последнего полного бэкапа.

3. **В чем разница между мониторингом и observability?**
   - **Мониторинг** отслеживает определённые метрики и показывает, что произошло.
   - **Observability** обеспечивает глубокую диагностику системы, показывая, почему и как произошли события, анализируя множество данных и метрик.