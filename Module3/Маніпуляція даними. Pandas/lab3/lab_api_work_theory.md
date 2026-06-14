# Pandas + API — Теорія

## 1. HTTP-запити

```python
import requests

# GET — отримати дані
response = requests.get('https://api.example.com/data')

# POST — створити
response = requests.post('https://api.example.com/data', json={'key': 'value'})

# PUT — оновити
response = requests.put('https://api.example.com/data/1', json={'key': 'new_value'})

# DELETE — видалити
response = requests.delete('https://api.example.com/data/1')
```

### Важливі атрибути response:
- `response.status_code` — HTTP статус (200=OK, 201=Created, 404=Not Found, 429=Rate Limited)
- `response.json()` — JSON -> Python dict/list
- `response.headers` — заголовки (Content-Type, Retry-After)
- `response.raise_for_status()` — викинути помилку при 4xx/5xx

## 2. JSON -> DataFrame

```python
# Простий JSON (список об'єктів)
df = pd.DataFrame(response.json())

# Вкладений JSON (розгорнути крапкою)
df = pd.json_normalize(response.json())

# Розгортання масиву всередині
df = pd.json_normalize(
    data,
    record_path='items',          # поле з масивом
    meta=['order_id', 'customer'] # поля на рівні вище
)
```

## 3. Пагінація

Більшість API повертають дані сторінками.

| Тип | Параметри | Приклад |
|-----|-----------|---------|
| Page-based | `?page=1&per_page=100` | JSONPlaceholder |
| Offset-based | `?offset=0&limit=100` | Zendesk |
| Cursor-based | `?cursor=abc123` | GitHub, Twitter |
| Link header | `Link: <url>; rel="next"` | GitHub API |

```python
# Page-based пагінація
all_data = []
for page in range(1, max_pages + 1):
    params = {'page': page, 'per_page': 100}
    response = requests.get(url, params=params)
    data = response.json()
    if not data:
        break
    all_data.extend(data)
```

## 4. Авторизація

```python
# Bearer Token
headers = {'Authorization': 'Bearer YOUR_TOKEN'}
response = requests.get(url, headers=headers)

# API Key
params = {'api_key': 'YOUR_KEY'}
response = requests.get(url, params=params)

# Basic Auth
from requests.auth import HTTPBasicAuth
response = requests.get(url, auth=HTTPBasicAuth('user', 'pass'))
```

## 5. Обробка помилок

```python
def safe_request(url, max_retries=3):
    for attempt in range(1, max_retries + 1):
        try:
            response = requests.get(url, timeout=10)
            if response.status_code == 429:  # Rate Limited
                time.sleep(2 ** attempt)
                continue
            response.raise_for_status()
            return response
        except requests.exceptions.Timeout:
            time.sleep(2 ** attempt)
    raise Exception(f'Failed after {max_retries} attempts')
```

## 6. Асинхронні запити (aiohttp)

```python
import aiohttp
import asyncio

async def fetch_all(urls):
    async with aiohttp.ClientSession() as session:
        tasks = [session.get(url) for url in urls]
        responses = await asyncio.gather(*tasks)
        return [await r.json() for r in responses]
```

## 7. API для практики (без авторизації)

| API | Опис | Ендпоінт |
|-----|------|----------|
| JSONPlaceholder | Тестові дані | `jsonplaceholder.typicode.com` |
| GitHub API | Репозиторії, issues | `api.github.com` |
| Open Library | Книги | `openlibrary.org/search.json` |
| CoinDesk | Bitcoin курс | `api.coindesk.com/v1/bpi` |
| Nationalize | Національність за ім'ям | `api.nationalize.io` |
| Random User | Генерація користувачів | `randomuser.me/api` |
| NASA API | Космічні знімки | `api.nasa.gov` (key: DEMO_KEY) |

## 8. ETL-патерн: API -> DWH

```
Extract:   requests.get(url) -> JSON
Transform: pd.json_normalize() -> cleaning -> enrichment
Load:      df.to_sql() / df.to_parquet() / df.to_csv()
```

## 9. Необхідні пакети

```bash
pip install requests pandas pyarrow aiohttp
```
