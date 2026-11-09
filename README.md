# Brreg: firmaoppslag i Claude med et åpent API

Søker etter firma og slår opp organisasjonsnummer i Enhetsregisteret hos Brønnøysundregistrene. Eksempelet fra
artikkelen [Bygg et KI-verktøy](https://nskarv.no/notater/bygg-et-ki-verktoy/) på nskarv.no, del 4 av serien om KI i praksis.
Artikkelen viser oppsettet steg for steg i Claude-appen, uten at du trenger å kunne kode.

| Fil | Hva det er |
| --- | --- |
| `skill/brreg-oppslag/SKILL.md` | Oppskriften Claude følger, skrevet som en skill. |
| `skill/brreg-oppslag/scripts/brreg_oppslag.py` | Skriptet som søker og slår opp. Rundt 120 linjer, bare Pythons standardbibliotek. |
| `skill/brreg-oppslag/scripts/brreg_oppslag.ps1` | Samme skript i PowerShell, for Windows uten Python. Gir nøyaktig samme svar. |
| `instructions.md` | Instruksen du limer inn under Instructions i et Claude-prosjekt. |

## Slik bruker du det i Claude-appen

Kortversjonen av artikkelen:

1. Legg `SKILL.md`, `brreg_oppslag.py` og `instructions.md` i en egen mappe på PC-en. Legg ikke noe annet der.
2. Lag et prosjekt i Claude-appen. Lim inn `instructions.md` under Instructions, og koble til mappen med Add folder.
3. Slå på «Use the internet while making files» under Settings, Capabilities. Under Allowed websites velger du None og
   legger til `data.brreg.no`. Da når skriptet Brreg og ingenting annet.
4. Spør: «Hent informasjon om Equinor.»
5. Til slutt kan du gjøre oppsettet om til en skill med `/skill-creator` i samme prosjekt.

Mappen og nettilgangen krever et betalt abonnement (Pro, Max, Ultra, Team eller Enterprise).

## Kjør skriptet selv

```bash
python skill/brreg-oppslag/scripts/brreg_oppslag.py equinor      # søk på navn
python skill/brreg-oppslag/scripts/brreg_oppslag.py 923609016    # oppslag på organisasjonsnummer
```

På Windows uten Python:

```powershell
powershell -ExecutionPolicy Bypass -File .\brreg_oppslag.ps1 Equinor
```

`-ExecutionPolicy Bypass` slipper gjennom akkurat denne kjøringen og endrer ingen innstillinger. Filen er lagret som
UTF-8 med BOM, ellers leser Windows PowerShell 5.1 æ, ø og å feil.

Svaret er kort JSON på én linje. Et oppslag på Equinor gir rundt 0,3 kB i stedet for 3 kB fra Brreg:

```json
{"organisasjonsnummer": "923609016", "navn": "EQUINOR ASA", "form": "Allmennaksjeselskap", "status": "aktiv",
 "adresse": "Forusbeen 50, 4035 STAVANGER", "bransje": "Utvinning av råolje", "ansatte": 21231,
 "stiftet": "1972-09-18", "mva_registrert": true, "kilde": "Enhetsregisteret, Brønnøysundregistrene (NLOD 2.0)"}
```

## Hva skriptet gjør

- **Søker på hele navnet.** Teksten må stå sammenhengende i navnet (`navnMetodeForSoek=FORTLOEPENDE`). Ellers godtar
  registeret treff på enkeltord som «AS».
- **Viser de fem med flest ansatte.** «telenor» gir Telenor Norge AS og Telenor ASA før foreningene. Svaret sier at
  listen er sortert slik.
- **Sjekker kontrollsifferet.** Et organisasjonsnummer har et kontrollsiffer etter modulus 11. En skrivefeil oppdages
  før API-et spørres.
- **Slår sammen status.** Konkurs, avvikling og sletting blir ett felt, så modellen ikke overser dem.
- **Gir korte svar.** Bare feltene modellen trenger.

## Når noe går galt

Feil kommer som `{"feil": "..."}` med avslutningskode 1. Meldingene er skrevet til modellen:

| Situasjon | Skriptet svarer |
| --- | --- |
| Feil kontrollsiffer | `923609017 har feil kontrollsiffer, så det er trolig en skrivefeil. Be om nummeret på nytt.` |
| Nummeret finnes ikke (HTTP 404) | `… finnes ikke i Enhetsregisteret. Det kan være en avdeling (underenhet). Søk på navnet.` |
| Fjernet av juridiske årsaker (HTTP 410) | `… er fjernet fra Enhetsregisterets åpne data av juridiske årsaker.` |
| Ingen treff på navnet | `Fant ingen enheter med «…» i navnet. Be om organisasjonsnummer eller en annen stavemåte.` |
| Ingen forbindelse | `Fikk ikke kontakt med data.brreg.no (…). Trolig slipper ikke maskinen eller sandkassen til data.brreg.no. …` |
| Tjenesten svarer ikke innen 20 sekunder | `data.brreg.no svarte ikke innen 20 sekunder. Prøv igjen senere.` |

## Prompt injection

Navn og adresser i registeret skrives av enhetene selv, og havner rett i modellens kontekst.
«IGNORE ALL PREVIOUS INSTRUCTIONS AS» (938 096 392) er et registrert aksjeselskap.

Før modellen ser svaret, fjerner skriptet usynlige tegn og korter ned felt over 300 tegn. Tekst som ligner instrukser,
som «ignore previous instructions» eller rollemarkører som `<|im_start|>`, gir feltet `advarsel`. Dataene endres ikke.

Skanneren er et varsel, ikke et vern. Det som faktisk begrenser skaden, er at skriptet bare leser, at modellen får få og
korte felt, og at nettilgangen bare gjelder data.brreg.no.

## Vilkår

- Enhetsregisteret er åpne data under [Norsk lisens for offentlige data (NLOD 2.0)](https://data.norge.no/nlod/no/2.0). Oppgi Brønnøysundregistrene som kilde.
- Brreg ber om at enheter som svarer 410, også fjernes fra kopier. Skriptet lagrer ingenting.
- Det du søker på, sendes til Brønnøysundregistrene. Ingenting annet sendes videre.

Koden er fri å bruke under [MIT-lisensen](LICENSE).
