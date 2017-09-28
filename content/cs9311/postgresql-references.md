---
title: Postgresql References
date: 2017-09-13T20:33:49+10:00
tags:
  - sql
  - postgresql
  - database
  - comp9311
---

## Constants

Type               | Example           | Note
------------------ | ----------------- | -----------------------
string             | 'hello, world'    | 不能用双引号
string with escape | E'hello, world\n' | '\n'会被当作换行输出，跟C语言中字符串类似
multiline string   | $$abc$$           | $$中间可以有字符，比如$func$abc$func$
boolean            | true, false       |
numeric            | 12.34             | 没有单引号以及不是boolean的都被视为numeric

双引号作为escape关键词的手段要少用（即尽量不要用关键词作为identifier）（而且双引号可以指定有大写字母的identifier，正常来说SQL的identifier是大小写不分的）

例如

```sql
create table "table"();  -- 可行的但不建议
create table "Table"();  -- 可行的但不建议
```

其他类型

```sql
type 'string'
'string'::type
CAST ( 'string' AS type )

-- 例如
date '2015-01-01'
12.34::float
```

## data types

除注明外，以下类型均是SQL标准支持，在定义table时，尽量使用SQL标准的类型。

Name                            | Aliases        | Note
------------------------------- | -------------- | ---------------
integer                         | int, int4      |
smallint                        | int2           |
bigint                          | int8           |
boolean                         | bool           |
character (n)                   | char (n)       |
character varying (n)           | varchar (n)    |
double precision                | float, float8  |
real                            | float4         |
numeric (p, s)                  | decimal (p, s) | p：总位数，s：小数点右边位数
date                            |                |
time [ without time zone ]      |                |
time with time zone             | timetz         |
timestamp [ without time zone ] |                |
timestamp with time zone        | timestamptz    |
serial                          | serial4        | 自增整数，非标准SQL
smallserial                     | serial2        | 自增整数，非标准SQL
bigserial                       | serial8        | 自增整数，非标准SQL

## Date time functions

Function                      | Returns     | Example                                            | Result
----------------------------- | ----------- | -------------------------------------------------- | ------------------------------------
current_date                  | date        |                                                    |
current_time                  | timetz      |                                                    |
current_timestamp             | timestamptz |                                                    | 2017-09-13 13:32:03.234753+10
localtime                     | time        |                                                    |
localtimestamp                | timestamptz |                                                    | 2017-09-13 13:32:03.234753
now()                         | timestamptz |                                                    | 2017-09-13 13:32:03.234753+10
timeofday()                   | text        |                                                    | Wed Sep 13 13:31:49.227326 2017 AEST
date_part(text, timestamp)    | float       | date_part('hour', timestamp '2001-02-16 20:38:40') | 20
extract(field from timestamp) | float       | extract(hour from timestamp '2001-02-16 20:38:40') | 20

## Custom functions

```sql
create or replacce function
f(arg1 type1, arg2 type2) returns type
as $$
... function body ...
$$ language language [ mode ];
```

mode可能的值为

- immutable: 不访问数据库 (fast)
- stable: 不修改数据库
- volatile: 修改数据库 (slow, default)

在语言为sql的时候只能通过$1, $2等访问参数。语言为plpgsql的时候可以通过参数名访问，也可以通过$1, $2访问，最好用一个下划线前缀如`_name`当作参数名避免冲突。

plpgsql时函数体如

```sql
create or replacce function
f(arg1 type1, arg2 type2) returns text
as $$
declare
    r       record;
    out     text := '';
    curbeer text := '';
    tasters text;
    sum     int;
    count   int;
begin
    for r in
        select * from AllRatings order by beer,taster
    loop
        if (r.beer <> curbeer) then
            if (curbeer <> '') then
                out := out || BeerDisplay(curbeer,sum/count,tasters);
            end if;
            curbeer := r.beer;
            sum := 0; count := 0; tasters := '';
        end if;
        sum := sum + r.rating;
        count := count + 1;
        tasters := tasters || ', ' || r.taster;
    end loop;
    -- finish off the last beer
    out := out || beerDisplay(curbeer,sum/count,tasters);
    return out;
end;
$$ language plpgsql;
```

declare与begin后都不加分号，end可加可不加。另外每个赋值语句也要加分号。control structure（如for, if）在没有end之前那些关键词都不加分号。（但其中的赋值语句要）

### Return setof

```sql
returns setof text
returns setof table(x int, y int) -- temporary table as type
create type pair as (x int, y int);
returns setof pair
--
return setof record -- avoid that
-- you will need a column definition list every time you call the function
-- as follows
select * from func() as f(x int, y varchar(30));
```

returns setof的函数中return也有所不同

```
RETURN NEXT expression;
RETURN QUERY query;
RETURN QUERY EXECUTE command-string;
```

`RETURN NEXT`可用于for循环中，比如

```
for r in
    select * from beer
loop
    return next r;
end loop;
```

它只是每一次都把一行的结果加到return的set中。

`RETURN QUERY`后面跟一个正常的query，`RETURN QUERY EXECUTE`后面则跟的是query的字符串。

## Custom aggregations

postgresql处理aggregate是如下的伪代码

```
state = initial state
for each item in group {
    state = sfunc(state, item)
}
return finalfunc(AggState)
```

比如avg的伪代码可以写成这样

```
state = (0, 0)
for each value in group {
    state = (state.x + value, state.y + 1)
}
return state.x / state.y
```

具体在postgresql中是这样定义的

```sql
create aggregate concat_with_comma (text) (
    stype     = text,
    initcond  = '',
    sfunc     = appendNext,
    finalfunc = finalText
);
```

其中，concat_with_comma是aggregation的函数名（比如内置的min，max），括号中的text是参数类型。stype是state的类型，initcond是state的初始值。sfunc和finalfunc都对应上面伪代码的位置，并且都是postgresql中的function。

## Window functions

是可以在某列上计算aggregate function（如min，max，avg）但又不group by。如下。

```sql
select round(avg(rating) over (partition by taster), 2), *
from allratings;
```

```
 round | taster |          beer          |          brewer          | rating
-------+--------+------------------------+--------------------------+--------
  2.00 | Adam   | Old                    | Toohey's                 |      4
  2.00 | Adam   | Victoria Bitter        | Carlton and United       |      1
  2.00 | Adam   | New                    | Toohey's                 |      1
  3.67 | Geoff  | Redback                | Matilda Bay Brewing      |      4
  3.67 | Geoff  | James Squire Pilsener  | Maltshovel Brewery       |      4
  3.67 | Geoff  | Empire                 | Carlton and United       |      3
  3.50 | Hector | Pale Ale               | Sierra Nevada            |      4
  3.50 | Hector | Fosters                | Carlton and United       |      3
```

group by的结果是。而且注意group by是没法select没有被group by或者aggregate的列的，比如beer和brewer。

```
 round | taster
-------+--------
  2.00 | Adam
  3.67 | Geoff
  3.50 | Hector
```

## With queries

会生成temporary tables供with语句后的语句使用。比如

```sql
WITH regional_sales AS (
        SELECT region, SUM(amount) AS total_sales
        FROM orders
        GROUP BY region
     ), top_regions AS (
        SELECT region
        FROM regional_sales
        WHERE total_sales > (SELECT SUM(total_sales)/10 FROM regional_sales)
     )
SELECT region,
       product,
       SUM(quantity) AS product_units,
       SUM(amount) AS product_sales
FROM orders
WHERE region IN (SELECT region FROM top_regions)
GROUP BY region, product;
```

### Recursive queries

With语句另外一个非常有用的地方在于它能够递归地查询，例如

```sql
WITH RECURSIVE t(n) AS (
    VALUES (1)
  UNION ALL
    SELECT n+1 FROM t WHERE n < 100
)
SELECT sum(n) FROM t;
```

定义与view类似，t是表名，n是列名。

union all前的是初始条件，这里是n这列初始化为1的意思，当然也可以写成`select 1`。

union all后面是recursive的部分，需要引用到t（要不然就不递归了）。递归部分的表t都是上一次产生的结果。比如说第一次执行的是`VALUES(1)`，第二次执行`SELECT n+1 FROM t WHERE n < 100`的时候t里就有一个元素是1，这次的执行结果产生一个值2，这个值作为下一次表t的值。

### Select a tree

再看一个实际一点的例子，我们有一个表存了一棵多叉树。是以parent指针这种方式存的。这种存法的好处是表非常简单，插入修改什么的都很方便。但其中有个问题是怎样算出每个节点到根的深度，或者这个表里有多棵树的话，怎么把单独的一棵树用一个query就查出来。

假设我们有这样一棵树。

<img alt='binary tree' src='/img/tree.png' width=200>

在数据库它的表示是

```sql
create table tree_node (
    id int primary key,
    parent_id int references tree_node(id)
);
```

```
 id | parent_id
----+-----------
  1 |
  2 |         1
  3 |         1
  4 |         2
  5 |         2
  6 |         3
  7 |         3
```

那么，从根（即id为1的点）把整棵树都拎出来并且顺便算出每个节点的深度的SQL如下。

```sql
with recursive children(id, parent_id, depth) as (
    select 1, null::int, 0
union all
    select t.id, t.parent_id, c.depth + 1
    from children c
    join tree_node t on c.id = t.parent_id
)
select * from children;
```

它从根开始，每次查到下一层的所有节点，并把深度加1。最后，叶子结点没有children，所以用inner join后select会return空集，所以到叶子结点后就会停止。结果如下

```
 id | parent_id | depth
----+-----------+-------
  1 |           |     0
  2 |         1 |     1
  3 |         1 |     1
  4 |         2 |     2
  5 |         2 |     2
  6 |         3 |     2
  7 |         3 |     2
```

## Create Assertion

constraint需要依赖于某一特定的table，而且语法非常简单。

## Joins

```sql
CREATE TABLE weather (
    city        varchar(80),
    temp_lo        int,        -- low temperature
    temp_hi        int,        -- high temperature
    prcp        real,        -- precipitation
    date        date
);

CREATE TABLE cities (
    name        varchar(80),
    location    point
);
```

这里的命名就是一个不好的例子。

```
     city      | temp_lo | temp_hi | prcp |    date
---------------+---------+---------+------+------------
 San Francisco |      46 |      50 | 0.25 | 1994-11-27
 San Francisco |      43 |      57 |    0 | 1994-11-29
 Hayward       |      37 |      54 |      | 1994-11-29

     name      | location
---------------+-----------
 San Francisco | (-194,53)
 Houston       | (-183,25)
```

### Cross join

```sql
SELECT w.city as weather_city, w.temp_lo, c.name as city_name, c.location
FROM weather w, cities c;
```

```
 weather_city  | temp_lo |   city_name   | location
---------------+---------+---------------+-----------
 San Francisco |      46 | San Francisco | (-194,53)
 San Francisco |      46 | Houston       | (-183,25)
 San Francisco |      43 | San Francisco | (-194,53)
 San Francisco |      43 | Houston       | (-183,25)
 Hayward       |      37 | San Francisco | (-194,53)
 Hayward       |      37 | Houston       | (-183,25)
```

相当于笛卡尔乘积，表A中所有可能值组合表B中所有可能值，结果条数为N * M。不常用，即使加上where条件，速度慢。

### Inner join

```sql
SELECT w.city as weather_city, w.temp_lo, c.name as city_name, c.location
FROM weather w
JOIN cities c on w.city = c.name;
```

```
 weather_city  | temp_lo |   city_name   | location
---------------+---------+---------------+-----------
 San Francisco |      46 | San Francisco | (-194,53)
 San Francisco |      43 | San Francisco | (-194,53)
```

只会出现key（即join的那一列）值分别都存在于两表的行，比如San Francisco都在weather和cities两表中。而Hayward只在weather中，而不在cities中，所以join之后就不出现了。Houston同理。

### Left (outer) join

```sql
SELECT w.city as weather_city, w.temp_lo, c.name as city_name, c.location
FROM weather w
LEFT JOIN cities c on w.city = c.name;
```

```
 weather_city  | temp_lo |   city_name   | location
---------------+---------+---------------+-----------
 San Francisco |      46 | San Francisco | (-194,53)
 San Francisco |      43 | San Francisco | (-194,53)
 Hayward       |      37 |               |
```

出现在左边表的key值都会出现在结果中，如果它不在右边表中，结果表中对应右边表的列都为null。比如Hayward那一行。

### Right (outer) join

```
weather_city  | temp_lo |   city_name   | location
---------------+---------+---------------+-----------
San Francisco |      43 | San Francisco | (-194,53)
San Francisco |      46 | San Francisco | (-194,53)
              |         | Houston       | (-183,25)
```

### Full (outer) join

```
 weather_city  | temp_lo |   city_name   | location
---------------+---------+---------------+-----------
 San Francisco |      46 | San Francisco | (-194,53)
 San Francisco |      43 | San Francisco | (-194,53)
 Hayward       |      37 |               |
               |         | Houston       | (-183,25)
```

Inner join与Left join较常用，看具体需要。

## Transactions

是一系列语句的集合，它们要么同时被成功执行，要么都没有被执行（比如其中一句执行出错了，前面成功的结果也不会写到数据库中）。

设想一个转账的情景。A给B转100元，A账户里要扣100元，B账户里加100元。如果A账户扣成功了，但B账户收钱时发生错误了没有加上100元。我们不希望这样的情况发生。所以希望当中间发生错误了，A的100元也不会被扣掉。

在transaction中，已经执行的操作在commit前对其他transactions都是不可见的。事实上，在postgresql中，每一个单独的语句都被作为是一个transaction来执行。而多语句的transaction以以下的语法实现。

```sql
BEGIN;
UPDATE accounts SET balance = balance - 100.00
    WHERE name = 'Alice';
UPDATE branches SET balance = balance - 100.00
    WHERE name = (SELECT branch_name FROM accounts WHERE name = 'Alice');
UPDATE accounts SET balance = balance + 100.00
    WHERE name = 'Bob';
UPDATE branches SET balance = balance + 100.00
    WHERE name = (SELECT branch_name FROM accounts WHERE name = 'Bob');
COMMIT;
```

## ACID

### Atomic

原子性，被transaction这个机制实现的

### Consistency

一致性，数据与DDL描述的是一致的，constraints比如foreign key都是不能违反的。

### Isolation

隔离性，即在transaction中，已经执行的操作在commit前对其他transactions都是不可见的。

### Durability

持久性，即transaction完成后，所有的变化都应该被保存在持久的介质上（如硬盘，而不是内存）
