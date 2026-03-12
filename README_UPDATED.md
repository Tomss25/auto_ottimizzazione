# 📊 Portfolio Optimization & Historical Data Engine

[![Streamlit App](https://static.streamlit.io/badges/streamlit_badge_black_white.svg)](https://your-app-url.streamlit.app)

Applicazione web professionale per **acquisizione dati finanziari** e **ottimizzazione quantitativa di portafoglio** con design light mode elegante.

## 🚀 Quick Start

### Deploy su Streamlit Cloud (Consigliato)

1. **Fork/Clone** questo repository
2. Vai su [share.streamlit.io](https://share.streamlit.io)
3. Clicca "New app"
4. Seleziona:
   - Repository: `tuo-username/ottimizzazione_auto`
   - Branch: `main`
   - Main file path: `app.py`
5. Click "Deploy"!

### Esecuzione Locale

```bash
# Clona il repository
git clone https://github.com/tuo-username/ottimizzazione_auto.git
cd ottimizzazione_auto

# Installa dipendenze
pip install -r requirements.txt

# Avvia l'app
streamlit run app.py
```

L'app si aprirà automaticamente su `http://localhost:8501`

## 🎯 Funzionalità Principali

### 📥 Modulo 1: Data Engine
- **Download Automatico**: Scarica dati da Yahoo Finance/Morningstar via ticker/ISIN
- **Upload Manuale**: Carica file CSV/Excel personalizzati
- **Timeframe Flessibile**: Giornaliero, Settimanale, Mensile
- **Alias Intelligenti**: `SP500` → `^GSPC`, `GOLD` → `GC=F`, ecc.

### 📈 Modulo 2: Portfolio Optimizer
- **Markowitz**: Ottimizzazione mean-variance (SLSQP)
- **Montecarlo**: 10,000 simulazioni stocastiche
- **GMV Shrinkage**: Ledoit-Wolf robust covariance
- **Min-CVaR**: Ottimizzazione tail risk (difesa eventi estremi)
- **Walk-Forward Backtest**: Confronto performance 4 modelli
- **Proiezione GBM**: Simulazione multi-scenario

### 📊 Visualizzazioni
- Performance base 100
- Matrice correlazione
- Equity curves comparative
- Drawdown analysis
- Cono di probabilità (P05/P50/P95)
- Pie charts allocazione

## 🔧 Configurazione

### Parametri Ottimizzazione
- **Orizzonte Rolling**: 1-10 anni (window size per ricalcolo pesi)
- **Frequenza Ribilanciamento**: Daily/Weekly/Monthly
- **Vincoli Peso**: Min/Max per singolo asset
- **Risk-Free Rate**: Tasso per calcolo Sharpe

### Ticker Supportati

#### Indici USA
- `^GSPC` - S&P 500
- `^NDX` - NASDAQ 100
- `^DJI` - Dow Jones
- `^VIX` - Volatilità

#### Indici Europa
- `^GDAXI` - DAX
- `^FCHI` - CAC 40
- `^STOXX50E` - Euro Stoxx 50
- `FTSEMIB.MI` - FTSE MIB

#### ETF Milano
- `SWDA.MI` - iShares MSCI World
- `EIMI.MI` - iShares EM
- `ENEL.MI` - Enel
- `ISP.MI` - Intesa Sanpaolo

#### Commodities & Crypto
- `GC=F` - Gold
- `CL=F` - Crude Oil
- `BTC-USD` - Bitcoin
- `ETH-USD` - Ethereum

## 📁 Struttura Repository

```
ottimizzazione_auto/
├── app.py                          # App principale
├── requirements.txt                # Dipendenze (versioni pinned)
├── requirements_minimal.txt        # Dipendenze (se problemi)
├── .streamlit/
│   └── config.toml                 # Configurazione tema
├── .gitignore                      # File da ignorare
└── README.md                       # Documentazione
```

## 📦 Dipendenze

### Principali
- **streamlit**: Framework web app
- **yfinance**: Download dati Yahoo Finance
- **pandas/numpy**: Manipolazione dati
- **scipy**: Ottimizzazione numerica
- **plotly**: Grafici interattivi
- **scikit-learn**: Ledoit-Wolf shrinkage
- **matplotlib/seaborn**: Visualizzazioni

### Opzionali
- **mstarpy**: Download dati Morningstar (gestito gracefully se assente)

## 🐛 Troubleshooting

### Problema: Loop infinito al caricamento
✅ **Risolto**: Rimosso `st.rerun()` non necessario

### Problema: ModuleNotFoundError
**Soluzione**: 
```bash
# Prova prima
pip install -r requirements.txt

# Se fallisce, usa la versione minimal
pip install -r requirements_minimal.txt
```

### Problema: mstarpy non si installa
**Soluzione**: L'app funziona comunque! Solo Yahoo Finance sarà disponibile (copre 95% dei casi)

### Problema: Upload CSV non funziona
**Verifica formato**:
- Prima colonna: `data` o `date`
- Formato data: GG/MM/AAAA o AAAA-MM-GG
- Separatore: `;` o `,`
- Usa **Adjusted Close** per calcoli corretti

## 📊 Formato File CSV/Excel

```
data         | Asset1    | Asset2    | Asset3
-------------|-----------|-----------|----------
01/01/2020   | 100.50    | 250.00    | 75.30
02/01/2020   | 101.20    | 251.50    | 76.10
```

**IMPORTANTE**: Usa **Adjusted Close** per includere dividendi/cedole!

## 🎨 Design System

- **Tema**: Light mode professionale
- **Colori**: Light Navy Blue (#1A365D) + neutrali
- **Tipografia**: Inter (UI) + JetBrains Mono (numeri)
- **Layout**: Responsive, mobile-friendly

## 📖 Metodologia Quantitativa

### 1. Markowitz (Mean-Variance)
Massimizza Sharpe Ratio. ⚠️ Sensibile a errori di stima rendimenti attesi.

### 2. Montecarlo
10,000 simulazioni casuali. Validazione statistica Markowitz.

### 3. GMV Shrinkage
Minimizza solo varianza. Matrice covarianza robusta via ML.

### 4. Min-CVaR
Ottimizza per 5% peggiori scenari. Difesa tail risk.

### 5. Walk-Forward Backtest
Test realistico con ribilanciamento dinamico.

### 6. Proiezione GBM
Simulazione stocastica con cono probabilità.

## ⚠️ Disclaimer

**IMPORTANTE**: Tool educativo/analitico. Non costituisce consulenza finanziaria.

- Performance passate ≠ rendimenti futuri
- I modelli sono semplificazioni con limitazioni
- Usa solo dati **Adjusted Close**
- Consulta sempre un professionista prima di investire

## 📧 Supporto

Per bug report o feature requests, apri una [Issue](https://github.com/tuo-username/ottimizzazione_auto/issues).

## 📄 Licenza

Fornito "as is" senza garanzie.

---

**Versione**: 1.0 Professional  
**Ultimo Aggiornamento**: Marzo 2026  
**Compatibilità**: Python 3.9+ | Streamlit 1.28+
