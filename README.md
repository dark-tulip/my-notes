# PostgreSQL
```sql
-- нужно для ускорения сортировки в памяти
SHOW work_mem;
SET work_mem = "16MB";

-- буфер write ahead lock - журнал пред записи запросов, нужен для ACID, после изменения сбрасываются на диск
SHOW wal_buffers;
SET wal_buffers = "8MB";


-- буфер postgresql нужен для кеширования (чтобы рабочий набор данных мог находится в кеше)
SHOW shared_buffers;
SET shared_buffers = "512MB";
```

### Немного про БД
- `WAL` - write ahead log. До начала транзакции операция записывается в журнал событий. Нужен для обеспечения ACID 
- `COMMIT` - фиксация данных, теперь видны всем пользователям
- В журналируемой таблице есть номер `lsn`, 
- В нежурналируемой `lsn` пустой `0/0` (автоматическая очистка данных после сбоя (TRUNCATE), при обычной остановке c командой stop - сохраняются), эти таблицы не реплицируются, так как нет событий в WAL файлах
- Временные таблицы - те же нежурналируемые таблицы, но удаляются после завершения сессии
- MVCC - Multiversion Concurrency Control
- `Update` - это тяжелая операция, при которой вставляется новая запись и удаляется старая (в случае наличия индекса + происходит вставка в индекс)

### Vacuum
- `VACUUM` - нужен для очистки пустых кортежей (или провести сборку мусора), удаление и сжатие пустых tuple-ов (пометить отрезки памяти которые можно перезаписать новыми данными)
- `VACUUM FULL` - полная перезапись и сжатие (компактинг таблицы, в основном только если удалили очень много ланных)
- `Autovacuum` - демон делающий все автоматически (отключать не рекомендуется)
- первое - чистит индексы
- второе - чистит таблицы


### Index
- Индекс - это структура данных (как оглавление в телефонном справочнике)
- Организованы в виде деревьев
- Позволяет найти значение без полного перебора
- Индексы - нужны для ускорения поиска в таблице (`select`), после каждой вставки (`insert\update`) индексы перестраиваются
- Для каждой таблицы и индекса выделяется отдельный файл in `pgdata/`
- По Primary Key` and `Unique` индекс составляется автоматически
- `SELECT amname FROM pg_am`; - какие типы индексов доступны на сервере

#### Show indexes in schema
```sql
-- postgreSQL
SELECT *, pg_size_pretty(pg_relation_size(indexrelname::text))
FROM pg_stat_all_indexes
WHERE schemaname = 'public';
```
#### Index types
- B-tree сбалансированное дерево, по умолчанию. Balanced Tree (<, >, <=, >=, =) LIKE (abc% НО НЕ %abc), индексирует NULL. O(log(N))
- Хеш Индекс поддерживает только '='. Сложность поиска O(1)
- GiST - обобщенное дерево поиска - несбалансирован
- GIN - обобченный обратный индекс - generalized inverted index (for range types, сложные регулярные выражения)
- SP-GiST - (GiST с двоичным разбиением пространства)
- BTIN - блочный диапазонный индекс

#### Методы сканирования данных
- `Index scan` - индексное (оптимизатор включает поиск, с заходом в основную таблицу за доп колонками)
- `Index only scan` - (без обращения к таблицам, используя карту видимости)
- `Bitmap scan` - сканирование по битовой карте версий строк
- `Sequential scan` - последовательное сканирование
- Nested loops - соединение вложенными циклами
- Hash join - соединение с помощью хэш таблицы (соответствия по ключам)
- Merge join - соединение заранее отсортированных наборов с помощью алгоритма слияния
<br>
Стоимость - `cost` (первым делом это планируемое время) - это кол-во времени для извлечения первой строки, вторая цифра - время на отдачу всех строк
### Data storage structure
- `Heap (куча записей) file` - это файл таблицы, список записей (структурирован как набор страниц)
- Таблица состоит из массива страниц (`pages`, размером страницы в 8 килобайт по умолчанию)
- Таблица по умолчанию до 1 ГБ
- Страница содержит ссылки на строки (CTID)
- FSM - free space map
- VM - visibility map, хранит бит актуальности на страницу


### Explain 
Аналитический инструмент для просмотра плана выполнения запроса
```
EXPLAIN QUERY
```
Прогон запроса, показывает план и реальность
```
EXPLAIN ANALYZE QUERY
```

