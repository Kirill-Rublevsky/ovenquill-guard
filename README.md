# ✦ OVENQUILL GUARD

Система безопасной модерации и аудита для приватного Minecraft-проекта OVENQUILL. Поддерживает Paper/Purpur 26.2 и сеть Velocity.

> **Статус:** первый рабочий инкремент 1.0.0-SNAPSHOT. Xray, CoreProtect и временные LuckPerms-профили ещё находятся в разработке.

## Возможности

- MOD MODE через `/mod on|off|status`.
- Полное разделение игрового и служебного инвентарей.
- SQLite-журнал фаз `ENTERING / ACTIVE / EXITING`.
- Автоматическое восстановление после сбоя или reconnect.
- PDC-маркеры и защита служебных предметов от переноса.
- Аудит команд, блоков, контейнеров и телепортаций.
- Маскирование паролей и токенов.
- Оценка риска опасных команд.
- Идемпотентная локальная очередь аудита.
- Асинхронная пакетная отправка в OVENQUILL CONTROL.
- Velocity Bridge с запретом перехода при активном MOD MODE.

## Требования

- Minecraft / Paper / Purpur 26.2
- Java 25
- Velocity 4.x для прокси-сети
- LuckPerms для управления доступом

## Установка

1. Скачайте оба JAR из корня репозитория.
2. Установите `guard-paper-1.0.0-SNAPSHOT.jar` на каждый Paper/Purpur backend.
3. Установите `guard-velocity-1.0.0-SNAPSHOT.jar` на Velocity.
4. Перезапустите серверы и укажите уникальный `guard.server-id`.
5. Настройте Velocity modern forwarding и сетевую изоляцию backend.

## Доступ через LuckPerms

Команда `/mod` не требует OP и по умолчанию никому не доступна.

```text
/lp user <ник> permission set ovenquill.guard.mod.use true
```

Отозвать доступ:

```text
/lp user <ник> permission unset ovenquill.guard.mod.use
```

Разрешение на `/guard status` выдаётся отдельно:

```text
/lp user <ник> permission set ovenquill.guard.admin.status true
```

Право `ovenquill.guard.mod.use` разрешает только вход в MOD MODE. Оно не выдаёт OP, wildcard `*` или сторонние административные команды.

## Безопасность состояния

Перед переключением исходное состояние фиксируется в SQLite с `synchronous=FULL`. Сохраняются инвентарь, броня, вторая рука, выбранный слот, опыт, здоровье, голод, игровой режим, координаты, полёт и эффекты. Исходный снимок не перезаписывается незавершённой операцией.

При выходе запись переводится в `EXITING`, служебные предметы удаляются, исходное состояние восстанавливается и только после этого запись удаляется. При аварии восстановление выполняется на следующем входе.

## Audit API

События сначала сохраняются локально, затем отправляются без блокировки игрового потока:

```text
POST /api/v1/guard/events/batch
Authorization: Bearer <machine-token>
```

API-ключ хранится только в переменной окружения `OVENQUILL_GUARD_TOKEN`. Команды авторизации маскируются. Ввод команды отмечается как `SUBMITTED`, а не как гарантированно выполненная операция.

## Модули

- `guard-common` — модель аудита, маскирование секретов, оценка риска.
- `guard-paper` — MOD MODE, восстановление, инструменты, SQLite и Guard API.
- `guard-velocity` — блокировка небезопасных переходов между backend.

## Исходный код

Архив `ovenquill-guard-source-1.0.0-SNAPSHOT-20261121.zip` содержит полный Gradle-проект. Сборка:

```powershell
gradle wrapper
.\gradlew.bat clean test build
```

## План до полной v1.0

LuckPerms-профили `JUNIOR_MOD / MODERATOR / SENIOR_MOD / OWNER`, CoreProtect API, расширенный аудит item transfer, Xray Monitor, Xray Replay, heartbeat/config API и Guard Dashboard.

Подозрение Xray никогда не должно считаться доказанным нарушением или приводить к автоматическому бану без проверки.

## Лицензия

Проект предназначен для инфраструктуры OVENQUILL. Отдельная open-source лицензия пока не назначена.
