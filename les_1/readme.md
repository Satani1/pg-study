# ДЗ-1

## Задание:
Посчитать количество поездок - `select count(*) from book.tickets;`

## Решение:
За основу взята миграция:

```
wget https://storage.googleapis.com/thaibus/thai_small.tar.gz && tar -xf thai_small.tar.gz && psql < thai.sql
```

Количество поездок = **5185505**

![Screenshot 2024-10-11 at 15.49.23.png](Screenshot%202024-10-11%20at%2015.49.23.png)