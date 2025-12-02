## 1. Вывести имена всех людей, которые есть в базе данных авиакомпаний

```
select name from Passenger
```
## 2. Вывести названия всеx авиакомпаний

```
SELECT name FROM Company
```
## 3. Вывести все рейсы, совершенные из Москвы

```
SELECT *
FROM Trip
WHERE town_from = "Moscow"
```

## 4. Вывести имена людей, которые заканчиваются на "man"
```
SELECT name
FROM Passenger
WHERE name LIKE '%man';
```

## 5. Вывести количество рейсов, совершенных на TU-134
```
SELECT COUNT(id) as count
FROM Trip
WHERE plane = 'TU-134'
```

## 6. Какие компании совершали перелеты на Boeing
```
SELECT DISTINCT name
FROM Company
	INNER JOIN Trip ON Trip.company = Company.id
WHERE plane = 'Boeing'
```

## 7. Вывести все названия самолётов, на которых можно улететь в Москву (Moscow)
```
SELECT DISTINCT plane
FROM Trip
WHERE town_to = 'Moscow';
```

## 8. В какие города можно улететь из Парижа (Paris) и сколько времени это займёт?
```
SELECT DISTINCT town_to,
	TIMEDIFF(time_in, time_out) as flight_time
FROM Trip
WHERE town_from = 'Paris'
```

## 9. Какие компании организуют перелеты из Владивостока (Vladivostok)?

```
SELECT name
FROM Company
	inner JOIN Trip on Trip.company = Company.id
WHERE Trip.town_from = 'Vladivostok'
```

## 10. Вывести вылеты, совершенные с 10 ч. по 14 ч. 1 января 1900 г.

```
SELECT *
FROM Trip
WHERE time_out >= '1900-01-01T10:00:00.000Z'
	and time_out <= '1900-01-01T14:00:00.000Z'
```

## 11. Выведите пассажиров с самым длинным ФИО. Пробелы, дефисы и точки считаются частью имени.

```
SELECT name
FROM Passenger
WHERE LENGTH(name) = (
		SELECT MAX(LENGTH(name))
		FROM Passenger
	)
```

## 12. Выведите идентификаторы всех рейсов и количество пассажиров на них. Обратите внимание, что на каких-то рейсах пассажиров может не быть. В этом случае выведите число "0".

```
SELECT t.id,
	COUNT(Pass_in_trip.passenger)
FROM Trip as t
	LEFT JOIN Pass_in_trip on Pass_in_trip.trip = t.id
GROUP BY t.id
```

## 13. Вывести имена людей, у которых есть полный тёзка среди пассажиров

```
SELECT name
From Passenger
GROUP BY name
HAVING COUNT(name) > 1
```

## 14. В какие города летал Bruce Willis

```
SELECT DISTINCT town_to
FROM Trip as t
	INNER JOIN Pass_in_trip as pt on t.id = pt.trip
	INNER JOIN Passenger as p ON pt.passenger = p.id
WHERE p.name = 'Bruce Willis'
```

## 15. Выведите идентификатор пассажира Стив Мартин (Steve Martin) и дату и время его прилёта в Лондон (London)

```
SELECT DISTINCT p.id,
	t.time_in
FROM Trip as t
	INNER JOIN Pass_in_trip as pt on t.id = pt.trip
	INNER JOIN Passenger as p ON pt.passenger = p.id
WHERE p.name = 'Steve Martin'
	AND t.town_to = 'London'
```

## 16. Вывести отсортированный по количеству перелетов (по убыванию) и имени (по возрастанию) список пассажиров, совершивших хотя бы 1 полет.

```
SELECT name,
	COUNT(pt.id) as count
FROM Passenger as p
	INNER JOIN Pass_in_trip as pt ON p.id = pt.passenger
GROUP BY p.name
ORDER BY count desc,
	p.name
```

## 17. Определить, сколько потратил в 2005 году каждый из членов семьи. В результирующей выборке не выводите тех членов семьи, которые ничего не потратили.

```
SELECT fm.member_name,
	fm.status,
	SUM(p.amount * p.unit_price) as costs
FROM FamilyMembers as fm
	INNER JOIN Payments as p ON p.family_member = fm.member_id
WHERE DATE(p.date) BETWEEN '2005-01-01' AND '2006-01-01'
GROUP BY fm.member_name,
	fm.status
```

## 18. Выведите имя самого старшего человека. Если таких несколько, то выведите их всех.

```
SELECT member_name
FROM FamilyMembers
WHERE birthday = (
		SELECT MIN(birthday)
		FROM FamilyMembers
	)
```

## 19. Определить, кто из членов семьи покупал картошку (potato)

```
SELECT DISTINCT status
from FamilyMembers
	inner JOIN Payments on Payments.family_member = FamilyMembers.member_id
WHERE Payments.good = (
		SELECT good_id
		FROM Goods
		WHERE good_name = 'potato'
	)
```

## 20. Сколько и кто из семьи потратил на развлечения (entertainment). Вывести статус в семье, имя, сумму

```
SELECT fm.status,
	fm.member_name,
	SUM(amount * unit_price) as costs
FROM FamilyMembers as fm
	INNER JOIN Payments as p on p.family_member = fm.member_id
	INNER JOIN Goods as g on g.good_id = p.good
	INNER JOIN GoodTypes as gt on gt.good_type_id = g.type
WHERE gt.good_type_name = 'entertainment'
GROUP BY fm.status,
	fm.member_name
```

## 21. Определить товары, которые покупали более 1 раза

```
SELECT good_name
FROM Goods as g
	INNER JOIN Payments as p ON p.good = g.good_id
GROUP BY good_name
HAVING COUNT(good_name) > 1
```

## 22. Найти имена всех матерей (mother)

```
SELECT member_name
FROM FamilyMembers
WHERE status = 'mother'
```

## 23. Найдите самый дорогой деликатес (delicacies) и выведите его цену

```
WITH new_table AS (
	SELECT *
	FROM GoodTypes
		INNER JOIN Goods on Goods.type = GoodTypes.good_type_id
		INNER JOIN Payments on Payments.good = Goods.good_id
)

SELECT good_name, unit_price
FROM new_table
WHERE good_type_name = 'delicacies'
	AND unit_price = (
		SELECT MAX(unit_price)
		FROM new_table
		WHERE good_type_name = 'delicacies'
	)
GROUP BY good_name,
	unit_price
```

## 24. Определить, кто и сколько потратил в июне 2005

```
SELECT member_name,
	SUM(amount * unit_price) as costs
FROM Payments
	INNER JOIN FamilyMembers ON member_id = family_member
WHERE date >= '2005-06-01T00:00:00.000Z'
	AND date < '2005-07-01T00:00:00.000Z'
GROUP BY member_name
```

## 25. Определить, какие товары не покупались в 2005 году

```
SELECT good_name
FROM Payments
	RIGHT JOIN Goods on good_id = good
WHERE good NOT IN (
		SELECT good
		FROM Payments
		WHERE EXTRACT(YEAR FROM date) = 2005
	)
	OR payment_id IS NULL;
```

## 26. Определить группы товаров, которые не приобретались в 2005 году

```
WITH new_table AS (SELECT * FROM Payments
RIGHT JOIN Goods on good_id = good
RIGHT JOIN GoodTypes ON good_type_id = type)

SELECT DISTINCT good_type_name FROM new_table
WHERE good_type_name NOT IN ( 
SELECT DISTINCT good_type_name FROM new_table
WHERE EXTRACT(YEAR FROM date) = 2005
)
GROUP BY good_type_name, date
```
