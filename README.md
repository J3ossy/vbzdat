# vbzdat

Aufgabe 6

![ERD](https://github.com/J3ossy/vbzdat/blob/master/Bookmarks/Assets/Aufgabe%206.PNG)

Aufgabe 7


```sql

SELECT     
  fsi.linie,     
  fsi.richtung,     
  fsi.fahrzeug,     
  fsi.kurs,     
  fsi.seq_von,     
  fsi.halt_id_von,     
  fsi.halt_id_nach,     
  fsi.halt_punkt_id_von,    
  fsi.halt_punkt_id_nach,     
  fsi.fahrt_id,     
  fsi.fahrweg_id,     
  fsi.fw_no,     
  fsi.fw_typ,     
  fsi.fw_kurz,     
  fsi.fw_lang,    
  fsi.betriebs_datum,    
  fsi.datumzeit_soll_an_von,     
  fsi.datumzeit_ist_an_von,     
  fsi.datumzeit_soll_ab_von,     
  fsi.datumzeit_ist_ab_von,     
  fsi.datum__nach,     
  TIMEDIFF (datumzeit_soll_an_von, datumzeit_ist_an_von) as timediff_an,    
  TIMESTAMPDIFF (SECOND, datumzeit_soll_an_von, datumzeit_ist_an_von) as timediff_an_seconds,   
  TIMEDIFF (datumzeit_soll_ab_von, datumzeit_ist_ab_von) as timediff_ab,   
  TIMESTAMPDIFF (SECOND, datumzeit_soll_ab_von, datumzeit_ist_ab_von) as timediff_ab_seconds,   
  TIMESTAMPDIFF (SECOND, datumzeit_soll_an_von, datumzeit_soll_ab_von) as halt_soll_time_seconds,   
  TIMESTAMPDIFF (SECOND, datumzeit_ist_an_von, datumzeit_ist_ab_von) as halt_ist_time_seconds 
FROM    
  fahrzeiten_soll_ist fsi 
WHERE linie = '4' AND betriebs_datum  = '2020-01-01' AND fahrweg_id = '99548'AND fahrt_id = '20001' 
ORDER By seq_von 
LIMIT 40000;
```

![Bild Aufgabe 7](https://github.com/J3ossy/vbzdat/blob/master/Bookmarks/Assets/Aufgabe%207.PNG)


Aufgabe 8
```sql
SELECT * FROM linie l WHERE linie = '4'
```

![Bild Aufgabe 8](https://github.com/J3ossy/vbzdat/blob/master/Bookmarks/Assets/Aufgabe%208.PNG)


Aufgabe 9

```sql

ALTER TABLE fahrzeiten_soll_ist ADD datumzeit_soll_nach_von DATETIME NULL;
ALTER TABLE fahrzeiten_soll_ist ADD datumzeit_ist_an_nach DATETIME NULL;
ALTER TABLE fahrzeiten_soll_ist ADD datumzeit_soll_ab_nach DATETIME NULL;
ALTER TABLE fahrzeiten_soll_ist ADD datumzeit_ist_ab_nach DATETIME NULL;

UPDATE fahrzeiten_soll_ist SET datumzeit_soll_an_nach = DATE_ADD(STR_TO_DATE(datum_nach,'%d.%m.%Y'), INTERVAL soll_an_nach SECOND);
UPDATE fahrzeiten_soll_ist SET datumzeit_ist_an_nach = DATE_ADD(STR_TO_DATE(datum_nach,'%d.%m.%Y'), INTERVAL ist_an_nach SECOND);
UPDATE fahrzeiten_soll_ist SET datumzeit_soll_ab_nach = DATE_ADD(STR_TO_DATE(datum_nach,'%d.%m.%Y'), INTERVAL soll_ab_nach SECOND);
UPDATE fahrzeiten_soll_ist SET datumzeit_ist_ab_nach = DATE_ADD(STR_TO_DATE(datum_nach,'%d.%m.%Y'), INTERVAL ist_ab_nach SECOND);


CREATE TABLE ankunftszeiten

SELECT 
	fsi.halt_punkt_id_nach AS haltepunkt_id,
	fsi.fahrweg_id,
	fsi.fahrt_id,
	fsi.datumzeit_ist_an_nach AS datumzeit_ist_an,
	fsi.datumzeit_soll_an_nach AS datumzeit_soll_an,
	fsi.datumzeit_soll_ab_nach AS datumzeit_soll_ab,
	TIMESTAMPDIFF (SECOND, datumzeit_soll_an_nach, datumzeit_ist_an_nach) as delay
FROM
	vbzdat.fahrzeiten_soll_ist fsi
WHERE
fsi.linie = 4

UNION 

SELECT 
	fsi.halt_punkt_id_von AS haltepunkt_id,
	fsi.fahrweg_id,
	fsi.fahrt_id,
	fsi.datumzeit_ist_an_von AS datumzeit_ist_an,
	fsi.datumzeit_soll_an_von AS datumzeit_soll_an,
	fsi.datumzeit_soll_ab_von AS datumzeit_soll_ab,
	TIMESTAMPDIFF (SECOND, datumzeit_soll_an_von, datumzeit_ist_an_von) as delay
FROM 
	vbzdat.fahrzeiten_soll_ist fsi
WHERE
	fsi.seq_von = 1
	AND fsi.linie = 4;

ALTER TABLE ankunftszeiten ADD id INT PRIMARY KEY AUTO_INCREMENT FIRST;
```


![Bild Aufgabe 9](https://github.com/J3ossy/vbzdat/blob/master/Bookmarks/Assets/Aufgabe%209.PNG)


Aufgabe 10 


```sql
SELECT DISTINCT 
    a.id,
    a.haltepunkt_id,
    h2.halt_lang,
    h.GPS_Latitude,
    h.GPS_Longitude,
    a.fahrt_id,
    l.linie,
    a.datumzeit_ist_an,
    a.datumzeit_soll_an,
    a.delay
FROM
    vbzdat.ankunftszeiten a
INNER JOIN vbzdat.haltepunkt h ON
    a.haltepunkt_id = h.halt_punkt_id
INNER JOIN vbzdat.haltestelle h2 ON
    h.halt_id = h2.halt_id
INNER JOIN vbzdat.linie l ON
    a.fahrweg_id = l.fahrweg_id
 ORDER BY 
 	delay DESC 
 LIMIT 20;
 
 ```
 Aufgabe 10
 
 ![Bild Aufgabe 10](https://github.com/J3ossy/vbzdat/blob/master/Bookmarks/Assets/Aufgabe%2010.PNG)
 
 Zusützliche Visualisierung:
 
 ![Bild Aufgabe 10](https://github.com/J3ossy/vbzdat/blob/master/Bookmarks/Assets/Aufgabe%2010_1.PNG)
 
 Aufgabe 11
 
```sql 
 SELECT DISTINCT 
    h.GPS_Latitude as lat,
    h.GPS_Longitude as lng,
    h2.halt_lang as name,
    null as color,
    null as note
FROM
    vbzdat.linie l
INNER JOIN vbzdat.ankunftszeiten a ON
    l.fahrweg_id = a.fahrweg_id
INNER JOIN vbzdat.haltepunkt h ON
    a.haltepunkt_id = h.halt_punkt_id
INNER JOIN vbzdat.haltestelle h2 ON
    h.halt_id = h2.halt_id
WHERE
    l.fw_no = 1;
  ```
 Export
 
  ![Bild Aufgabe 11](https://github.com/J3ossy/vbzdat/blob/master/Bookmarks/Assets/Aufgabe%2011_1.PNG)
  ![Bild Aufgabe 11](https://github.com/J3ossy/vbzdat/blob/master/Bookmarks/Assets/Aufgabe%2011_2.PNG)
  ![Bild Aufgabe 11](https://github.com/J3ossy/vbzdat/blob/master/Bookmarks/Assets/Aufgabe%2011_3.PNG)
  ![Bild Aufgabe 11](https://github.com/J3ossy/vbzdat/blob/master/Bookmarks/Assets/Aufgabe%2011_4.PNG)
  
  Visualisierung 
  ![Bild Aufgabe 11](https://github.com/J3ossy/vbzdat/blob/master/Bookmarks/Assets/Aufgabe%2011_5.PNG)
 