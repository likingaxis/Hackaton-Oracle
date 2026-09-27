# Hackathon Oracle — AI Race Engineer Copilot

**Team Struffoli:** Valerio, Alfredo, Samuele, Luca  
**Progetto:** AI Race Engineer Copilot per la Scuderia Tor Vergata, costruito su piattaforma OCI Enterprise AI.

---

## 📂 Struttura del Repository

- **`COPIONI/`**
  - `Copione LUCA.md`: copione dettagliato per la presentazione.
- **`SLIDES/SLIDE FINALI/`**
  - `AI_Race_Engineer_Copilot.html`: presentazione finale interattiva (Reveal.js), autonoma e apribile direttamente da browser.
  - `AI_Race_Engineer_Copilot.pdf`: versione PDF esportata delle slide.
  - `slides.qmd`: sorgente Quarto della presentazione.
  - `_quarto.yml`, `theme.scss`, `finali.scss`, `fonts.css`: file di configurazione, stile e font.
  - `assets/`: cartella contenente foto, icone SVG vettoriali, schemi architetturali e video demo.
- **`SLIDES/SLIDE FONTI DA PRENDERE/`**
  - `SLIDE.md`: bozza dei contenuti e riferimenti delle slide.
  - `Assets/`: asset originali ed immagini di supporto.

---

## 🚀 Visualizzazione delle Slide

Per visualizzare la presentazione:
1. Apri il file `SLIDES/SLIDE FINALI/AI_Race_Engineer_Copilot.html` con un qualunque browser web.
2. Controlli principali:
   - **Frecce direzionali**: navigazione avanti / indietro
   - **F**: modalità a schermo intero
   - **S**: console relatore con note e timer
   - **O / ESC**: panoramica delle slide

---

## 🛠️ Ricompilazione con Quarto

Per ricompilare le slide dopo eventuali modifiche:
```bash
cd "SLIDES/SLIDE FINALI"
quarto render slides.qmd
```
