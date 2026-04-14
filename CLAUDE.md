# Rottien & Hiirten Pyydystys – CLAUDE.md

## Projektin kuvaus

PWA-sovellus rottien ja hiirten pyydystystapahtumien kirjaamiseen. Käyttäjä kirjaa saaliit päivämäärällä, sovellus näyttää reaaliaikaisen listan ja tilastot. Toimii mobiilissa offline-tilassa.

## Tiedostorakenne

```
index.html          – koko sovellus, yksi tiedosto
firebase-config.js  – Firebase-projektin tunnisteet (ei salaisia avaimia)
manifest.json       – PWA-manifest
sw.js               – Service worker, välimuisti
icon-192.png        – PWA-ikoni
icon-512.png        – PWA-ikoni
CLAUDE.md           – tämä tiedosto
```

## Teknologiat

- **Frontend**: Vanilla JS + HTML/CSS, ei bundleria, ei frameworkia
- **Tietokanta**: Firebase Firestore (reaaliaikainen, `onSnapshot`)
- **Auth**: Firebase Anonymous Authentication
- **PWA**: Service worker + Web App Manifest
- **Hosting**: GitHub Pages

## Firebase-projekti

- **Projekti**: `rottien-pyydystys`
- **Kokoelma**: `saaliit`
- **Dokumentin kentät**:
  - `aika` – Firestore Timestamp
  - `laji` – string, sallitut arvot: `'rotta'` | `'hiiri'`
  - `uid` – string, kirjautuneen käyttäjän Firebase UID

## Firestore Security Rules

Säännöt rajoittavat pääsyn kirjautuneille käyttäjille. Tärkeimmät periaatteet:
- `read`: vaatii `request.auth != null`
- `create`: validoi kentät (`laji`, `aika`, `uid`) + tarkistaa että `uid == request.auth.uid`
- `delete`: vain oman merkinnän voi poistaa (`resource.data.uid == request.auth.uid`)
- `update`: estetty kokonaan (`if false`)

## Arkkitehtuuripäätökset

- **Yksi HTML-tiedosto**: helppo jakaa ja deployata, ei build-prosessia
- **Anonyymi auth**: ei vaadi käyttäjätunnuksia, silti Firestore-säännöt toimivat
- **DOM-rakentaminen**: lista rakennetaan `createElement`+`textContent`-metodeilla, ei `innerHTML`-interpolaatiolla (XSS-suojaus)
- **`confirm()` korvattu**: poistovahvistus käyttää omaa modal-dialogia selaimen natiivin sijaan

## Service Worker

- Välimuistiavain: `rotat-v2` – **muista päivittää** (`rotat-v3` jne.) aina kun tiedostoja muutetaan, jotta käyttäjät saavat uuden version
- Välimuistissa: `./`, `index.html`, `manifest.json`, `firebase-config.js`
- Strategia: network-first, fallback välimuistiin

## Tietoturvahuomiot

- `firebase-config.js` sisältää API-avaimen selkotekstinä – tämä on normaalia Firebasessa, turvallisuus perustuu Firestore-sääntöihin
- Älä lisää Firestore-sääntöihin `allow read, write: if true` – se avaa datan kaikille
- `laji`-kenttä validoidaan sekä clientissä (`SALLITUT_LAJIT`-lista) että Firestore-säännöissä

## Kehitysmuistio

- Ei erillistä build-vaihetta – muokkaa tiedostoja suoraan ja pushaa GitHubiin
- GitHub Pages deployaa automaattisesti `main`-haarasta
- Testaa PWA-toiminnallisuus mobiililla tai Chromen DevToolsin Application-välilehdellä
