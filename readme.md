# Infra ja devops

## Kuvaus projektista

Itseäni alkoi kiinnostamaan Openshift ja koodin kehittäminen Lenni Laukkasen luennon jälkeen. Kokeilin DevOps tehtävässä 
tehdä palvelimen Javalla (Spring), MySql serverin (Openshift) sekä käyttöliittymän (React). Laitoin koodit Githubiin
ja sen jälkeen Openshiftissä ilmoitin, että buildaa kontit sieltä. Tällä tavalla kaikki toimi, mutta en itse siis tehnyt
kontin koodia, enkä saanut sitä automaattisesti komentokehotteesta debloyattua.

Tällä kertaa ajatus oli aika samanlainen, mutta toisin kuin tehtävässä, halusin nyt tehdä itse kontit sekä linkittää
imaget suoraan Openshiftin repositorioon. Tällä tavalla, aina kun kontin buildaan ja pusken -> debloyaa automaattisesti
uuden sovelluksen käyttöön. Sovelluksessani piti myös olla käyttäjien hallinta ulkoisen tahon puolelta. Tähän valitsin 
Googlen Firebasen, jossa käyttäjä luotaisiin ja varmennettaisiin aina kirjautumisen / rekisteröinnin yhteydessä. 

## Aloittaminen

Päätin aloittaa projektin tekemisen projektin luomisella Firebaseen. Minulla oli jo käyttäjätiedot palveluun olemassa, joten
sisäänkirjautumisen jälkeen aloitin luomalla uuden projektin palveluun. Projektin luomisen jälkeen mahdollistin autentikoinnin
palvelussa (Authentication), lisäsin autentikointitavaksi sähköpostiosoitteen ja projektinäkymästä (Project Overview) sain 
tarvittavat tiedot ohjelmallista yhteydenottoa varten.

| ![kuva1.jpg](https://github.com/niikari/ohjelmistotekniikoiden-seminaari/blob/main/photos/authentication_firebase.JPG?raw=true) |
|:--:|
| *Käyttäjätiedot* |

Kuvassa näkyvissä testausta varten luotu käyttäjä (sittemmin poistettu).

| ![kuva2.jpg](https://github.com/niikari/ohjelmistotekniikoiden-seminaari/blob/main/photos/authentication_method.JPG?raw=true) |
|:--:|
| *Sähköposti/Salasana tunnistautuminen mahdollistettu* |

|![kuva3.jpg](https://github.com/niikari/ohjelmistotekniikoiden-seminaari/blob/main/photos/authentication_web.JPG?raw=true) |
|:--:|
| *Tiedot palvelusta, kuten tarvittava API avain* |

## Palvelin

Palvelimen pohjaksi valitsin Java Spring Bootin (Maven). Aloitin luomalla itselleni uuden projektin [Spring initializerilla](https://start.spring.io/).
Lopullinen palvelimen koodi löydettävissä [täältä](https://github.com/niikari/tori_backend) (dev -branch).

Palvelimelle luotu mahdollista tulevaa kehittämistä varten useampi luokka sekä repositoriot näille. Ajan säästämisen vuoksi, en käyttänyt aikaani RestControllerien 
luomiseen vaan käytin Springin tarjoamaa Rest Data Api -palvelua hyväksi, jonka avulla sain automaattisesti luotua rajapinanat mm. käyttäjien lisäämiseksi
tietokantaan. Firebasen autentikoinnin mukaan ottamiseksi löysin Googlen avulla hyvän sivun avukseni: [Authentication with Firebase Auth and Spring security](https://medium.com/comsystoreply/authentication-with-firebase-auth-and-spring-security-fcb2c1dc96d).

Palvelimen koodauksen aloituksen yhteydessä loin heti paikallisen git repositorion sekä Githubiin tälle yhteyden. Koodatessa aina seuraavaa ominaisuutta, tein
ensin tästä oman haaransa, ja mikäli haaran kanssa tuli ongelmia - palasin aina edelliseen toimivaan versioon. Tätä jatkoin, kunnes omasta mielestäni koodi
oli tarpeeksi toimiva tehdäkseni siitä Docker imagen. 

## Palvelimen koodi Docker imageksi

Aloitin kontin tekemisen jar-tiedoston luomisella. Käytin koodaamiseen Eclipsen EE editionia ja täältä painamalla hiiren oikeaa näppäintä projektin päällä
löysin run -valikosta maven build... -kohdan. Antamalla täällä "goaliksi" *package* ja painamalla run - aloitti jar-tiedoston luomisen testaamalla toiminnan
ja tämän jälkeen luomalla jar-tiedoston (/target/backend_openshift-0.0.1-SNAPSHOT.jar) projektin kansioon. 

Tämän jälkeen loin Dockerfile -tiedoston ja sisällöksi asetin:

```
FROM openjdk:latest

EXPOSE 8080

WORKDIR /app

COPY /target/backend_openshift-0.0.1-SNAPSHOT.jar .

CMD ["java", "-jar", "backend_openshift-0.0.1-SNAPSHOT.jar"]
```

Sitten edessä olikin imagen rakentaminen, tämän toteutin komennolla (samasta sijainnista Dockerfilen kanssa):

	$ docker build . -t javabackend

"Hetken" lataamisen jälkeen kokeilin palvelimen toimintaa komennolla:

	$ docker run -it -p 8080:8080 javabackend

Palvelin lähti käyntiin ilman virheviestejä ja yhteyden ottaminen osoitteessa http://localhost:8080/api toimi oikein. Sammutin palvelimen
painamalla CTRL + C

| ![kuva4.jpg](https://github.com/niikari/ohjelmistotekniikoiden-seminaari/blob/main/photos/docker_javabackend_v1.JPG?raw=true) |
|:--:|
| *Kontissa pyörivä palvelin ja testattu yhteyttä selaimella* |

## Openshift ja Docker imagen yhdistäminen kehityputkeksi

Otaverkon tarjoamaan Openshiftiin kirjauduttuani loin uuden projektin: niilesseminaari.

Tämän jälkeen palasin takaisin komentokehotteelle ja jatkoin kirjautumalla Otaverkon imageregistryyn ensin hakemalla autentikointi
avaimen ja tämän jälkeen kirjoittamalla komennon:

	$ docker login default-route-openshift-image-registry.apps.hhocp.otaverkko.fi

Kirjauduttani onnistuneesti palveluun jatkoin aiemmin tehdyn imagen parissa. Lisäsin ensin imageen tagin, joka kertoo mihin tämä image
push -komennon jälkeen ladataan ja tämän jälkeen latasin imagen Otaverkon registryyn:

	$ docker tag javabackend default-route-openshift-image-registry.apps.hhocp.otaverkko.fi/niilesseminaari/backend:latest
	
	$ docker push default-route-openshift-image-registry.apps.hhocp.otaverkko.fi/niilesseminaari/backend

Tästä siirryin takaisin Openshiftin projektiin ja sieltä kohtaan +Add. Painoin valikosta löytyvää *Container images* -kohtaa ja siirryin 
etsimään juuri lisäämääni imagea osaksi projektiani.

| ![kuva5.jpg](https://github.com/niikari/ohjelmistotekniikoiden-seminaari/blob/main/photos/openshift_backend_image_add.JPG?raw=true) |
|:--:|
| *Image löytyi onnistuneesti, portti asetettu 8080 ja create* | 

Tämän jälkeen tarkistin, että kontti pyörii oikein ja, että siihen saa yhteyttä (tässä vaiheessa autentikointia ei tarvita, sallii
kaikki pyynnöt).

| ![kuva6.jpg](https://github.com/niikari/ohjelmistotekniikoiden-seminaari/blob/main/photos/openshift_backend_running_v.JPG?raw=true) |
|:--:|
| *Backend sovellus onnistuneesti julkaistu* |

Tämän jälkeen aloin miettimään onko konttini koodi nyt tarpeeksi hyvä, vai voisiko julkaisua vieläkin helpottaa. Teemun vinkillä päädyin
tekemään jar-tiedoston luomisen imagen luonnin yhteydessä. Tällä tavalla ei jatkossa enää tarvetta suorittaa maven buildia eclipsessa
jokaisen muutoksen jälkeen.

Dockerfile sisällöksi tuli lopulta (muutaman epäonnistumisen jälkeen):

```
FROM maven:3.5-jdk-8-alpine as builder

WORKDIR /app

COPY . .

RUN mvn clean package

FROM openjdk:latest

EXPOSE 8080

WORKDIR /app

COPY --from=builder /app/target/backend_openshift-0.0.1-SNAPSHOT.jar .

CMD ["java", "-jar", "backend_openshift-0.0.1-SNAPSHOT.jar"]
```

Nyt samassa tiedostossa luodaan kaksi imagea. Ensimmäinen image buildaa sovelluksen ja luo jar-tiedoston, jonka seuraava image ottaa
sitten ainoaksi tiedostokseen ja ajaa lopuksi tiedoston. Testasin lopuksi toiminnan - toimi.

Lopuksi vielä suljin palvelimellla kaikki reitit. Mikäli tietoja palvelimelta halutaan, on annettava Firebasen kirjautumisen yhteydessä
saatava token (idToken) jokaisen pyynnön headerissa.

```
@Configuration
@EnableWebSecurity
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {

	@Override
    protected void configure(HttpSecurity http) throws Exception {
		http.csrf().disable().cors().and().authorizeRequests()
                .anyRequest()
                .authenticated();

        http.oauth2ResourceServer()
                .jwt();
    }
```

Pushasin muokatun imagen Openshiftiin -> kun kontti oli jo linkitetty, niin buildasi automaattisesti uuden sovelluksen käyttöön ilman 
katkosta -> lopputulos tämän osalta saavutettu.

## Käyttöliittymä React

Aloitin uuden React sovelluksen ja latasin samalla käyttööni mui -komponenttikirjaston sekä Firebase -kirjaston.
Luotuun projektiin lisäsin juurihakemistoon kaksi uutta tiedostoa: .env (ympäristömuuttujat) ja .dockerignore (mitkä tiedostot jätetään
pois kun konttiin tiedostoja kopioidaan Dockerfilessa). 

Ympäristömuuttujien pitää Reactissa alkaa aina "REACT_APP_MUUTTUJANIMI". Lisäsin tiedostoon aiemmin Firebasesta saamani tiedot, kuten
API avaimen ja osoitteen. Lisäksi laitoin yhdeksi muuttujaksi REACT_APP_BACKEND_URL, myöhempää käyttöä varten. Alkuun tässä muuttujassa 
oli Openshiftissä pyörivän palvelimen osoite.

Käyttöliittymästä oli tarkoitus tehdä mahdollisimman yksinkertainen (ajan säästämisen vuoksi). Ainoat mitkä olivat tärkeitä, olivat
kirjautumissivu sekä rekisteröintisivu. Sovelluksen lopullinen koodi löytyy [täältä](https://github.com/niikari/tori_frontend). 

Lopulta tein Dockerfilen ja loin tästä paikallisen imagen:

```
FROM node:17-alpine

WORKDIR /app

COPY package.json .

RUN npm install

COPY . .

EXPOSE 3000

CMD ["npm", "start"]
```

| ![kuva7.jpg](https://github.com/niikari/ohjelmistotekniikoiden-seminaari/blob/main/photos/docker_react_v1.JPG?raw=true) |
|:--:|
| *Kontissa pyörivä React sovellus ja testattu selaimella* |

## React sovelluksen vieminen Openshiftiin

Seuraavaksi tein samoin kuin palvelimen imagen kanssa - linkitin imagen nimeen haluamani paikan osoitteen Openshiftin palvelimella:

	$ docker tag frontend default-route-openshift-image-registry.apps.hhocp.otaverkko.fi/niilesseminaari/frontend:latest

	// Tämän jälkeen puskin imagen 

	$ docker push default-route-openshift-image-registry.apps.hhocp.otaverkko.fi/niilesseminaari/frontend

Kun lautaus oli valmis siirryin takaisin Openshiftiin. Suoritin jälleen saman vaiheen kuin palvelimen latauksen yhteydessä (Container
image). Nimesin kontin 'appfrontend', kerroin portiksi 3000 -> create.

| ![kuva8.jpg](https://github.com/niikari/ohjelmistotekniikoiden-seminaari/blob/main/photos/openshift_frontend_v1.JPG?raw=true) |
|:--:|
| *Kontti liitetty mukaan verkkoon, error viesti näkyvissä* |

Tarkastin virhelokin ja sieltä löytyi seuraava virhe:

```
Error: EACCES: permission denied, open '/app/public/env.js'
at Object.openSync (node:fs:586:3)
at Object.writeFileSync (node:fs:2171:35)
at Object.<anonymous> (/app/node_modules/react-dotenv/src/cli.js:37:4)
at Module._compile (node:internal/modules/cjs/loader:1099:14)
at Object.Module._extensions..js (node:internal/modules/cjs/loader:1153:10)
at Module.load (node:internal/modules/cjs/loader:975:32)
at Function.Module._load (node:internal/modules/cjs/loader:822:12)
at Function.executeUserEntryPoint [as runMain] (node:internal/modules/run_main:77:12)
at node:internal/main/run_main_module:17:47 {
errno: -13,
syscall: 'open',
code: 'EACCES',
path: '/app/public/env.js'
}
```

Ongelmana oli kontin sisäiset käyttäjäoikeudet. Kansio, joka konttiin luotiin (/app) on luoutu root -käyttäjänä. Kontin suoritusta
ei tehdä root -käyttäjänä.

Yritin ratkoa tätä ongelmaa, mutta en siinä lopulta onnistunut. Päätin lisätä käyttöliittymän Openshiftiin sen tarjoaman Github
"builderin" avulla. Tämän jälkeen laitoin kontin käynnistyksen yhteydessä tarvittavat ympäristömuuttujat paikoilleen (.env -tiedoston
sisältö). REACT_APP_BACKEND_URL yritin ensin laittaa vain kontin nimen (koska on Openshiftissä samassa verkossa): 'backend:8080', mutta
tämä ei toiminut -> käyttöliittymä ei löytänyt palvelinta. Päädyin lopulta laittamaan oikean fyysisen url-osoitteen - toimi.

| ![kuva9.jpg](https://github.com/niikari/ohjelmistotekniikoiden-seminaari/blob/main/photos/openshift_frontend_env.JPG?raw=true) |
|:--:|
| *Ympäristömuuttujien asettaminen ja tallennus* |

## Lopputulos

Lähdin testaamaan lopputulosta ja tässä vaiheessa kohtasin ongelman. Palvelimen kontin kellonaika on virheellinen, heittää
useammalla tunnilla. Tarkalleen 3 tunnin heitto. Kun lähden rekisteröimään käyttäjää käyttöliittymän puolelta: annan 
sähköpostiosoitteen, salasanan ja tämän jälkeen käyttöliittymä luo onnistuneesti käyttäjän Firebasen palveluun. Rekisteröinti
palauttaa sovellukselle idtokenin, jonka lähetän palvelimelle (/api/users) headerissa yhdessä uuden käyttäjän sähköpostiosoitteen
kanssa. Nyt tämä vaihe antoi virheen:

```
POST https://backend-niilesseminaari.apps.hhocp.otaverkko.fi/api/users net::ERR_CERT_AUTHORITY_INVALID
```

Tarkistin virheviestin ja tämä voi ilmeisesti johtua kahdesta syystä: tuosta kellonajan virheestä tai sitten siitä, että 
koska tämä palvelin toimii sertifikoimattomassa (itse sertifikoitu) url-osoitteessa. Päätin jättää ongelman ratkaisun kesken (aika).

Teoriassa kaikki toimii niinkuin pitääkin:

Käyttäjä voi rekisteröityä käyttäjäksi -> lisää käyttäjän Firebaseen luodun projektin käyttäjäksi -> lisää samalla käyttäjän
sähköpostin mukaan palvelimen tietokantaan -> tallentaa käyttäjän tiedot (kuten idtokenin, refreshtokenin) sovelluksen
tilan käyttöön.

Käyttäjä voi kirjautua palveluun -> Firebase tarkistaa sähköpostiosoitteen ja salasanan olevan oikeat -> palauttaa käyttäjän
tiedot (kuten idtokenin ja refreshtokenin) -> näiden tietojen avulla palvelimelta voidaan hakea tietoja laittamalla pyyntöön
header -kohtaan idtokenin.

| ![kuva10.jpg](https://github.com/niikari/ohjelmistotekniikoiden-seminaari/blob/main/photos/end_part1.JPG?raw=true) |
|:--:|
| *Käyttäjän rekisteröiminen (ei käyttäjää vielä Firebase projektissa)* |

| ![kuva11.jpg](https://github.com/niikari/ohjelmistotekniikoiden-seminaari/blob/main/photos/end_part2.JPG?raw=true) |
|:--:|
| *Rekisteröinti tehty, käyttäjä lisätty Firebase projektiin ja kirjauduttu sisään sovellukseen* |

| ![kuva12.jpg](https://github.com/niikari/ohjelmistotekniikoiden-seminaari/blob/main/photos/end_part3.JPG?raw=true) |
|:--:|
| *Käyttäjän tiedot lisätty palvelimelle ja sovelluksella käytössä tokenit sekä sähköpostiosoite* |























