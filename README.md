# vbzdat Migration/Visualisierung Prüfung Datenbank Sandro Sacher

## Aufgabe 6
Erstellen Sie ein Entity-Relation-Ship Diagramm (z.B. mit DBeaver)

![ERD](https://github.com/J3ossy/vbzdat/blob/master/Bookmarks/Assets/Aufgabe%206.PNG)

## Aufgabe 7
Für Ihre Linie und einen bestimmten Fahrweg und Datum:
Abgabe: Abfrage (SQL) und Abfrageergebnis

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

### Ergebnis:

![Bild Aufgabe 7](https://github.com/J3ossy/vbzdat/blob/master/Bookmarks/Assets/Aufgabe%207.PNG)


## Aufgabe 8
Erstellen Sie aus der Linientabelle eine Abfrage mit allen Varianten Ihrer Linie
Abgabe: Abfrage (SQL) und Abfrageergebnis.

```sql
SELECT
    l.fahrweg_id,
    l.linie,
    l.richtung,
    l.fw_no,
    l.fw_lang
FROM
    vbzdat.linie l
WHERE
    l.linie = 4;
```

### Ergebnis:

![Bild Aufgabe 8](https://github.com/J3ossy/vbzdat/blob/master/Bookmarks/Assets/Aufgabe%208.PNG)


## Aufgabe 9
Erstellen Sie aus der Ankunftszeitentabelle eine Abfrage für Ihre Linie und einen bestimmten Fahrweg und Datum Linie.
Abgabe: Abfrage (SQL) und Abfrageergebnis

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

### Ergebnis:

![Bild Aufgabe 9](https://github.com/J3ossy/vbzdat/blob/master/Bookmarks/Assets/Aufgabe%209.PNG)


## Aufgabe 10 
Abfrage für Ihre Linie mit den Verspätungen gemäss Aufgabestellung.
Abgabe: Abfrage (SQL) und Abfrageergebnis

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
### Ergebnis:
 
 ![Bild Aufgabe 10](https://github.com/J3ossy/vbzdat/blob/master/Bookmarks/Assets/Aufgabe%2010.PNG)
 
 Zusützliche Visualisierung:
 
 ![Bild Aufgabe 10](https://github.com/J3ossy/vbzdat/blob/master/Bookmarks/Assets/Aufgabe%2010_1.PNG)
 
 ## Aufgabe 11
 PrintScreen des Export Assistenten, CSV-Datei-Inhalt und Print-Screen der Karte.
 
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
### Export:
 
 ![Bild Aufgabe 11](https://github.com/J3ossy/vbzdat/blob/master/Bookmarks/Assets/Aufgabe%2011_1.PNG)
 ![Bild Aufgabe 11](https://github.com/J3ossy/vbzdat/blob/master/Bookmarks/Assets/Aufgabe%2011_2.PNG)
 ![Bild Aufgabe 11](https://github.com/J3ossy/vbzdat/blob/master/Bookmarks/Assets/Aufgabe%2011_3.PNG)
 ![Bild Aufgabe 11](https://github.com/J3ossy/vbzdat/blob/master/Bookmarks/Assets/Aufgabe%2011_4.PNG)
  
 ### CSV Dateiinhalt 
  
 ![Bild Aufgabe 11](https://github.com/J3ossy/vbzdat/blob/master/Bookmarks/Assets/Aufgabe%2011_csv.PNG)
  
### Visualisierung 
 
 ![Bild Aufgabe 11](https://github.com/J3ossy/vbzdat/blob/master/Bookmarks/Assets/Aufgabe%2011_5.PNG)
 
## Aufgabe 12
Pivot Abfrage für Fahrplan	einer	Linie.
Abgabe: Abfrage (SQL) und Abfrageergebnis


```sql  
 SELECT
	h2.halt_lang,
	(cast(a.datumzeit_soll_an as time)) as zeit_soll_an
FROM 
	vbzdat.ankunftszeiten a 
inner join vbzdat.haltepunkt h on 
	h.halt_punkt_id =a.haltepunkt_id
inner join vbzdat.haltestelle h2 on 
	h.halt_id = h2.halt_id 
WHERE 
		cast(a.datumzeit_soll_an as DATE) = "2020-01-02"
 	and a.fahrt_id in (20001)
 ORDER BY a.fahrt_id, cast(a.datumzeit_soll_an as time);
   ```
   
 ### Ergebnis:
 
![Bild Aufgabe 12](https://github.com/J3ossy/vbzdat/blob/master/Bookmarks/Assets/Aufgabe%2012.PNG)

## Aufgabe 13
Nächste Station Abgabe:
Abfrage (SQL) und Abfrageergebnis 

```sql  
 SELECT halt_punkt_id,
        h.GPS_Latitude,
        h.GPS_Longitude,      
        p.distance_unit
                 * DEGREES(ACOS(LEAST(1.0, COS(RADIANS(p.latpoint))
                 * COS(RADIANS(h.GPS_Latitude))
                 * COS(RADIANS(p.longpoint) - RADIANS(h.GPS_Longitude))
                 + SIN(RADIANS(p.latpoint))
                 * SIN(RADIANS(h.GPS_Latitude))))) AS distance_in_km
  FROM haltepunkt  AS h
  JOIN (
        SELECT  47.3699111 AS latpoint,  8.5259085 AS longpoint,
                50.0 AS radius,      111.045 AS distance_unit
    ) AS p ON 1=1
  WHERE GPS_Latitude
     BETWEEN p.latpoint  - (p.radius / p.distance_unit)
         AND p.latpoint  + (p.radius / p.distance_unit)
    AND GPS_Longitude 
     BETWEEN p.longpoint - (p.radius / (p.distance_unit * COS(RADIANS(p.latpoint))))
         AND p.longpoint + (p.radius / (p.distance_unit * COS(RADIANS(p.latpoint))))
  ORDER BY distance_in_km
  LIMIT 4;
 ```
 
 ### Abfrageergebnis
 
 ![Bild Aufgabe 13](https://github.com/J3ossy/vbzdat/blob/master/Bookmarks/Assets/Aufgabe%2013_1.PNG)
 
 
 ### Visualisierung der 4 nächsten Stationen
 

 ![Bild Aufgabe 13](https://github.com/J3ossy/vbzdat/blob/master/Bookmarks/Assets/Aufgabe%2013.PNG)
 
 
## Aufgabe 14
Distanz zwischen zwei Haltestellen:	
Abfrage (SQL) und Abfrageergebnis

### Referenzpunkt Bahnhofquai/HB

```sql 
SELECT 
  name, 
   ( 6371 * acos( cos( radians(47.37738) ) * cos( radians( a.lat ) ) 
   * cos( radians(a.lng) - radians(8.541672)) + sin(radians(47.37738)) 
   * sin( radians(a.lat)))) AS distance 
FROM Aufgabe_14 a 
ORDER BY distance;
 ```
 
 ### Abfrageergebnis
 
 ![Bild Aufgabe 13](https://github.com/J3ossy/vbzdat/blob/master/Bookmarks/Assets/Aufgabe%2014.PNG)
