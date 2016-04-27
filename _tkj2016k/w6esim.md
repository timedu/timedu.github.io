---
layout: page
title: Esimerkkejä
---

Esimerkit perustuvat OrientDB 
-[käsikirjaan](http://orientdb.com/docs/last/index.html) 
sekä 
[PHP-ajurin](https://github.com/orientechnologies/PhpOrient) 
mukana tulevaan dokumentaatioon. 


## Yhteydenotto tietokantaan


### Peruskäyttö

~~~
$client = new PhpOrient('localhost', 2424);
$client->dbOpen( 'GratefulDeadConcerts', 'admin', 'admin' );
~~~

Tehtäväpohjassa oleva yhteydenotto tietokantaan on peruskäytön näkökulmasta hieman ylimitoitettu tarjoten mahdollisuuden myös hallinta -toimintojen käyttöön (esim. uuden tietokannan perustaminen).
 
### Hallinta-toimintojen käyttö

~~~
$client = new PhpOrient();
$client->hostname = 'localhost';
$client->port     = 2424;
$client->username = 'root';
$client->password = 'root_pass';
$client->connect();
~~~

## Kyselyt

Tietokantaan voidaan kohdistaa yksinkertaisia kysely -operaatioita perinteisen SQL:n tapaan 
[SELECT](http://orientdb.com/docs/last/SQL-Query.html)-komennolla, 
esim:

~~~
SELECT name, job, @rid AS id
FROM   Employee 
WHERE  city.name = 'Rome'
ORDER BY name
~~~

Kysely kohdistuu `Employee`-luokan dokumentteihin. `SELECT`-osassa oleva `@rid` ("record id") viittaa dokumentin tietokantatunnisteeseen, jolle kyselyssä määritellään (viitattava) nimi `id`. `WHERE` -osassa olevassa ehtolauseessa käytetään dokumentin `city`-ominaisuuttta, joka on linkki toiseen dokumenttiin. Linkatun dokumentin `name`-ominaisuuttta käytetään kyselyehdossa ilman erillistä liitos -operaatiota, mitä relaatiomalli edellyttää vastaavissa tilanteissa. Myös komennon `SELECT`-osassa voidaan viitata linkattujen dokumenttien ominaisuuksiin.

PHP-ohjelmassa edellinen kysely voidaan toteuttaa seuraavasti:

~~~
$city = 'Rome';
$employees = $client->query(''
      . ' SELECT   name, job, @rid AS id'
      . ' FROM     Employee'
      . " WHERE    city.name = '$city'"
      . ' ORDER BY name'
);

~~~
 
Kysely palauttaa `$employees`-muuttujaan `Record`-olioita sisältävän taulukon. `Record` sisältää metodeja, joita käyttäen esim. twig-template osaa poimia tarvitsemansa tiedot.

Dokumentti voidaan hakea tietokannasta tietokantatunnuksella seuraavasti:

~~~
$id = '#2:34';
$employee = $client->query(''
      . ' SELECT   name, job, @rid AS id, city.name AS city_name'
      . ' FROM     Employee'
      . " WHERE    @rid = $id"
)[0];

~~~

Tämäkin kysely palauttaa taulukon, mutta sen sisältönä on ainoastaan yksi `Record`-luokan olio,  joka voidaan poimia taulukosta indeksillä `0`.

Yksittäinen ominaisuus saadaan oliosta viittaamalla kyselyn `SELECT`-osan tunnisteita vastaaviin attribuutteihin, esim. `$employee->name`. Ominaisuuksien nimillä indeksoitu taulukko voidaan täten muodostaa seuraavasti:

~~~
$emp = [
	"name" => $employee->name,
	"job" => $employee->job, 
	"id" => $employee->id,
 	"city_name" => $employee->city_name
];

~~~

Edellinen hoituu myös yhdellä sijoituslauseella:

~~~
$emp = $employee->getOData();
~~~
 

## Lisäys 

Tietueen lisäys voidaan suorittaa
[INSERT](http://orientdb.com/docs/last/SQL-Insert.html)-komennolla. Jos tietue on gaafin solmu, voidaan käyttää myös komentoa
[CREATE VERTEX](http://orientdb.com/docs/last/SQL-Create-Vertex.html). Kaarien lisäykseen on käytettävissä komento 
[CREATE EDGE](http://orientdb.com/docs/last/SQL-Create-Edge.html).

Lisäys voidaan toteutta perinteisen SQL:n tapaan:

~~~
INSERT INTO Person (name, surname) 
VALUES ('Jay', 'Miner')
~~~

Edellisen sijaan voidaan käyttää myös seuraavaa muotoa:

~~~
INSERT INTO Person 
CONTENT {"name": "Jay", "surname": "Miner"}
~~~

Jälkimmäistä muotoa voidaan käyttää myös lisättäessä graafien solmuja (`Person` on tällöin `V`-luokan aliluokka):

~~~
CREATE VERTEX  Person 
CONTENT {"name": "Jay", "surname": "Miner"}
~~~


PHP -ohjelmassa edellisen komennon voi toteuttaa esim. seuraavasti:

~~~
$person = $client->command(''
      . 'CREATE VERTEX Person CONTENT '
      . json_encode([
              'name' => "Jay",
              'surname' => "Miner"
       ])
);

~~~

Edellä PHP:n `json_encode`-funktiolla muunnetaan taulukko json -merkkijonoksi. `command`-metodi palauttaa tietokantaan talletetun tietueen (`Record`-olio).

Graafin kaari voidaan lisätä seuraavasti:

~~~
CREATE EDGE Watched FROM #10:3 TO #11:4
~~~

Vastaava komento PHP -ohjelmassa voi olla esim. seuraava:

~~~
$client->command(''
       . 'CREATE EDGE Watched '
       . "FROM {$person->getRid()} "
       . "TO {$movie->getRid()} ");
~~~


## Muutos

Ks. [UPDATE](http://orientdb.com/docs/last/SQL-Update.html).

UPDATE-komento voidaan suoritaa PHP-ohjelmassa INSERT-komennon tapaan tietokanta-asiakkaan `command`-metodilla. 


## Poisto

Ks. 
[DELETE](http://orientdb.com/docs/last/SQL-Delete.html), 
[DELETE VERTEX](http://orientdb.com/docs/last/SQL-Delete-Vertex.html),
[DELETE EDGE](http://orientdb.com/docs/last/SQL-Delete-Edge.html).

DELETE-komennot voidaan suoritaa PHP-ohjelmassa INSERT-komennon tapaan tietokanta-asiakkaan `command`-metodilla. 


## Transaktiot

Yhteenkuuluvat tietokantakomennot voidaan suorittaa nippuna esim. tietokanta-asiakkaan `sqlBatch`-metodilla:

~~~
$cmd = 'BEGIN;' .
       'LET a = CREATE VERTEX SET script = TRUE;' .
       'LET b = SELECT FROM V LIMIT 1;' .
       'CREATE EDGE FROM $a TO $b;' .
       'COMMIT;';

$client->sqlBatch( $cmd );
~~~


