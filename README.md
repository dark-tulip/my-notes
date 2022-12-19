# PostgreSQL
- MVCC - Multiversion Concurrency Control
- Update - это тяжелая операция, при которой вставляется новая запись и удаляется старая (в случае наличия индекса + происходит вставка в индекс)
- VACUUM - нужен дл очистки пустых кортежей (или провести сборку мусора), удаление и сжатие пустых tuple-ов
- 1 - чистит индекса
- 2 - чистит таблицы

Индексы - нужны для ускорения поиска в таблице (select), после каждой вставки (insert) индексы перестраиваются ()
