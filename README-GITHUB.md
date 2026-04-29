# 🚌 EAV Bus Manager - Sistema GitHub

**Repository:** `Ropagnozzi/EAV-bus-data`

Sistema di gestione paline e interventi per linee bus Campania (1, 2, 3, 5, 6).
Sincronizzazione bidirezionale tra versione Desktop (pianificazione) e Mobile (esecuzione).

---

## 📁 STRUTTURA REPOSITORY

```
EAV-bus-data/
├── config.json                 ← Configurazione tipi manutenzione
├── paline.json                 ← Dati fissi paline (coordinate, percorsi)
├── interventi/
│   ├── 2025-04-29.json        ← Interventi di oggi
│   ├── 2025-04-28.json        ← Interventi ieri
│   └── ...
└── README.md                   ← Questo file
```

---

## 🔑 CONFIGURAZIONE INIZIALE

### Per Ropagnozzi (proprietario):

1. **Crea repository su GitHub:**
   ```
   Nome: EAV-bus-data
   Visibilità: Private (solo team vede i dati)
   Description: "Sistema gestione paline linee bus Campania"
   ```

2. **Carica i file iniziali:**
   - `config.json`
   - `paline.json`
   - `README.md`

3. **Genera Personal Access Token:**
   - Vai a: Settings → Developer settings → Personal access tokens
   - Crea nuovo token con permessi: `repo`, `read:user`
   - **Salva il token** (serve per Desktop e Mobile)

---

## 📊 FLUSSO DATI

### GIORNATA OPERATIVA:

```
┌─────────────────────────────────────────────────────────────┐
│ MATTINA: DESKTOP pianifica                                   │
├─────────────────────────────────────────────────────────────┤
│ 1. Apri Desktop v2                                          │
│ 2. Scegli DATA (es: 2025-04-29)                            │
│ 3. Scegli PALINE da visitare (checkbox)                     │
│ 4. Clicca "Prepara sessione"                               │
│ 5. Desktop genera file: interventi/2025-04-29.json         │
│ 6. Desktop fa COMMIT su GitHub                              │
│ 7. Desktop genera QR CODE                                   │
└─────────────────────────────────────────────────────────────┘
                           ↓
┌─────────────────────────────────────────────────────────────┐
│ GIORNATA: MOBILE esegue                                      │
├─────────────────────────────────────────────────────────────┤
│ 1. Mobile scansiona QR code da Desktop                       │
│ 2. Mobile scarica lista da GitHub                            │
│ 3. Mobile mostra ELENCO paline                              │
│ 4. Per ogni palina:                                         │
│    - Clicca → Apre Google Maps                             │
│    - Raggiunto → Clicca "Report"                           │
│    - Compila: Tipo manutenzione                            │
│    - Scatta: Foto Fronte (camera)                          │
│    - Scatta: Foto Retro (camera)                           │
│    - Aggiungi note (se "Altro")                            │
│    - Salva → Prossima palina                               │
│ 5. Fine giornata → "Termina sessione"                      │
│ 6. Mobile fa COMMIT su GitHub con dati                     │
└─────────────────────────────────────────────────────────────┘
                           ↓
┌─────────────────────────────────────────────────────────────┐
│ SERA: DESKTOP sincronizza                                    │
├─────────────────────────────────────────────────────────────┤
│ 1. Desktop rileva nuovo commit da Mobile                    │
│ 2. Desktop scarica dati aggiornati                          │
│ 3. Desktop mostra REPORT giornaliero                        │
│ 4. Desktop salva in memoria i dati                          │
│ 5. Desktop aggiorna STATO paline                            │
└─────────────────────────────────────────────────────────────┘
```

---

## 📄 FORMATO FILE INTERVENTO

**File:** `interventi/2025-04-29.json`

```json
{
  "data": "2025-04-29",
  "operatore": "Nome Operatore",
  "linea": "2",
  "paline_pianificate": 5,
  "paline_completate": 5,
  "interventi": [
    {
      "id_palina": "2_001",
      "nome_palina": "Napoli - Stazione Centrale",
      "lat": 40.8531,
      "lng": 14.2696,
      "tipo_manutenzione": 1,
      "tipo_manutenzione_label": "Riparazione",
      "foto_fronte": "data:image/jpeg;base64,/9j/4AAQSkZJRg...",
      "foto_retro": "data:image/jpeg;base64,/9j/4AAQSkZJRg...",
      "note": "",
      "timestamp": "2025-04-29T14:35:22Z",
      "completato": true
    },
    {
      "id_palina": "2_002",
      "nome_palina": "Napoli - Piazza Garibaldi",
      "lat": 40.8549,
      "lng": 14.2698,
      "tipo_manutenzione": 6,
      "tipo_manutenzione_label": "Altro",
      "foto_fronte": "data:image/jpeg;base64,/9j/4AAQSkZJRg...",
      "foto_retro": "data:image/jpeg;base64,/9j/4AAQSkZJRg...",
      "note": "Vetro rotto, segnalare a manutenzione specializzata",
      "timestamp": "2025-04-29T15:12:45Z",
      "completato": true
    }
  ],
  "timestamp_creazione": "2025-04-29T18:30:00Z",
  "sincronizzato": true
}
```

---

## 🔐 TIPI DI MANUTENZIONE

```
ID  Nome                    Descrizione
───────────────────────────────────────────────────
1   Riparazione            Riparazione componenti
2   Sostituzione palina    Sostituzione completa palina
3   Sostituzione palo      Sostituzione palo
4   Verniciatura           Verniciatura/ritintegrazione
5   Sostituzione tabella   Sostituzione tabella orari
6   Altro                  Altro (specificare note)
```

---

## 🛠️ INTEGRAZIONE GITHUB API

### Endpoint per Desktop:

```javascript
// Leggere paline
GET https://api.github.com/repos/Ropagnozzi/EAV-bus-data/contents/paline.json
Header: Authorization: token YOUR_GITHUB_TOKEN

// Scrivere intervento
PUT https://api.github.com/repos/Ropagnozzi/EAV-bus-data/contents/interventi/2025-04-29.json
Header: Authorization: token YOUR_GITHUB_TOKEN
Body: {
  "message": "Interventi 29/04/2025",
  "content": "BASE64_ENCODED_JSON",
  "branch": "main"
}

// Leggere interventi (per Mobile)
GET https://api.github.com/repos/Ropagnozzi/EAV-bus-data/contents/interventi/
Header: Authorization: token YOUR_GITHUB_TOKEN
```

---

## 📱 VERSIONI

### Desktop v2:
- Calendario (selezione data)
- Selezione paline (checkbox)
- Sincronizzazione GitHub
- Generazione QR code
- Report giornaliero

### Mobile v1:
- Ricezione QR code
- Elenco paline
- Google Maps integrato
- Camera foto automatica
- Report con foto base64
- Sincronizzazione GitHub

---

## ✅ CHECKLIST SETUP

- [ ] Repository creato su GitHub
- [ ] Token GitHub generato
- [ ] File config.json caricato
- [ ] File paline.json caricato
- [ ] Desktop v2 pronto
- [ ] Mobile v1 pronto
- [ ] Testing completo
- [ ] Operai addestrati

---

## 🚀 AVVIO OPERATIVO

1. **Giorno 1:** Desktop riceve lista paline da pianificare
2. **Giorno 1 sera:** Desktop invia QR a Mobile via WhatsApp/Email
3. **Giorno 2 mattina:** Mobile operai ricevono QR, scansionano
4. **Giorno 2:** Mobile compila dati
5. **Giorno 2 sera:** Mobile invia dati a GitHub
6. **Giorno 3 mattina:** Desktop sincronizza e mostra report

---

## 📞 SUPPORTO

**Documenti di riferimento:**
- GitHub API: https://docs.github.com/en/rest
- Google Maps API: https://developers.google.com/maps
- WebRTC Camera: https://developer.mozilla.org/en-US/docs/Web/API/MediaDevices/getUserMedia

**Contatto:**
- Owner GitHub: Ropagnozzi
- Project Manager: Roberto
