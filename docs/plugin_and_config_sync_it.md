# Guida alla Sincronizzazione di Plugin e Configurazioni

Questa guida spiega come funziona la **Sincronizzazione Personalizzazioni (Beta3)** in
Self-hosted LiveSync, perché i plugin non si installano automaticamente, e il flusso
corretto per trasferire plugin e impostazioni tra dispositivi.

---

## Come funziona davvero

Quando un elemento arriva dal dispositivo remoto durante la replica, LiveSync aggiorna
il suo elenco interno di configurazioni note — **non scrive nulla sul disco
automaticamente**. Non esiste nessuna installazione automatica in background. La
scrittura su disco richiede uno dei due metodi descritti di seguito.

La finestra di Sincronizzazione Personalizzazioni si apre dall'icona nella barra
laterale oppure dal pannello comandi
(`Self-hosted LiveSync: Open customization sync`).

---

## Le due strategie di sincronizzazione

### 1. Selettiva / Selettiva con Contrassegno — manuale, decidi tu caso per caso

Questa è la modalità **predefinita** per ogni elemento. La riga mostra un'interfaccia
di confronto con un menu a tendina per selezionare il dispositivo sorgente e dei
indicatori di stato. Non succede nulla finché non scegli una sorgente e clicchi
su applica.

Usala quando vuoi controllo pieno su cosa viene aggiornato e quando.

### 2. Automatica — gestita da HiddenFileSync, senza intervento manuale

Quando imposti un elemento su Automatico (✨), i suoi file vengono affidati al modulo
HiddenFileSync e copiati su disco automaticamente ad ogni replica. **Devi configurare
questa modalità esplicitamente su ogni dispositivo.** Quando la selezioni, ti viene
chiesta un'azione iniziale:

| Opzione | Cosa fa |
|---|---|
| `↑ Overwrite Remote` | Carica subito la versione di questo dispositivo nel DB |
| `↓ Overwrite Local` | Scarica subito la versione remota su questo dispositivo |
| `⇅ Use newer` | Confronta i timestamp e prende la versione più recente |

> **Nota:** La modalità Automatica copia i file su disco ma non ricarica i plugin di
> Obsidian. Dopo un aggiornamento via modalità Automatica potrebbe essere necessario
> riavviare Obsidian una volta.

---

## Riferimento modalità per elemento

Clicca il pulsante emoji a sinistra di ogni riga per cambiare la modalità.

| Emoji | Modalità | Comportamento |
|---|---|---|
| 🔀 | **Selettiva** | Predefinita. Mostra l'interfaccia di confronto completa. Nulla si sincronizza finché non applichi manualmente. |
| ✨ | **Automatica** | Affidata a HiddenFileSync. I file si sincronizzano automaticamente ad ogni replica. |
| ⛔ | **Ignora** | L'elemento viene completamente saltato — non scansionato, non mostrato, non sincronizzato. |
| 🚩 | **Selettiva con Contrassegno** | Uguale a Selettiva, ma l'elemento viene selezionato dal pulsante "Seleziona Contrassegnati Lucenti". Usala per marcare gli elementi che vuoi applicare in blocco con un clic. |

---

## Riferimento pulsanti

| Pulsante | Cosa fa |
|---|---|
| **Scan changes** | Legge ogni file in `.obsidian/` su questo dispositivo e lo scrive nel database locale, etichettato con il nome di questo dispositivo. Eseguilo sul dispositivo sorgente prima di sincronizzare. |
| **Sync once** | Avvia una replica singola tra il database locale e il server remoto. Usalo dopo Scan changes sulla sorgente, e prima di Refresh sulla destinazione. |
| **Refresh** | Rilegge il database locale e ricostruisce l'elenco visualizzato. Usalo dopo la sincronizzazione per vedere gli stati aggiornati. Non legge il file system. |
| **Reload** | *(Solo modalità Manutenzione)* Svuota l'intero elenco in memoria e lo ricostruisce da zero. Un reset più profondo di Refresh. |
| **Select All Shiny** | Per ogni elemento, seleziona automaticamente il dispositivo remoto con la versione modificata più di recente. |
| **🚩 Select Flagged Shiny** | Come Select All Shiny, ma solo per gli elementi in modalità Selettiva con Contrassegno (🚩). Gli elementi in modalità Selettiva semplice vengono ignorati. |
| **Deselect all** | Annulla tutte le selezioni. Nessuna applicazione verrà effettuata. |
| **Apply All Selected** | Scrive i file del dispositivo selezionato su disco per ogni elemento con una sorgente selezionata, poi ricarica i plugin interessati. |

---

## Indicatori di stato

Quando selezioni un dispositivo remoto dal menu a tendina di una riga, compaiono tre
indicatori di stato:

| Indicatore | Etichetta | Significato |
|---|---|---|
| 📅 | `Local only` | L'elemento esiste solo su questo dispositivo; il remoto non ha nulla. |
| 📅 | `Remote only` | Il remoto ha questo elemento ma non questo dispositivo. Applica è attivo. |
| 📅 | `Newer (Xm Ys)` | La versione remota è più recente di quel delta. Applica è attivo. |
| 📅 | `Older (Xm Ys)` | La versione remota è più vecchia. Puoi comunque applicarla per tornare indietro. |
| 📅 | `Same` | I timestamp sono entro 10 secondi l'uno dall'altro. |
| 📄 | `Same` | Tutti i contenuti dei file sono identici byte per byte. |
| 📄 | `Same or local only` | Nessun contenuto differisce; alcuni file potrebbero esistere solo in locale. |
| 📄 | `Different` | Almeno un file ha contenuto diverso. Applica e confronto (⮂) sono attivi. |
| 📄 | `Mixed` | Alcuni file corrispondono, altri differiscono, altri mancano da un lato. |
| 🏷️ | `Same` / `Lower` / `Higher` | Confronto della versione del manifesto del plugin. |

### "All the same or non-existent"

Questo messaggio appare quando **non ci sono altri dispositivi da confrontare** per
quell'elemento. Significa una delle seguenti cose:

- Nessun altro dispositivo ha ancora sincronizzato questo elemento (il DB remoto non
  ha voci per esso).
- La copia di ogni altro dispositivo è identica e "Nascondi elementi non applicabili"
  è attivo.
- L'elemento semplicemente non esiste ancora nel database.

Se lo vedi sul dispositivo di destinazione dopo la sincronizzazione, i dati non sono
ancora arrivati — esegui di nuovo **Sync once** e poi **Refresh**.

> **Se lo vedi su tutti gli elementi del desktop:** è normale — il desktop sta cercando
> i dati degli altri dispositivi. Significa che il tablet non ha ancora eseguito
> Scan changes. Vai sul tablet e segui i passi della sezione seguente.

---

## Flusso: inviare plugin dal desktop al tablet per la prima volta

### Opzione A — Manuale (trasferimento una tantum)

**Sul desktop (sorgente):**

1. Apri Sincronizzazione Personalizzazioni.
2. Clicca **Scan changes** — carica lo stato di `.obsidian/` nel DB locale.
3. Clicca **Sync once** — invia il DB al server remoto.

**Sul tablet (destinazione):**

4. Apri Sincronizzazione Personalizzazioni.
5. Clicca **Sync once** — scarica i dati dal server remoto.
6. Clicca **Refresh** — l'elenco dovrebbe ora mostrare il nome del desktop in ogni
   riga.
7. Ogni riga dovrebbe mostrare `Newer (...)` o `Remote only` negli indicatori di stato.
8. Clicca **Select All Shiny** per selezionare automaticamente la versione desktop per
   ogni elemento.
9. Clicca **Apply All Selected** — i plugin vengono scritti su disco e ricaricati.
10. Se un elemento `CONFIG` è stato applicato, Obsidian chiederà di riavviare.

### Opzione B — Automatica (continua, senza intervento)

Fai questo una volta per ogni elemento sul **tablet**:

1. Apri Sincronizzazione Personalizzazioni sul tablet.
2. Completa prima il trasferimento manuale sopra, in modo che l'elemento esista
   localmente.
3. Clicca il pulsante modalità (🔀) sulla riga dell'elemento.
4. Seleziona **✨ Automatica**.
5. Scegli **`↓ Overwrite Local`** se il desktop ha sempre la versione autorevole,
   oppure **`⇅ Use newer`** se entrambi i dispositivi possono aggiornare l'elemento
   indipendentemente.

Da questo momento, quell'elemento si sincronizza automaticamente ad ogni replica senza
aprire la finestra.

---

## Lista di controllo prerequisiti

Se non appare nulla o gli stati sono sbagliati, controlla prima queste cose:

- [ ] **I nomi dispositivo sono unici su ogni dispositivo** (Impostazioni LiveSync →
  Generale). Ogni dispositivo deve avere un nome diverso. Se due dispositivi
  condividono lo stesso nome si sovrascrivono a vicenda nel DB e producono messaggi
  di log confusi come `STORAGE -x> DB:ix:desktop/plugin_data/...: (config) already
  deleted (Not found on database)`. Quel messaggio significa che il dispositivo che
  scansiona non ha trovato voci nel DB con il proprio nome — quasi sempre causato da
  un nome dispositivo duplicato.
- [ ] **Il nome dispositivo è impostato** su entrambi i dispositivi. Senza un nome,
  `scanAllConfigFiles` si interrompe silenziosamente e non carica nulla.
- [ ] **Sincronizzazione Personalizzazioni è abilitata** nelle Impostazioni LiveSync
  (`usePluginSync: true`).
- [ ] Il desktop ha eseguito **Scan changes** almeno una volta. Finché non lo fa, il
  DB remoto non ha voci e il tablet mostra "All the same or non-existent" per tutto.
- [ ] Nessun elemento sul tablet è impostato su **⛔ Ignora** quando ti aspetti che
  si sincronizzi.

---

## Reimpostare lo stato memorizzato di un dispositivo (in caso di corruzione)

Se le voci di un dispositivo nel DB sono obsolete, errate o provengono da un
dispositivo rinominato:

1. Apri Sincronizzazione Personalizzazioni.
2. Abilita la **Modalità Manutenzione** (casella in fondo alla finestra).
3. Appare un selettore "Delete All of [dispositivo]" in cima.
4. Scegli il nome del dispositivo da pulire e clicca 🗑️.
5. Sul dispositivo interessato: clicca **Scan changes** (ricarica lo stato attuale del
   disco), poi **Sync once**, poi **Refresh**.

Fai questo solo se gli stati sono genuinamente sbagliati. Non è necessario per una
prima configurazione normale.

---

## Risoluzione problemi

| Sintomo | Causa probabile | Soluzione |
|---|---|---|
| "All the same or non-existent" su tutti gli elementi | Il desktop non ha eseguito Scan changes, o la sincronizzazione non è completata | Esegui Scan changes sul desktop → Sync once su entrambi → Refresh sul tablet |
| Gli elementi appaiono ma Applica non fa nulla | La modalità dell'elemento è ⛔ Ignora | Cambia la modalità in 🔀 Selettiva |
| I file del plugin appaiono ma il plugin non si carica | Obsidian necessita di un riavvio dopo la scrittura dei file | Riavvia Obsidian sul tablet |
| Un elemento in modalità Automatica non si sincronizza | L'azione iniziale non è mai stata eseguita | Elimina l'impostazione di modalità, reimpostala su Automatica, scegli un'azione iniziale |
| Il nome del dispositivo manca dal menu a tendina | Quel dispositivo non ha mai eseguito Scan changes | Esegui Scan changes su quel dispositivo |
