---
layout: page
title: Esimerkki
---

Lähtökohtana (Versio 0) on html -lomakkeen sisältävä käyttöliittymä (`index.php`) ja lomakkeen kutsuma palvelu (`service.php`), joka tulostaa lomakkeen välittämän datan NetBeansin Output -ikkunaan ja ohjaa sen jälkeen käsittelyn takaisin käyttöliittymälle.

Välivaiheiden kautta päädytään ratkaisuun (Versio 6), jossa käyttöliittymään syötetty data välitetään palvelulle `json`-muodossa `XMLHttpRequest`-objektia käyttäen. Palvelu selvittää, onko käyttöliittymän kahteen tekstikenttään syötetty samat arvot. Selvityksen tulos palautetaan käyttöliittymälle `json`-muodossa. Käyttöliittymän koodi rakentaa paluuviestin saapumisen jälkeen elementin, jonka sisältönä on vastaanotettu tulos, ja sijoittaa elementin sivulle.


Versioiden koodi löytyy
[GitHubista](https://github.com/timedu/weo/tree/gh-pages/demot/form2xhr).

Selainpään osalta esimerkkiin liittyviä asioista käsitellään kurssimateriaalin (Web-selainohjelmointi) luvuissa 
[5. DOM](http://web-selainohjelmointi.github.io/#5-DOM) 
ja 
[7. Keskustelu palvelimen kanssa](http://web-selainohjelmointi.github.io/#7-Keskustelu-palvelimen-kanssa).

## Versio 0


### index.php (käyttöliittymä)

Ottaa vastaan käyttäjältä datan ja lähettää sen `POST`-pyynnön mukana osoitteeseen `service.php` käyttäjän klikattua submit-nappia.

{% highlight html %}

    <body>
        <form method="POST" action="service.php">
            data 1: <input name="data1"/><br/>
            data 2: <input name="data2"/><br/>
            <input type="submit" value="Lähetä"/>
        </form>
    </body>

{% endhighlight %}

### service.php

Lukee POST-pyynnön välittämän datan ja tulostaa sen NetBeansin Output-ikkunaan sekä ohjaa käsittelyn takaisin käyttöliittymälle.

{% highlight php startinline=true %}

<?php
	
    $httpBody = file_get_contents('php://input');
    debug($httpBody);
    header('Location: index.php');

    function debug($data) {
       file_put_contents('php://stderr', $data . PHP_EOL);
    }

{% endhighlight %}



Jos käyttöliittymän tekstikenttiin syötetään arvot `a` ja `b`, Output -ikkunaan tulostuu `data1=a&data2=b`.


## Versio 1


### index.php

Näyttää ulospäin samalta kuin edellinen versio, mutta lähettää datan käyttäen `XMLHttpRequest`-oliota käyttäjän klikattua sivulla olevaa nappia.


{% highlight html %}

    <body>

        data 1: <input id="data1"/><br/>
        data 2: <input id="data2"/><br/>
        <input id="button" type="button" value="Lähetä"/>

        <script>
            window.onload = function () {
                document.getElementById('button').onclick = sendData;
            };
            
            function sendData() {
                var data =
                        'data1=' + document.getElementById('data1').value +
                        '&' +
                        'data2=' + document.getElementById('data2').value;

                var xhr = new XMLHttpRequest();
                xhr.open('POST', 'service.php');
                xhr.setRequestHeader("Content-type", "application/x-www-form-urlencoded");
                xhr.send(data);                
            }
        </script>

    </body>

{% endhighlight %}


Sivulla ei ole `form`-elementtiä eikä `submit`-nappia, jonka on korvannut "tavallinen" `button`. JavaScript -koodi on sidottu napin `click`-tapahtumaan. 

### service.php

On lähes sama kuin edellinen versio. Ei kuitenkaan sisällä uudelleenohjaavaa lausetta.

{% highlight php %}

<?php

    $httpBody = file_get_contents('php://input');
    debug($httpBody);

    function debug($data) {
        file_put_contents('php://stderr', $data . PHP_EOL);
    }

{% endhighlight %}

## Versio 2


### index.php

Lähettää datan "lomake-muodon" sijaan `json`-muodossa. On muuten sama kuin edellinen versio.

{% highlight html %}

        <script>

            window.onload = function () {
                document.getElementById('button').onclick = sendData;
            };
            
            function sendData() {

                var data = {};                
                data.data1 = document.getElementById('data1').value;
                data.data2 = document.getElementById('data2').value;
                
                var xhr = new XMLHttpRequest();
                xhr.open('POST', 'service.php');
                xhr.setRequestHeader("Content-type", "application/json");
                
                xhr.send(JSON.stringify(data));                
            }

        </script>
        
{% endhighlight %}


### service.php

On sama kuin edellinen versio. Jos käyttöliittymän tekstikenttiin syötetään arvot `a` ja `b`, NetBeansin Output -ikkunaan tulostuu nyt `{"data1":"a","data2":"b"}`.


## Versio 3

### index.php

On sama kuin edellinen versio.

### service.php

Muuntaa pyynnön mukana saadun datan merkkijonoilla indeksoiduksi taulukoksi ja tulostaa alkiot erillisinä Netbeansin Output-ikkunaan.

{% highlight php %}

<?php

    $httpBody = file_get_contents('php://input');
    $data = json_decode($httpBody, true);

    debug($data['data1']);
    debug($data['data2']);

    function debug($data) {
        file_put_contents('php://stderr', $data . PHP_EOL);
    }

{% endhighlight %}


Jos käyttöliittymän tekstikenttiin syötetään arvot `a` ja `b`,  Output -ikkunaan tulostuu nyt `a` ja `b` omille riveilleen. 


## Versio 4


### index.php

Pyynnön lähetyksen lisäksi käsittelee vasteen mukana saadun datan tulostaen sen selaimen konsolille. Jos tekstikenttiin on syötetty saman arvot, konsolille tulostuu `{"tulos":"samat"}` muutoin `{"tulos":"erit"}`.

{% highlight html %}

        <script>

            window.onload = function () {
                document.getElementById('button').onclick = sendData;
            };
            
            function sendData() {

                var data = {};                
                data.data1 = document.getElementById('data1').value;
                data.data2 = document.getElementById('data2').value;
                
                var xhr = new XMLHttpRequest();
                xhr.onload = receiveData;
                xhr.open('POST', 'service.php');
                xhr.setRequestHeader("Content-type", "application/json");
                
                xhr.send(JSON.stringify(data));                
            }
            
            function receiveData() {                
                console.log(this.responseText);
            }
            
        </script>

{% endhighlight %}

### service.php

Rakentaa vasteen ja tulostaa sen `json`-muodossa. 

{% highlight php %}

<?php

    $httpBody = file_get_contents('php://input');
    $input = json_decode($httpBody, true);

    $output = [];

    if ($input['data1'] == $input['data2']){
        $output['tulos'] = 'samat';
    } else {
        $output['tulos'] = 'erit';
    }

    header('Content-type: application/json');
    echo json_encode($output);

{% endhighlight %}


## Versio 5


### index.php

Muuntaa vastaanotetun `json`-muotoisen merkkijonon JavaScript -objektiksi ja  tulostaa konsolille ainoastaan `tulos`-ominaisuuden arvon.

{% highlight html %}

        <script>

            window.onload = function () {
                document.getElementById('button').onclick = sendData;
            };
            
            function sendData() {

                var data = {};                
                data.data1 = document.getElementById('data1').value;
                data.data2 = document.getElementById('data2').value;
                
                var xhr = new XMLHttpRequest();
                xhr.onload = receiveData;
                xhr.open('POST', 'service.php');
                xhr.setRequestHeader("Content-type", "application/json");
                
                xhr.send(JSON.stringify(data));                
            }
            
            function receiveData() {
                var response = JSON.parse(this.responseText)                
                console.log(response.tulos);
            }
            
        </script>

{% endhighlight %}

Konsolille tulostuu joko `samat` tai `erit` riippuen tekstikenttiin syötetyistä arvoista.


### service.php

On sama kuin edellinen versio.


## Versio 6

### index.php

JavaSript on erillisessä tiedostossa. Käyttöliittymässä on `div`-elementti tuloksen esittämistä varten.

{% highlight html %}

    <body>

        data 1: <input id="data1"/><br/>
        data 2: <input id="data2"/><br/>
        <input id="button" type="button" value="Lähetä"/>
        
        <div id="tulos"></div>

        <script src="index.js"></script>

    </body>

{% endhighlight %}

### index.js

Esittää tuloksen sivulla konsolitulostuksen sijaan.

{% highlight javascript %}

    window.onload = function () {
      document.getElementById('button').onclick = sendData;
    };

    function sendData() {

      var data = {};
      data.data1 = document.getElementById('data1').value;
      data.data2 = document.getElementById('data2').value;

      var xhr = new XMLHttpRequest();
      xhr.onload = receiveData;
      xhr.open('POST', 'service.php');
      xhr.setRequestHeader("Content-type", "application/json");

      xhr.send(JSON.stringify(data));
    }

    function receiveData() {

      var response = JSON.parse(this.responseText);

      var h3 = document.createElement('h3');
      h3.textContent = response.tulos;

      var div = document.getElementById('tulos');
      div.innerHTML = '';
      div.appendChild(h3);

    }

{% endhighlight %}

Muodostaa `h3`-elementin ja asettaa sen `div`-elementin sisällöksi.


### service.php

On sama kuin edellisessä versiossa.


