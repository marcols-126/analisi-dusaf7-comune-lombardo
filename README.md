# analisi-dusaf7-comune-lombardo
Strumento di Processing QGIS per l'analisi dell'uso del suolo basato su DUSAF 7.0 - Comuni lombardi
# Analisi DUSAF 7.0 - Comune Lombardo

Strumento di Processing per **QGIS 3.40+** che automatizza l'analisi dell'uso del suolo per qualsiasi Comune lombardo, basato sul dataset regionale **DUSAF 7.0** e sui confini amministrativi **ISTAT 2026**. Sviluppato come progetto finale del corso *IA applicata ai GIS* sul caso del Comune di Tromello (PV).

Il flusso copre l'intera pipeline - validazione input, riproiezione su EPSG:32632, fix geometries, clip sul confine comunale, audit numerico **QC-4**, calcolo superfici e tematizzazione automatica - producendo CSV, GeoPackage e log di audit pronti per l'istruttoria PGT.

## Caratteristiche principali

- **Tool di Processing nativo**: si carica in QGIS via *Processing → Crea nuovo script*, con GUI integrata.
- **Protocollo QC-4**: confronto automatico tra superficie DUSAF calcolata e perimetro ISTAT con tolleranza configurabile (default 1,0 m²); blocca il flusso e segnala discrepanze tra fonti cartografiche.
- **Riusabile su qualsiasi Comune lombardo**: il nome del Comune è un parametro di input con autocompletamento.
- **Fallback robusto sui nomi dei campi**: gestisce le varianti tra rilasci ISTAT e regionali.
- **Output multipli**: GeoPackage tematizzato, CSV delle superfici per classe e macro-categoria, layer slivers per QC.

## Prerequisiti

- QGIS 3.40 o superiore (testato anche su 3.44)
- I due layer di base **già caricati nel progetto QGIS attivo**:
  - `DUSAF7` - [Geoportale Regione Lombardia](https://www.geoportale.regione.lombardia.it/)
  - `Com01012026_WGS84` - [Confini amministrativi ISTAT 2026](https://www.istat.it/notizia/confini-delle-unita-amministrative-a-fini-statistici-al-1-gennaio-2018-2/)
- La cartella `stili/` di questo repository copiata nella cartella del progetto QGIS

## Installazione

1. Clona o scarica questo repository
2. Apri QGIS e il tuo progetto
3. Carica i due layer prerequisito (`DUSAF7`, `Com01012026_WGS84`)
4. Copia la cartella `stili/` nella cartella del progetto QGIS
5. *Processing → Strumenti → Cassetta degli strumenti → Script → Crea nuovo script*
6. Incolla il contenuto di `analisi_dusaf7_comune_lombardo.py` e salva
7. Lo strumento appare in *Cassetta degli strumenti → Script → Analisi Territoriale → Analisi DUSAF 7 - Comune Lombardo*

## Uso

1. Doppio click sullo strumento
2. Inserisci il nome del Comune (autocompletamento)
3. Imposta l'area minima slivers (default 1,0 m²)
4. *Esegui*

I risultati vengono salvati nella cartella `output_dusaf_<comune>/` del progetto:
- `<comune>_dusaf7_output.gpkg` - GeoPackage con DUSAF clip + superfici + slivers + confine fix
- `<comune>_dusaf7_superfici.csv` - tabella superfici per classe e macro-categoria

## Protocollo QC-4

Il nodo critico del flusso confronta:
- **Superficie del perimetro ISTAT** del Comune (calcolata in EPSG:32632)
- **Superficie DUSAF calcolata** dopo clip e fix geometries
- **Somma delle percentuali** (`pct_dusaf`) sul totale comunale

Se la differenza supera la tolleranza configurata (`AUDIT_TOLERANCE_M2 = 1.0`) il flusso emette un `[DATA AUDIT WARNING]` con i valori puntuali e invita a verificare confine ISTAT, CRS, geometrie, copertura DUSAF e slivers. La somma `pct_dusaf` deve essere coerente con 100,00 % entro tolleranza 0,0001 %.

## Licenza

Distribuito con licenza MIT - vedi [LICENSE](LICENSE).

## Autore

Marco Stefano La Sala - progetto finale del corso *IA applicata ai GIS* (2026).
