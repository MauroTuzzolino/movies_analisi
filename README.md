# Cosa fa guadagnare un film? Analisi di 3.200 titoli con Power BI

Progetto di analisi dati svolto in Power BI su un dataset di film (budget, incassi, generi, voti di critica e pubblico). L'obiettivo è rispondere a tre domande concrete e costruire una dashboard adatta alla condivisione su LinkedIn.

---

## 1. Obiettivo e domande

Partendo da un dataset reale, sporco e incompleto, il progetto vuole mostrare l'intero percorso di un'analisi: pulizia dei dati, modellazione, misure e comunicazione dei risultati.

| # | Domanda | Pagina della dashboard |
|---|---------|------------------------|
| 1 | Quali generi hanno il ritorno sull'investimento (incasso/budget) più alto? | Ritorno per genere |
| 2 | Un budget maggiore garantisce incassi maggiori, o il rapporto cambia per fasce di budget? | Budget e incassi |
| 3 | I voti della critica sono allineati a quelli del pubblico e agli incassi? | Critica e pubblico |

A queste si aggiunge una **copertina** con i numeri chiave.

## 2. Dati

- **Fonte:** `movies.json` dal repository [vega/vega-datasets](https://github.com/vega/vega-datasets).
- **Dimensione originale:** 3.201 righe e 16 colonne.
- **Contenuto:** titolo, incasso USA e mondiale, vendite DVD, budget di produzione, data di uscita, classificazione MPAA, durata, distributore, origine della storia, genere, tipo creativo, regista, voto Rotten Tomatoes (scala 1-100), voto IMDB (scala 1-10) e numero di voti IMDB.
- **Periodo coperto:** film dal 1915 al 2011, con la grande maggioranza dagli anni '90 in poi.
- **Formato:** il file era disponibile solo in JSON ed è stato convertito in CSV (Python/pandas) senza alcuna pulizia, per fare tutta la preparazione in Power Query.

## 3. Strumenti

- Power BI Desktop, Power Query (linguaggio M) e DAX
- Python/pandas, usato solo per la conversione da JSON a CSV e per verificare indipendentemente i numeri della dashboard
- Supporto di Claude (Anthropic) come tutor: pianificazione dei passaggi, controllo dei risultati e revisione

## 4. Come è stato svolto

### 4.1 Selezione delle colonne

Prima di pulire, ho deciso quali colonne servivano alle tre domande, per ridurre il lavoro successivo.

| Decisione | Colonne | Motivo |
|-----------|---------|--------|
| Eliminate | `US DVD Sales`, `Running Time min`, `Director`, `Distributor` | Vuoti rispettivamente nell'82%, 62%, 42% e 7% delle righe, e non servono alle domande |
| Tenute come filtro | `MPAA Rating` | Utile per segmentare |
| Tenute come opzionali | `Source`, `Creative Type` | Maggiore varietà per eventuali approfondimenti |
| Tenute | `Title`, `US Gross`, `Worldwide Gross`, `Production Budget`, `Release Date`, `Major Genre`, `Rotten Tomatoes Rating`, `IMDB Rating`, `IMDB Votes` | Necessarie alle domande |

### 4.2 Pulizia dei dati in Power Query

1. **Tipi di dato.** `IMDB Rating` usa il punto come separatore decimale, che il Power BI in italiano interpreta male: la colonna è stata convertita in numero decimale rispettando il formato originale.
2. **Date errate.** 22 film d'epoca avevano l'anno sbagliato di un secolo (per esempio *Via col vento* risultava del 2039). Poiché l'ultima data legittima è il 2011, ogni anno successivo è stato ridotto di 100. Dopo la correzione il periodo va dal 1915 al 2011. Molti film d'epoca hanno il 31 dicembre come data, che è un segnaposto: per le analisi temporali si usa l'anno.
3. **Duplicati.** Ci sono 24 titoli ripetuti, ma nessun doppione reale: sono remake o film omonimi di anni diversi (per esempio *Casino Royale* 1967 e 2006). La coppia titolo + anno identifica ogni film in modo univoco. Nessuna riga è stata eliminata per questo motivo.
4. **Valori mancanti.**
   - Eliminata l'unica riga senza titolo (3.200 righe finali).
   - Gli incassi a zero (47 film con incasso USA e mondiale entrambi a 0) sono stati trasformati in vuoti, perché indicano un dato mancante e non un incasso nullo.
   - I vuoti nelle colonne di testo (`Major Genre`, `MPAA Rating`, `Source`, `Creative Type`) sono diventati `Non specificato`.
   - I voti mancanti di Rotten Tomatoes (880) e IMDB (213) sono rimasti vuoti, senza imputazioni inventate.

### 4.3 Colonne calcolate

| Colonna | Scopo |
|---------|-------|
| `Anno`, `Decennio` | Analisi temporale |
| `Ritorno` | (incasso mondiale − budget) / budget |
| `Fascia Budget` | Cinque fasce: sotto 1M, 1-10M, 10-30M, 30-70M, oltre 70M |
| `Voto IMDB su 100` | Porta il voto del pubblico sulla scala di Rotten Tomatoes |
| `Incasso solo USA` | Segnala i film con incasso mondiale identico a quello USA |
| `Fascia Voto Critica` | Quattro fasce: sotto 40, 40-59, 60-79, 80 e oltre |

Le formule sono riportate in appendice.

### 4.4 Modello e misure

Nel modello i campi non sommabili (anno, voti, ritorno) sono stati impostati su "Non riepilogare", e i formati sono stati sistemati (valuta e percentuali). Misure DAX create: `Numero Film`, `Ritorno Mediano`, `Voto Critica Medio`, `Voto Pubblico Medio`, `% Film sopra Budget`, `Budget Mediano`, `Incasso Mediano`, `Divario Critica-Pubblico`.

### 4.5 Scelte metodologiche

- **Mediana invece di media.** Il ritorno ha valori estremi (*Paranormal Activity*, costato 15.000 dollari, ha un ritorno di circa 12.917 volte il budget), quindi la media sarebbe ingannevole.
- **Filtro `Incasso solo USA = No`.** Per il 40% circa dei film l'incasso mondiale coincide con quello USA, quasi sempre perché il dato estero manca (il 93-97% nei film degli anni '30-'50, il 23% negli anni 2000). Per confrontare film con lo stesso tipo di dato, la dashboard usa i **1.921 film** con incasso mondiale distinto da quello USA. Di questi, il 94% è del 1990 o successivo.
- **Soglia di 30 film per genere,** per evitare conclusioni su campioni troppo piccoli.
- **Almeno 1.000 voti IMDB** nella pagina su critica e pubblico, per escludere punteggi poco affidabili. Il campione diventa di **1.388 film**.

## 5. Risultati

### 5.1 Ritorno per genere

Con il filtro dei 1.921 film, il ritorno mediano complessivo è **+150%** (l'incasso è circa 2,5 volte il budget) e il **79%** dei film incassa più del proprio budget.

| Genere | Ritorno mediano |
|--------|-----------------|
| Horror | +216% |
| Adventure | +206% |
| Romantic Comedy | +171% |
| Action | +168% |
| Comedy | +139% |
| Drama | +113% |
| Thriller/Suspense | +89% |

**Conclusione:** l'horror è il genere più redditizio in mediana e il thriller il meno redditizio tra quelli analizzati. Controllando i film dal 2000 in poi l'horror resta primo (+200%), mentre l'avventura scende al terzo posto (+143%): la leadership dell'horror è robusta, quella dell'avventura meno.

**Nota:** includendo tutti i film, e non solo quelli con dati affidabili, il dramma scende a +38% e il western a +3%. Una parte di quel divario dipende dagli incassi esteri mancanti.

### 5.2 Budget e incassi

| Fascia budget | Film | Budget mediano | Incasso mediano | Ritorno mediano | % sopra budget |
|---------------|-----:|---------------:|----------------:|----------------:|---------------:|
| Sotto 1M | 62 | 0,34M | 3,5M | +959% | 92% |
| 1-10M | 287 | 4,8M | 16,2M | +258% | 80% |
| 10-30M | 589 | 18,0M | 45,3M | +157% | 77% |
| 30-70M | 597 | 45,0M | 95,0M | +116% | 77% |
| Oltre 70M | 385 | 90,0M | 228,9M | +125% | 83% |

**Conclusione:** un budget più alto porta incassi più alti in valore assoluto, ma non un ritorno più alto per dollaro investito. Il ritorno scende fino alla fascia 30-70M e poi risale leggermente per i blockbuster, che sono anche i più "sicuri" (83% sopra budget). La forma a U regge anche limitandosi ai film dal 2000 in poi (+70% nella fascia 30-70M, +123% oltre 70M). I due punti isolati del grafico a dispersione sono *Avatar* e *Titanic*.

**Cautela:** la fascia sotto 1M ha solo 62 film ed è probabilmente un campione di successi, perché per questi film il dato estero esiste di solito solo se hanno avuto distribuzione internazionale. Non va usata per trarre conclusioni.

### 5.3 Critica, pubblico e incassi

Su 1.388 film, il voto medio della critica è **53,6** e quello del pubblico **64,1**, con un divario di **−10,5** punti. La correlazione tra i due voti è circa **0,77**.

| Fascia voto critica | Film | Voto critica | Voto pubblico | Budget mediano | Incasso mediano | Ritorno mediano | % sopra budget |
|---------------------|-----:|-------------:|--------------:|---------------:|----------------:|----------------:|---------------:|
| Sotto 40 | 469 | 22,4 | 53,7 | 38,0M | 62,5M | +77% | 70% |
| 40-59 | 292 | 48,8 | 63,4 | 37,0M | 82,8M | +127% | 76% |
| 60-79 | 314 | 69,6 | 69,2 | 35,0M | 98,0M | +165% | 85% |
| 80 e oltre | 313 | 88,6 | 75,1 | 22,0M | 110,4M | +371% | 91% |

**Conclusioni:**

- Critica e pubblico sono correlati: quando i critici apprezzano un film, di solito piace anche al pubblico.
- Il divario non è uniforme. Sui film bocciati dalla critica il pubblico è molto più generoso (+31 punti), nella fascia 60-79 i due voti coincidono, e sui film più apprezzati il pubblico è più freddo (−14 punti circa). I voti del pubblico sono molto più "compressi" (deviazione standard 11,6 contro 26,9).
- I film più apprezzati dalla critica rendono di più (+371% contro +77%), ma costano anche meno (budget mediano 22M contro 38M), quindi parte del ritorno più alto è un effetto del budget.
- Il divario per genere è massimo nel thriller (−18,6) e nell'horror (−16,9), minimo nell'avventura (−5,4) e nel dramma (−5,8). Va letto con cautela: questi generi hanno anche voti critici più bassi, dove il divario è comunque più ampio, e con questa analisi non si può separare l'effetto del genere da quello del livello del voto.

## 6. Limiti dell'analisi

- **Il ritorno è lordo,** non un utile: l'incasso al botteghino non tiene conto della quota trattenuta dalle sale né dei costi di marketing.
- **Il dato sugli incassi esteri è incompleto,** soprattutto per i film più vecchi. Per questo l'analisi principale si limita ai film con incasso mondiale disponibile.
- **I due voti non sono confrontabili in modo diretto:** Rotten Tomatoes è una percentuale di recensioni positive, IMDB una media di voti degli utenti. Il divario di 10 punti non va letto come "i critici sono più severi" in senso stretto.
- **Il dataset si ferma al 2011** e ha alcuni errori residui (per esempio voti IMDB implausibili per alcuni film), quindi i casi più estremi non vanno usati come esempi.
- **Correlazione non è causalità:** nessuna delle relazioni trovate stabilisce un rapporto di causa ed effetto.

## 7. Problemi incontrati e soluzioni

| Problema | Soluzione |
|----------|-----------|
| Date d'epoca nel futuro (fino al 2046) | Correzione dell'anno sottraendo 100 per le date oltre il 2011 |
| Zeri negli incassi che falserebbero il ritorno | Trasformati in vuoti |
| Incasso mondiale uguale a quello USA | Flag `Incasso solo USA` e filtro di pagina |
| Grafico a dispersione: errore tra "Valori" e assi non riepilogati | Riepilogo su Somma con titolo e anno come chiave univoca |
| Punti schiacciati nel grafico a dispersione | Scala logaritmica su entrambi gli assi |
| Colonne delle fasce ordinate per valore | Ordinamento per fascia, con numeri nelle etichette |

## Appendice: formule

**Power Query (M)**

```
// Correzione data (colonna personalizzata)
if Date.Year([Release Date]) > 2011 then Date.AddYears([Release Date], -100) else [Release Date]

// Anno
Date.Year([Release Date])

// Decennio
Number.RoundDown([Anno] / 10) * 10

// Ritorno
([Worldwide Gross] - [Production Budget]) / [Production Budget]

// Fascia Budget
if [Production Budget] = null then null
else if [Production Budget] < 1000000 then "1. Sotto 1M"
else if [Production Budget] < 10000000 then "2. 1-10M"
else if [Production Budget] < 30000000 then "3. 10-30M"
else if [Production Budget] < 70000000 then "4. 30-70M"
else "5. Oltre 70M"

// Voto IMDB su 100
Number.Round([IMDB Rating] * 10, 0)

// Incasso solo USA
if [Worldwide Gross] = null then null
else if [Worldwide Gross] = [US Gross] then "Sì"
else "No"

// Fascia Voto Critica
if [Rotten Tomatoes Rating] = null then null
else if [Rotten Tomatoes Rating] < 40 then "1. Sotto 40"
else if [Rotten Tomatoes Rating] < 60 then "2. 40-59"
else if [Rotten Tomatoes Rating] < 80 then "3. 60-79"
else "4. 80 e oltre"
```

**DAX**

```
Numero Film = COUNTROWS(Movies)
Ritorno Mediano = MEDIAN(Movies[Ritorno])
Voto Critica Medio = AVERAGE(Movies[Rotten Tomatoes Rating])
Voto Pubblico Medio = AVERAGE(Movies[Voto IMDB su 100])
Budget Mediano = MEDIAN(Movies[Production Budget])
Incasso Mediano = MEDIAN(Movies[Worldwide Gross])
Divario Critica-Pubblico = [Voto Critica Medio] - [Voto Pubblico Medio]
% Film sopra Budget = DIVIDE(COUNTROWS(FILTER(Movies, Movies[Ritorno] > 0)), COUNT(Movies[Ritorno]))
```
