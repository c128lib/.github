## Hi there 👋

![c128lib logo 40 cols](./resources/logo-40.png "c128lib logo 40 column")![c128lib logo 80 cols](./resources/logo-80.png "c128lib logo 80 column")

Here you'll find set of useful KickAssembler libraries for native Commodore 128-development as well as a few supporting tools.

This work is strongly inspired by the [c64lib](https://github.com/c64lib) project (created by [Maciej Małecki](https://github.com/maciejmalecki)), extending it where necessary to support the functionality of the C128.

I'm also building a memory map reference available [here](https://c128lib.github.io/Reference).

Thank you for been here, if you want to contribute (to the project and/or the reference guide) you're welcome, skills required are:
* 6502/8502 assembly and architecture
* KickAssembler
* (obviusly) Git commit/pull/push/pull request and integration with GitHub

In this case, send me an email (raffaele.intorcia at gmail.com) with your GitHub account name.

If you are already a developer team member you can start with [team discussion page](https://github.com/orgs/c128lib/teams) or [project page](https://github.com/orgs/c128lib/projects/1)

## C128lib main description [ITA] (ENG follows)
### Intro libreria - Perchè?
Ho scelto di iniziare a scrivere questa libreria semplicemente perché volevo cimentarmi nella creazione di un gioco dedicato al Commodore 128. Mi sono trovato molto bene nell'utilizzare c64lib per realizzare giochi e programmi per C64 e volevo fare qualcosa di simile. Ho preso fortemente spunto da questa libreria per iniziare, è stato molto importante averla come riferimento ma ora lo sviluppo sta procedendo su una strada indipendente.

Ritengo sia fondamentale avere una base a disposizione per questa macchina perché è più complessa rispetto al C64 e un livello di astrazione semplifica molto. Inoltre nella maggior parte dei casi ne beneficia anche la leggibilità del codice.
Si tratta di un progetto opensource con licenza MIT, liberamente consultabile su GitHub e usabile nei propri progetti. La partecipazione allo sviluppo non è aperta a tutti ma su invito, come tutti i progetti a più mani è necessaria collaborazione e coordinamento degli sforzi.

### Funzionalità fornite
La libreria è suddivisa in sotto progetti che hanno ambiti specifici.

Il progetto **common** definisce le label per gli elementi non riferiti a specifici chip del sistema (riferimenti di memoria del kernal, dello screen editor ecc...). Inoltre definisce alcune funzionalità e macro di base per rendere più veloce la stesura del codice (copia di aree di memoria, trattamento della screen memory, gestione dello stack ecc...).

Il progetto **chipset** prevede le label e le macro utilizzate nello sviluppo di codice con specifici chip. Sono presenti i chip Cia, Sid, Vic (con supporto agli shadow register), Vdc e Mmu.
Esistono alcune macro per lavorare agevolmente con gli sprite, facendo riferimento diretto ai registri del Vic. Non manca la gestione del processore e delle aree di memoria (es. attivazione o disattivazione Rom).

Sta nascendo il progetto **framework**, ultimo in ordine di creazione. Lo scopo del progetto è fornire funzionalità più complesse, rispetto a quelle presenti nel common, che potenzialmente coprono più aspetti. A fronte di singole macro, è possibile svolgere azioni che coinvolgono più chip e più registri, in modo da dare una funzionalità completa.

A corredo di questi progetti, è in fase di realizzazione una memory reference per enumerare e descrivere le locazioni di memoria e i registri più importanti del C128.

Non escludo che possano nascere altri progetti, ma per ora c'è già molta carne al fuoco.

### Modalità uso
L'utilizzo della libreria è possibile con una serie di prerequisiti:
* utilizzo di KickAssembler come compilatore
* utilizzo di Gradle come build system

Il file build.gradle dovrà avere una struttura simile a questa:
```
plugins {
// importazione del plugin retro-assembler
    id "com.github.c64lib.retro-assembler" version "1.5.3"
}

repositories {
    mavenCentral()
}

retroProject {
    dialect = "KickAssembler"
    dialectVersion = "5.25"
// Indicazione del folder delle librerie (utilizzato dalle clausole import)
    libDirs = ["..", ".ra/deps/c128lib"]

// Indicazione delle librerie di c128lib da scaricare da GitHub (<organization>/<progetto>)
// e versione di riferimento
    libFromGitHub "c128lib/common", "0.5.0"
}
```

L'esecuzione del comando gradle farà partire lo scaricamento delle librerie indicate se non presenti e poi eseguirà la compilazione del progetto.

Nei file sorgente, le librerie si importano con il comando #import <path-libreria>. Si possono importare solo i file che definiscono le label (es "vic2.asm") o quelli che definiscono le macro (es. "vic2-global.asm"). 

### Differenza complessità codice
Per dare un chiaro esempio di aumentata leggibilità, partiamo dal codice sottostante:
```
    lda #$3F
    sta $D500
```
Conoscendo a memoria i registri del C128 si capisce su quale chip si sta lavorando. Sempre a memoria, il parametro $3F dovrebbe far capire le impostazioni che saranno attivate.

Considerando però questo codice:

```
c128lib_SetMMULoadConfiguration(Mmu.RAM0 | Mmu.ROM_HI_RAM | Mmu.ROM_MID_RAM | Mmu.ROM_LOW_RAM | Mmu.IO_RAM)
```
Risulta più leggibile il fatto che:
* si sta impostando la configurazione della memoria operando sul registro della MMU
* si sta attivando il primo blocco di RAM (cioè i primi 64 KB)
* la Screen Editor Rom è disattivata (su $C000-$FFFF ci sarà la RAM)
* la upper Basic Rom è disattivata (su $8000-$BFFF ci sarà la RAM)
* la lower Basic Rom è disattivata (su $4000-$7FFF ci sarà la RAM)
* la IO/ROM è disattivata (su $D000-$DFFF ci sarà la RAM)

Il risultato della macro è lo stesso del codice sopra indicato ma ci sono molte informazioni in più che facilitano la stesura del codice e, soprattutto, la rilettura a distanza di tempo.

Come esempio di semplificazione, vediamo il codice necessario a modificare background e foreground color nello schermo a 80 colonne. Si tratta della macro SetBackgroundForegroundColor() presente in [chipset/vdc.asm](https://github.com/c128lib/chipset/blob/e6bec215c7df912c3c9527aaef640fca74fec476/lib/vdc.asm#L130).
Scrivere da zero la funzionalità richiede 17 istruzioni. La macro permette di scrivere una sola riga, specificando solo il codice dei colori.

## C128lib main description [ENG]
### Intro - Why?
I chose to start writing this library simply because I wanted to try my hand at creating a game dedicated to the Commodore 128. I really enjoyed using c64lib to make games and programs for the C64 and I wanted to do something similar. I took a strong cue from this library to start with, it was very important to have it as a reference but now the development is proceeding on an independent road.

I think it is essential to have a base available for this machine because it is more complex than the C64 and a level of abstraction simplifies a lot. Furthermore, in most cases the readability of the code also benefits.
It is an open source project with MIT license, freely available on GitHub and usable in your own projects. Participation in development is not open to all but by invitation, as with all collaborative projects collaboration and coordination of efforts is required.

### Features provided
The library is divided into sub-projects that have specific scopes.

The **common** project defines the labels for the elements not related to specific chips of the system (memory references of the kernal, of the screen editor etc...). It also defines some basic functions and macros to speed up coding (copying memory areas, handling screen memory, stack management, etc...).

The **chipset** project provides the labels and macros used in the development of code with specific chips. There are the Cia, Sid, Vic (with shadow register support), Vdc and Mmu chips.
There are some macros to work easily with sprites, directly referencing the registers of the Vic. Do not miss the management of the processor and memory areas (e.g. activation or deactivation of Rom).

The **framework** project is being born, last in order of creation. The aim of the project is to provide more complex features, compared to those present in the common, which potentially cover more aspects. Against single macros, it is possible to carry out actions involving multiple chips and multiple registers, in order to give a complete functionality.

In support of these projects, a memory reference is being developed to enumerate and describe the most important memory locations and registers of the C128.

I'm not ruling out the possibility of other projects, but for now there's already a lot of irons on the fire.

### Usage mode
Using the library is possible with some prerequisites:
* use of KickAssembler as compiler
* use of Gradle as build system

The build.gradle file should have a structure similar to this:
```
plugins {
// importing the retro-assembler plugin
     id "com.github.c64lib.retro-assembler" version "1.5.3"
}

repositories {
     mavenCentral()
}

retroProject {
     dialect = "KickAssembler"
     dialectVersion = "5.25"
// Indication of the library folder (used by the import clauses)
     libDirs = ["..", ".ra/deps/c128lib"]

// Indication of the c128lib libraries to download from GitHub (<organization>/<project>)
// and reference version
     libFromGitHub "c128lib/common", "0.5.0"
}
```

Running the gradle command will start the download of the indicated libraries if not present and then will compile the project.

In source files, libraries are imported with the #import <library-path> command. Only files that define labels (eg "vic2.asm") or those that define macros (eg "vic2-global.asm") can be imported.

### Code complexity difference
To give a clear example of increased readability, let's start with the code below:
```
     lda #$3F
     sta $D500
```
Knowing C128 registers, you understand which chip you are working on. Again, the $3F parameter should make it clear which settings will be activated.

But considering this code:

```
c128lib_SetMMULoadConfiguration(Mmu.RAM0 | Mmu.ROM_HI_RAM | Mmu.ROM_MID_RAM | Mmu.ROM_LOW_RAM | Mmu.IO_RAM)
```
It is more readable that:
* the memory configuration is being set by operating on the MMU register
* the first block of RAM is being activated (i.e. the first 64 KB)
* Screen Editor Rom is disabled ($C000-$FFFF will have RAM)
* upper Basic Rom is disabled (on $8000-$BFFF there will be RAM)
* lower Basic Rom is disabled (on $4000-$7FFF there will be RAM)
* IO/ROM is disabled ($D000-$DFFF will have RAM)

The result of the macro is the same as the code indicated above but there is much more information that facilitates the drafting of the code and, above all, the re-reading after some time.

As an example of simplification, let's see the code needed to change the background and foreground color in the 80-column screen. This is the SetBackgroundForegroundColor() macro present in [chipset/vdc.asm](https://github.com/c128lib/chipset/blob/e6bec215c7df912c3c9527aaef640fca74fec476/lib/vdc.asm#L130).
Writing the feature from scratch requires 17 instructions. The macro allows you to write a single line, specifying only the color code.
