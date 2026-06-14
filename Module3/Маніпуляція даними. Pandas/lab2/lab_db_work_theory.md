# Pandas + Бази даних — Теорія

## 1. Підключення до БД через SQLAlchemy

SQLAlchemy — стандартний спосіб підключення Python до реляційних БД.

```python
from sqlalchemy import create_engine

# SQLite (файл або in-memory)
engine = create_engine('sqlite:///path/to/db.db')
engine = create_engine('sqlite:///:memory:')

# PostgreSQL
engine = create_engine('postgresql://user:pass@host:5432/dbname')

# MySQL
engine = create_engine('mysql+pymysql://user:pass@host:3306/dbname')

# MSSQL
engine = create_engine('mssql+pyodbc://user:pass@host:1433/dbname?driver=ODBC+Driver+18+for+SQL+Server')
```

## 2. Читання даних

| Метод | Опис | Приклад |
|-------|------|---------|
| `pd.read_sql(sql, engine)` | SQL-запит | `pd.read_sql('SELECT * FROM t', engine)` |
| `pd.read_sql_query(sql, engine)` | Те саме, явно | `pd.read_sql_query('SELECT * FROM t WHERE id > 5', engine)` |
| `pd.read_sql_table(table, engine)` | Вся таблиця | `pd.read_sql_table('employees', engine)` |

### Параметри:
- `params={'key': value}` — параметризовані запити (SQL injection protection)
- `index_col='col'` — колонка як індекс DataFrame
- `chunksize=N` — читання частинами (ітератор)
- `columns=['a', 'b']` — лише вказані колонки (для read_sql_table)

## 3. Запис даних

```python
df.to_sql('table_name', engine, if_exists='replace', index=False)
```

### `if_exists`:
- `'append'` — додати до існуючої таблиці
- `'replace'` — видалити і створити заново
- `'fail'` — помилка, якщо таблиця існує

### Корисні параметри:
- `chunksize=N` — писати частинами (для великих даних)
- `dtype={'col': sqlalchemy.types.VARCHAR(100)}` — явний тип даних
- `schema='schema_name'` — схема (для PostgreSQL)

## 4. Транзакції

```python
# Автоматичний commit/rollback
with engine.begin() as conn:
    conn.execute(text("INSERT INTO t VALUES (1, 'a')"))
    conn.execute(text("INSERT INTO t VALUES (2, 'b')"))

# Ручне керування
conn = engine.connect()
trans = conn.begin()
try:
    conn.execute(text("DELETE FROM t WHERE id = 1"))
    trans.commit()
except:
    trans.rollback()
finally:
    conn.close()
```

## 5. Інспекція БД

```python
from sqlalchemy import inspect

inspector = inspect(engine)
tables = inspector.get_table_names()
columns = inspector.get_columns('table_name')
```

## 6. Оптимізація

- **Агрегацію робіть на стороні БД** (GROUP BY в SQL, а не groupby в Pandas)
- **Читайте тільки потрібні колонки** (SELECT col1, col2)
- **Використовуйте chunksize** для великих таблиць
- **Параметризуйте запити** — безпека + кешування плану виконання

## 7. Типовий ETL-патерн

```
Source DB → pd.read_sql() → Transform (Pandas) → df.to_sql() → Target DB
```

## 8. Необхідні пакети

| БД | Пакет | URI |
|----|-------|-----|
| SQLite | вбудовано | `sqlite:///file.db` |
| PostgreSQL | `psycopg2-binary` | `postgresql://user:pass@host/db` |
| MySQL | `pymysql` | `mysql+pymysql://user:pass@host/db` |
| MSSQL | `pyodbc` | `mssql+pyodbc://user:pass@host/db` |
