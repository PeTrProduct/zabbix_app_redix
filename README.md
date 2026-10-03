# Шаблон мониторинга СХД RAIDIX

Шаблон использует Resp API для обнаружения получения статусов объектов:
- Узлы кластера
- Физические диски
- RAID массивы
- LUN

Дополнительно, для LUN выполняется сбор статистики по IOPS (усредненное значение за минуту по 1-секундным трейсам)

## Установка и настройка

1)	В RAIDIX создать пользователя с правами чтения, который будет использоваться для мониторинга
2)	Импортировать в Zabbix шаблон мониторинга «RAIDIX by HTTP» (template_app_raidix.yaml).
3)	Добавить все узлы кластера RAIDIX в Zabbix. Рекомендуется назначить на хост шаблон «ICMP Ping» для проверки сетевой доступности.
4)	На каждый хост добавить макросы:

 - **{$RAIDIX.AUTH.USER}** – имя пользователя в RAIDIX
 - **{$RAIDIX.AUTH.PASSWORD}** – пароль
 - **{$RAIDIX.URL.HOST}** – URL web-консоли администрирования RAIDIX (рекомендуется без закрывающего слеша)

5) Назначить на хосты шаблон «RAIDIX by HTTP». В течение нескольких минут последовательно выполняется обнаружение компонентов RAIDIX и настройка связанных элементов данных и триггеров. Проверить наличие новых элементов можно на странице «Latest data» для каждого хоста RAIDIX (раздел «Monitoring»).

---

## Macros used

| Macro | Description | Value |
| ----- | ----------- | ----- |
 | `{$RAIDIX.AUTH.PASSWORD}` | API user password | `xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx` |
 | `{$RAIDIX.AUTH.USER}` | API user name | `monitoring` |
 | `{$RAIDIX.URL.HOST}` | API URL | `http://127.0.0.1` |

## Triggers

| Name | Expression | Severity |
| ---- | ---------- | -------- |
 | RAIDIX: API service not available | `last(/RAIDIX by HTTP/raidix.api.available) <> 200` | high |
 | RAIDIX: Cluster node not OK | `last(/RAIDIX by HTTP/raidix.cluster.node.status)<>"OK"` | high |

## Data Items

| Name | Type | Key |
| ---- | ---- | --- |
 | RAIDIX: API availability status | script | `raidix.api.available` |
 | RAIDIX: Get cluster node data | script | `raidix.cluster.node` |
 | RAIDIX: Cluster node: Status | dependent | `raidix.cluster.node.status` |
 | RAIDIX: Get Drive data | script | `raidix.drive.data` |
 | RAIDIX: Get LUN data | script | `raidix.lun.data` |
 | RAIDIX: Get RAID data | script | `raidix.raid.data` |

## LLD rule RAIDIX: Drive Discovery

#### Trigger prototypes for RAIDIX: Drive Discovery

| Name | Expression | Severity |
| ---- | ---------- | -------- |
 | Drive [{#RAIDIX.DRIVE.ID}] status not OK | `last(/RAIDIX by HTTP/raidix.drive.status.[{#RAIDIX.DRIVE.ID}])<>"ok"` | warning |

### Item prototypes for RAIDIX: Drive Discovery

| Name | Type | Key |
| ---- | ---- | --- |
 | Drive [{#RAIDIX.DRIVE.ID}] status | dependent | `raidix.drive.status.[{#RAIDIX.DRIVE.ID}]` |

## LLD rule RAIDIX: LUN Discovery

#### Trigger prototypes for RAIDIX: LUN Discovery

| Name | Expression | Severity |
| ---- | ---------- | -------- |
 | LUN [{#RAIDIX.LUN.ID}] status not OK | `last(/RAIDIX by HTTP/raidix.lun.status.[{#RAIDIX.LUN.ID}])<>"ok"` | high |

### Item prototypes for RAIDIX: LUN Discovery

| Name | Type | Key |
| ---- | ---- | --- |
 | LUN [{#RAIDIX.LUN.ID}]: Read IOPS (average to minute) | dependent | `raidix.lun.performance.read.[{#RAIDIX.LUN.ID}]` |
 | LUN [{#RAIDIX.LUN.ID}]: Write IOPS (average to minute) | dependent | `raidix.lun.performance.write.[{#RAIDIX.LUN.ID}]` |
 | LUN [{#RAIDIX.LUN.ID}]: Get IOPS data | script | `raidix.lun.performance.[{#RAIDIX.LUN.ID}]` |
 | LUN [{#RAIDIX.LUN.ID}] status | dependent | `raidix.lun.status.[{#RAIDIX.LUN.ID}]` |

## LLD rule RAIDIX: RAID Discovery

#### Trigger prototypes for RAIDIX: RAID Discovery

| Name | Expression | Severity |
| ---- | ---------- | -------- |
 | RAID [{#RAIDIX.RAID.ID}] status not OK | `last(/RAIDIX by HTTP/raidix.raid.status.[{#RAIDIX.RAID.ID}])<>"ok"` | high |

### Item prototypes for RAIDIX: RAID Discovery

| Name | Type | Key |
| ---- | ---- | --- |
 | RAID [{#RAIDIX.RAID.ID}] status | dependent | `raidix.raid.status.[{#RAIDIX.RAID.ID}]` |

