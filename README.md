# clickhouse-vs-reindexer

[трансляция](https://youtube.com/live/krXPKdiVums)

Задача: замерить Reindexer по условиям из первой заметки, т.к. во второй заметке ClickHouse показал очень плохие результаты.

1. [Делаем быстрый поиск по турам на основе ClickHouse](https://habr.com/ru/articles/324846/)
2. [Как мы выбирали между Elastic и Tarantool, а сделали свою (самую быструю) in-memory БД. С Join и полнотекстовым поиском](https://habr.com/ru/articles/346884/)

## как поднять ClickHouse в docker?

```bash
$ docker run -d --name some-clickhouse-server --ulimit nofile=262144:262144 clickhouse/clickhouse-server
```

Команда docker run -d используется для запуска контейнера в фоновом режиме. Здесь -d означает “detached”, что в переводе с английского означает “отсоединенный”.

```bash
$ docker run --name some-clickhouse-server -it --rm --ulimit nofile=262144:262144 -p 8123:8123 -p 9000:9000 clickhouse/clickhouse-server
```

--rm - удаляет конейнер после остановки, -it : -t выводит в tty(консоль) вывод, i- позволяет взаимодействовать с контейнером через консоль

или

```bash
$ cd ch-and-grafana && docker-compose up -d
```

## примеры применения ClickHouse

https://github.com/ClickHouse/examples

## что быстрее? clickhouse interface (formally native interface) vs database/sql interface

While the database/sql provides a database-agnostic interface, allowing developers to abstract their data store, it enforces some typing and query semantics that impact performance. For this reason, the client-specific API should be used where performance is important. However, users who wish to integrate ClickHouse into tooling, which supports multiple databases, may prefer to use the standard interface.

Both interfaces encode data using the native format and native protocol for communication. Additionally, the standard interface supports communication over HTTP.

## чем отличается ClickHouse от других баз данных?

ClickHouse отличается от других систем управления базами данных (СУБД) по нескольким ключевым параметрам:

1. **Архитектура**: ClickHouse использует колоночно-ориентированную архитектуру, которая спроектирована для улучшения производительности запросов и сокращения требований к хранению. Это контрастирует с MySQL, PostgreSQL и MongoDB, которые все являются строково-ориентированными.

2. **Производительность запросов**: ClickHouse оптимизирован для аналитических запросов, что делает его идеальным для компаний, которым нужно быстро обрабатывать большие наборы данных. MySQL и PostgreSQL - это более универсальные базы данных, которые могут обрабатывать как транзакционные, так и аналитические рабочие нагрузки, но они могут не работать так же хорошо, как ClickHouse, для аналитических запросов.

3. **Масштабируемость**: ClickHouse разработан для горизонтального масштабирования, позволяя компаниям распределять свои данные по нескольким серверам для улучшения производительности и емкости.

4. **Типы данных**: ClickHouse поддерживает широкий спектр типов данных, включая примитивные типы, массивы и сложные структуры.

5. **Интеграция с другими технологиями**: ClickHouse хорошо интегрируется с другими популярными технологиями и инструментами, включая Apache Kafka, Apache Spark и Grafana.

6. **Открытый исходный код и сообщество**: ClickHouse - это проект с открытым исходным кодом с растущим сообществом разработчиков и пользователей.

## чем отличается колоночно-ориентированная база данных от строково-ориентированной?

Колоночно-ориентированные и строково-ориентированные базы данных отличаются способом организации и хранения данных:

1. **Строково-ориентированные базы данных**: В этих базах данных данные хранятся построчно, так что все столбцы одной строки хранятся вместе. Это обеспечивает отличную производительность при запросе нескольких столбцов одной строки, что типично для приложений онлайн-транзакционной обработки (OLTP). Примеры строково-ориентированных баз данных: Postgres, MySQL.

2. **Колоночно-ориентированные базы данных**: В этих базах данных данные организуются по полям, причем все данные, связанные с одним полем, хранятся рядом друг с другом в памяти. Это обеспечивает преимущества в производительности при выполнении запросов к данным. Они оптимизированы для эффективного чтения и вычисления столбцов. Примеры колоночно-ориентированных баз данных: Redshift, BigQuery, Snowflake.

Важно отметить, что выбор между колоночно-ориентированной и строково-ориентированной базой данных в значительной степени зависит от конкретных требований к приложению. Строковые базы данных обычно лучше подходят для транзакционных систем, где часто требуется доступ ко всем полям записи. Колоночные базы данных лучше подходят для аналитических систем, где часто выполняются агрегатные запросы к большому числу записей, но только к небольшому числу полей.

## Reindexer

The core is written in C++ and the application level API is in Go.

Reindexer - встраиваемая документо-ориентированная СУБД, которая работает с in-memory базами данных и опционально может использовать дисковое хранилище. Основная отличительная особенность reindexer — быстрый поиск по сложным запросам.

![](./assets/reindexer-benchmarks)

## pv

```bash
$ cat tours.csv | pv | clickhouse --client --query 'INSERT INTO tours FORMAT CSVWithNames'
524MiB 0:00:04 [ 123MiB/s]
```

Если у вас ещё не уставлена утилита pv — рекомендую поставить, она позволяет печатать прогресс исполнения операций и показывает, сколько байт прошло через пайп.

## Выводы

Сравнение тёплого с мягким. Reindexer идеально применим для каталога товаров, ClickHouse - для аналитики и выборки туров для путешествий.

## Продолжение про ClickHouse

- Опыт работы с Clickhouse и умение строить MaterializedView, понимание принципа работы движка MergeTree

- вторичные индексы в ClickHouse?

- В ClickHouse нельзя модифицировать данные?

Да, в ClickHouse нет встроенной поддержки операций UPDATE или DELETE, как в традиционных реляционных базах данных1. Это означает, что вы не можете напрямую изменить существующие данные в таблице. Однако, есть способы обойти это ограничение.

Например, вы можете использовать механизмы TTL (Time to Live), чтобы автоматически удалять старые данные. Кроме того, вы можете создать новую таблицу с измененными данными, а затем переименовать ее, чтобы заменить старую таблицу.

Также стоит отметить, что ClickHouse предоставляет функцию ALTER TABLE UPDATE, которая позволяет изменять данные в таблице. Однако эта функция работает путем создания новой таблицы с измененными данными и замены старой таблицы на новую.

Важно понимать, что эти обходные пути могут быть неэффективными для больших объемов данных и могут потребовать значительного объема дискового пространства.

Кроме того, ClickHouse использует свою собственную модель данных с подходом CRDT для обновления данных. Это позволяет ClickHouse эффективно обрабатывать большие объемы данных и обеспечивать высокую производительность.

- ClickHouse Keeper заменил ZooKeeper (который на Java)

- Есть hash-join по ключу, join по условию пока нет?

Вы можете выполнить JOIN по ключу, но не можете выполнить JOIN по условию, такому как BETWEEN. Однако, есть обходные пути для выполнения операций JOIN с условием BETWEEN. Например, вы можете создать подзапрос с JOIN и затем применить условие BETWEEN во внешнем запросе. Но это не то же самое, что непосредственное выполнение JOIN по условию BETWEEN.

"Словари" - эффективный способ реализации hash-join. В качестве источника для этих словарей можно указать какую угодно базу: ODBC, MySQL, текстовый файл, HTTP и executable источники.

- Visual Interfaces from Third-party Developers

  - [Make Your Company Data Driven. Connect to any data source, easily visualize, dashboard and share your data.](https://github.com/getredash/redash)
  - [polyglot, lightweight high-performance observability stack with ClickHouse storage, designed to ingest and analyze logs, metrics and traces](https://github.com/metrico/qryn)
  - [Open source APM: OpenTelemetry traces, metrics, and logs](https://github.com/uptrace/uptrace)

- `SELECT ... FORMAT TabSeparatedWithNames`

```go
package main

import (
	"encoding/csv"
	"fmt"
	"strings"
)

func main() {
	data := `column1	column2	column3
value1	value2	value3
value4	value5	value6`

	reader := csv.NewReader(strings.NewReader(data))
	reader.Comma = '\t' // Установите разделитель на табуляцию

	records, err := reader.ReadAll()
	if err != nil {
		fmt.Println(err)
		return
	}

	for i, record := range records {
		if i == 0 {
			fmt.Println("Header:", record)
		} else {
			fmt.Println("Record:", record)
		}
	}
}
```

или с применением [`dataframe-go`](https://pkg.go.dev/github.com/rocketlaunchr/dataframe-go) (аналог `pandas`)

- Узнаём, как всё устроено

`SHOW CREATE TABLE visits_table`

```sql
  CREATE TABLE visits_table
    (UserID UInt64, ...) ENGINE =
    MergeTree(EventDate,
    intHash32(UserID),
    (EventDate, StartURLDomain, intHash32(UserID)),
    8192)
```

Q: как узнать, что сэмплирование идёт по идентификаторам пользователя?

A: В вашем примере создания таблицы `visits_table` в ClickHouse, ключ сэмплирования определяется вторым аргументом функции `MergeTree`. В данном случае, ключ сэмплирования - это `intHash32(UserID)`, что означает, что сэмплирование будет происходить по идентификаторам пользователя. Функция `intHash32` применяется к столбцу `UserID`, что означает, что идентификаторы пользователей будут хешироваться перед использованием в качестве ключа сэмплирования. Это может быть полезно для обеспечения равномерного распределения данных при сэмплировании. Таким образом, если вы выполняете запрос с использованием оператора `SAMPLE`, ClickHouse будет выбирать подмножество данных на основе хеш-значений идентификаторов пользователей.

- "Слишком" большие данные, тогда `SAMPLE` & `OFFSET`

```sql
  SELECT
    count()*10 as visits,
    sum(PageViews) as hits,
    uniq(UserID) as users,
    URL as url
  FROM visits_table
  SAMPLE 1/10
  GROUP BY url
```

```sql
  SELECT
    count() as visits,
    sum(PageViews) as hits,
    uniq(UserID) as users,
    URL as url
  FROM visits_table
  SAMPLE 1/10 OFFSET {i}/10
  GROUP BY url
```

- А если данные не сэмплированы, то можно самому

```sql
  SELECT
    count() as visits,
    sum(PageViews) as hits,
    uniq(UserID) as users,
    URL as url
  FROM visits_table
  WHERE intHash32(UserID) % 10 = i
  GROUP BY url
```

- Временная таблица

```bash
$ head -n 3 user_ids
1234
4321
3333

$ clickhouse_client --query="SELECT count() FROM visits_table WHERE UserID in users" --external --file=user_ids --name=users --type=UInt64
```

- Модификаторы функции -If, -Array

```sql
  SELECT
    count() as visits,
    sum(PageViews) as hits,
    countIf(IsMobile) as mobile_visits,
    sumIf(PageViews, IsMobile) as mobile_hits,
    mobile_visits/visits as mobile_visits_share,
    mobile_hits/hits as mobile_hits_share
  FROM visits_table
  FORMAT TabSeparatedWithNames
```

```sql
  SELECT
    sumArray(ProductsPrices) as revenue
  FROM visits_table
  FORMAT TabSeparatedWithNames
```

- Расшифровываем IDs - словари

`dictGetString('my_dict', 'description', id)`

- Разработка в консольном клиенте

```bash
$ clickhouse_client --multiline
```

- Системные таблицы

  - `system.query_log` - логи пользователей
  - `system.functions` - список имплементированных функций
  - `system.settings` - текущие настройки

- argMin, argMax

Стандарный SQL:

```sql
  SELECT PurchaseId FROM visits_table
  WHERE DateTime = (SELECT min(DateTime) FROM visits_table)

```

ClickHouse:

```sql
  SELECT argMin(PurchaseId, DateTime) FROM visits_table
```

(вернуть PurchaseId при минимальном времени DateTime)

- Сила массивов

arrayJoin, groupArray, arrayMap, arrayFilter

выручка от проданных книг V1

```sql
  SELECT
    sumArray(
      arrayFilter(
        price, name -> LIKE %book%,
        PruductsPrices, ProductsNames))
  FROM visits_table

```

выручка от проданных книг V2

```sql
  SELECT sum(price)
  FROM visits_table
  ARRAY JOIN
    ProductsNames as name,
    PruductsPrices as price
  WHERE name LIKE %book%
```
