# Gjennomstrømningsvarmer med PID-kontroll

Semesterprosjekt i emnet **52TE01I – Måle- og reguleringsteknikk** (10 studiepoeng) ved Fagskolen Rogaland, linje Automatisering.

Prosjektet utvikler og analyserer en matematisk modell av en gass-gjennomstrømningsvarmer for utendørs dusj, og sammenligner P-, PI-, PD- og PID-regulering i MATLAB/Simulink.

## Innhold

- [Bakgrunn](#bakgrunn)
- [Problemstilling](#problemstilling)
- [System og modell](#system-og-modell)
- [Mappestruktur](#mappestruktur)
- [Forutsetninger](#forutsetninger)
- [Slik kjører du simuleringen](#slik-kjører-du-simuleringen)
- [Regulatorparametere](#regulatorparametere)
- [Resultater](#resultater)
- [Forfattere](#forfattere)
- [Veileder](#veileder)
- [Lisens](#lisens)

## Bakgrunn

Prosjektets opprinnelige mål var å bygge en komplett temperaturregulator basert på mikrokontroller (ESP32-C3). På grunn av frafall av et gruppemedlem ble omfanget justert til modellering, simulering og analyse av temperaturregulering i MATLAB/Simulink.

## Problemstilling

**Hovedproblemstilling:** Hvordan kan utgående vanntemperatur i en gjennomstrømningsvarmer reguleres ved hjelp av PID-basert regulering, og hvordan påvirkes reguleringskvaliteten av variasjoner i vannmengde og innløpstemperatur?

**Delproblemstillinger:**
- Hvordan kan en gjennomstrømningsvarmer modelleres som en termisk prosess basert på energibalanse?
- Hvordan påvirker endringer i vannmengde reguleringssystemets dynamikk og stabilitet?
- Hvordan håndterer regulatoren forstyrrelser i innløpstemperatur?
- Hvordan påvirker pådragsmetning reguleringskvaliteten og statisk avvik?
- Hvilke regulatorinnstillinger gir tilfredsstillende respons for en rask termisk prosess?

## System og modell

Prosessen er en gass-gjennomstrømningsvarmer (32 kW, maks 16 L/min) for utendørs dusj. Modellen er basert på energibalanse:

```
C · dT/dt = η · P − Q_tap + mdot · cp · (Tin − T)
```

Energibalansen er implementert som en `MATLAB Function`-blokk (`HeaterODE`) inne i Simulink-modellen. PID-regulatoren, summasjonspunkt, forstyrrelser og scope-blokker er plassert på toppnivå i samme `.slx`-fil.

**Parameterverdier brukt i modellen:**

| Parameter | Symbol | Verdi | Enhet |
|---|---|---|---|
| Spesifikk varmekapasitet vann | cp | 4186 | J/(kg·K) |
| Tetthet vann | ρ | 1,0 | kg/L |
| Forbrenningsvirkningsgrad | η | 0,85 | – |
| Vannvolum i veksler | V | 0,25 | L |
| Termisk kapasitet vann | C | 1046,5 | J/K |
| Varmetapskoeffisient | UA_loss | 10 | W/K |
| Omgivelsestemperatur | T_amb | 10 | °C |
| Maks varmeeffekt | P_max | 32 000 | W |

## Mappestruktur

```
.
├── README.md                                 # Dette dokumentet
├── gjennomstromningsvarmer.slx               # Simulink-modell med HeaterODE, PID og forstyrrelser
├── rapport/
│   └── Rapport_Gjennomstromningsvarmer.docx
├── presentasjon/
│   └── presentasjon_Gjennomstromningsvarmer.pptx
```

> Juster filnavn etter ditt faktiske oppsett før du pusher.

## Forutsetninger

- MATLAB R2023a eller nyere
- Simulink
- Control System Toolbox (for `PID Controller`-blokken)

## Slik kjører du simuleringen

1. Åpne `gjennomstromningsvarmer.slx` i MATLAB.
2. Verifiser at PID-blokken har parametersettet du vil teste (se tabell under).
3. Trykk **Run** i Simulink-verktøylinjen.
4. Temperaturrespons, pådrag og feil vises i Scope-blokkene.

For å veksle mellom P-, PI-, PD- og PID-modus endres Kp-, Ki- og Kd-verdiene direkte i PID-blokken.

## Regulatorparametere

Verdier valgt gjennom manuell tuning etter metoden i pensum [1, kap. 1.4.5]:

| Regulator | Kp | Ki | Kd | N (derivatfilter) |
|---|---|---|---|---|
| P | 0,03 | 0 | 0 | – |
| PI | 0,03 | 0,03 | 0 | – |
| PD | 0,03 | 0 | -0,002 | 2 |
| PID | 0,03 | 0,03 | -0,002 | 2 |

Anti-windup: clamping med utgangsbegrensning [0, 1].

## Resultater

Sprangrespons ved SP = 40 °C, Tin = 10 °C, q = 8 L/min:

| Regulator | Stasjonærfeil [°C] | Overshoot [°C] | Innsvingingstid [s] |
|---|---|---|---|
| P | 12,3 | Ingen | Permanent avvik |
| PI | 0 | 1,9 | 4 |
| PD | 12,3 | Ingen | Permanent avvik |
| PID | 0 | 1,9 | 4 |

**Hovedfunn:**
- Kun regulatorer med I-ledd (PI og PID) oppnår null stasjonærfeil. P og PD stabiliserer seg rundt 27,7 °C — pådraget u = Kp · e = 0,03 × 12,3 ≈ 37 % er akkurat nok til å holde likevekt, men ikke nok til å nå settpunktet.
- PI gir raskest innsvinging i dette tilfellet (≈ 4 s, overshoot 1,9 °C). PID gir samme ytelse her, men er mer robust mot forstyrrelser og endringer i prosessdynamikken.
- Forstyrrelse (q: 8 → 12 L/min ved t = 15 s) kompenseres av PI/PID innen ca. 6 sekunder. P og PD får et varig nytt avvik — stasjonærfeilen øker fra ca. 12,3 °C til ca. 15,2 °C.
- Ved uoppnåelig settpunkt (SP = 60 °C, q = 16 L/min, Tin = 5 °C) kreves ~61 kW, mens tilgjengelig effekt er ~27 kW. Uten anti-windup bygger integralleddet seg opp ubegrenset; med clamping responderer regulatoren umiddelbart når belastningen synker.

## Forfattere

- **Stian Emil Johansen** — Moreld Apply, feltingeniør. Fagbrev automatiker.
- **Håkon Hjertnes Simonsen** — Solstad Offshore, skipselektriker / teknisk inspektør. Elektriker med gruppe L.

## Veileder

Hafrun Hauksdottir, Fagskolen Rogaland.

## Lisens

Dette er en studentinnlevering ved Fagskolen Rogaland og publiseres for vurdering og dokumentasjon. All gjenbruk utenfor studiesammenheng krever samtykke fra forfatterne.
