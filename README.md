# Setup Mac Apple Silicon con mise

Un solo file, `config.toml`, descrive il software e la configurazione del Mac. Lo stesso comando serve per la prima installazione e per gli aggiornamenti successivi.

## Installazione e aggiornamenti

Al primo utilizzo completa la preparazione descritta sotto. Per installare o aggiornare il setup, chiudi le app da aggiornare, compreso Warp, e apri **Terminale macOS**:

```sh
cd ~
mise bootstrap --yes
```

Il comando installa ciò che manca, applica i blocchi shell, aggiorna runtime, pacchetti, Oh My Zsh ed estensioni VS Code, salva un checkpoint e verifica la configurazione. `--yes` evita le conferme di mise, ma non le eventuali richieste di password o autorizzazioni macOS.

## Preparazione del primo Mac

1. Usa un Mac **Apple Silicon**, con i Command Line Tools Apple installati. Se mancano, esegui `xcode-select --install` e attendi il completamento.
2. Installa manualmente [mise](https://mise.jdx.dev/getting-started.html), versione **2026.10.7 o successiva**, e verifica che `mise --version` funzioni nel Terminale.
3. Salva il file di questo progetto come `~/.config/mise/config.toml`, creando la cartella se necessario. Se esiste già una configurazione, integra i contenuti prima di sostituirla. Usa i percorsi standard; configurazioni personalizzate di mise, XDG o `ZDOTDIR` richiedono adattamento.
4. Leggi il file e autorizzalo una volta:

```sh
mise trust ~/.config/mise/config.toml
```

Poi esegui il comando principale. Non anteporre `sudo`: mise richiederà i privilegi quando servono. Non occorre installare Homebrew né copiare script o cartelle aggiuntive.

Al termine apri una nuova sessione Warp. Se macOS non usa ancora la nuova shell di login, esci e rientra nella sessione utente. Completa i login delle app e il primo avvio di OrbStack per Docker.

## Software incluso

| Categoria | Contenuto |
| --- | --- |
| Runtime | Bun, Node, pnpm, Python, uv, Rust con rustfmt, clippy e rust-src |
| CLI | Git, curl, zsh, GitHub CLI, ripgrep, jq, fd, fzf, zoxide, Claude Code, Codex |
| App | Warp, VS Code, Chrome, Yaak, OrbStack, Claude, ChatGPT |
| Shell | Oh My Zsh, tema robbyrussell, plugin Git, zoxide con comando j, integrazione fzf fuori da Warp, attivazione mise e zsh come shell di login |
| Estensioni VS Code | Codex, Claude Code, Rust Analyzer, Python, Pylance, Even Better TOML |
| Configurazione iniziale | Git, ignore globale, SSH GitHub, Warp e VS Code |

## Cosa significa «latest»

Tutti i runtime e pacchetti configurati richiedono `latest`. **Node segue la release stabile corrente, non soltanto LTS.** Ogni bootstrap completo esegue un aggiornamento esplicito; per i runtime consulta nuovamente le versioni remote e conserva quelle precedenti per i progetti che le usano. I progetti possono fissare le proprie versioni.

Per app e pacchetti, latest è la versione disponibile nel catalogo supportato, che può arrivare dopo la release del produttore. Mise può saltare app aperte, versioni già più recenti o non confrontabili. Le app già gestite da Homebrew restano di competenza di Homebrew. Un’app esistente non adottabile produce un errore invece di essere sostituita forzatamente. [Regole di adozione e aggiornamento](https://mise.jdx.dev/bootstrap/packages/brew.html).

Il setup aggiorna **solo il software elencato nel TOML**, non macOS o tutte le app del computer.

## Aggiornamenti automatici

Nel file globale sono attivi:

```toml
[settings]
self_update.auto = "1d"
tool_update.global_auto = "24h"
upgrade.auto_prune = false
```

Mise controlla il proprio aggiornamento prima dei comandi interattivi idonei, al massimo una volta al giorno. Per i runtime globali, controlla al prompt della shell attivata oppure all’avvio tramite shim/`mise exec`; al prompt l’aggiornamento avviene in background. Non è un timer che gira a Mac inattivo. Le versioni esatte e quelle definite dai progetti restano protette. Le vecchie versioni vengono conservate.

L’aggiornamento automatico rispetta le condizioni native, incluse modalità offline/CI e l’eventuale età minima delle release; un controllo giornaliero non garantisce di installare una release appena pubblicata. Controlla `mise doctor` per eventuali errori.

App/cask, pacchetti bootstrap, Oh My Zsh ed estensioni VS Code continuano ad aggiornarsi con `mise bootstrap --yes` o `mise run setup:update`. Fonti: [auto-update runtime](https://mise.jdx.dev/dev-tools/#automatic-tool-updates), [impostazioni mise](https://mise.jdx.dev/configuration/settings.html#self_update-auto).

## Personalizzazione

I file Git, ignore globale, SSH, Warp e VS Code vengono inizializzati **solo se assenti**. Le impostazioni già presenti vengono conservate, salvo gli alias Git elencati sotto e due opzioni gestite a ogni applicazione dei dotfiles: `rerere.enabled=true` e `rerere.autoupdate=false`. Git può riutilizzare risoluzioni di conflitti già registrate, lasciandole da controllare prima dello staging. Gli altri default del TOML non sono imposti su file esistenti. Nessuna pulizia dei profili o dei dati delle app è prevista.

I blocchi marcati in `~/.zprofile` e `~/.zshrc` sono gestiti da mise: a ogni applicazione vengono aggiornati, mentre il testo esterno resta intatto. L’attivazione copre zsh login e interattivo. Per script che non leggono questi file usa `mise exec -- comando`.

Modifica `~/.config/mise/config.toml` per cambiare pacchetti, versioni o blocchi shell, poi ripeti il comando principale. Per personalizzare preferenze già inizializzate modifica il relativo file o usa l’app. `.zshrc.local` e `.gitconfig.local` sono estensioni locali facoltative, caricate dai default del setup e non tracciate. Modificare la copia nella cartella di questo progetto non modifica automaticamente quella installata.

## Uso della shell

Zoxide impara le cartelle visitate: usa `j nome` per saltare a una cartella e `ji nome` per sceglierla con fzf. Visita prima le nuove cartelle con `cd`; il database è locale. In Warp usa `j` o `ji`: le completions speciali Space+Tab di zoxide non sono supportate dal terminale. Nelle sessioni zsh con editor di riga attivo, fuori da Warp, fzf aggiunge ricerca cronologia (`Ctrl-R`), selezione file (`Ctrl-T`) e cartelle (`Alt-C`). Warp mantiene i propri controlli di input.

Apri una nuova sessione per caricare le modifiche. Plugin e tema vanno impostati nel blocco TOML prima di caricare Oh My Zsh; `.zshrc.local` viene letto dopo.

## Alias Git

Gli alias sono comandi Git reali (`git st`, non soltanto alias della shell). Il bootstrap applica questi nomi anche a un gitconfig esistente, preservando identità, include e alias con altri nomi:

| Comando | Scopo |
| --- | --- |
| `git st`, `git co`, `git br`, `git ci` / `git cm` | Stato, checkout, branch e commit |
| `git cb` | Nome del branch corrente |
| `git amend` | Modifica l’ultimo commit |
| `git unstage` | Rimuove file dallo staging |
| `git unstash` | Applica l’ultimo stash e lo rimuove in caso di successo |
| `git last`, `git ls1` | Mostra l’ultimo commit |
| `git ls`, `git lsf`, `git lsn N` | Grafo con autore e data; solo prima linea di parentela; ultimi N commit |
| `git ll`, `git ld`, `git hist` | Log con statistiche; grafo compatto; branch ordinati per ultimo commit |
| `git showname`, `git diffname`, `git diffs` | Nomi dei file del commit; nomi dei file modificati; diff dello staging |
| `git fetchp`, `git pullr` | Fetch con pruning; pull con rebase |
| `git sha`, `git shal` | Hash breve o completo di HEAD |
| `git stashow N`, `git stashowname N`, `git stapply N` | Mostra, elenca i file o applica lo stash per indice (default 0) |
| `git shallow-clone`, `git partial-clone` | Clone con profondità 1 o filtro `tree:0` |
| `git aliases` | Elenca gli alias configurati |

## Controlli e problemi

Per controllare lo stato senza avviare un aggiornamento:

```sh
cd ~
mise run setup:verify
```

Il task controlla diagnostica mise, stato bootstrap, cronologia e disponibilità dei runtime in una nuova shell. Non verifica account, licenze o autenticazione SSH/GitHub. Per Docker, dopo aver aperto OrbStack, usa `docker info` e `docker compose version`.

Per un’anteprima:

```sh
mise bootstrap --dry-run
mise upgrade --dry-run
mise bootstrap packages upgrade --dry-run
```

L’anteprima bootstrap non esegue gli hook e non espande gli aggiornamenti del task finale; i due comandi successivi mostrano i piani di aggiornamento. `mise install` o `mise dot apply` da soli non sostituiscono il flusso completo.

Se il setup si interrompe, leggi l’errore, risolvi il prerequisito o il conflitto e rilancia `mise bootstrap --yes`: le fasi già completate restano applicate. Non cancellare configurazioni o dati per superare un conflitto. Dopo un aggiornamento app, macOS può richiedere nuovamente alcuni permessi.

## Configurazione facoltativa

Esegui i comandi seguenti da HOME (`cd ~`) dopo il bootstrap.

### Accesso SSH a GitHub

```sh
mise run setup:ssh
```

`mise run setup:ssh` usa `~/.ssh/id_ed25519_github` se presente, altrimenti riutilizza `~/.ssh/id_ed25519` (anche se creata da GitHub CLI); crea una chiave dedicata soltanto se entrambe mancano, verifica che abbia una passphrase e che la coppia di chiavi corrisponda, poi usa il Portachiavi Apple e copia la chiave pubblica negli appunti. Aggiungila nelle impostazioni SSH di GitHub. Una chiave esistente non viene sovrascritta. Il file SSH iniziale seleziona la stessa chiave. Se hai già un file SSH personalizzato, verifica che `IdentityFile` nel blocco GitHub punti alla chiave realmente registrata: il setup conserva quel file. La presenza della chiave sul sito GitHub non corregge un percorso locale errato.

### Identità Git personale e di lavoro

Per usare email diverse nei repository personali e di lavoro:

```sh
mise run setup:git-identity
```

Il task chiede nome, email personale, email di lavoro e percorso assoluto della cartella di lavoro. Salva i valori in `.gitconfig.local` e `.gitconfig-work`, non tracciati, e configura un include condizionale Git: i repository sotto la cartella scelta usano l’email di lavoro, gli altri quella personale. Le impostazioni del singolo repository possono prevalere. Il task non modifica i commit già esistenti. Il percorso e le email restano locali e non sono incorporati nel TOML riutilizzabile.

### Checkpoint e sincronizzazione dei dotfiles

Il servizio `mise-history` salva automaticamente la cronologia locale del TOML, di `.zprofile` e di `.zshrc`. Non traccia le chiavi private né tutte le preferenze delle app. Non inserire segreti nei file tracciati.

Per sincronizzare, crea un repository GitHub **privato e vuoto**, esegui `mise run setup:github`, quindi controlla `mise dot status` e `mise dot history` prima di `mise run dot:push`. Il push invia anche i checkpoint precedenti. Il collegamento iniziale non pubblica e la sincronizzazione resta manuale.

## Comandi di manutenzione

| Comando | Scopo |
| --- | --- |
| `mise run setup:update` | Aggiorna il software già installato senza riapplicare i blocchi shell; già incluso nel bootstrap completo |
| `mise run dot:save` | Salva un checkpoint locale |
| `mise run dot:push` | Salva e pubblica la cronologia sul remoto configurato |
| `mise run dot:pull` | Recupera i file dal remoto; esegui poi il bootstrap per applicare il setup ricevuto |
| `mise dot rollback ~/.zshrc --dry-run` | Mostra il ripristino di una versione precedente; togli `--dry-run` per applicarlo |
| `mise dot undo` | Annulla l’ultimo ripristino |

## File del progetto

- **`config.toml`**: unico file necessario al setup.
- **`README.md`**: istruzioni d’uso.
- **`AGENTS.md`**: indicazioni per lavorare sul repository.

Riferimenti: [bootstrap mise](https://mise.jdx.dev/bootstrap.html), [dotfiles e blocchi gestiti](https://mise.jdx.dev/dotfiles.html), [upgrade runtime](https://mise.jdx.dev/cli/upgrade.html).
