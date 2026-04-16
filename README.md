# Gjennomstrømningsvarmer med PID-kontroll

Semesterprosjekt i emnet **52TE01I – Måle- og reguleringsteknikk** (10 studiepoeng) ved Fagskolen Rogaland, linje Automatisering.

Prosjektet utvikler og analyserer en matematisk modell av en gass-gjennomstrømningsvarmer for utendørs dusj, og sammenligner P-, PI-, PD- og PID-regulering i MATLAB/Simulink og Python.

## Innhold

- [Bakgrunn](#bakgrunn)
- [Problemstilling](#problemstilling)
- [System og modell](#system-og-modell)
- [Mappestruktur](#mappestruktur)
- [Forutsetninger](#forutsetninger)
- [Slik kjører du simuleringene](#slik-kjører-du-simuleringene)
- [Resultater](#resultater)
- [Forfattere](#forfattere)
- [Veileder](#veileder)
- [Lisens](#lisens)

## Bakgrunn

Prosjektets opprinnelige mål var å bygge en komplett temperaturregulator basert på mikrokontroller (ESP32-C3). På grunn av frafall av et gruppemedlem ble omfanget justert til modellering, simulering og analyse av temperaturregulering i MATLAB/Simulink og Python.

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
├── README.md                       # Dette dokumentet
├── rapport/
│   └── Rapport_Gjennomstromningsvarmer.docx
├── simulink/
│   ├── heater_model.slx            # Simulink-modell (hoved)
│   ├── HeaterODE.m                 # MATLAB Function-blokk (energibalanse)
│   └── run_simulation.m            # Kjøreskript med parametersett
├── python/
│   ├── heater_sim.py               # Python-implementasjon av modellen
│   ├── pid.py                      # PID-regulator med anti-windup
│   ├── plots.py                    # Plotting av sprangrespons og forstyrrelse
│   └── requirements.txt
├── figurer/                        # Genererte plott brukt i rapporten
└── vedlegg/
    ├── forprosjektrapport.pdf
    └── statusrapporter/
```

> Juster stinavn etter ditt faktiske oppsett før du pusher.

## Forutsetninger

**MATLAB/Simulink:**
- MATLAB R2023a eller nyere
- Simulink
- Control System Toolbox (for PID Controller-blokken)

## Slik kjører du simuleringene

**MATLAB/Simulink:**

```matlab
cd simulink
run_simulation
```

## Regulatorparametere

Verdier valgt gjennom manuell tuning etter metoden i pensum [1, kap. 1.4.5]:

| Regulator | Kp | Ki | Kd | N (derivatfilter) |
|---|---|---|---|---|
| P | 0,3 | 0 | 0 | – |
| PI | 0,3 | 0,1 | 0 | – |
| PD | 0,3 | 0 | 0,2 | 100 |
| PID | 0,3 | 0,1 | 0,2 | 100 |

Anti-windup: clamping med utgangsbegrensning [0, 1].

## Resultater

Sprangrespons ved SP = 40 °C, Tin = 10 °C, q = 8 L/min:

| Regulator | Stasjonærfeil [°C] | Overshoot [°C] | Innsvingingstid [s] |
|---|---|---|---|
| P | 1,87 | 0 | Aldri (permanent) |
| PI | 0 | 4,7 | 14 |
| PD | 2 | 0 | Aldri (permanent) |
| PID | 0 | 3,8 | 14 |

**Hovedfunn:**
- Kun regulatorer med I-ledd (PI og PID) oppnår null stasjonærfeil.
- PID gir lavere overshoot enn PI fordi D-leddet demper stigningen nær settpunktet.
- Forstyrrelse (q: 8 → 12 L/min) kompenseres av PI/PID innen ca. 7 sekunder; P og PD får varig avvik.
- Ved uoppnåelig settpunkt (SP = 60 °C, q = 16 L/min, Tin = 5 °C) kreves ~61 kW, mens tilgjengelig effekt er ~27 kW. Uten anti-windup bygger integralleddet seg opp ubegrenset; med clamping responderer regulatoren umiddelbart når belastningen synker.

## Forfattere

- **Stian Emil Johansen** — Moreld Apply, feltingeniør. Fagbrev automatiker.
- **Håkon Hjertnes Simonsen** — Solstad Offshore, skipselektriker / teknisk inspektør. Elektriker med gruppe L.

## Veileder

Hafrun Hauksdottir, Fagskolen Rogaland.

## Lisens

Dette er en studentinnlevering ved Fagskolen Rogaland og publiseres for vurdering og dokumentasjon. All gjenbruk utenfor studiesammenheng krever samtykke fra forfatterne.
