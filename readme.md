# Infra ja devops

## Kuvaus projektista

Tavoitteena minulla oli tehdä projekti, jossa koodia voisi jatkuvasti kehittää, versioida sekä julkaista helposti ja nopeasti.
Julkaisupaikaksi valitsin tunnilla käsitellyn Openshiftin (Otaverkko). Lisäksi päätin, että käyttäjien rekisteröiminen ja 
tunnistaminen tapahtuu jonkin ulkopuolisen toimijan avulla - tähän valitsin juuri käyttämäni Google Firebasen.

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
Lopullinen palvelimen koodi löydettävissä [täältä](https://github.com/niikari/tori_backend).

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
