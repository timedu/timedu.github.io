---
layout: page
title: Yleistä
---

# Osa 6: Hybriditietokannat/OrientDB


OrientDB:n 
[Getting Started -sivu](http://orientdb.com/getting-started/)
ohjaa tutustumaan tietokantaan lyhyen johdantokurssin avulla. Vapaasti käytettävä Udemy -alustalla toteutettu 
[online-kurssi](https://www.udemy.com/orientdb-getting-started/)
sisältää 21 luentoa (yhteensä 2h) ja niihin perustuvia monivalintatehtäviä. Tekstimuotoisen 
[käsikirjan](http://orientdb.com/docs/last/index.html) keskeisimpiä lukuja lienevät *1. Getting Started*, *2. Data Modeling* ja *5. SQL Reference*.

## Tietokantaohjelmiston asennus

Käsikirjan ensimmäinen luku sisältää ohjeet tietokantaohjelmiston asentamiseen. Binääriasennukseen tarvittavan paketin eri käyttöjärjestelmille voi ladata 
[Download -sivulta](http://orientdb.com/download/). Erillisiä ajureita ei tarvitse ladata/asentaa. Tehtävässä käytettävä ajuri
([PhpOrient](https://github.com/orientechnologies/PhpOrient))
on mukana tehtäväpohjassa. Ajuri kuitenkin edellyttää, että PHP:n socket-laajennus on aktivoitu[^1].

[^1]: Esim. windows-ympäristössä tiedostossa `php.ini` rivin `;extension=php_sockets.dll` alusta poistetaan kommentointimerkki (`;`).

## Tietokantapalvelimen käynnistys

Tietokantaohjelmiston asennuksen jälkeen palvelinohjelmiston voi käynnistää asennuspaketin mukana tulevilla komentotiedostoilla `server.sh` tai `server.bat` (ks. 
[Running the OrientDB Server](http://orientdb.com/docs/last/Tutorial-Run-the-server.html)). Ohjelmistoon sisältyy kaksi hallintatyökalua, joista toinen on komentopohjainen 
([console](http://orientdb.com/docs/last/Tutorial-Run-the-console.html)) 
ja toinen selaimen kautta käytettävä web-sovellus 
([Studio](http://orientdb.com/docs/last/Tutorial-Run-the-studio.html)). 

## Tietokannan perustaminen

Tehtävissä käytettävät tietokannat, `kurssit_1` ja `kurssit_2` voi perustaa esim. **Studion** login -ikkunan kautta (*New DB* -painonappi). 

Tietokantojen rakenteen ja sisällön määrittelevät komentotiedostot löytyvät tehtäväpohjasta. Ne voidaan suorittaa kopioimalla tiedostojen sisältö komentopohjaisen **konsolin** kehote -merkin perään. 

