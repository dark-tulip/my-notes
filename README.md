# PostgreSQL

### Немного про БД
- WAH - write ahead log. До начала транзакции операция записывается в журнал событий. Нужен для обеспечения ACID 
- MVCC - Multiversion Concurrency Control
- Update - это тяжелая операция, при которой вставляется новая запись и удаляется старая (в случае наличия индекса + происходит вставка в индекс)

### Vacuum
- VACUUM - нужен дл очистки пустых кортежей (или провести сборку мусора), удаление и сжатие пустых tuple-ов (пометить отрезки памяти которые можно перезаписать новыми данными)
- VACUUM FULL - полная перезапись и сжатие (компактинг таблицы, в основном только если удалили очень много ланных)
- Autovacuum - демон делающий все автоматически (отключать не рекомендуется)
- 1 - чистит индекса
- 2 - чистит таблицы


### Index
- Индекс - это структура данных
- Позволяет найти значение без полного перебора
- Индексы - нужны для ускорения поиска в таблице (`select`), после каждой вставки (`insert\update`) индексы перестраиваются
- Для каждой таблицы и индекса выделяется отдельный файл in `pgdata/`
- По Primary Key and Unique индекс составляется автоматически
- SELECT amname FROM pg_am; - какие типы индексов доступны на сервере

#### Show indexes in schema
```psql
SELECT *, pg_size_pretty(pg_relation_size(indexrelname::text))
FROM pg_stat_all_indexes
WHERE schemaname = 'public';
```
#### Index types
- B-tree сбалансированное дерево, по умолчанию
Balanced Tree (<, >, <=, >=, =) LIKE (abc% НО НЕ %abc), индексирует NULL
O(log(N))


- Хеш Индекс поддерживает только '='
Сложность поиска O(1)

- GiST - обобщенное дерево поиска - несбалансирован
- GIN - обобченный обратный индекс - generalized inverted index (for range types, сложные регулярные выражения)
- SP-GiST - (GiST с двоичным разбиением пространства)
- BTIN - блочный диапазонный индекс


#### Методы сканирования данных
- Index scan - индексное (оптимизатор включает поиск)
- Index only scan - (без обращения к таблицам, используя карту видимости)
- Bitmap scan - сканирование по битовой карте версий строк
- Sequential scan - последовательное сканирование

### Data storage structure
- Heap (куча записей) file - это файл таблицы, список записей (структурирован как набор страниц)
- Таблица состоит из массива страниц (pages, размером страницы в 8 килобайт по умолчанию)
- Таблица по умолчанию до 1 ГБ
- Страница содержит ссылки на строки (CTID)


FSM - free space map
VM - visibility map, хранит бит актуальности на страницу


### Explain 
Аналитический инструмент для просмотра плана выполнения запроса
```
EXPLAIN QUERY
```
Прогон запроса, показывает план и реальность
```
EXPLAIN ANALYZE QUERY
```

