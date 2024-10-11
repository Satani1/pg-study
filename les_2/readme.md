# ДЗ-2

## Решение:

Создаем схему и таблицу для тестирования

```sql
create schema if not exists t_mvcc;

create table if not exists t_mvcc.goods_stock
(
    good_id      bigint,
    warehouse_id bigint,
    amount       bigint
);
```

Наполняем данными таблицу

```sql

insert into t_mvcc.goods_stock(id, warehouse_id, amount)
values (1, 10, 250),
       (2, 20, 300),
       (3, 10, 50),
       (4, 20, 0);
```

Проверяем уровень изоляции.

```sql
show transaction isolation level;
```

![Screenshot 2024-10-11 at 18.34.10.png](..%2F..%2F..%2F..%2F..%2Fvar%2Ffolders%2Fb6%2Fth0bz2317cx_3r9688j88wvw0000gn%2FT%2FTemporaryItems%2FNSIRD_screencaptureui_7rmJJv%2FScreenshot%202024-10-11%20at%2018.34.10.png)

Одновременно в двух консолях начинаем транзакцию.
В первой вставляем запись

```sql
-- Первая консоль
begin;
insert into t_mvcc.goods_stock(good_id, warehouse_id, amount)
values (5, 20, 1000);
```

Во второй консоли делаем выборку из таблицы.

```sql
-- Вторая консоль
begin;
select *
from t_mvcc.goods_stock;
```

Ожидаем только 4 записи, так как на уровне изоляции `read commited` можем читать из таблиц только зафиксированные
данные.  
Из-за того, что первая транзакция еще не закоммитилась, получим 4 записи:

![Screenshot 2024-10-11 at 18.37.50.png](Screenshot%202024-10-11%20at%2018.37.50.png)


Завершим первую транзакцию коммитом и после этого, сделав повторную выборку всех данных, получим 5 строк:
![Screenshot 2024-10-11 at 18.43.26.png](Screenshot%202024-10-11%20at%2018.43.26.png)

Меняем уровень изоляции на `repeatable read` и проверяем

```sql
set default_transaction_isolation = 'repeatable read';
show transaction isolation level;
```

Пробуем снова через 2 консоли начать одновременные транзакции:
В первой вставляем запись

```sql
-- Первая консоль
begin;
insert into t_mvcc.goods_stock(good_id, warehouse_id, amount)
values (6, 30, 5);
```

Во второй консоли делаем выборку из таблицы.

```sql
-- Вторая консоль
begin;
select *
from t_mvcc.goods_stock;
```

Во второй транзакции не видно новой записи, тк данный уровень изоляции запрещает фантомные чтения.

То есть мы видим состояние таблицы на момент начала второй транзакции. Строка уже есть, но не зафиксирована.

![Screenshot 2024-10-11 at 18.49.40.png](Screenshot%202024-10-11%20at%2018.49.40.png)

Если завершить первую транзакцию, но не завершить вторую. То при повторном запросе select во второй транзакции, будем
получать данные на момент старта транзакци ---> все те же 5 строк

Новую строку можно будет увидеть, только если начать новую транзакцию.


