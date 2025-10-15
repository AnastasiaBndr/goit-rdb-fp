## Завантаження даних

```
CREATE SCHEMA IF NOT EXISTS `pandemic`;
USE `pandemic`;

CREATE TABLE IF NOT EXISTS `pandemic`.`countries` (
  `id` INT NOT NULL AUTO_INCREMENT,
  `Entity` VARCHAR(60) NULL,
  `Code` VARCHAR(10) NULL,
  PRIMARY KEY (`id`));
  
  SELECT COUNT(*) FROM infectious_cases;

```
  
## Нормалізація таблиць

Вставляємо у countries відповідні значення із таблиці infectious_cases

```

  INSERT INTO countries (Entity,`Code`)
  SELECT DISTINCT Entity,`Code`
  FROM infectious_cases;

```
  
Змінюємо назву таблиці щоб тепер працювати у ній. Додаємо новий стовпчик
```
ALTER TABLE infectious_cases
RENAME TO infections;
```
```
ALTER TABLE infections 
ADD COLUMN country_id INT;
```
Робимо join та вставляємо id країн які співпадають з entity та code

```
SET SQL_SAFE_UPDATES = 0;

UPDATE infections i
JOIN countries c ON i.Entity = c.Entity AND i.Code = c.Code
SET i.country_id = c.id;

SET SQL_SAFE_UPDATES = 1;
```

Додаємо foreign_key

```
ALTER TABLE infections
ADD CONSTRAINT fk_country
FOREIGN KEY (country_id)
REFERENCES countries(id);
```

Видаляємо ствопці entity та code

```
ALTER TABLE infections
DROP COLUMN entity,
DROP COLUMN `code`;
```

## Аналіз даних

```
SELECT country_id, MIN(Number_rabies) as min , 
AVG(Number_rabies) as `avg`, 
MAX(Number_rabies) as max
from infections
WHERE Number_rabies !=''
GROUP BY country_id
ORDER BY `avg` DESC
LIMIT 10;
```

## Побудова колонки різниці в роках

```
SELECT (MAKEDATE(`Year`,1)) as `January 1`, 
curdate() as `current date`, 
TIMESTAMPDIFF(YEAR,MAKEDATE(`Year`,1), curdate()) as difference 
from infections;
```

## Власна функція

```
DELIMITER //

CREATE FUNCTION YearsDifference(input_year TEXT)
RETURNS TEXT
DETERMINISTIC 
NO SQL
BEGIN
    DECLARE result TEXT;
    SET result = TIMESTAMPDIFF(YEAR,MAKEDATE(input_year,1), curdate());
    RETURN result;
END //

DELIMITER ;
```

Використання:

```
SELECT YearsDifference(`Year`) as `Years difference` from infections;
```