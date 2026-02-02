## 1. GitLab CI/CD vs GitHub Actions

Оба инструмента очень похожи (оба используют YAML и концепцию Runner/Action), но есть нюансы:

| Критерий | GitLab CI/CD | GitHub Actions |
| --- | --- | --- |
| **Интеграция** | Все в одном (Registry, CI, CD, Security) | Глубокая интеграция с репозиториями GitHub |
| **Повторное использование** | Через ключевое слово `include` и шаблоны | Через **Actions** (готовые блоки в Marketplace) |
| **Runner** | Собственный мощный агент (GitLab Runner) на Go | Hosted runners или Self-hosted агенты |
| **Сильная сторона** | Гибкое управление окружениями (Environments) | Огромное комьюнити и готовые плагины (Actions) |

---

## 2. Зачем в 2026 году использовать Jenkins?

Кажется, что Jenkins устарел, но его выбирают в трех случаях:

1. **Сложная логика (Groovy):** Если твой пайплайн — это не просто "собери и кати", а сложный алгоритм с ветвлениями, циклами и зависимостями от 10 внешних систем, написать это на Groovy (в Jenkins) проще, чем городить сотни строк в YAML.
2. **Legacy и On-premise:** Огромные корпорации с 10-летней историей сидят на Jenkins. Переписывать тысячи пайплайнов на GitLab CI — это годы работы.
3. **Плагины:** У Jenkins есть плагин для *всего*. Вообще для всего, что было создано в IT за последние 20 лет.

---

## 3. Пример GitLab CI Пайплайна

В GitLab файл называется `.gitlab-ci.yml`. Давай напишем стандартный пайплайн для сборки Docker-образа и деплоя в K8s.

```yaml
# Список стадий (очередность выполнения)
stages:
  - build
  - test
  - deploy

# Переменные (можно задавать в интерфейсе GitLab для безопасности)
variables:
  IMAGE_NAME: my-registry.com/my-app
  IMAGE_TAG: $CI_COMMIT_SHORT_SHA

# Сборка образа
build_job:
  stage: build
  image: docker:24.0.5
  services:
    - docker:24.0.5-dind # Docker-in-Docker
  script:
    - docker login -u $REGISTRY_USER -p $REGISTRY_PASS my-registry.com
    - docker build -t $IMAGE_NAME:$IMAGE_TAG .
    - docker push $IMAGE_NAME:$IMAGE_TAG

# Тестирование (запускается параллельно, если несколько раннеров)
test_job:
  stage: test
  script:
    - echo "Running unit tests..."
    - sleep 5 # Имитация тестов

# Деплой в Kubernetes (только при пуше в ветку main)
deploy_prod:
  stage: deploy
  image: bitnami/kubectl:latest
  script:
    - kubectl config use-context my-cluster-context
    - sed -i "s|__IMAGE__|${IMAGE_NAME}:${IMAGE_TAG}|g" k8s/deployment.yaml
    - kubectl apply -f k8s/
  only:
    - main
  environment:
    name: production

```

---

## 4. Как это работает: "под капотом"

* **Stages:** Если `build` упадет, `test` и `deploy` не запустятся.
* **Artifacts:** Если на шаге сборки ты создал файл (например, `.jar`), ты можешь передать его на следующий шаг с помощью `artifacts`.
* **Runners:** GitLab ищет доступный агент (Runner) с нужным тегом (например, `docker` или `linux`) и отправляет ему инструкции.

---

## 5. Что такое GitHub Actions (кратко)

В GitHub вместо `script` ты часто видишь `uses`:

```yaml
- name: Login to DockerHub
  uses: docker/login-action@v2  # Это готовый чужой код (Action) из Marketplace
  with:
    username: ${{ secrets.DOCKERHUB_USERNAME }}

```

Это делает GitHub Actions похожим на конструктор LEGO.

---

### Темы для Senior уровня:

* **Dynamic Pipelines:** Когда GitLab генерирует YAML-конфиг "на лету" в зависимости от того, какие файлы изменились.
* **Canvas/Directed Acyclic Graph (DAG):** Когда стадии не идут строго друг за другом, а образуют сложную сеть зависимостей (ключевое слово `needs`).

**Теперь мы точно готовы к Разделу VI: Observability (Prometheus, Grafana, ELK)? Или хочешь попробовать написать пайплайн для GitHub Actions для сравнения?**