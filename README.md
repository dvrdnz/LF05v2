# LF05v2 
### Begleitmaterial zum Lernfeld LF05v2 – Software zur Verwaltung von Daten anpassen
###  Einstieg in MySQL Workbench

Dieses Begleitmaterial führt euch praxisnah in den Umgang mit **MySQL Workbench** ein – dem grafischen Werkzeug zur Verwaltung von MySQL-Datenbanken. Anhand eines realen Datensatzes deutscher Städte lernt ihr, wie man eine relationale Datenbank anlegt, externe Daten im CSV-Format importiert und die gespeicherten Informationen mit SQL-Abfragen auswertet. Im zweiten Teil wird gezeigt, wie sich zwei Tabellen per `JOIN` miteinander verknüpfen lassen, um komplexere Fragen an den Datenbestand zu stellen. Abschließend werden fortgeschrittene Abfragen vorgestellt, die SQL mit geografischer Distanzberechnung kombinieren. Ziel ist es, ein grundlegendes Verständnis dafür zu entwickeln, wie Datenbanksoftware zur strukturierten Verwaltung und Auswertung von Daten eingesetzt wird. Die Motivation für dieses Projekt entstand aus der Begeisterung für Mathematik – besonders die fortgeschrittenen Abfragen am Ende zeigen, wie sich mathematische Konzepte wie Trigonometrie und die Haversine-Formel direkt in SQL anwenden lassen.

## Voraussetzungen

* MySQL Server 8.0 und MySQL Workbench CE müssen installiert sein
* Die Dateien **[german_cities.csv](https://raw.githubusercontent.com/dvrdnz/LF05v2/main/german_cities.csv)** und **[lebendgeborene_2023.csv](https://raw.githubusercontent.com/dvrdnz/LF05v2/main/lebendgeborene_2023.csv)**  aus dem Ordner `datasets/` dieses Repositories..

---

## Vorbereitung

> 1. Kopiere die Datei `datasets/german_cities.csv` in das folgende Verzeichnis:

```plain
C:\ProgramData\MySQL\MySQL Server 8.0\Uploads
```

> 2. Starte MySQL Workbench CE.

---

## Datenbank anlegen

> 3. Gib diesen Befehl ein, um eine Datenbank zu erstellen:

```sql
CREATE DATABASE deutsche_staedte
```

> 4. Wähle im Navigator-Fenster „Schemas" und klicke auf das Aktualisieren-Symbol.

![Navigator-Fenster mit Schemas](https://github.com/dvrdnz/LF05v2/blob/main/image-1.png?raw=true)

**Nun sollte die angelegte Datenbank sichtbar sein.**

---

## Datenbank befüllen

> 5. Um mit der angelegten Datenbank arbeiten zu können, muss sie wie folgt zur Nutzung ausgewählt werden:

```sql
USE deutsche_staedte
```

> 6. Nun kann eine Tabelle angelegt werden. Entsprechend der bereitgestellten `.csv`-Datei werden auch die Datentypen der Spalten direkt definiert:

```sql
CREATE TABLE german_cities (
    id INT PRIMARY KEY,
    name VARCHAR(100) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci,
    district VARCHAR(100) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci,
    state VARCHAR(100) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci,
    population INT,
    lat FLOAT,
    lon FLOAT,
    area FLOAT
) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

> Das Attribut `CHARACTER SET` ermöglicht mit der Kennung `utf8mb4` die saubere Darstellung von Umlauten, während `COLLATE` eine robuste Sortierfunktionalität gewährleistet.

> 7. Anschließend kann auf die bereitgestellte `.csv`-Datei verwiesen werden, um deren Inhalt in die Tabelle einzuspeisen:

```sql
LOAD DATA INFILE 'C:/ProgramData/MySQL/MySQL Server 8.0/Uploads/german_cities.csv'
INTO TABLE german_cities
FIELDS TERMINATED BY ',' 
ENCLOSED BY '"' 
LINES TERMINATED BY '\n'
IGNORE 1 ROWS
(id, name, district, state, population, @lat, @lon, @area)
SET lat = NULLIF(@lat, ''),
    lon = NULLIF(@lon, ''),
    area = NULLIF(@area, '');
```

### Die Datenbank ist nun bereit.

---

### Beispielabfrage

```sql
-- Wählt die Top 10 der bevölkerungsreichsten Städte in Nordrhein-Westfalen aus
SELECT name, district, population FROM german_cities
WHERE state = 'Nordrhein-Westfalen'
ORDER BY population DESC
LIMIT 10;
```

#### Syntaxerklärung

| SQL-Teil | Bedeutung |
| :--- | :--- |
| `SELECT name, district, population` | Wählt die anzuzeigenden Spalten aus. |
| `FROM german_cities` | Gibt die Zieltabelle an. |
| `WHERE state = 'Nordrhein-Westfalen'` | Filtert die Ergebnisse auf das Bundesland Nordrhein-Westfalen. |
| `ORDER BY population DESC` | Sortiert die Ergebnisse nach der Spalte `population` absteigend (**DESCending**) – die größten Werte (höchste Bevölkerung) stehen oben. |
| `LIMIT 10` | Begrenzt die Ausgabe auf die ersten 10 Zeilen. |

---

## Zweite Tabelle anlegen

Für die JOIN-Abfragen wird eine zweite Tabelle benötigt:

> 8. Kopiere die Datei `datasets/lebendgeborene_2023.csv` aus diesem Repository in das folgende Verzeichnis:

```plain
C:\ProgramData\MySQL\MySQL Server 8.0\Uploads
```

> 9. Gib diesen Befehl ein:

```sql
CREATE TABLE lebendgeborene_2023 (
    ags_code VARCHAR(8) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci NOT NULL PRIMARY KEY,
    region_name VARCHAR(100) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci,
    clean_name VARCHAR(100) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci,
    region_typ VARCHAR(50) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci,
    lebendgeborene VARCHAR(10)
) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

**Nach dem Aktualisieren sollte die neue Tabelle im Navigator sichtbar sein. Diese muss nun mit Daten befüllt werden.**

> 10. Führe dafür folgende Befehlsfolge aus:

```sql
LOAD DATA INFILE 'C:/ProgramData/MySQL/MySQL Server 8.0/Uploads/lebendgeborene_2023.csv'
INTO TABLE lebendgeborene_2023
CHARACTER SET utf8mb4
FIELDS TERMINATED BY ',' 
OPTIONALLY ENCLOSED BY '"'
LINES TERMINATED BY '\n'
IGNORE 1 ROWS
(ags_code, region_name, clean_name, region_typ, lebendgeborene);
```

### Beide Tabellen sind nun bereit – die JOIN-Abfragen können beginnen.

---

# Der JOIN-Befehl

Mit `JOIN` verbindet man Daten aus zwei oder mehr Tabellen in einer Datenbank, um sie in einem einzigen Ergebnisdatensatz abzufragen und anzuzeigen.

### Die drei JOIN-Typen im Überblick

Je nach gewünschtem Ergebnis gibt es verschiedene JOIN-Varianten. Der Unterschied liegt darin, welche Zeilen im Ergebnis erhalten bleiben, wenn es in einer Tabelle **keinen passenden Eintrag** in der anderen gibt:

| JOIN-Typ | Verhalten |
| :--- | :--- |
| **INNER JOIN** | Gibt nur Zeilen zurück, die in **beiden** Tabellen einen übereinstimmenden Wert haben. Städte ohne Geburtendaten würden z.B. nicht erscheinen. |
| **LEFT JOIN** | Gibt **alle Zeilen der linken Tabelle** zurück, auch wenn kein passender Eintrag in der rechten Tabelle existiert. Fehlende Werte werden als `NULL` dargestellt. |
| **RIGHT JOIN** | Gibt **alle Zeilen der rechten Tabelle** zurück – das Spiegelbild des LEFT JOIN. In der Praxis wird stattdessen oft einfach die Tabellenreihenfolge getauscht und ein LEFT JOIN verwendet. |

#### Beispiel mit Phantasiedaten

Um den Unterschied zu veranschaulichen, betrachte folgende zwei Mini-Tabellen:

**Tabelle A: `german_cities` (Auszug)**
| name | population |
| :--- | :--- |
| Berlin | 3645000 |
| München | 1488000 |
| Phantasiastadt | 50000 |

**Tabelle B: `lebendgeborene_2023` (Auszug)**
| clean_name | lebendgeborene |
| :--- | :--- |
| Berlin | 32000 |
| München | 15000 |
| Köln | 18000 |

Je nach JOIN-Typ fällt das Ergebnis unterschiedlich aus:

**INNER JOIN** – nur Zeilen, die in *beiden* Tabellen vorkommen:
| name | lebendgeborene |
| :--- | :--- |
| Berlin | 32000 |
| München | 15000 |

**LEFT JOIN** – alle Städte aus `german_cities`, fehlende Geburtendaten als `NULL`:
| name | lebendgeborene |
| :--- | :--- |
| Berlin | 32000 |
| München | 15000 |
| Phantasiastadt | NULL |

**RIGHT JOIN** – alle Einträge aus `lebendgeborene_2023`, fehlende Städte als `NULL`:
| name | lebendgeborene |
| :--- | :--- |
| Berlin | 32000 |
| München | 15000 |
| NULL | 18000 |

---

### Beispielabfrage mit INNER JOIN

```sql
-- Geburten in kreisfreien Städten
SELECT g.name, g.state, g.population, l.lebendgeborene
FROM german_cities g
INNER JOIN lebendgeborene_2023 l ON g.name = l.clean_name
WHERE l.region_typ = 'kreisfreie Stadt'
ORDER BY l.lebendgeborene DESC
LIMIT 10;
```

#### Syntaxerklärung

| SQL-Teil | Bedeutung |
| :--- | :--- |
| `SELECT g.name, g.state, g.population, l.lebendgeborene` | Wählt die Spalten `name`, `state`, `population` aus `german_cities` und `lebendgeborene` aus `lebendgeborene_2023` für die Anzeige aus. |
| `FROM german_cities g` | Gibt die erste Zieltabelle `german_cities` an; `g` ist der Alias für kürzere Schreibweise. |
| `INNER JOIN lebendgeborene_2023 l ON g.name = l.clean_name` | Verknüpft `german_cities` mit `lebendgeborene_2023` (Alias `l`). Nur Zeilen, bei denen `g.name` mit `l.clean_name` übereinstimmt, werden berücksichtigt. |
| `WHERE l.region_typ = 'kreisfreie Stadt'` | Filtert auf Einträge, bei denen der Regionstyp `kreisfreie Stadt` ist. |
| `ORDER BY l.lebendgeborene DESC` | Sortiert absteigend nach der Anzahl der Geburten – Städte mit den meisten Geburten stehen oben. |
| `LIMIT 10` | Begrenzt die Ausgabe auf die ersten 10 Zeilen. |

---

### Beispielabfrage mit INNER JOIN: Geburtenrate

```sql
-- Geburtenrate pro 1.000 Einwohner
SELECT g.name, g.state, g.population, l.lebendgeborene,
       ROUND((l.lebendgeborene / g.population) * 1000, 2) AS geburten_pro_1000
FROM german_cities g
INNER JOIN lebendgeborene_2023 l ON g.name = l.clean_name
WHERE l.region_typ = 'kreisfreie Stadt' AND l.lebendgeborene IS NOT NULL
ORDER BY geburten_pro_1000 DESC
LIMIT 10;
```

#### Syntaxerklärung

| SQL-Teil | Bedeutung |
| :--- | :--- |
| `ROUND((l.lebendgeborene / g.population) * 1000, 2) AS geburten_pro_1000` | Berechnet die Geburtenrate pro 1.000 Einwohner und rundet das Ergebnis auf zwei Dezimalstellen. Das Ergebnis wird als neue Spalte `geburten_pro_1000` ausgegeben. |
| `INNER JOIN lebendgeborene_2023 l ON g.name = l.clean_name` | Verknüpft beide Tabellen; nur Städte, die in beiden Tabellen vorhanden sind, erscheinen im Ergebnis. |
| `WHERE l.region_typ = 'kreisfreie Stadt'` | Filtert auf kreisfreie Städte. |
| `AND l.lebendgeborene IS NOT NULL` | Schließt Einträge ohne Geburtendaten aus, damit die Division nicht fehlschlägt. |
| `ORDER BY geburten_pro_1000 DESC` | Sortiert absteigend nach der berechneten Geburtenrate. |
| `LIMIT 10` | Begrenzt die Ausgabe auf die ersten 10 Zeilen. |

---

### Beispielabfrage mit LEFT JOIN

```sql
-- Top 10 kreisfreie Städte in Bayern nach Geburtenrate
SELECT g.name, g.state, g.population, l.lebendgeborene,
       ROUND((l.lebendgeborene / g.population) * 1000, 2) AS geburten_pro_1000
FROM german_cities g
LEFT JOIN lebendgeborene_2023 l ON g.name = l.clean_name
WHERE g.state = 'Bayern' AND l.region_typ = 'kreisfreie Stadt' AND l.lebendgeborene IS NOT NULL
ORDER BY geburten_pro_1000 DESC
LIMIT 10;
```

#### Syntaxerklärung

| SQL-Teil | Bedeutung |
| :--- | :--- |
| `SELECT g.name, g.state, g.population, l.lebendgeborene, ROUND(...) AS geburten_pro_1000` | Wählt die Spalten `name`, `state`, `population` aus `german_cities`, `lebendgeborene` aus `lebendgeborene_2023` und berechnet die Geburtenrate pro 1.000 Einwohner als `geburten_pro_1000`. |
| `FROM german_cities g` | Gibt die linke Zieltabelle `german_cities` an, mit dem Alias `g`. |
| `LEFT JOIN lebendgeborene_2023 l ON g.name = l.clean_name` | Alle Städte aus `german_cities` bleiben erhalten, auch wenn keine Geburtendaten vorliegen (`l.lebendgeborene` wäre dann `NULL`). Die Verknüpfung erfolgt über `g.name = l.clean_name`. |
| `WHERE g.state = 'Bayern' AND l.region_typ = 'kreisfreie Stadt' AND l.lebendgeborene IS NOT NULL` | Filtert auf Bayern und kreisfreie Städte. **Hinweis:** Die Bedingung `IS NOT NULL` schließt Städte ohne Geburtendaten aus – damit verhält sich diese Abfrage in der Praxis wie ein INNER JOIN. Der LEFT JOIN wäre erst dann relevant, wenn man das `IS NOT NULL` weglässt und stattdessen auch Städte ohne Daten anzeigen möchte. |
| `ORDER BY geburten_pro_1000 DESC` | Sortiert absteigend nach der Geburtenrate. |
| `LIMIT 10` | Begrenzt die Ausgabe auf die Top 10. |

---

### Beispielabfrage mit RIGHT JOIN

```sql
-- Top 10 kreisfreie Städte in Hessen nach Geburtenrate
SELECT g.name, g.state, g.population, l.lebendgeborene,
       ROUND((l.lebendgeborene / g.population) * 1000, 2) AS geburten_pro_1000
FROM lebendgeborene_2023 l
RIGHT JOIN german_cities g ON l.clean_name = g.name
WHERE g.state = 'Hessen' AND l.region_typ = 'kreisfreie Stadt' AND l.lebendgeborene IS NOT NULL
ORDER BY geburten_pro_1000 DESC
LIMIT 10;
```

#### Syntaxerklärung

| SQL-Teil | Bedeutung |
| :--- | :--- |
| `FROM lebendgeborene_2023 l` | Gibt die linke Ausgangstabelle `lebendgeborene_2023` an, mit dem Alias `l`. |
| `RIGHT JOIN german_cities g ON l.clean_name = g.name` | Alle Städte aus `german_cities` (rechte Tabelle) bleiben erhalten, auch wenn keine Geburtendaten vorliegen. Die Verknüpfung erfolgt über `l.clean_name = g.name`. |
| `WHERE g.state = 'Hessen' AND l.region_typ = 'kreisfreie Stadt' AND l.lebendgeborene IS NOT NULL` | Filtert auf Hessen, kreisfreie Städte und vorhandene Geburtendaten. |
| `ORDER BY geburten_pro_1000 DESC` | Sortiert absteigend nach der Geburtenrate. |
| `LIMIT 10` | Begrenzt die Ausgabe auf die Top 10. |

---

## Fortgeschrittene Beispielabfrage: Geografischer Radius

Kombiniert man JOIN mit der sogenannten **Haversine-Formel**, lassen sich auch geografische Fragen beantworten – zum Beispiel: „Welche kreisfreien Städte in Bayern liegen im Umkreis von 100 km um München, und wie hoch ist dort die Geburtenrate?" Dafür werden die in `german_cities` gespeicherten Koordinaten (`lat`, `lon`) direkt in der SQL-Abfrage zur Distanzberechnung verwendet.

```sql
-- Geburtenrate in kreisfreien Städten Bayerns im Umkreis von 100 km um München
SELECT g.name, g.state, g.population, l.lebendgeborene,
       ROUND((l.lebendgeborene / g.population) * 1000, 2) AS geburten_pro_1000,
       ROUND(
           6371 * acos(
               cos(radians(48.137154)) * cos(radians(g.lat)) * 
               cos(radians(g.lon) - radians(11.576124)) + 
               sin(radians(48.137154)) * sin(radians(g.lat))
           ), 2
       ) AS distance_km
FROM german_cities g
LEFT JOIN lebendgeborene_2023 l ON g.name = l.clean_name
WHERE g.state = 'Bayern' 
  AND l.region_typ = 'kreisfreie Stadt' 
  AND l.lebendgeborene IS NOT NULL
  AND (
      6371 * acos(
          cos(radians(48.137154)) * cos(radians(g.lat)) * 
          cos(radians(g.lon) - radians(11.576124)) + 
          sin(radians(48.137154)) * sin(radians(g.lat))
      )
  ) < 100
ORDER BY geburten_pro_1000 DESC
LIMIT 10;
```

### Syntaxerklärung: Haversine-Formel für die Distanzberechnung

Die Haversine-Formel wird zur Berechnung der **kürzesten Entfernung zwischen zwei Punkten auf einer Kugeloberfläche** verwendet. Sie berechnet hier die Distanz jeder Stadt (`g.lat`, `g.lon`) zum Zentrum von München.

| SQL-Teil | Bedeutung und Funktion |
| :--- | :--- |
| `SELECT g.name, g.state, g.population, l.lebendgeborene, ROUND(...) AS geburten_pro_1000, ROUND(...) AS distance_km` | Wählt die relevanten Spalten aus und berechnet die Geburtenrate pro 1.000 Einwohner sowie die Entfernung zu München in Kilometern. |
| `6371` | Die **Konstante für den Erdradius** in Kilometern. Sie wird mit dem errechneten Winkel multipliziert, um die tatsächliche Entfernung zu erhalten. |
| `6371 * acos(...)` | Berechnet die Entfernung zwischen München (48.137154°N, 11.576124°O) und der jeweiligen Stadt (`g.lat`, `g.lon`) mithilfe der Haversine-Formel. |
| `RADIANS(Wert)` | Wandelt die geografischen Koordinaten von Grad in **Radiant** um – notwendig für die trigonometrischen Funktionen `COS` und `SIN`. |
| `COS(radians(48.137154))` | Verwendet die festen Koordinaten des Zielpunkts **München** in der Formel. |
| `COS(radians(g.lat))` | Verwendet den dynamischen Breitengrad der aktuellen Stadt aus `german_cities`. |
| `ACOS(...)` | Die **Arcus-Cosinus-Funktion** berechnet den Winkel zwischen zwei Punkten auf der Kugeloberfläche. |
| `... AS distance_km` | Speichert das Ergebnis der Distanzberechnung als neue Spalte `distance_km`. |
| `AND (...) < 100` | Dieselbe Distanzberechnung wird als **Filterbedingung** verwendet, um nur Städte anzuzeigen, die **weniger als 100 km** von München entfernt sind. |
| `ORDER BY geburten_pro_1000 DESC` | Sortiert absteigend nach der Geburtenrate. |
| `LIMIT 10` | Begrenzt die Ausgabe auf die ersten 10 Zeilen. |

---

## Fortgeschrittene Beispielabfrage: Städte nahe Berlin mit hoher Geburtenrate

```sql
-- Top 5 kreisfreie Städte nahe Berlin mit Geburtenrate >= 8
SELECT g.name, g.state, g.population, l.lebendgeborene,
       -- 1. Berechnung der Geburtenrate pro 1.000 Einwohner
       ROUND((l.lebendgeborene / g.population) * 1000, 2) AS geburten_pro_1000,
       -- 2. Berechnung der Entfernung von Berlin in Kilometern (Haversine-Formel)
       ROUND(
           6371 * acos(
               cos(radians(52.520008)) * cos(radians(g.lat)) * cos(radians(g.lon) - radians(13.404954)) + 
               sin(radians(52.520008)) * sin(radians(g.lat))
           ), 2
       ) AS distance_km
FROM german_cities g
LEFT JOIN lebendgeborene_2023 l ON g.name = l.clean_name
WHERE l.region_typ = 'kreisfreie Stadt' 
  AND l.lebendgeborene IS NOT NULL
  -- 3. Filterung: Schränkt die Ergebnisse auf Städte mit Rate >= 8 ein
  AND (l.lebendgeborene / g.population) * 1000 >= 8 
ORDER BY distance_km ASC
LIMIT 5;
```

#### Syntaxerklärung

Diese Abfrage filtert Städte nach einer hohen Geburtenrate und sortiert das Ergebnis nach der geografischen Entfernung zu Berlin.

| SQL-Teil | Bedeutung und Funktion |
| :--- | :--- |
| **Feste Koordinaten** | Die Werte `52.520008` (Breitengrad) und `13.404954` (Längengrad) definieren den Ausgangspunkt Berlin. Die Distanz wird von jeder Stadt zu diesem festen Punkt berechnet. |
| `SELECT g.name, g.state, ... AS distance_km` | Wählt die anzuzeigenden Spalten aus, inklusive der berechneten Felder `geburten_pro_1000` und `distance_km`. |
| `6371 * acos(...)` | **Haversine-Formel:** Berechnet die kürzeste Entfernung in Kilometern zwischen Berlin und jeder Stadt in der Tabelle (`g.lat`, `g.lon`). |
| `FROM german_cities g LEFT JOIN lebendgeborene_2023 l ON g.name = l.clean_name` | Die Zieltabelle ist `german_cities g`. Der **LEFT JOIN** verknüpft diese mit `lebendgeborene_2023 l`, wobei alle Städte aus `german_cities` erhalten bleiben. |
| `WHERE ... AND (l.lebendgeborene / g.population) * 1000 >= 8` | Filtert auf kreisfreie Städte mit vorhandenen Geburtendaten und einer Geburtenrate von **mindestens 8** pro 1.000 Einwohner. |
| `ORDER BY distance_km ASC` | Sortiert **aufsteigend** nach der Entfernung zu Berlin – die nächstgelegenen Städte erscheinen zuerst. |
| `LIMIT 5` | Begrenzt die Ausgabe auf die **fünf** Städte, die die Kriterien erfüllen und Berlin am nächsten liegen. |

---

## Datenquellen und Lizenz

Die Rohdaten stammen vom deutschen Open-Data-Portal **GovData.de**. Dort veröffentlichte Datensätze stehen in der Regel unter einer von zwei Lizenzen:

**DL-Zero** (Datenlizenz Deutschland – Zero): Freie Nutzung ohne Pflicht zur Quellenangabe – ein Hinweis wird dennoch empfohlen.

**DL-DE/BY** (Datenlizenz Deutschland – Namensnennung): Nutzung erlaubt, Quellenangabe verpflichtend.

Die Dateien `german_cities.csv` und `lebendgeborene_2023.csv` wurden auf Basis dieser Rohdaten bereinigt und angepasst, um sie für dieses Begleitmaterial direkt verwendbar zu machen.

---

## Lizenz

Dieses Repository dient ausschließlich Lern- und Demonstrationszwecken und steht unter der [MIT-Lizenz](https://github.com/dvrdnz/LF05v2/blob/main/LICENSE)
