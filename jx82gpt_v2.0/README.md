# jx82gpt

Assistente da terminale per studio e lavoro quotidiano.

Il programma propone, spiega e prepara. Decidi sempre tu: nessun comando viene eseguito e nessun file viene scritto senza la tua conferma.

---

## Requisiti

- Go 1.21 o superiore (per compilare)
- Ollama installato e in esecuzione, per il modello locale (predefinito)
- Solo per la modalità cloud (`-a`): una chiave API di Anthropic o OpenAI

## Installazione

```bash
ollama pull devstral:24b          # oppure: ollama pull qwen3-coder:30b
go build -o jx82gpt jx82gpt.go
./jx82gpt -h
```

Per una build più piccola:

```bash
go build -trimpath -ldflags="-s -w" -o jx82gpt jx82gpt.go
```

---

## Uso rapido

```bash
./jx82gpt "cos'è il comando grep?"        # una domanda
./jx82gpt -i                               # chat continua, scrivi exit per uscire
cat errore.log | ./jx82gpt "perché fallisce?"   # analisi di un file o di un output
./jx82gpt -n -i                            # simulazione: non esegue nulla
```

---

## Opzioni (flag)

| Flag | Cosa fa | Esempio |
|------|---------|---------|
| `-i` | Avvia la chat interattiva. Scrivi `exit` o `quit` per uscire. | `./jx82gpt -i` |
| `-m` | Sceglie il modello. Vale per Ollama e per il cloud. | `./jx82gpt -m qwen3-coder:30b "domanda"` |
| `-n` | Simulazione (dry-run). Ti chiede comunque conferma, ma se approvi mostra solo che il comando non è stato eseguito. | `./jx82gpt -n -i` |
| `-p` | Nome del progetto. I file creati finiscono in `<work_root>/<nome>/`. Se non lo indichi, viene creato un progetto con data e ora. | `./jx82gpt -p esercizio1 -i` |
| `-w` | Cartella generale dove stanno tutti i progetti. | `./jx82gpt -w ~/progetti -i` |
| `-a` | Usa un modello cloud invece di Ollama. All'avvio chiede conferma, perché i messaggi lasciano la tua macchina. | `./jx82gpt -a -provider anthropic -i` |
| `-provider` | Sceglie il fornitore cloud: `anthropic` (predefinito con `-a`) oppure `openai`. Funziona solo insieme a `-a`. | `./jx82gpt -a -provider openai "domanda"` |
| `-h` | Mostra il menu di aiuto. | `./jx82gpt -h` |

Note sui flag:

- Senza `-a` il programma usa sempre Ollama, anche se nel file di configurazione hai scritto un altro provider.
- `-a` non è compatibile con l'input da pipe. Usa il terminale.
- Con `-m` il modello indicato ha la precedenza su quello del file di configurazione.

---

## Come si risponde alle domande

Il programma ti chiede conferma in tre situazioni:

1. **Comando shell** (blocchi `bash`, `sh` o `zsh`). Vedi la shell che verrà usata, la cartella di lavoro e il comando completo. Rispondi `s` per eseguirlo, invio per saltarlo.
2. **File proposti** (formato `FILE: percorso` seguito da un blocco di codice). Vedi l'elenco con dimensioni e se il file esiste già. Rispondi `s` per salvare, invio per annullare.
3. **File già esistente**. Scegli `o` per sovrascrivere, `c` per salvare una copia con data e ora, invio per saltare. Il default è saltare.

Le risposte accettate per "sì" sono `s`, `si`, `sì`, `y` e `yes`. Qualsiasi altra cosa, compreso invio, vale come "no".

L'output di un comando approvato torna nella conversazione, così puoi chiedere spiegazioni nel turno successivo.

---

## Configurazione

Il file opzionale è `~/.jx82mygpt/config.json`. Puoi indicare un percorso diverso con la variabile `JX82_CONFIG`.

```json
{
  "provider": "ollama",
  "model": "qwen3-coder:30b",
  "ollama_url": "http://127.0.0.1:11434",
  "temperature": 0.1,
  "timeout_seconds": 600,
  "shell_timeout_seconds": 300,
  "max_messages": 40,
  "spend_cap_tokens": 200000,
  "work_root": "/home/tu/jx82-progetti",
  "log_path": "/home/tu/.jx82mygpt/log.jsonl"
}
```

| Chiave | Significato |
|--------|-------------|
| `model` | Modello predefinito (puoi sovrascriverlo con `-m`). |
| `ollama_url` | Indirizzo del server Ollama. |
| `temperature` | Creatività delle risposte locali: valori bassi danno risposte più prevedibili. |
| `timeout_seconds` | Tempo massimo per una risposta del modello. |
| `shell_timeout_seconds` | Tempo massimo per l'esecuzione di un comando. |
| `max_messages` | Quanti messaggi tenere in memoria nella conversazione. |
| `spend_cap_tokens` | Tetto di token per sessione in modalità cloud. `0` significa nessun tetto. |
| `work_root` | Cartella generale dei progetti. |
| `log_path` | File del registro delle azioni. |

Ordine di precedenza: flag della riga di comando, poi `config.json`, poi valori predefiniti.

### Prompt personalizzato

Crea `~/.jx82mygpt/prompt.txt` per cambiare il comportamento del modello. Se il file non c'è o è vuoto, il programma usa il prompt incorporato.

### Modelli predefiniti

- Locale: `devstral:24b`
- Cloud con Anthropic: `claude-sonnet-5-5`
- Cloud con OpenAI: `gpt-4o`

Se usi un modello locale diverso, indicalo con `-m` oppure in `config.json`.

---

## Variabili d'ambiente

| Variabile | Uso |
|-----------|-----|
| `ANTHROPIC_API_KEY` | Chiave per il cloud Anthropic. |
| `OPENAI_API_KEY` | Chiave per il cloud OpenAI. |
| `JX82_CONFIG` | Percorso alternativo del file di configurazione. |
| `NO_COLOR` | Se impostata, disattiva i colori. |

Le chiavi non vengono mai mostrate a schermo né scritte nel registro. I comandi approvati non le ricevono nel loro ambiente.

---

## File e cartelle

| Percorso | Contenuto |
|----------|-----------|
| `~/.jx82mygpt/config.json` | Configurazione (facoltativa). |
| `~/.jx82mygpt/prompt.txt` | Prompt personalizzato (facoltativo). |
| `~/.jx82mygpt/log.jsonl` | Registro di ogni domanda, decisione, comando e file. |
| `~/jx82-progetti/<progetto>/` | File creati dal modello, per progetto. |

---

## Sicurezza

- Nulla viene eseguito o scritto senza conferma esplicita.
- I file possono stare solo dentro la cartella del progetto. Percorsi assoluti, `..` e symlink vengono rifiutati.
- I file più grandi di 1 MB vengono rifiutati.
- Un comando che non parte o supera il tempo massimo viene segnalato, non ignorato.
- Con input da pipe le conferme ricevono risposta vuota, quindi nulla viene eseguito o salvato.
- Il prompt predefinito non fornisce codice pensato per attaccare sistemi di terzi. Usa il programma per studio, difesa e laboratori autorizzati.

---

## Risoluzione dei problemi

| Messaggio | Cosa fare |
|-----------|-----------|
| `Ollama non raggiungibile` | Avvia Ollama con `ollama serve`. |
| `modello ... non trovato` | Installa il modello con `ollama pull <nome>`. |
| `variabile ... non impostata` | Imposta la chiave API nell'ambiente prima di usare `-a`. |
| `Permission denied` | Esegui `chmod +x jx82gpt`. |
| Caratteri strani nei menu | Imposta `NO_COLOR=1` oppure usa un terminale con UTF-8. |
| `con -a non è supportato l'input da pipe` | Lancia il programma da terminale, senza pipe. |
