# fluxcd_vds

GitOps-репозиторий для управления Kubernetes-кластером с использованием FluxCD.

## О проекте

Репозиторий реализует подход **GitOps**: все изменения инфраструктуры и приложений происходят через Git.

FluxCD автоматически синхронизирует состояние Kubernetes-кластера с этим репозиторием.

---

## Структура репозитория

```
.
├── apps/
│   ├── base/          # Базовые HelmRelease/Kustomize для приложений
│   ├── production/    # Патчи для production окружения
│   └── staging/       # Патчи для staging окружения
│
├── clusters/
│   ├── production/    # Конфигурация production кластера
│   └── staging/       # Конфигурация staging кластера (с bootstrap Flux)
│
├── infrastructure/
│   ├── cert-manager/      # cert-manager + ClusterIssuer
│   ├── repositories/      # HelmRepository источники
│   └── servers-transport/ # Доп. настройки
│
└── README.md
```

---

## Приложения

Базовые приложения находятся в `apps/base/`:

- flask-app
- mailu
- matrix
- mousebook
- moviecat
- vless

Каждое приложение описано через:
- `release.yaml` (HelmRelease)
- `kustomization.yaml`

Окружения (`production`, `staging`) накладывают патчи:
- настройки ресурсов
- домены
- переменные окружения

---

## Окружения

### Staging

- содержит bootstrap Flux (`flux-system`)
- используется для тестирования

### Production

- использует готовую инфраструктуру
- подключает:
  - приложения (`apps.yaml`)
  - cert-manager
  - Helm репозитории

---

## Infrastructure

### cert-manager

- автоматическая выдача TLS сертификатов
- staging и production ClusterIssuer

### repositories

- Helm репозитории для всех сервисов

### servers-transport

- кастомные настройки сетевого уровня

---

## Как это работает

1. Изменения пушатся в Git
2. Flux отслеживает репозиторий
3. Flux применяет изменения в кластер

---

## Управление

Проверка состояния:

```bash
flux get all
```

Принудительная синхронизация:

```bash
flux reconcile source git flux-system
```

---

## Добавление нового приложения

1. Создать папку в `apps/base/<app>`
2. Добавить:
   - `release.yaml`
   - `kustomization.yaml`
3. Добавить патчи в:
   - `apps/staging/`
   - `apps/production/`
4. Подключить в `apps.yaml` кластера

---

## Безопасность

Рекомендуется:

- использовать SOPS или Sealed Secrets
- ограничивать доступ к кластеру
- хранить чувствительные данные вне Git

---

## 📝 Лицензия

MIT