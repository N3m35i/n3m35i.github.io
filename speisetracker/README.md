# 🏠 SpeiseTracker — Setup Firebase

## File dell'app
- `index.html` — l'intera app
- `manifest.json` — PWA manifest
- `sw.js` — service worker (offline)
- *(opzionale)* `icon-192.png`, `icon-512.png` — icone per la schermata home

---

## 1. Crea il progetto Firebase

1. Vai su [console.firebase.google.com](https://console.firebase.google.com)
2. **Aggiungi progetto** → nome (es. `speisetracker`) → disabilita Analytics → **Crea**
3. Menu laterale → **Build → Firestore Database** → **Crea database**
   - Modalità: **Test** (per iniziare)
   - Region: **eur3 (europe-west)**
4. Torna alla home del progetto → icona `</>` (Web app) → registra con nickname `speisetracker-web`
5. Copia il blocco `firebaseConfig` che appare

---

## 2. Incolla la config in index.html

Apri `index.html` e cerca il blocco (circa riga 10 dello script):

```javascript
const firebaseConfig = {
  apiKey:            "INSERISCI_API_KEY",
  authDomain:        "INSERISCI_AUTH_DOMAIN",
  projectId:         "INSERISCI_PROJECT_ID",
  ...
};
```

Sostituiscilo con i tuoi valori reali da Firebase Console.

---

## 3. Regole Firestore (Security Rules)

Nella Firebase Console → Firestore → **Regole**, incolla:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    // Solo chi conosce l'URL può leggere/scrivere
    match /{document=**} {
      allow read, write: if true;
    }
  }
}
```

> ⚠️ Queste regole "open" vanno bene per uso familiare privato.
> Se vuoi più sicurezza, abilita Firebase Authentication e limita l'accesso.

---

## 4. Pubblica su GitHub Pages

```bash
git init
git add .
git commit -m "SpeiseTracker init"
git branch -M main
git remote add origin https://github.com/TUO-UTENTE/speisetracker.git
git push -u origin main
```

Poi su GitHub → **Settings → Pages → Source: main / root** → Salva.
L'app sarà disponibile su `https://tuo-utente.github.io/speisetracker/`

---

## 5. Installa come app (PWA)

**Android (Chrome):** apri l'URL → menu ⋮ → *"Aggiungi a schermata Home"*

**iPhone (Safari):** apri l'URL → tasto Condividi → *"Aggiungi a schermata Home"*

---

## Come funziona il database condiviso

- **Utenti**: salvati come documento unico `config/users` in Firestore. Chiunque aggiunga un nuovo utente lo vede subito su tutti i device.
- **Spese**: collection `expenses`, con listener realtime. Quando qualcuno aggiunge o elimina una spesa, tutti i device connessi si aggiornano **in tempo reale** senza ricaricare.
- **Lista della spesa**: solo in memoria di sessione, si resetta alla chiusura del browser (come richiesto).
