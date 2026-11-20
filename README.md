# ✦ OVENQUILL GUARD

Paper/Purpur 26.2 + Velocity moderation and security plugin for OVENQUILL.

> **Current build:** 1.1.0-SNAPSHOT. Requires Java 25, LuckPerms and CoreProtect for Xray Replay.

## Что изменено в 1.1

- В MOD MODE служебный инвентарь полностью пустой.
- /mod больше не выдаёт предметы и сам не включает spectator.
- /mod on временно подключает все permissions из настраиваемой группы LuckPerms.
- /mod off снимает временные разрешения и восстанавливает исходное состояние.
- Добавлен Xray Replay через публичный CoreProtect API.

## Установка

1. Установите `guard-paper-1.1.0-SNAPSHOT.jar` на каждый Paper/Purpur backend.
2. Установите `guard-velocity-1.0.0-SNAPSHOT.jar` на Velocity.
3. Перезапустите серверы.
4. В `config.yml` задайте уникальный `guard.server-id` и имя LP-группы.

## Настройка LuckPerms

Право на включение режима:

```text
/lp user <ник> permission set ovenquill.guard.mod.use true
```

Создайте отдельную группу функций модератора:

```text
/lp creategroup ovenquill-moderator
/lp group ovenquill-moderator permission set essentials.tp true
/lp group ovenquill-moderator permission set essentials.vanish true
/lp group ovenquill-moderator permission set ovenquill.guard.xray.inspect true
```

Конфигурация:

```yaml
permissions:
  provider: luckperms
  moderator-group: ovenquill-moderator
```

При `/mod on` плагин читает итоговые permissions этой группы, включая наследуемые, и подключает их игроку временно. Игрок не становится постоянным участником группы. При `/mod off`, выходе или восстановлении разрешения снимаются.

## Xray Replay

Команда работает только в активном MOD MODE и при наличии `ovenquill.guard.xray.inspect` в группе:

```text
/xray алмаз 2h
/xray незерит 30m
/xray diamond_ore 1d
/xray stop
```

- `алмаз` ищет сразу `DIAMOND_ORE` и `DEEPSLATE_DIAMOND_ORE`.
- `незерит` ищет `ANCIENT_DEBRIS`.
- Можно указать любой Minecraft Material.
- Время: секунды `s`, минуты `m`, часы `h`, дни `d`; максимум 30 дней.

Поиск выполняется асинхронно в базе CoreProtect. После поиска модератор переводится в spectator и телепортируется к первой записи. Он может свободно летать. ПКМ переносит к следующей записи. В чат выводятся материал, игрок, дата, время, мир и координаты. В конце списка либо после `/xray stop` возвращаются прежние режим игры и позиция.

## Безопасность

Перед MOD MODE исходное состояние фиксируется в SQLite с `synchronous=FULL`: инвентарь, броня, offhand, опыт, здоровье, голод, режим игры, координаты, полёт и эффекты. Незавершённое состояние восстанавливается после reconnect или сбоя. HTTP-аудит выполняется вне игрового потока, а команды с паролями и токенами маскируются.

## Файлы

- `guard-paper-1.1.0-SNAPSHOT.jar` — актуальный Paper/Purpur plugin.
- `guard-velocity-1.0.0-SNAPSHOT.jar` — совместимый Velocity Bridge.
- `ovenquill-guard-source-1.1.0-SNAPSHOT-20261121.zip` — полный Gradle-проект.

## Статус

Сборка прошла `clean test build`. Полная v1.0 всё ещё требует расширенного item-transfer аудита, Xray-аналитики подозрительности и Guard Dashboard. Xray Replay показывает факты CoreProtect и не выносит автоматический вердикт о нарушении.
