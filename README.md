# vbzdat


OK

```sql

-- Create a 1:1 example with auto and person

DROP SCHEMA IF EXISTS `ex01`;
-- In MySQL, the schema is the synonym for the database
-- DROP DATABASE [IF EXISTS] database_name;

CREATE SCHEMA `ex01` DEFAULT CHARACTER SET utf8 ;
USE `ex01`;

CREATE TABLE `auto` (
  `anr` int(11) NOT NULL,
  `marke` varchar(45) NOT NULL,
  `typ` varchar(45) DEFAULT NULL,
  `baujahr` int(11) DEFAULT NULL,
  PRIMARY KEY (`anr`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

