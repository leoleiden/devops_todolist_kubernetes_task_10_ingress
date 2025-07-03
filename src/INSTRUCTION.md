# Django ToDo List - Kubernetes Deployment Instructions

## Передумови

Перед початком переконайтеся, що у вас встановлені:
- Docker
- kind (Kubernetes in Docker)
- kubectl
- Git

## Крок 1: Клонування репозиторію

```bash
git clone <your-forked-repository>
cd <repository-name>
```

## Крок 2: Створення Kubernetes кластера

Використовуйте наданий конфігураційний файл для створення кластера:

```bash
kind create cluster --config=cluster.yml
```

Перевірте, що кластер створений успішно:

```bash
kubectl cluster-info
kubectl get nodes
```

## Крок 3: Розгортання застосунку

Запустіть bootstrap скрипт для розгортання всіх необхідних компонентів:

```bash
chmod +x bootstrap.sh
./bootstrap.sh
```

Цей скрипт розгорне:
- MySQL базу даних з усіма необхідними ресурсами
- ToDo застосунок з конфігураціями
- Ingress контролер NGINX

## Крок 4: Створення Ingress маніфесту

Створіть директорію для Ingress файлів:

```bash
mkdir -p .infrastructure/ingress
```

Створіть файл `.infrastructure/ingress/ingress.yml` з наступним вмістом:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: todo-app-ingress
  namespace: app
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /$1
spec:
  ingressClassName: nginx
  rules:
  - host: localhost
    http:
      paths:
      - path: /(.*)
        pathType: Prefix
        backend:
          service:
            name: todo-app-service
            port:
              number: 8080
```

## Крок 5: Застосування Ingress

```bash
kubectl apply -f .infrastructure/ingress/ingress.yml
```

## Валідація змін

### 1. Перевірка статусу подів

```bash
kubectl get pods -A
```

Всі поди повинні мати статус `Running`.

### 2. Перевірка сервісів

```bash
kubectl get svc -A
```

### 3. Перевірка Ingress

```bash
kubectl get ingress -n app
kubectl describe ingress todo-app-ingress -n app
```

### 4. Перевірка доступності застосунку

Відкрийте браузер та перейдіть на:
- `http://localhost`

Ви повинні побачити головну сторінку ToDo застосунку.

### 5. Тестування функціональності

1. **Головна сторінка**: `http://localhost/` - повинна завантажуватися без помилок
2. **API endpoint**: `http://localhost/api/` - повинен показувати API інтерфейс
3. **Реєстрація**: `http://localhost/auth/register/` - сторінка реєстрації
4. **Вхід**: `http://localhost/auth/login/` - сторінка входу

### 6. Перевірка консолі браузера

Відкрийте Developer Tools (F12) і перевірте:
- Немає помилок 404 у вкладці Network
- Всі статичні файли (CSS, JS) завантажуються успішно
- Немає помилок у консолі

## Очистка ресурсів

Для видалення кластера після тестування:

```bash
kind delete cluster
```

## Troubleshooting

### Якщо поди не запускаються:

```bash
kubectl describe pod <pod-name> -n <namespace>
kubectl logs <pod-name> -n <namespace>
```

### Якщо Ingress не працює:

```bash
kubectl describe ingress todo-app-ingress -n app
kubectl logs -n ingress-nginx -l app.kubernetes.io/name=ingress-nginx
```

### Якщо застосунок недоступний:

1. Перевірте, чи працює Ingress контролер:
   ```bash
   kubectl get pods -n ingress-nginx
   ```

2. Перевірте налаштування сервісу:
   ```bash
   kubectl get svc -n app
   ```

3. Перевірте логи застосунку:
   ```bash
   kubectl logs -n app -l app=todo-app
   ```

## Структура проекту

```
├── cluster.yml                    # Конфігурація Kind кластера
├── bootstrap.sh                   # Скрипт для розгортання
├── requirements.txt               # Python залежності
├── manage.py                      # Django management скрипт
├── settings.py                    # Налаштування Django
├── .infrastructure/
│   ├── mysql/                     # MySQL конфігурації
│   ├── app/                       # Застосунок конфігурації
│   └── ingress/
│       └── ingress.yml            # Ingress конфігурація
└── INSTRUCTION.md                 # Цей файл
```

## Перевірка успішності

Завдання вважається виконаним успішно, якщо:
- ✅ Кластер Kind створений з конфігурації cluster.yml
- ✅ Bootstrap скрипт виконаний без помилок
- ✅ Ingress файл створений в правильній директорії
- ✅ Ingress має 1 HTTP правило з path-based маршрутизацією
- ✅ Застосунок доступний на http://localhost
- ✅ Немає помилок 404 у консолі браузера
- ✅ Всі сторінки застосунку працюють коректно
