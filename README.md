# Analisi DUSAF 7.0 - Comune Lombardo

Strumento di Processing per **QGIS 3.40+** che automatizza l'analisi dell'uso del suolo per qualsiasi Comune lombardo, basato sul dataset regionale **DUSAF 7.0** e sui confini amministrativi **ISTAT 2026**. Sviluppato come progetto finale del corso *IA applicata ai GIS* sul caso del Comune di Tromello (PV).

Il flusso copre l'intera pipeline: validazione input, riproiezione su EPSG:32632, fix geometries, clip sul confine comunale, audit numerico **QC-4**, calcolo superfici e tematizzazione automatica. Produce CSV, GeoPackage e log di audit pronti per l'istruttoria PGT.

## Caratteristiche principali

- **Tool di Processing nativo**: si carica in QGIS via *Processing → Crea nuovo script*, con GUI integrata.
- **Protocollo QC-4**: confronto automatico tra superficie DUSAF calcolata e perimetro ISTAT con tolleranza configurabile, default 1,0 m²; segnala nel log di Processing eventuali discrepanze tra fonti cartografiche e richiede una verifica metodologica prima dell'uso del dato.
- **Riusabile su qualsiasi Comune lombardo**: il nome del Comune è un parametro di input con autocompletamento e validazione.
- **Fallback robusto sui nomi dei campi**: gestisce le varianti tra rilasci ISTAT e regionali.
- **Output multipli**: GeoPackage tematizzato, CSV delle superfici per classe DUSAF, layer slivers per QC.

## Prerequisiti

- QGIS 3.40 o superiore, testato anche su QGIS 3.44.
- I due layer di base già caricati nel progetto QGIS attivo:
  - `DUSAF7` - Geoportale Regione Lombardia
  - `Com01012026_WGS84` - Confini amministrativi ISTAT 2026
- La cartella `stili/` di questo repository copiata nella cartella del progetto QGIS.

## Struttura consigliata della cartella progetto

```text
cartella_progetto/
│
├─ progetto.qgz
├─ analisi_dusaf7_comune_lombardo.py
├─ ISTRUZIONI_ANALISI_DUSAF7.txt
│
├─ stili/
│  ├─ Confine.qml
│  ├─ DUSAF7 - clip QC.qml
│  ├─ DUSAF7 - superfici.qml
│  ├─ DUSAF7.qml
│  └─ QC slivers DUSAF7.qml
│
├─ DUSAF7/
└─ Limiti01012026/
```

Le cartelle `DUSAF7/` e `Limiti01012026/` possono contenere i dati scaricati ed estratti dall'utente. I dati non sono inclusi nel repository per ridurre il peso del pacchetto e perché devono essere scaricati dalle fonti ufficiali.

## Installazione

1. Clona o scarica questo repository.
2. Apri QGIS e salva il progetto nella cartella di lavoro.
3. Scarica ed estrai i dati DUSAF 7.0 e i confini ISTAT 2026.
4. Carica nel progetto QGIS i due layer prerequisito:
   - `DUSAF7`
   - `Com01012026_WGS84`
5. Copia la cartella `stili/` nella cartella del progetto QGIS.
6. Apri *Processing → Cassetta degli strumenti*.
7. Vai in *Script → Crea nuovo script*.
8. Incolla il contenuto di `analisi_dusaf7_comune_lombardo.py` e salva.
9. Lo strumento appare in:

```text
Cassetta degli strumenti → Script → Analisi Territoriale → Analisi DUSAF 7 - Comune Lombardo
```

## Uso

1. Fai doppio click sullo strumento.
2. Inserisci il nome del Comune, selezionandolo tra i valori suggeriti dall'autocompletamento.
3. Imposta l'area minima degli slivers, default `1.0 m²`.
4. Premi *Esegui*.

Lo script non parte se il Comune digitato non corrisponde a un valore valido presente nel layer `Com01012026_WGS84`.

## Output

I risultati vengono salvati nella cartella del progetto:

```text
output_dusaf7_<comune>/
```

con file nominati con timestamp:

```text
<comune>_dusaf7_<timestamp>.gpkg
<comune>_dusaf7_superfici_<timestamp>.csv
```

Il CSV contiene le superfici per classe DUSAF:

```text
codice_dusaf
descrizione
area_m2
area_ha
pct_dusaf
pct_comune
```

Il GeoPackage contiene i layer principali del workflow:

```text
dusaf7_<comune>_superfici
dusaf7_<comune>_clip_qc
confine_<comune>_fix
qc_slivers_<comune>
```

## Stili QML

La cartella `stili/` deve contenere:

```text
Confine.qml
DUSAF7 - clip QC.qml
DUSAF7 - superfici.qml
DUSAF7.qml
QC slivers DUSAF7.qml
```

Se la cartella `stili/` non è presente nella cartella del progetto QGIS, lo script funziona comunque, ma i layer vengono caricati senza simbologia personalizzata.

## Protocollo QC-4

Il nodo critico del flusso confronta:

- **Superficie del perimetro ISTAT** del Comune, calcolata in EPSG:32632.
- **Superficie DUSAF calcolata** dopo clip e fix geometries.
- **Somma delle percentuali** (`pct_dusaf`) sul totale comunale.

La tolleranza configurata è:

```python
AUDIT_TOLERANCE_M2 = 1.0
```

Se la differenza supera la tolleranza configurata, il flusso emette un `[DATA AUDIT WARNING]` con i valori puntuali e invita a verificare:

- confine ISTAT;
- CRS;
- geometrie;
- copertura DUSAF;
- slivers.

La somma `pct_dusaf` deve essere coerente con `100,00 %` entro tolleranza `0,0001 %`.

## Workflow sintetico

```text
Caricamento layer nel progetto QGIS
        ↓
Validazione nome Comune
        ↓
Fix geometries input
        ↓
Riproiezione in EPSG:32632
        ↓
Estrazione Comune
        ↓
Clip DUSAF sul confine comunale
        ↓
Fix geometries post-clip
        ↓
Controllo slivers
        ↓
Dissolve per classe DUSAF
        ↓
Calcolo superfici m², ha e %
        ↓
Data Audit QC-4
        ↓
Esportazione GeoPackage e CSV
        ↓
Applicazione stili QML
```

## File principali

`analisi_dusaf7_comune_lombardo.py`

Script PyQGIS completo da installare nel Toolbox Processing di QGIS.

`ISTRUZIONI_ANALISI_DUSAF7.txt`

File di istruzioni operative sintetiche per utenti QGIS.

`stili/`

Cartella con gli stili QML applicati automaticamente agli output.

## Note metodologiche

Lo script è stato sviluppato per garantire un flusso ripetibile e controllabile. Prima di qualsiasi operazione di clip o calcolo, le geometrie vengono corrette tramite `native:fixgeometries`. Il calcolo delle superfici avviene in EPSG:32632, con unità metriche e conversione in ettari.

Il controllo sugli slivers segnala le geometrie residue inferiori o uguali alla soglia impostata dall'utente, default `1.0 m²`, senza rimuoverle automaticamente dal calcolo. Questo consente di mantenere tracciabilità completa del dato e di valutare manualmente eventuali anomalie.

## Licenza

Distribuito con licenza GNU Affero General Public License v3.0 (AGPL-3.0) - vedi LICENSE.

Questa licenza mantiene il codice sorgente aperto e impone che eventuali modifiche o riusi distribuiti, anche in contesti di servizio, rispettino gli obblighi della licenza AGPL-3.0.

## Autore

Marco Stefano La Sala - progetto finale del corso *IA applicata ai GIS* (2026).
