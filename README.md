[![Latest Release](https://img.shields.io/github/v/tag/NikitaLGit/k8s_viewer?sort=semver&label=release&color=6f42c1)](https://github.com/NikitaLGit/k8s_viewer/releases)
[![Build Status](https://img.shields.io/github/actions/workflow/status/NikitaLGit/k8s_viewer/build.yml?label=build)](https://github.com/NikitaLGit/k8s_viewer/actions/workflows/build.yml)
[![License: MIT](https://img.shields.io/github/license/NikitaLGit/k8s_viewer?color=blue)](LICENSE)
[![Go](https://img.shields.io/github/go-mod/go-version/NikitaLGit/k8s_viewer?filename=hub%2Fgo.mod&logo=go&label=go)](hub/go.mod)
[![Last commit](https://img.shields.io/github/last-commit/NikitaLGit/k8s_viewer)](https://github.com/NikitaLGit/k8s_viewer/commits/main)
[![Stack](https://img.shields.io/badge/stack-Go%20%C2%B7%20React%20%C2%B7%20SQLite-informational)](#51-стек)
[![Image](https://img.shields.io/badge/image-distroless%20%C2%B7%20non--root-2ea44f?logo=docker)](Dockerfile)
[![Deploy](https://img.shields.io/badge/deploy-air--gapped%20ready-success)](#-развёртывание-в-kubernetes)

<img width="1672" height="941" alt="image" src="https://github.com/user-attachments/assets/4e4fb2a5-1f86-4130-a88a-bc8991132bdf" />

> Один браузерный экран вместо десятков kubeconfig'ов, переключений
> контекста `kubectl` и отдельного SSH-клиента. Смотрите и правите ресурсы, открываете терминалы к подам и нодам, работаете с файлами, видите топологию, метрики и дрейф конфигурации, пользуетесь набором инструментов сопровождения и администрируете доступ — всё в одном месте, в реальном времени.

**Содержание:** [Что это и зачем](#1-что-это-и-зачем) · [Преимущества](#2-кому-и-чем-полезно-преимущества) · [Возможности](#3-основные-возможности-приложения) · [Инструменты](#4-инструменты-меню-tools) · [Установка](#-установка-и-первоначальная-настройка) · [Техспецификация](#5-техническая-спецификация)

---

## 1. Что это и зачем

**k8s-viewer** — production-хаб для управления несколькими Kubernetes-кластерами через единый веб-интерфейс. Разворачивается одним самодостаточным контейнером внутри вашей инфраструктуры и не требует, чтобы у оператора был прямой сетевой доступ к API-серверам кластеров.

### Проблема
- Кластеров много, они в разных сетях, часть достижима только через SSH-бастион.
- У каждого инженера — свой набор kubeconfig'ов, контексты `kubectl` путаются, доступы разрозненны.
- Диагностика ноды (нагрузка, диски, systemd, dmesg) требует ручного SSH мимо всех инструментов.
- Нет единого журнала: кто что перезапустил, смасштабировал, удалил.
- Резервные копии, дрейф конфигурации, версии образов по флоту — считаются вручную.

### Решение
Единая точка входа с ролевым доступом, аудитом и realtime-обновлением. Кластеры подключаются либо через SSH-туннель (двойной прыжок через бастион), либо через лёгкого агента внутри кластера (обратный WebSocket-туннель). Оператору достаточно браузера.

---

## 2. Кому и чем полезно. Преимущества

| Роль | Что получает |
|---|---|
| **Дежурный / SRE** | Обзор всего флота на одном экране, быстрый перезапуск/масштабирование, exec и SSH прямо из браузера, диагностика ноды на уровне ОС. |
| **Администратор** | Управление пользователями и кластерами, журнал аудита, запись сессий, серверные настройки и сроки хранения без редеплоя. |
| **Инженер сопровождения** | Отслеживание версий образов относительно эталона, синхронизация справочников, наблюдение за ресурсами, дрейф конфигурации, работа с БД, бэкапы. |
| **Руководитель / презентации** | Инфографика: диаграммы, графики и списки по данным флота для отчётов — офлайн, без интернета. |

### Чем отличается от «просто kubectl» и от Rancher
- **SSH-native** — прямой терминал в ноду, systemd-статус, файловый менеджер на admin-ноде, диагностика ОС (load / память / диски / failed-юниты / dmesg). Ни один kubeconfig-инструмент этого не видит.
- **Работает через бастион** — двойной SSH-прыжок к недоступным напрямую кластерам; либо агент из кластера, без входящих доступов вообще.
- **Backup-first** — вид Velero/k8up + восстановление из UI + верификация в S3.
- **Cross-cluster** — поиск по всему флоту (Ctrl+K), YAML-diff одного ресурса между кластерами.
- **Дрейф конфигурации без Git** — фоновые снапшоты «желаемого» состояния и timeline расхождений.
- **Совместные сессии** — терминал можно показать коллеге по ссылке (read-only, Teleport-style); видно присутствие («кто какой кластер смотрит»).
- **Полностью автономно** — не требует внешних сервисов. Опциональны только локальный реестр (для эталона версий) и локальный Ollama (для AI-подсказок по YAML).
- **Один артефакт** — фронт вшит в бинарь, образ distroless / non-root / read-only rootfs. Минимальная поверхность атаки.

---

## 3. Основные возможности приложения

### Обзор флота
<img width="1911" height="609" alt="image" src="https://github.com/user-attachments/assets/130f407d-08b3-4345-b1c6-c04bf23e4be5" />

- **Cluster overview** — состояние всех кластеров, нод и нагрузок на одном экране; спарклайны CPU/Memory в реальном времени, бейдж устаревшего бэкапа, прогноз заполнения PVC, поиск по кластерам.
- **Cluster-wide search (Ctrl+K)** — поиск ресурсов по всему флоту с инвертированным фильтром типов.
- **Присутствие** — видно, кто из коллег какой кластер сейчас смотрит.

### Кластер и ресурсы
<img width="1908" height="881" alt="image" src="https://github.com/user-attachments/assets/870493f1-79d2-4a2a-87e5-7a5481757ac2" />

- **ClusterView** — список ресурсов выбранного типа: фильтры по namespace/статусу, сортировка колонок, группировка по namespace, экспорт в CSV. Поды, деплойменты, statefulset'ы, сервисы, ingress и произвольные CRD.
- **Detail (карточка ресурса)** — обогащённый обзор со связанными кликабельными объектами, правка YAML с подсветкой, side-by-side YAML-diff между кластерами, просмотр логов (подсветка, follow/pause, поиск), live-карточка подов (Terminating/новые сразу по WebSocket).
- **Действия** — restart, scale, rollout history и rollback, delete, apply произвольного YAML; контекстное меню действий в строке таблицы.

### Терминалы и файлы
<img width="1041" height="579" alt="image" src="https://github.com/user-attachments/assets/f8e820f1-f27f-40c1-9e44-5ad913fd3de1" />

- **Exec в под** — полноценный PTY-терминал (xterm.js): ANSI-цвета, resize, tab-completion.
- **SSH в ноду** — терминал к узлу прямо в браузере (hub → бастион → admin-нода → нода); отдельный shell на admin-ноде, выбор пользователя.
- **Плавающий док терминалов** — ресайзящиеся окна, таскбар сессий снизу; сессии переживают навигацию, их можно держать несколько одновременно и сворачивать.
- **Файловый менеджер** — просмотр, загрузка и выгрузка файлов на admin-ноде.
- **Node OS Health** — load, память, диски, failed systemd-юниты, dmesg — через SSH.

### Визуализация и диагностика
<img width="1513" height="811" alt="image" src="https://github.com/user-attachments/assets/ff305373-719d-4b1a-aeab-31057bd116c3" />

- **Топология** — SVG force-directed граф (Deployment→Pods, Pod→Owner+Node, Service→Pods, Ingress→Services), zoom/pan, клик→переход.
- **Service map** — карта связей сервисов, выведенная из конфигов (env/args/envFrom → DNS сервисов), подписи на рёбрах.
- **Метрики и события** — CPU/Mem в реальном времени, история за 48 ч, поток событий кластера.

### Совместная работа и безопасность
<img width="962" height="811" alt="image" src="https://github.com/user-attachments/assets/ef8bed18-d1a6-4516-8adf-9426c183ba1c" />

- **Session sharing** — расшаривание терминала по ссылке, read-only зритель.
- **Запись сессий** — exec/SSH-сессии записываются в формате asciinema и доступны к просмотру.
- **Аудит** — журнал всех изменяющих операций с фильтрами и экспортом в CSV.
- **RBAC** — роли viewer / operator / admin / супер-админ.
- **i18n** — полный перевод EN/RU, тёмная и светлая темы.
- **Учебник** — встроенная база знаний со скриншотами (супер-админ может править статьи).

---

## 4. Инструменты (меню Tools)

| Инструмент | Что делает | Что хранит |
|---|---|---|
| **Version Tracker** | Показывает, какие версии образов запущены во всех кластерах сразу; группирует одинаковые сервисы по репозиторию; сравнивает с эталоном из реестра (Harbor/Nexus) и подсвечивает отставание. | Скан идёт на лету за один проход по флоту; эталон берётся из реестра. |
| **Dictionary Sync (НСИ)** | Сверка и загрузка справочников нормативно-справочной информации по кластерам; фоновые задачи синхронизации с пошаговым прогрессом. | `dict_checks` (история сверок), `dict_oid_mapping` (маппинг OID→справочник), `dict_tasks` / `dict_task_steps` (задачи и шаги). |
| **Resource Watch** | Правила слежения за ресурсами и алерты при срабатывании условия. | `watches` (правила), `watch_alerts` (сработавшие алерты). |
| **Config Drift** | Фоновые снапшоты «желаемой» конфигурации нагрузок и timeline расхождений; side-by-side diff и YAML любого события — без Git. | `config_snapshots` (снапшот пишется только при реальном изменении конфига), `drift_enabled` (по каким кластерам включено). |
| **Databases (PostgreSQL)** | Панель БД: Overview (соединения, репликация, блокировки, WAL, топ-запросы), Explorer (дерево схемы, структура таблиц), Query (read-only SQL-консоль). Killer-фича — джойн соединений на Pod IP. | `db_targets` (цели подключения и учётные данные); просмотр строк и запросы — только чтение, в отдельной транзакции, всё под аудитом. |
| **Infographic** | Диаграммы, графики и списки по данным флота для отчётов и презентаций. Полностью офлайн — рендер и экспорт (SVG/PNG) без интернета и CDN. | Шаблоны и настройки инфографики; движок рендерит локально. |
| **Backups (Velero / k8up)** | Вид резервных копий, восстановление из UI, диагностика и верификация в S3, CRUD расписаний VeleroSchedule. | Ничего не хранит: читает CRD кластера и напрямую S3. |
| **Runbooks** | Каталог сценариев (public/private, владелец-или-админ), запуск на admin-ноде кластера, live-вывод по WebSocket, интерактивный stdin (для команд с паролем). | `runbooks` (сценарии и их видимость/режим). |
| **AI YAML assistant** | Валидация и правка YAML через локальный Ollama (streaming); опционально. | Не хранит; обращается к локальному Ollama. |
| **Policy scan** | Встроенный линтер YAML (pure-Go, 11 правил), без внешних зависимостей. | Не хранит; проверка на лету. |
| **Export / Import config** | Перенос списка кластеров и параметров подключения (только admin). | Экспорт/импорт содержимого таблицы `clusters`. |

---

## 🚀 Установка и первоначальная настройка

Хаб — это один самодостаточный образ (фронт вшит в бинарь). Ставится тремя способами: витрина локально, деплой в Kubernetes через `kubectl`, или air-gapped-импорт образа.

### Требования
- Для сборки из исходников: **Go ≥ 1.26**, **Node ≥ 20**. Либо только **Docker** — тогда сборка идёт внутри образа.
- Для деплоя: доступ `kubectl` к кластеру с StorageClass (по умолчанию **Longhorn**, RWO).

### Вариант A — быстрый старт локально (demo-флот)
```bash
git clone https://github.com/NikitaLGit/k8s_viewer.git && cd k8s_viewer

make build     # собрать SPA в hub/web/dist и скомпилировать хаб
make run       # поднять всё на http://localhost:8080 (HUB_DEV=true, HUB_SOURCE=fake)
```
Откройте <http://localhost:8080>, войдите **admin / admin** (сидируется при первом запуске). Demo-флот работает без ключей и без доступа к настоящим кластерам.

Горячая перезагрузка фронта при разработке:
```bash
make run    # терминал 1 — API на :8080
make dev    # терминал 2 — Vite dev-сервер (проксирует /api и WS на :8080)
```

### Вариант B — контейнер
```bash
# сборка (контекст — корень репозитория)
make image REGISTRY=<registry>:<port> TAG=1.0.0
# или напрямую:
docker build -t <registry>/k8s-viewer-hub:1.0.0 -f Dockerfile .

# локальный запуск
docker run --rm -p 8080:8080 -e HUB_ADMIN_PASSWORD=change-me \
  -v k8s-viewer-data:/data <registry>/k8s-viewer-hub:1.0.0
# …или: docker compose up --build
```
Образ работает как non-root (uid 65532), read-only rootfs, без шелла; SQLite живёт на томе `/data`. Проба готовности — `GET /healthz`.

### <a id="-развёртывание-в-kubernetes"></a>Вариант C — развёртывание в Kubernetes

Образ на каждый git-тег `v*` собирает CI (`.github/workflows/build.yml`) и публикует в GHCR:
`ghcr.io/nikitalgit/k8s-viewer:latest`.

**Разово:** после первого прогона Actions сделайте пакет публичным (GitHub → Packages → `k8s-viewer` → Package settings → *Change visibility → Public*), либо оставьте приватным и добавьте `imagePullSecret`.

```bash
git clone https://github.com/NikitaLGit/k8s_viewer.git && cd k8s_viewer
kubectl apply -f deploy/k8s/
kubectl -n k8s-viewer rollout status deploy/hub
kubectl -n k8s-viewer port-forward svc/hub 8080:80   # → http://localhost:8080 (admin / admin)
```
Манифесты создают namespace `k8s-viewer`, ConfigMap, PVC (Longhorn, RWO), **безправный** ServiceAccount (хабу не нужен доступ к API кластера, где он живёт — он управляет *удалёнными* кластерами), Deployment (1 реплика, стратегия Recreate — единственный писатель SQLite), Service и Ingress.

**Приватный GHCR-пакет** (вместо публичного):
```bash
kubectl -n k8s-viewer create secret docker-registry ghcr \
  --docker-server=ghcr.io --docker-username=<github-user> --docker-password=<PAT-с-read:packages>
# затем в deploy/k8s/50-deployment.yaml под spec.template.spec добавить:
#   imagePullSecrets: [{ name: ghcr }]
```

**Air-gapped-кластер** (нет egress к ghcr.io):
```bash
# на машине с Docker:
docker build -t k8s-viewer:1.0.0 -f Dockerfile . && docker save k8s-viewer:1.0.0 -o k8s-viewer.tar
scp k8s-viewer.tar user@<node>:~/
# на каждой ноде k3s:
sudo k3s ctr images import ~/k8s-viewer.tar
# нацелить Deployment на локальный тег:
sed -i 's#ghcr.io/nikitalgit/k8s-viewer:latest#k8s-viewer:1.0.0#' deploy/k8s/50-deployment.yaml
sed -i 's/imagePullPolicy: Always/imagePullPolicy: IfNotPresent/' deploy/k8s/50-deployment.yaml
```

### Первоначальная настройка

1. **Смените пароль администратора.** По умолчанию — `admin / admin`. Меню профиля (справа сверху) → *Сменить пароль*. Либо засидируйте нестандартный пароль при первом запуске:
   ```bash
   kubectl -n k8s-viewer create secret generic hub-secrets \
     --from-literal=HUB_ADMIN_USER=admin --from-literal=HUB_ADMIN_PASSWORD='<strong>'
   kubectl -n k8s-viewer rollout restart deploy/hub
   ```
2. **Заведите пользователей и роли.** Меню профиля → *Пользователи*: добавление/удаление, роли viewer / operator / admin, сброс паролей.
3. **Смонтируйте SSH-ключ** (для режима `ssh`). Один ed25519-ключ, авторизованный и на бастионе, и на admin-нодах кластеров:
   ```bash
   kubectl -n k8s-viewer create secret generic hub-ssh-key --from-file=id=/path/to/hub_key
   kubectl -n k8s-viewer rollout restart deploy/hub
   ```
   > `kubectl apply -f deploy/k8s/` **не трогает** секреты (шаблоны убраны в `examples/`). Если ключ пуст/невалиден — хаб стартует, но ssh-jump отключён, кластеры уйдут в Degraded.
4. **Зарегистрируйте кластер** в UI кнопкой **+ Add**:
   - **SSH jump** — данные бастиона и admin-ноды; хаб откроет `hub → бастион → admin-нода`, прочитает kubeconfig с admin-ноды и погонит весь API-трафик через туннель (при необходимости — с `--tls-server-name`). Туннель самовосстанавливается (keepalive + backoff).
   - **Agent** — задайте pre-shared токен; в БД хранится только его SHA-256-хеш. Тот же токен передаётся агенту (`AGENT_TOKEN`) в манифестах `deploy/agent/`. Агент держит обратный WS-туннель — входящий доступ к кластеру не нужен.
5. **Настройте сроки хранения** (при необходимости). Настройки → *Хранение данных* — правится на живом поде без редеплоя (метрики 48 ч, дрейф 14 д, аудит 30 д, записи 14 д и т. д.).

### Обновление
```bash
cd k8s_viewer && git pull
kubectl apply -f deploy/k8s/
kubectl -n k8s-viewer rollout restart deploy/hub   # imagePullPolicy: Always
kubectl -n k8s-viewer rollout status  deploy/hub
```
Встроенные статьи учебника и новые возможности подтягиваются автоматически; статьи, отредактированные супер-админом, при обновлении сохраняются.

---

## 5. Техническая спецификация

### 5.1 Стек

- **Бэкенд:** Go, `client-go`, чистый-Go SQLite (сборка с `CGO_ENABLED=0`). REST + WebSocket API, собственные аутентификация и RBAC.
- **Фронтенд:** React + Vite + TypeScript + react-router-dom v7. Собирается в статику и **вшивается в Go-бинарь** через `//go:embed`.
- **Образ:** многослойная сборка → distroless (`gcr.io/distroless/static-debian12:nonroot`), non-root uid 65532, read-only rootfs, один порт `:8080`.
- **Агент** (для agent-режима): отдельный Go-модуль/бинарь, работает как Pod внутри целевого кластера.

### 5.2 Архитектура подключения

```
browser
  ↕ HTTPS/WSS
внешний reverse-proxy (TLS)
  ↕ HTTP/WS
Ingress кластера, где живёт хаб
  ↕
hub pod
  ↕ SSH        ↕ SSH               ↕ WS (обратный туннель)
бастион      admin-нода (kubc)       agent pod (в целевом кластере)
              ↕ HTTPS               ↕ HTTPS (localhost)
         kube-apiserver         kube-apiserver
```

**Три режима кластера** (`clusters.mode`):
- `fake` — demo-кластер, генерация в памяти, ключи не нужны (для витрины/разработки).
- `ssh` — двойной SSH-прыжок: один ed25519-ключ на оба хопа (бастион и admin-нода), все API-запросы и чтение kubeconfig идут через туннель. Опциональная верификация хостов через `known_hosts`.
- `agent` — обратный WebSocket-туннель из кластера; аутентификация по pre-shared токену (в БД хранится только его **SHA-256-хеш**, `clusters.agent_token_hash`); токен передаётся в заголовке `Authorization`, не в URL.

**Ключевые решения по надёжности:**
- **Warm-pool Manager** — по одному supervisor на кластер с backoff и кэшем карточек; подключение (dial) никогда не происходит внутри HTTP-хендлера, поэтому флот и ресурсы отдаются мгновенно.
- **Polling-watch вместо long-lived watch** — переспрашивание списка раз в ~2 с и diff; устойчиво к обрывам SSH, первый проход отдаётся сразу.
- **watchmux** — один общий поллер на `(кластер, тип, namespace)` обслуживает всех подписчиков; diff по стабильному `ItemID` элемента.
- **Exec через SPDY** поверх SSH-туннеля.

### 5.3 Хранение данных — что, где, как

Всё состояние — в **SQLite** на одном томе (Longhorn PVC, ReadWriteOnce). Три отдельные базы:

| Файл | Назначение |
|---|---|
| `hub.db` | Основное состояние: пользователи, сессии, кластеры, runbooks, метрики, снапшоты конфигов, справочники, watch'и, БД-цели, учебник, геймификация. |
| `audit.db` | Журнал аудита (отдельная база — изоляция и своя ротация). |
| `recordings.db` | Записи exec/SSH-сессий (asciinema). |

**Основные таблицы `hub.db`:**

- **Доступ:** `users` (bcrypt-хеш пароля, роль, full_name), `sessions` (хеш токена, TTL, created_at), `meta` (ключ-значение: маркеры сидов, сроки хранения и пр.).
- **Кластеры:** `clusters` (параметры подключения, режим, `agent_token_hash`).
- **Метрики:** `metrics_history` (кольцевой буфер CPU/Mem, ретенция 48 ч), `cluster_pvc` (одна строка на кластер, upsert — потребление PVC, «состояние, а не временной ряд»).
- **Дрейф:** `config_snapshots` (YAML нормализованного «желаемого» состояния, пишется только при изменении), `drift_enabled`.
- **Справочники (НСИ):** `dict_checks`, `dict_oid_mapping`, `dict_tasks`, `dict_task_steps`.
- **Наблюдение:** `watches`, `watch_alerts`.
- **БД-панель:** `db_targets` (цели и учётные данные PostgreSQL).
- **Совместные сессии:** `session_shares` (токен, ключ сессии, срок).
- **Runbooks:** `runbooks`.
- **Учебник:** `docs` — статьи (RU/EN). Встроенные статьи сидируются из бандла; флаг `edited` защищает статьи, отредактированные супер-админом, от перезаписи при обновлении.

**Как обслуживается объём (важно на большом флоте):**
- Хранение растёт как `кластеры × нагрузки × частота × ретенция`. Толстые колонки пишутся экономно: PVC — раз в 10 мин (не 30 с), снапшот дрейфа — только при реальном изменении (перед хешированием снимаются `status`, `resourceVersion` и служебные поля, чтобы status-апдейты контроллеров не плодили записи).
- **Возврат места ОС:** все базы переведены на `auto_vacuum=INCREMENTAL`; `store.ReclaimSpace` конвертирует старые базы (первый вызов — полный `VACUUM`) и вызывается после каждой prune-джобы. Проверка освобождения смотрит реальный размер файла и `-wal`, а не `page_count`.
- **Сроки хранения — настройка в БД, а не константа**: правятся в UI (Настройки → Хранение данных), применяются на живом поде без редеплоя. Значения по умолчанию: метрики 48 ч, дрейф 14 д, аудит 30 д, записи 14 д, сверки/алерты 14 д.

**Фоновые процессы:**
- сбор метрик — каждые 30 с; сбор PVC — каждые 10 мин;
- детектор дрейфа — снапшоты конфигов периодически, запись только при изменении;
- суточная ротация аудита и prune датасетов по текущим срокам хранения.

### 5.4 Аутентификация и RBAC

- Пароли — **bcrypt**; сессия — хеш токена в SQLite, cookie, TTL 12 ч; смена пароля инвалидирует сессии.
- Роли: **viewer** (чтение, watch, логи) < **operator** (+ restart/scale/apply/delete/exec) < **admin** (+ пользователи и кластеры); отдельно **супер-админ** — владелец инсталляции (серверные настройки, правка учебника).
- WebSocket-хендшейк — по одноразовому тикету (`POST /api/ws/ticket`, TTL 30 с); при 401 клиент переподключается автоматически.
- Защита от перебора: lockout после 5 неудач на 15 мин, аудит неуспешных входов; минимальная длина пароля 8, пароль ≠ username; предупреждение при дефолтном `admin/admin`.

### 5.5 API (обзор)

REST под `/api/...` + WebSocket под `/api/ws/...`. Основные группы:
- **auth:** login / logout / me / password / ws-ticket.
- **clusters:** CRUD + connect / health / topology / events / metrics / counts / resources (list/get/yaml/apply/restart/scale/delete) / backups / restore / velero-schedules / node exec / files / servicegraph / node health / metrics history / drift.
- **ws:** fleet-stream, resources (snapshot+deltas), events, follow logs, exec-PTY, ssh-PTY, runbook live output, session-share viewer, agent reverse tunnel.
- **tools/admin:** audit (+ filters), runbooks CRUD/run, policy check, AI yaml, db-панель (targets, discover, overview, activity, pods, settings, databases, tables, structure, rows, query), users CRUD.

### 5.6 Деплой и CI

- **Единый образ** (фронт вшит) → k8s-манифесты: namespace, ConfigMap (env, без секретов), PVC (Longhorn RWO), ServiceAccount (без ClusterRole — хаб не ходит в свой k8s), Deployment (1 реплика, стратегия Recreate — единственный писатель SQLite), Service, Ingress.
- **Секреты создаются вручную** (SSH-ключ, пароль админа) и не входят в репозиторий.
- **CI:** сборка образа по git-тегу `v*` → GitHub Container Registry; деплой — `rollout restart` (imagePullPolicy: Always).
- **Память:** `GOMEMLIMIT` держится строго ниже `limits.memory` пода (heap ≠ RSS: goroutine-стеки, SSH-буферы, page-cache SQLite).

### 5.7 Версионирование и информирование

- Версия инжектится при сборке (`-ldflags`), видна в интерфейсе как бейдж.
- **Changelog** (клик по версии) — «git log приложения»: сырые коммиты, файл `changelog.json` автоматически регенерируется из истории git перед сборкой релиза (скрипт `scripts/gen-changelog.sh`, шаг в CI).
- **What's new** — двуязычные (RU/EN) человекочитаемые заметки о значимых изменениях (`release_notes.json`); всплывают один раз при обновлении версии.

---

## Лицензия

[MIT](LICENSE) © 2025–2026 NikitaLGit

---

k8s-viewer превращает разрозненную работу с парком кластеров — kubeconfig'и, `kubectl`-контексты, ручной SSH, отдельные скрипты для бэкапов, версий и дрейфа — в **единую браузерную панель** с ролевым доступом, аудитом и обновлением в реальном времени. Работает там, где кластеры недоступны напрямую (через бастион или агента), не тянет внешних зависимостей и разворачивается одним защищённым контейнером.
