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

{% highlight php %}

<?php

    $client = new PhpOrient('localhost', 2424);
    $client->dbOpen( 'GratefulDeadConcerts', 'admin', 'admin' );

{% endhighlight %}

Tehtäväpohjassa oleva yhteydenotto tietokantaan on peruskäytön näkökulmasta hieman ylimitoitettu tarjoten mahdollisuuden myös hallinta -toimintojen käyttöön (esim. uuden tietokannan perustaminen).
 
### Hallinta-toimintojen käyttö

{% highlight php %}

<?php

    $client = new PhpOrient();
    $client->hostname = 'localhost';
    $client->port     = 2424;
    $client->username = 'root';
    $client->password = 'root_pass';
    $client->connect();

{% endhighlight %}

## Kyselyt

Tietokantaan voidaan kohdistaa yksinkertaisia kysely -operaatioita perinteisen SQL:n tapaan 
[SELECT](http://orientdb.com/docs/last/SQL-Query.html)-komennolla, 
esim:

{% highlight sql %}

<?php

    SELECT name, job, @rid AS id
    FROM   Employee 
    WHERE  city.name = 'Rome'
    ORDER BY name

{% endhighlight %}

Kysely kohdistuu `Employee`-luokan dokumentteihin. `SELECT`-osassa oleva `@rid` ("record id") viittaa dokumentin tietokantatunnisteeseen, jolle kyselyssä määritellään (viitattava) nimi `id`. `WHERE` -osassa olevassa ehtolauseessa käytetään dokumentin `city`-ominaisuuttta, joka on linkki toiseen dokumenttiin. Linkatun dokumentin `name`-ominaisuuttta käytetään kyselyehdossa ilman erillistä liitos -operaatiota, mitä relaatiomalli edellyttää vastaavissa tilanteissa. Myös komennon `SELECT`-osassa voidaan viitata linkattujen dokumenttien ominaisuuksiin.

PHP-ohjelmassa edellinen kysely voidaan toteuttaa seuraavasti:

{% highlight php %}

<?php

    $city = 'Rome';
    $employees = $client->query(''
      . ' SELECT   name, job, @rid AS id'
      . ' FROM     Employee'
      . " WHERE    city.name = '$city'"
      . ' ORDER BY name'
);

{% endhighlight %}
 
Kysely palauttaa `$employees`-muuttujaan `Record`-olioita sisältävän taulukon. `Record` sisältää metodeja, joita käyttäen esim. twig-template osaa poimia tarvitsemansa tiedot.

Dokumentti voidaan hakea tietokannasta tietokantatunnuksella seuraavasti:

{% highlight php %}

<?php

    $id = '#2:34';
    $employee = $client->query(''
      . ' SELECT   name, job, @rid AS id, city.name AS city_name'
      . ' FROM     Employee'
      . " WHERE    @rid = $id"
    )[0];

{% endhighlight %}

Tämäkin kysely palauttaa taulukon, mutta sen sisältönä on ainoastaan yksi `Record`-luokan olio,  joka voidaan poimia taulukosta indeksillä `0`.

Yksittäinen ominaisuus saadaan oliosta viittaamalla kyselyn `SELECT`-osan tunnisteita vastaaviin attribuutteihin, esim. `$employee->name`. Ominaisuuksien nimillä indeksoitu taulukko voidaan täten muodostaa seuraavasti:

{% highlight php %}

<?php

    $emp = [
	    "name" => $employee->name,
	    "job" => $employee->job, 
	    "id" => $employee->id,
 	    "city_name" => $employee->city_name
    ];

{% endhighlight %}

Edellinen hoituu myös yhdellä sijoituslauseella:

{% highlight php %}

<?php

    $emp = $employee->getOData();

{% endhighlight %}

### Graafeihin kohdistuvat kyselyt

Edellä oli esimerkki, miten dokumentin `LINK`-tyyppisen ominaisuuden kautta voidaan viitata toisen dokumentin ominaisuukseen. Seuraavssa on sama esimerkki kuitenkin niin, että `Employee` ja `City` ovat graafitietokannan solmuja siten, että niiden välillä on `Lives`-kaari ("employee lives city"):

{% highlight php %}

<?php

    $id = '#2:34';
    $employee = $client->query(''
      . ' SELECT   name, job, @rid AS id, OUT(Lives).name AS city_name'
      . ' FROM     Employee'
      . " WHERE    @rid = $id"
    )[0];

{% endhighlight %}

[OUT](http://orientdb.com/docs/last/SQL-Functions.html#out)
-funktiolla löydetään `Employee`-solmuun `Lives`-kaaren kautta liittyvät `City`-solmut.  

Edellä esitettyjen sijoituslauseiden jälkeen `$employee` sisältää `Record`-olion, jonka `city_name`-ominaisutena on taulukko. Jos tavoitteena on muodostaa `$employee`-muuttujan sisällöksi "normalisoitu" rakenne, niin sen voi tehdä tässä esim. seuraavasti:

{% highlight php %}

<?php

    if (count($employee->city_name)) {
       $employee->city_name = $employee->city_name[0];
    }

{% endhighlight %}

Edellä taulukko siis korvataan taulukon ensimmäisellä alkiolla (oletetaan, että employee:llä on korkeintaan yksi city).



## Lisäys 

Tietueen lisäys voidaan suorittaa
[INSERT](http://orientdb.com/docs/last/SQL-Insert.html)-komennolla. Jos tietue on gaafin solmu, voidaan käyttää myös komentoa
[CREATE VERTEX](http://orientdb.com/docs/last/SQL-Create-Vertex.html). Kaarien lisäykseen on käytettävissä komento 
[CREATE EDGE](http://orientdb.com/docs/last/SQL-Create-Edge.html).

Lisäys voidaan toteutta perinteisen SQL:n tapaan:

{% highlight sql %}

    INSERT INTO Person (name, surname) 
    VALUES ('Jay', 'Miner')

{% endhighlight %}

Edellisen sijaan voidaan käyttää myös seuraavaa muotoa:

{% highlight sql %}

    INSERT INTO Person 
    CONTENT {"name": "Jay", "surname": "Miner"}

{% endhighlight %}

Jälkimmäistä muotoa voidaan käyttää myös lisättäessä graafien solmuja (`Person` on tällöin `V`-luokan aliluokka):

{% highlight sql %}

    CREATE VERTEX  Person 
    CONTENT {"name": "Jay", "surname": "Miner"}

{% endhighlight %}


PHP -ohjelmassa edellisen komennon voi toteuttaa esim. seuraavasti:

{% highlight php %}

<?php

    $person = $client->command(''
      . 'CREATE VERTEX Person CONTENT '
      . json_encode([
              'name' => "Jay",
              'surname' => "Miner"
       ])
    );

{% endhighlight %}

Edellä PHP:n `json_encode`-funktiolla muunnetaan taulukko json -merkkijonoksi. `command`-metodi palauttaa tietokantaan talletetun tietueen (`Record`-olio).

Graafin kaari voidaan lisätä seuraavasti:

{% highlight sql %}

    CREATE EDGE Watched FROM #10:3 TO #11:4

{% endhighlight %}

Vastaava komento PHP -ohjelmassa voi olla esim. seuraava:

{% highlight php %}

<?php

    $client->command(''
       . 'CREATE EDGE Watched '
       . "FROM {$person->getRid()} "
       . "TO {$movie->getRid()} ");

{% endhighlight %}


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

{% highlight php %}

<?php

    $cmd = 'BEGIN;' .
       'LET a = CREATE VERTEX SET script = TRUE;' .
       'LET b = SELECT FROM V LIMIT 1;' .
       'CREATE EDGE FROM $a TO $b;' .
       'COMMIT;';

    $client->sqlBatch( $cmd );

{% endhighlight %}


