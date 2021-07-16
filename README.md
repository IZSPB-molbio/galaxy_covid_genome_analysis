# Analisi genomi SARS-CoV-2 su Galaxy Europe

<!-- TOC START min:2 max:3 link:true asterisk:false update:true -->
- [Introduzione](#introduzione)
- [COVID-19: variation analysis on ARTIC PE data](#covid-19-variation-analysis-on-artic-pe-data)
    - [Workflow overview](#workflow-overview)
    - [Preparazione read dataset e file accessori](#preparazione-read-dataset-e-file-accessori)
    - [Avviare il workflow](#avviare-il-workflow)
    - [Output](#output)
- [COVID-19: consensus construction](#covid-19-consensus-construction)
    - [Avviare il workflow](#avviare-il-workflow-1)
    - [Output](#output-1)
- [PANGOLIN & NextClade](#pangolin--nextclade)
- [Sottomissione a I-Co-Gen](#sottomissione-a-i-co-gen)
- [Sottomissione a GISAID](#sottomissione-a-gisaid)
<!-- TOC END -->

## Introduzione

Per analizzare i dati di sequenziamento e ottenere i consensus FASTA vengono usati due workflow in sequenza, <u>già importati nell'account parisi.izspb@gmail.com su [usegalaxy.eu](https://usegalaxy.eu)</u>:

- **COVID-19: variation analysis on ARTIC PE data** per ottenere la lista delle varianti per ogni campione, a partire dalle read grezze;
- **COVID-19: consensus construction** per ottenere i FASTA consensus dalle varianti.

Fonte:

- https://covid19.galaxyproject.org/artic/#a-galaxy-workflow-for-the-analysis-of-illumina-paired-end-sequenced-artic-amplicon-data

## COVID-19: variation analysis on ARTIC PE data

### Workflow overview

Aprire immagine in una nuova scheda!

<img src="https://covid19.galaxyproject.org/assets/img/wf.14034fd2.png"
     alt="COVID-19: variation analysis on ARTIC PE data" />

### Preparazione read dataset e file accessori

#### Caricamento collection

##### Caricare fastq grezzi

Per un qualche motivo, caricare i fastq direttamente come collection non funziona. Pertanto è necessario caricare i fastq prima come regular file e poi creare una collection a partire da questi. Pertanto:

- Selezionare Upload Data > Regular > Choose local files
- Selezionare tutti i file (R1 e R2)
- Selezionare Start
- Al termine dell'upload, cliccare su Close

##### Creare collection

Video tutorial: https://www.youtube.com/watch?v=6toVj35q1r0&t=3s

- Dalla History corrente, selezionare il simbolo "check" per effettuare operazioni su dataset multipli
- Selezionare tutti i file fastq
- Dal menù a tendina "For all selected..." selezionare "Build list of dataset pairs"

A questo punto bisogna definire un criterio di definizione dei file fastq forward e dei file reverse per poi appaiarli. I predefiniti `_1` e `_2` non funzionano perché i nostri fastq sono definiti con `_R1` e `_R2`, che vanno quindi opportunamente inseriti (`_R1` al posto di `_1` e `_R2` al posto di `_2`). Dopo aver fatto ciò, le coppie di file dovrebbero già comparire, ognuna con una casella "Pair these datasets" al centro. Cliccando su "Auto-pair" gli appaiamenti dovrebbero comparire. Le caselle centrali conterranno il nome del campione, ad esempio da `PT669_S30_L001_R1_001.fastq.gz` e `PT669_S30_L001_R2_001.fastq.gz` verrà suggerito il nome campione `PT669_S30_L001_001.fastq.gz`. <u>Potrebbe valer la pena cambiare (manualmente) i nomi campioni assegnati automaticamente, in quanto questi nomi verranno usati ad es. per i consensus FASTA finali</u> (quindi ad es cambiare `PT669_S30_L001_001.fastq.gz` in `PT669`). 

- Assegnare un nome alla collection, ad es `run_210715`
- Selezionare "Create collection"

La collection così generata potrà essere usata come input per il workflow.

#### File accessori

Oltre alla collection di read grezze, il workflow richiede dei file accessori, inclusi in questa repository e scaricati dal sito di riferimento di questo workflow (quindi è indifferente se vengono usate le versioni caricate qui o quelle scaricate seguendo i rispettivi link). Questi file saranno caricati su Galaxy all'avvio del workflow (sezione successiva).

- [Sequenza di riferimento in formato FASTA](https://www.ncbi.nlm.nih.gov/nuccore/NC_045512.2?report=fasta) - file NC_045512.2.fasta
- [BED file](https://covid19.galaxyproject.org/artic/nCoV-2019_v3.bed) con le posizioni dei primer ARTIC V3 sulla sequenza di riferimento - file nCoV-2019_v3.bed
- [Amplicon info file](https://covid19.galaxyproject.org/artic/ARTIC_SARS_CoV-2_amplicon_info_v3.tsv) con le info sugli appaiamenti dei primer - file ARTIC_SARS_CoV-2_amplicon_info_v3.tsv.

### Avviare il workflow

- Selezionare (dal menù in alto bordato in blu) "Workflow"
- Cliccare sul tasto Play blu sulla destra del workflow "COVID-19: variation analysis on ARTIC PE data"

Apparirà la schermata per impostare il workflow. Da qui vanno caricati i file accessori per la pipeline (per ognuno, cliccare su "Upload datasets" e caricare il file; dopodiché, cliccare su "Browse datasets" e selezionare il file caricato):

- "Fasta sequence of SARS-CoV-2": file NC_045512.2.fasta in questa repository (o scaricato seguendo il link nella sezione precedente)
- "Paired collection": dovrebbe comparire automaticamente la collection appena generata
- "ARTIC primer BED": file nCoV-2019_v3.bed in questa repository (o scaricato seguendo il link nella sezione precedente)
- "ARTIC primers to amplicon assignments": file ARTIC_SARS_CoV-2_amplicon_info_v3.tsv in questa repository (o scaricato seguendo il link nella sezione precedente)

A questo punto il workflow sarà pronto per essere avviato tramite il pulsante Play blu in alto a destra.

### Output

La pipeline produrrà molti output. Quelli più essenziali per capire la buona riuscita del run sono:

- **Preprocessing and mapping reports** fornisce info utili su numero di read, copertura del genoma etc 
- **Final SNPEff annotated variants** (collection di VCF annotati, uno per campione) sarà <u>l'input della pipeline per ottenere i consensus FASTA</u>.

## COVID-19: consensus construction

Il file di input di questa pipeline è la collection di VCF generata dal workflow precedente (**Final SNPEff annotated variants**). 

### Avviare il workflow

- Selezionare (dal menù in alto bordato in blu) "Workflow"
- Cliccare sul tasto Play blu sulla destra del workflow "COVID-19: consensus construction"

Apparirà la schermata per impostare il workflow.

- "Variant calls": selezionare **Final SNPEff annotated variants**
- "aligned reads data for depth calculation": selezionare **ivar removereads on collection...**.
- "Reference genome": dovrebbe già esserci il file NC_045512.2.fasta, se no riselezionarlo.

### Output

Output: `Multisample consensus FASTA`.

## PANGOLIN & NextClade

- https://pangolin.cog-uk.io/
- https://clades.nextstrain.org/

L'analisi su Nextclade indicherà quali genomi escludere (inizialmente) dall'upload su I-Co-Gen. Campioni che dall'analisi Nextclade risultano flaggati in rosso per **Mixed sites** o **Missing data** dovrebbero essere scartati. 

## Sottomissione a I-Co-Gen

Per la sottomissione a I-Co-Gen si può seguire la procedura riportata nel documento IRIDA21-ICoGen.pdf (in questa repository) a partire da pagina 22 (sezione "Invio batch mediante ftps"). Alcune note:

- Per collegarsi via FTP al server ISS bisogna aver installato un client tipo FileZilla
- Le credenziali per l'accesso sono su MEGA, nel file `Covid/icogen/icogen_batch.txt`.
- Per ottenere la lista dei nomi dei file da inserire nel file batch.csv si può usare il tool Galaxy "Extract element identifiers of a list collection" (nella sezione "Collection Operations"). Usando questo tool sulla collection generata all'inizio del workflow si ricava la lista dei campioni, che si può usare come base per immettere i nomi dei file fastq nel file batch.csv.
- Per generare l'archivio per la sottomissione, è importante selezionare tutti i file (batch.csv + tutti i fastq), cliccare col tasto destro e generare un archivio zip.

## Sottomissione a GISAID

Per la sottomissione a GISAID bisogna generare la tabella per la sottomissione e gli ID appropriati e creare una copia del `Multisample consensus FASTA` con i nuovi ID. Può essere utile in questo senso usare alcuni tool disponibili qui: https://users-birc.au.dk/palle/php/fabox/ 

Una possibile procedura potrebbe prevedere:

- **Estrazione degli ID da sostituire** con il tool [FASTA header extractor](https://users-birc.au.dk/palle/php/fabox/header_extractor.php)
- Utilizzare gli ID estratti per generare i nuovi ID, ovvero per passare da `PT669_S30_L001_001.fastq` a `hCoV-19/Italy/APU-IZSPB-669PT/2021`.
- **Sostituzione degli ID** con il tool [header replacer](https://users-birc.au.dk/palle/php/fabox/header_replacer.php). In particolare:
    - nella pagina del tool, caricare il multifasta
    - incollare gli ID estratti dal multifasta nel box "Optional: input header search terms (to search the old headers)"
    - incollare i nuovi ID nel box "Input the new headers, one per sequence"
    - cliccare su "replace headers".
    
