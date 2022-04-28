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








