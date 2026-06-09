"""
Portfolio Optimization & Historical Data Engine
================================================
Unified App: Data Acquisition + Portfolio Optimization
Light Mode Professional Theme

Run with:
    streamlit run app.py
"""

import streamlit as st
import yfinance as yf
try:
    import mstarpy
    MSTARPY_AVAILABLE = True
except ImportError:
    MSTARPY_AVAILABLE = False
import pandas as pd
import numpy as np
from datetime import datetime, timedelta
import re
from scipy.optimize import minimize
import plotly.graph_objects as go
import streamlit.components.v1 as components
from sklearn.covariance import LedoitWolf
import seaborn as sns
import matplotlib.pyplot as plt

# ══════════════════════════════════════════════════════════════════
#  PAGE CONFIG
# ══════════════════════════════════════════════════════════════════
st.set_page_config(
    page_title="Portfolio Optimizer & Data Engine",
    page_icon="📊",
    layout="wide",
    initial_sidebar_state="expanded",
)

# ══════════════════════════════════════════════════════════════════
#  THEME TOKENS - LIGHT MODE
# ══════════════════════════════════════════════════════════════════
BG_PRIMARY      = "#F8F9FA"
BG_SECONDARY    = "#FFFFFF"
BG_CARD         = "#FFFFFF"
TEXT_PRIMARY    = "#1A202C"
TEXT_SECONDARY  = "#4A5568"
TEXT_MUTED      = "#718096"
COLOR_HIGHLIGHT = "#1A365D"  # Light Navy Blue
COLOR_ACCENT    = "#2C5282"
BORDER_COLOR    = "#E2E8F0"
CHART_COLORS    = ["#1A365D", "#2C5282", "#3182CE", "#4299E1", "#63B3ED", "#90CDF4", "#BEE3F8", "#E6F2FF"]

COLOR_GREEN     = "#38A169"
COLOR_RED       = "#E53E3E"
COLOR_GOLD      = "#D69E2E"
BG_CARD_HOVER   = "#F7FAFC"
BORDER_RADIUS   = "12px"

PLOTLY_LAYOUT = dict(
    template="plotly_white",
    paper_bgcolor=BG_SECONDARY,
    plot_bgcolor=BG_SECONDARY,
    font=dict(color=TEXT_PRIMARY, family="Inter, sans-serif", size=12),
    xaxis=dict(gridcolor=BORDER_COLOR, linecolor=BORDER_COLOR,
               tickcolor=BORDER_COLOR, tickfont=dict(color=TEXT_SECONDARY),
               zerolinecolor=BORDER_COLOR),
    yaxis=dict(gridcolor=BORDER_COLOR, linecolor=BORDER_COLOR,
               tickcolor=BORDER_COLOR, tickfont=dict(color=TEXT_SECONDARY),
               zerolinecolor=BORDER_COLOR),
    legend=dict(bgcolor=BG_SECONDARY, bordercolor=BORDER_COLOR,
                font=dict(color=TEXT_SECONDARY)),
    margin=dict(l=40, r=20, t=40, b=40),
    hoverlabel=dict(bgcolor=BG_CARD, bordercolor=BORDER_COLOR,
                    font=dict(color=TEXT_PRIMARY)),
    colorway=CHART_COLORS
)

# ══════════════════════════════════════════════════════════════════
#  CSS INJECTION - LIGHT MODE
# ══════════════════════════════════════════════════════════════════
st.markdown(f"""
<style>
@import url('https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700&family=JetBrains+Mono:wght@400;500;700&display=swap');

html, body, [data-testid="stAppViewContainer"] {{
    background-color: {BG_PRIMARY} !important;
    color: {TEXT_PRIMARY};
    font-family: 'Inter', sans-serif;
}}
[data-testid="stHeader"] {{
    background-color: {BG_SECONDARY} !important;
    border-bottom: 1px solid {BORDER_COLOR};
}}
[data-testid="stToolbar"] {{ display: none; }}
.block-container {{
    padding-top: 1.5rem !important;
    padding-bottom: 2rem !important;
    max-width: 1400px;
}}

/* Sidebar */
[data-testid="stSidebar"] {{
    background-color: {BG_SECONDARY} !important;
    border-right: 2px solid {BORDER_COLOR};
}}
[data-testid="stSidebar"] .stMarkdown p,
[data-testid="stSidebar"] label,
[data-testid="stSidebar"] .stSelectbox label {{
    color: {TEXT_SECONDARY} !important;
    font-size: 0.82rem;
    text-transform: uppercase;
    letter-spacing: 0.06em;
    font-weight: 600;
}}
[data-testid="stSidebar"] .stSlider > div > div > div > div {{
    background-color: {COLOR_HIGHLIGHT} !important;
}}
[data-testid="stSidebar"] hr {{
    border-color: {BORDER_COLOR} !important;
    margin: 1rem 0;
}}

section.main div[data-testid="stSlider"] label p {{
    color: {TEXT_PRIMARY} !important;
}}

/* Text Area (Sidebar) */
.stTextArea textarea {{
    background-color: {BG_SECONDARY} !important;
    color: {TEXT_PRIMARY} !important;
    border: 2px solid {BORDER_COLOR} !important;
    border-radius: {BORDER_RADIUS} !important;
    font-family: 'JetBrains Mono', monospace;
}}
.stTextArea textarea:focus {{
    border-color: {COLOR_HIGHLIGHT} !important;
    box-shadow: 0 0 0 2px rgba(26, 54, 93, 0.1) !important;
}}

/* KPI tiles (Custom) */
.kpi-tile {{
    background: linear-gradient(135deg, {BG_CARD} 0%, #F7FAFC 100%);
    border: 2px solid {BORDER_COLOR};
    border-radius: {BORDER_RADIUS};
    padding: 1.4rem 1.6rem;
    text-align: center;
    position: relative;
    overflow: hidden;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.04);
    transition: all 0.3s ease;
}}
.kpi-tile:hover {{
    box-shadow: 0 4px 12px rgba(26, 54, 93, 0.1);
    transform: translateY(-2px);
}}
.kpi-tile::before {{
    content: '';
    position: absolute;
    top: 0; left: 0; right: 0;
    height: 3px;
    background: linear-gradient(90deg, {COLOR_HIGHLIGHT}, {COLOR_ACCENT});
}}
.kpi-label {{
    font-size: 0.72rem;
    font-weight: 700;
    color: {TEXT_MUTED};
    text-transform: uppercase;
    letter-spacing: 0.08em;
    margin-bottom: 0.6rem;
}}
.kpi-value {{
    font-size: 1.8rem;
    font-weight: 700;
    color: {TEXT_PRIMARY};
    font-family: 'JetBrains Mono', monospace;
    line-height: 1.1;
}}
.kpi-value.positive {{ color: {COLOR_GREEN}; }}
.kpi-value.negative {{ color: {COLOR_RED}; }}
.kpi-sub {{
    font-size: 0.72rem;
    color: {TEXT_MUTED};
    margin-top: 0.4rem;
}}

/* Standard Streamlit Metrics Styling */
[data-testid="stMetric"] {{
    background: {BG_CARD} !important;
    border: 2px solid {BORDER_COLOR} !important;
    border-radius: {BORDER_RADIUS} !important;
    padding: 1rem 1.2rem !important;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.04);
}}
[data-testid="stMetricLabel"] {{
    color: {TEXT_MUTED} !important;
    font-size: 0.72rem !important;
    text-transform: uppercase;
    letter-spacing: 0.07em;
    font-weight: 600;
}}
[data-testid="stMetricValue"] {{
    color: {TEXT_PRIMARY} !important;
    font-family: 'JetBrains Mono', monospace !important;
    font-size: 1.5rem !important;
    font-weight: 700 !important;
}}

/* Tabs */
[data-testid="stTabs"] [role="tablist"] {{
    background: {BG_SECONDARY};
    border-radius: {BORDER_RADIUS};
    border: 2px solid {BORDER_COLOR};
    padding: 6px;
    gap: 6px;
}}
[data-testid="stTabs"] button[role="tab"] {{
    border-radius: 8px !important;
    color: {TEXT_SECONDARY} !important;
    font-size: 0.85rem !important;
    font-weight: 600 !important;
    padding: 8px 18px !important;
    transition: all 0.2s ease;
    border: none !important;
}}
[data-testid="stTabs"] button[role="tab"]:hover {{
    background: {BG_CARD_HOVER} !important;
}}
[data-testid="stTabs"] button[role="tab"][aria-selected="true"] {{
    background: {COLOR_HIGHLIGHT} !important;
    color: white !important;
    box-shadow: 0 2px 8px rgba(26, 54, 93, 0.2);
}}

/* Buttons */
.stButton > button {{
    background: linear-gradient(135deg, {COLOR_HIGHLIGHT} 0%, {COLOR_ACCENT} 100%) !important;
    color: white !important;
    border: none !important;
    border-radius: {BORDER_RADIUS} !important;
    font-weight: 600 !important;
    font-size: 0.9rem !important;
    letter-spacing: 0.04em;
    padding: 0.7rem 1.6rem !important;
    transition: all 0.2s ease;
    box-shadow: 0 4px 6px rgba(26, 54, 93, 0.2);
}}
.stButton > button:hover {{
    transform: translateY(-2px);
    box-shadow: 0 6px 12px rgba(26, 54, 93, 0.3);
}}
div.stDownloadButton > button {{
    background: transparent !important;
    color: {COLOR_HIGHLIGHT} !important;
    border: 2px solid {COLOR_HIGHLIGHT} !important;
    border-radius: {BORDER_RADIUS} !important;
    font-weight: 600 !important;
}}
div.stDownloadButton > button:hover {{
    background: {COLOR_HIGHLIGHT} !important;
    color: white !important;
}}

/* Tables */
.stDataFrame, .stTable {{
    border-radius: {BORDER_RADIUS} !important;
    overflow: hidden !important;
    border: 2px solid {BORDER_COLOR};
}}
thead tr th {{
    background: linear-gradient(135deg, {COLOR_HIGHLIGHT} 0%, {COLOR_ACCENT} 100%) !important;
    color: white !important;
    font-size: 0.75rem !important;
    text-transform: uppercase;
    letter-spacing: 0.07em;
    font-weight: 700;
    border-bottom: none !important;
    padding: 12px !important;
}}
tbody th {{
    color: {TEXT_PRIMARY} !important;
    font-weight: 600 !important;
    background: {BG_SECONDARY} !important;
    border-bottom: 1px solid {BORDER_COLOR} !important;
}}
tbody tr td {{
    color: {TEXT_PRIMARY} !important;
    font-family: 'JetBrains Mono', monospace;
    font-size: 0.82rem !important;
    background: {BG_SECONDARY} !important;
    border-bottom: 1px solid {BORDER_COLOR} !important;
}}
tbody tr:hover td, tbody tr:hover th {{
    background: {BG_CARD_HOVER} !important;
}}

/* Alerts */
[data-testid="stAlert"] {{
    border-radius: {BORDER_RADIUS} !important;
    border: 2px solid !important;
}}

/* File uploader */
[data-testid="stFileUploader"] {{
    background: {BG_CARD} !important;
    border: 2px dashed {BORDER_COLOR} !important;
    border-radius: {BORDER_RADIUS} !important;
    transition: border-color 0.2s;
}}
[data-testid="stFileUploader"]:hover {{
    border-color: {COLOR_HIGHLIGHT} !important;
}}

/* Selectbox / multiselect */
[data-testid="stMultiSelect"] > div,
[data-testid="stSelectbox"] > div {{
    background: {BG_CARD} !important;
    border-color: {BORDER_COLOR} !important;
    border-radius: {BORDER_RADIUS} !important;
    color: {TEXT_PRIMARY} !important;
}}

/* Progress bar */
[data-testid="stProgress"] > div > div {{
    background: {COLOR_HIGHLIGHT} !important;
    border-radius: 4px !important;
}}

/* Divider */
hr {{
    border: none;
    border-top: 2px solid {BORDER_COLOR};
    margin: 2rem 0;
}}

/* Methodology box */
.methodology-box {{
    background: {BG_CARD};
    border: 2px solid {BORDER_COLOR};
    border-left: 4px solid {COLOR_HIGHLIGHT};
    border-radius: {BORDER_RADIUS};
    padding: 1.8rem 2rem;
    margin-top: 1rem;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.04);
}}
.methodology-box h3 {{
    color: {TEXT_PRIMARY};
    font-size: 1.1rem;
    font-weight: 700;
    margin-bottom: 1rem;
}}
.methodology-box h4 {{
    color: {COLOR_HIGHLIGHT};
    font-size: 0.9rem;
    font-weight: 600;
    margin: 1rem 0 0.5rem;
    text-transform: uppercase;
    letter-spacing: 0.05em;
}}
.methodology-box p {{
    color: {TEXT_SECONDARY};
    font-size: 0.88rem;
    line-height: 1.8;
}}

/* Status badge */
.status-badge {{
    display: inline-flex;
    align-items: center;
    gap: 8px;
    background: {BG_CARD};
    border: 2px solid {BORDER_COLOR};
    border-radius: 24px;
    padding: 4px 14px;
    font-size: 0.75rem;
    font-weight: 600;
    color: {TEXT_SECONDARY};
}}
.status-dot {{
    width: 8px; height: 8px;
    border-radius: 50%;
    animation: pulse 2s infinite;
}}
@keyframes pulse {{
    0%, 100% {{ opacity: 1; }}
    50% {{ opacity: 0.5; }}
}}

/* Section header */
.section-header {{
    font-size: 0.75rem;
    font-weight: 700;
    color: {TEXT_MUTED};
    text-transform: uppercase;
    letter-spacing: 0.1em;
    margin-bottom: 1rem;
    padding-bottom: 0.6rem;
    border-bottom: 2px solid {BORDER_COLOR};
}}

/* Code block styling */
.stCode {{
    background: {BG_CARD} !important;
    border: 2px solid {BORDER_COLOR} !important;
    border-radius: {BORDER_RADIUS} !important;
}}
</style>
""", unsafe_allow_html=True)


# ══════════════════════════════════════════════════════════════════
#  SESSION STATE INITIALIZATION
# ══════════════════════════════════════════════════════════════════
if 'data_loaded' not in st.session_state:
    st.session_state.data_loaded = False
if 'df_historical' not in st.session_state:
    st.session_state.df_historical = None
if 'selected_assets' not in st.session_state:
    st.session_state.selected_assets = []
if 'data_source' not in st.session_state:
    st.session_state.data_source = None


# ══════════════════════════════════════════════════════════════════
#  UI HELPERS
# ══════════════════════════════════════════════════════════════════
def render_topbar(title: str, subtitle: str, data_loaded: bool = False):
    status_color = COLOR_GREEN if data_loaded else COLOR_GOLD
    status_text  = "Data Loaded"  if data_loaded else "Awaiting Data"
    st.markdown(f"""
    <div style="display:flex; align-items:flex-end; justify-content:space-between;
                padding:1rem 0 1.8rem 0; border-bottom:2px solid {BORDER_COLOR};
                margin-bottom:2rem;">
        <div>
            <div style="font-size:1.8rem; font-weight:700; color:{COLOR_HIGHLIGHT};
                        letter-spacing:-0.03em; line-height:1.1;">
                {title}
            </div>
            <div style="font-size:0.85rem; color:{TEXT_SECONDARY}; margin-top:6px; font-weight:500;">
                {subtitle}
            </div>
        </div>
        <div style="display:flex; align-items:center; gap:14px; flex-shrink:0;">
            <div class="status-badge">
                <div class="status-dot" style="background:{status_color};"></div>
                {status_text}
            </div>
            <div style="font-size:0.72rem; color:{TEXT_MUTED}; text-align:right;">
                Unified Platform<br>
                <span style="color:{COLOR_HIGHLIGHT}; font-weight:600;">v1.0 Professional</span>
            </div>
        </div>
    </div>
    """, unsafe_allow_html=True)


def kpi_tile(label: str, value: str, sub: str = "", positive=None):
    color_class = "positive" if positive is True else "negative" if positive is False else ""
    sub_html = f'<div class="kpi-sub">{sub}</div>' if sub else ""
    st.markdown(f"""
    <div class="kpi-tile">
        <div class="kpi-label">{label}</div>
        <div class="kpi-value {color_class}">{value}</div>
        {sub_html}
    </div>
    """, unsafe_allow_html=True)


def kpi_row(metrics: list):
    cols = st.columns(len(metrics))
    for col, m in zip(cols, metrics):
        with col:
            kpi_tile(m.get("label", ""), m.get("value", "—"),
                     m.get("sub", ""), m.get("positive", None))


def allocation_table(assets: list, weights):
    df = pd.DataFrame({"Asset": assets, "Peso": weights}) \
           .sort_values("Peso", ascending=False).reset_index(drop=True)
    df["Allocazione"] = (df["Peso"] * 100).map(lambda x: f"{x:.2f} %")
    st.table(df[["Asset", "Allocazione"]])


# ══════════════════════════════════════════════════════════════════
#  DATA ACQUISITION FUNCTIONS (FROM SERIE STORICHE)
# ══════════════════════════════════════════════════════════════════
ALIAS_MAP = {
    "SP500": "^GSPC", "S&P500": "^GSPC", "NASDAQ": "^NDX", "NASDAQ100": "^NDX",
    "DOWJONES": "^DJI", "DAX": "^GDAXI", "CAC40": "^FCHI", "ESTX50": "^STOXX50E",
    "EUROSTOXX": "^STOXX50E", "VIX": "^VIX", "GOLD": "GC=F", "OIL": "CL=F",
    "BITCOIN": "BTC-USD", "BTC": "BTC-USD", "EURUSD": "EURUSD=X"
}


def get_data_yahoo(ticker, start_dt):
    try:
        df = yf.download(ticker, start=start_dt, progress=False)
        if not df.empty:
            col = 'Adj Close' if 'Adj Close' in df.columns else 'Close'
            series = df[col].squeeze()
            if isinstance(series, pd.Series): 
                return series.ffill()
    except: 
        return None
    return None


def get_data_morningstar(isin, start_dt, end_dt):
    try:
        fund = mstarpy.Funds(term=isin, country="it")
        history = fund.nav(start_date=start_dt, end_date=end_dt, frequency="daily")
        if history:
            df = pd.DataFrame(history)
            df['date'] = pd.to_datetime(df['date'])
            df.set_index('date', inplace=True)
            series = df['nav']
            series.index = series.index.normalize().tz_localize(None)
            return series
    except: 
        return None
    return None


def fetch_historical_data(tickers_input, years, timeframe="Giornaliero"):
    """
    Scarica dati storici con timeframe specificato.
    
    Args:
        tickers_input: Lista di ticker/ISIN
        years: Numero di anni di storico
        timeframe: "Giornaliero", "Settimanale", "Mensile"
    
    Returns:
        DataFrame con dati storici al timeframe richiesto
    """
    start_date = datetime.now() - timedelta(days=years*365)
    end_date = datetime.now()
    all_series = {}
    
    # Mapping timeframe per yfinance
    interval_map = {
        "Giornaliero": "1d",
        "Settimanale": "1wk",
        "Mensile": "1mo"
    }
    yf_interval = interval_map.get(timeframe, "1d")
    
    for t in tickers_input:
        series = None
        
        # Prova con Yahoo Finance usando l'interval specificato
        try:
            df = yf.download(t, start=start_date, interval=yf_interval, progress=False)
            if not df.empty:
                col = 'Adj Close' if 'Adj Close' in df.columns else 'Close'
                series = df[col].squeeze()
                if isinstance(series, pd.Series): 
                    series = series.ffill()
        except: 
            pass
        
        # Fallback Morningstar (solo giornaliero disponibile, poi resample)
        if series is None and MSTARPY_AVAILABLE:
            try:
                fund = mstarpy.Funds(term=t, country="it")
                history = fund.nav(start_date=start_date, end_date=end_date, frequency="daily")
                if history:
                    df = pd.DataFrame(history)
                    df['date'] = pd.to_datetime(df['date'])
                    df.set_index('date', inplace=True)
                    series = df['nav']
                    series.index = series.index.normalize().tz_localize(None)
                    
                    # Resample se necessario
                    if timeframe == "Settimanale":
                        series = series.resample('W').last().ffill()
                    elif timeframe == "Mensile":
                        series = series.resample('ME').last().ffill()
            except: 
                pass
        
        if series is not None:
            series.name = t 
            all_series[t] = series
        else:
            st.warning(f"⚠️ Dati non trovati per {t}")
    
    if all_series:
        df_daily = pd.DataFrame(all_series).ffill().dropna()
        return df_daily
    return None


# ══════════════════════════════════════════════════════════════════
#  CHART BUILDERS (FROM OTTIMIZZATORE)
# ══════════════════════════════════════════════════════════════════
def _base_fig(**kwargs) -> go.Figure:
    return go.Figure(layout=go.Layout(**{**PLOTLY_LAYOUT, **kwargs}))


def pie_chart(labels, values, title="Asset Allocation") -> go.Figure:
    fig = go.Figure()
    fig.add_trace(go.Pie(
        labels=labels, values=values, hole=0.52,
        marker=dict(line=dict(color=BG_CARD, width=2)),
        textinfo="label+percent", textfont=dict(size=12),
        hovertemplate="<b>%{label}</b><br>Peso: %{percent}<br><extra></extra>",
        sort=True, direction="clockwise",
    ))
    fig.update_layout(**{**PLOTLY_LAYOUT,
        "title": dict(text=title, font=dict(size=14, color=TEXT_PRIMARY, weight=600), x=0.5, xanchor="center"),
        "showlegend": True, "margin": dict(l=10, r=10, t=50, b=10),
    })
    return fig


def equity_line_chart(nav_df: pd.DataFrame, title="Equity Line Comparativa (Base 100)") -> go.Figure:
    fig = _base_fig(title=dict(text=title, font=dict(size=14, color=TEXT_PRIMARY, weight=600), x=0))
    for i, col in enumerate(nav_df.columns):
        fig.add_trace(go.Scatter(
            x=nav_df.index, y=nav_df[col], name=col,
            mode="lines", line=dict(width=2.5),
            hovertemplate=f"<b>{col}</b><br>%{{x|%b %Y}}<br>NAV: %{{y:.2f}}<extra></extra>",
        ))
    fig.update_layout(
        xaxis=dict(**PLOTLY_LAYOUT["xaxis"], title=""),
        yaxis=dict(**PLOTLY_LAYOUT["yaxis"], title="NAV (Base 100)"),
        legend=dict(orientation="h", yanchor="bottom", y=1.02, xanchor="right", x=1),
        hovermode="x unified",
    )
    return fig


def _hex_to_rgba(hex_color: str, alpha: float = 0.15) -> str:
    h = hex_color.lstrip("#")
    r, g, b = int(h[0:2], 16), int(h[2:4], 16), int(h[4:6], 16)
    return f"rgba({r},{g},{b},{alpha})"


def drawdown_chart(nav_df: pd.DataFrame, title="Drawdown Analysis") -> go.Figure:
    fig = _base_fig(title=dict(text=title, font=dict(size=14, color=TEXT_PRIMARY, weight=600), x=0))
    dd_colors = CHART_COLORS
    for i, col in enumerate(nav_df.columns):
        dd = (nav_df[col] - nav_df[col].cummax()) / nav_df[col].cummax() * 100
        color = dd_colors[i % len(dd_colors)]
        fig.add_trace(go.Scatter(
            x=dd.index, y=dd.values, name=col,
            mode="lines", line=dict(color=color, width=2),
            fill="tozeroy", fillcolor=_hex_to_rgba(color, 0.1),
            hovertemplate=f"<b>{col}</b><br>%{{x|%b %Y}}<br>DD: %{{y:.2f}}%<extra></extra>",
        ))
    fig.update_layout(
        yaxis=dict(**PLOTLY_LAYOUT["yaxis"], title="Drawdown (%)"),
        hovermode="x unified",
    )
    return fig


# ══════════════════════════════════════════════════════════════════
#  OPTIMIZATION FUNCTIONS (FROM OTTIMIZZATORE)
# ══════════════════════════════════════════════════════════════════
def get_optimal_weights(mu, sigma, min_weight, max_weight, rf):
    """Markowitz: Ottimizzazione Matematica (SLSQP)"""
    num_assets = len(mu)
    theoretical_min_max_weight = 1.0 / num_assets
    actual_max_weight = max_weight if max_weight >= theoretical_min_max_weight else theoretical_min_max_weight + 0.01
    args = (mu, sigma, rf)
    constraints = ({'type': 'eq', 'fun': lambda x: np.sum(x) - 1})
    bounds = tuple((min_weight, actual_max_weight) for _ in range(num_assets))
    initial_guess = [1./num_assets] * num_assets

    def neg_sharpe(w, mu, sigma, rf):
        ret = np.sum(mu * w)
        vol = np.sqrt(np.dot(w.T, np.dot(sigma, w)))
        if vol <= 0: return 1e6
        return -(ret - rf) / vol

    try:
        res = minimize(neg_sharpe, initial_guess, args=args, bounds=bounds,
                       constraints=constraints, method='SLSQP',
                       options={'ftol': 1e-9, 'disp': False})
        return res.x if res.success else None
    except:
        return None


def get_montecarlo_weights(mu, sigma, min_weight, max_weight, rf, num_sims=5000):
    """Montecarlo: Ottimizzazione via Simulazione Casuale"""
    num_assets = len(mu)
    weights = np.random.random((num_sims, num_assets))
    weights = weights / np.sum(weights, axis=1)[:, np.newaxis]
    mask = ((np.max(weights, axis=1) <= (max_weight + 0.01)) &
            (np.min(weights, axis=1) >= (min_weight - 0.01)))
    valid_weights = weights[mask]
    if len(valid_weights) == 0:
        return None
    port_ret = np.dot(valid_weights, mu)
    port_var = np.sum(np.dot(valid_weights, sigma) * valid_weights, axis=1)
    port_vol = np.sqrt(port_var)
    sharpe_ratios = (port_ret - rf) / port_vol
    return valid_weights[np.argmax(sharpe_ratios)]


def get_gmv_weights(sigma, min_weight, max_weight):
    """Global Minimum Variance (GMV): Minimizza esclusivamente il rischio"""
    num_assets = len(sigma)
    theoretical_min_max_weight = 1.0 / num_assets
    actual_max_weight = max_weight if max_weight >= theoretical_min_max_weight else theoretical_min_max_weight + 0.01
    
    args = (sigma,)
    constraints = ({'type': 'eq', 'fun': lambda x: np.sum(x) - 1})
    bounds = tuple((min_weight, actual_max_weight) for _ in range(num_assets))
    initial_guess = [1./num_assets] * num_assets

    def port_var(w, sig):
        return np.dot(w.T, np.dot(sig, w))

    try:
        res = minimize(port_var, initial_guess, args=args, bounds=bounds,
                       constraints=constraints, method='SLSQP',
                       options={'ftol': 1e-9, 'disp': False})
        return res.x if res.success else None
    except:
        return None


def get_cvar_weights(returns_matrix, min_weight, max_weight, alpha=0.05):
    """Ottimizzazione Min-CVaR (Expected Shortfall)"""
    num_assets = returns_matrix.shape[1]
    num_scenarios = returns_matrix.shape[0]
    
    theoretical_min_max_weight = 1.0 / num_assets
    actual_max_weight = max_weight if max_weight >= theoretical_min_max_weight else theoretical_min_max_weight + 0.01

    def cvar_objective(params):
        w = params[:-1]
        gamma = params[-1]
        port_returns = np.dot(returns_matrix, w)
        shortfall = -port_returns - gamma
        shortfall = np.maximum(shortfall, 0)
        return gamma + np.sum(shortfall) / (alpha * num_scenarios)

    initial_w = [1./num_assets] * num_assets
    initial_gamma = -np.percentile(np.dot(returns_matrix, initial_w), alpha * 100)
    initial_params = np.append(initial_w, initial_gamma)

    bounds = tuple((min_weight, actual_max_weight) for _ in range(num_assets)) + ((None, None),)
    constraints = ({'type': 'eq', 'fun': lambda x: np.sum(x[:-1]) - 1})

    try:
        res = minimize(cvar_objective, initial_params, method='SLSQP', bounds=bounds,
                       constraints=constraints, options={'ftol': 1e-7, 'disp': False})
        return res.x[:-1] if res.success else None
    except:
        return None


def prep_data(df, assets, lookback, freq):
    """Funzione helper per preparare i dati."""
    valid_assets = [c for c in assets if c in df.columns]
    if not valid_assets: return None, None, "Nessun asset valido."
    df = df[valid_assets]
    if df.empty or len(df) < 10: return None, None, "Dati insufficienti."
    df_res = df.resample(freq).last().dropna()
    returns = df_res.pct_change().dropna()
    if returns.empty: return None, None, "Impossibile calcolare rendimenti."
    ann_factor = 252 if freq == 'D' else 52 if freq == 'W' else 12
    mu    = returns.mean() * ann_factor
    sigma = returns.cov()  * ann_factor
    return mu, sigma, (df_res, returns, ann_factor)


def portfolio_metrics(weights, mu, sigma, rf) -> dict:
    p_ret    = float(np.sum(mu * weights))
    p_vol    = float(np.sqrt(np.dot(weights.T, np.dot(sigma, weights))))
    p_sharpe = (p_ret - rf) / p_vol if p_vol > 0 else 0.0
    return {"return": p_ret, "volatility": p_vol, "sharpe": p_sharpe}


def compute_nav(returns_df: pd.DataFrame, base: float = 100.0) -> pd.DataFrame:
    return (1 + returns_df).cumprod() * base


def max_drawdown(nav_series: pd.Series) -> float:
    rolling_max = nav_series.cummax()
    return float(((nav_series - rolling_max) / rolling_max).min())


# ══════════════════════════════════════════════════════════════════
#  SIDEBAR - DATA ACQUISITION MODULE
# ══════════════════════════════════════════════════════════════════
with st.sidebar:
    st.markdown(f"""
    <div style="padding:0.8rem 0 1.8rem 0; border-bottom:2px solid {BORDER_COLOR}; margin-bottom:1.8rem;">
        <div style="font-size:1.2rem; font-weight:700; color:{COLOR_HIGHLIGHT}; letter-spacing:-0.02em;">
            Portfolio Optimizer
        </div>
        <div style="font-size:0.72rem; font-weight:600; color:{COLOR_ACCENT};
                    text-transform:uppercase; letter-spacing:0.12em; margin-top:4px;">
            Data Engine · Quant Models
        </div>
    </div>
    """, unsafe_allow_html=True)

    st.markdown('<div class="section-header">📊 Modulo 1: Acquisizione Dati</div>', unsafe_allow_html=True)
    
    # Warning mstarpy solo se non disponibile
    if not MSTARPY_AVAILABLE:
        st.info("ℹ️ Morningstar non disponibile. Funzionerà solo Yahoo Finance (ticker standard).", icon="ℹ️")
    
    # Tab per scelta sorgente dati
    data_source_tab = st.radio(
        "Scegli Sorgente Dati",
        ["Download Automatico (Ticker/ISIN)", "Upload File CSV/Excel"],
        label_visibility="collapsed"
    )
    
    st.markdown("<br>", unsafe_allow_html=True)
    
    # Inizializzazione variabili per evitare NameError
    uploaded_file = None
    tickers_input = []
    
    if data_source_tab == "Download Automatico (Ticker/ISIN)":
        st.markdown("**Lista Tickers / ISIN**")
        raw_input = st.text_area(
            "Inserisci codici separati da spazi o righe", 
            value="^GSPC\nSWDA.MI\nEIMI.MI\nGC=F", 
            height=120,
            label_visibility="collapsed"
        )
        
        years = st.selectbox("Orizzonte Temporale", [1, 3, 5, 10, 20], index=2)
        
        st.markdown("**Timeframe Dati**")
        data_freq = st.selectbox(
            "Frequenza acquisizione",
            ["Giornaliero", "Settimanale", "Mensile"],
            index=0,
            help="Frequenza dei dati storici da scaricare",
            label_visibility="collapsed"
        )
        
        st.caption("💡 Il timeframe selezionato qui verrà usato per tutti i calcoli e grafici")
        
        raw_tokens = re.findall(r"[\w\.\-\^\=]+", raw_input.upper())
        tickers_input = []
        for t in raw_tokens:
            if t in ALIAS_MAP: 
                tickers_input.append(ALIAS_MAP[t])
            else: 
                tickers_input.append(t)
        
        st.caption(f"✅ {len(tickers_input)} strumenti rilevati")
        
    else:
        uploaded_file = st.file_uploader(
            "Carica File CSV/Excel",
            type=["csv", "xlsx", "xls"],
            help="File con colonna data e prezzi/NAV degli asset"
        )
    
    st.markdown("<br>", unsafe_allow_html=True)
    st.markdown('<div class="section-header">⚙️ Modulo 2: Parametri Ottimizzazione</div>', unsafe_allow_html=True)
    
    lookback = st.slider("Orizzonte Rolling (Anni)", 1, 10, 3,
                         help="Finestra dati passati per ricalcolare i pesi.")
    freq = st.selectbox("Frequenza Ribilanciamento", ["D", "W", "M"], index=2,
                        format_func=lambda x: {"D": "Giornaliero", "W": "Settimanale", "M": "Mensile"}[x])
    
    st.markdown("<br>", unsafe_allow_html=True)
    st.markdown('<div class="section-header">🔒 Vincoli Portafoglio</div>', unsafe_allow_html=True)
    
    min_weight = st.slider("Peso Minimo Asset", 0.0, 0.2, 0.0, step=0.01)
    max_weight = st.slider("Peso Massimo Asset", 0.1, 1.0, 0.40, step=0.05)
    rf = st.number_input("Tasso Risk Free (%)", 0.0, 10.0, 3.0, step=0.1) / 100
    
    st.markdown("<br>", unsafe_allow_html=True)
    
    # PULSANTE PRINCIPALE
    generate_and_optimize = st.button(
        "🚀 GENERA ED OTTIMIZZA SERIE STORICHE",
        type="primary",
        use_container_width=True
    )
    
    st.markdown(f"""
    <div style="margin-top:2rem; padding-top:1rem; border-top:2px solid {BORDER_COLOR};">
        <div style="font-size:0.68rem; color:{TEXT_MUTED}; line-height:1.7; text-align:center;">
            Modello quantitativo · Solo uso analitico<br>
            Non costituisce consulenza finanziaria
        </div>
    </div>
    """, unsafe_allow_html=True)


# ══════════════════════════════════════════════════════════════════
#  MAIN AREA - TOPBAR
# ══════════════════════════════════════════════════════════════════
render_topbar(
    title="Portfolio Optimization & Historical Data Engine",
    subtitle="Unified Platform: Data Acquisition → Portfolio Optimization → Risk Analysis",
    data_loaded=st.session_state.data_loaded
)


# ══════════════════════════════════════════════════════════════════
#  DATA LOADING LOGIC
# ══════════════════════════════════════════════════════════════════
if generate_and_optimize:
    with st.spinner("⏳ Caricamento e preparazione dati in corso..."):
        
        if data_source_tab == "Download Automatico (Ticker/ISIN)":
            if not tickers_input:
                st.error("❌ Inserisci almeno un ticker o ISIN valido.")
                st.stop()
            
            df_data = fetch_historical_data(tickers_input, years, data_freq)
            
            if df_data is not None and not df_data.empty:
                st.session_state.df_historical = df_data
                st.session_state.selected_assets = df_data.columns.tolist()
                st.session_state.data_loaded = True
                data_source_name = "Yahoo Finance" if not MSTARPY_AVAILABLE else f"Yahoo/Morningstar ({data_freq})"
                st.session_state.data_source = data_source_name
                st.success(f"✅ Dati scaricati con successo · {len(df_data.columns)} asset · {len(df_data)} periodi · Timeframe: {data_freq}")
            else:
                st.error("❌ Impossibile scaricare i dati. Verifica i ticker inseriti.")
                st.stop()
                
        else:
            if uploaded_file is None:
                st.error("❌ Carica un file CSV o Excel dalla sidebar.")
                st.stop()
            
            try:
                uploaded_file.seek(0)
                if uploaded_file.name.endswith('.csv'):
                    try:
                        df = pd.read_csv(uploaded_file, sep=';', decimal=',', thousands='.')
                        if df.shape[1] < 2: raise ValueError
                    except:
                        uploaded_file.seek(0)
                        df = pd.read_csv(uploaded_file, sep=',', decimal='.')
                else:
                    df = pd.read_excel(uploaded_file)
                
                df.columns = df.columns.str.strip()
                date_col = next((c for c in df.columns if c.lower() in
                                 ["data", "date", "timestamp", "time", "datetime"]), None)
                
                if date_col is None:
                    st.error("❌ Colonna Data non trovata nel file.")
                    st.stop()
                
                df[date_col] = pd.to_datetime(df[date_col], dayfirst=True, errors='coerce')
                df = (df.dropna(subset=[date_col])
                        .set_index(date_col)
                        .sort_index()
                        .select_dtypes(include=[np.number]))
                
                if df.empty:
                    st.error("❌ Nessun dato numerico trovato nel file.")
                    st.stop()
                
                st.session_state.df_historical = df
                st.session_state.selected_assets = df.columns.tolist()
                st.session_state.data_loaded = True
                st.session_state.data_source = "Upload CSV/Excel"
                st.success(f"✅ File caricato con successo · {len(df.columns)} asset · {len(df)} periodi")
                
            except Exception as e:
                st.error(f"❌ Errore nel parsing del file: {e}")
                st.stop()


# ══════════════════════════════════════════════════════════════════
#  LANDING STATE (NO DATA)
# ══════════════════════════════════════════════════════════════════
if not st.session_state.data_loaded:
    st.markdown(f"""
    <div style="text-align:center; padding:6rem 2rem;">
        <div style="font-size:4rem; margin-bottom:1.5rem;">📊</div>
        <div style="font-size:1.3rem; font-weight:700; color:{COLOR_HIGHLIGHT}; margin-bottom:0.8rem;">
            Benvenuto nel Portfolio Optimizer
        </div>
        <div style="font-size:0.95rem; color:{TEXT_SECONDARY}; max-width:600px;
                    margin:0 auto; line-height:1.9;">
            Configura i parametri nella <b>sidebar</b> e premi il pulsante<br>
            <b>"GENERA ED OTTIMIZZA SERIE STORICHE"</b> per iniziare l'analisi.
        </div>
        <div style="margin-top:3rem; padding:2rem; background:{BG_CARD}; 
                    border:2px solid {BORDER_COLOR}; border-radius:{BORDER_RADIUS};
                    max-width:700px; margin-left:auto; margin-right:auto;">
            <div style="font-size:0.85rem; color:{TEXT_SECONDARY}; line-height:1.8; text-align:left;">
                <b style="color:{COLOR_HIGHLIGHT};">Opzione 1: Download Automatico</b><br>
                Inserisci ticker (es. ^GSPC, SWDA.MI) o ISIN e l'app scaricherà automaticamente
                i dati storici da Yahoo Finance e Morningstar.<br><br>
                <b style="color:{COLOR_HIGHLIGHT};">Opzione 2: Upload File</b><br>
                Carica un file CSV o Excel con una colonna <code>data</code> e colonne per ciascun asset.
                Usa <b>Adjusted Close</b> per includere dividendi e cedole.
            </div>
        </div>
    </div>
    """, unsafe_allow_html=True)
    st.stop()


# ══════════════════════════════════════════════════════════════════
#  DATA LOADED - SHOW ANALYSIS
# ══════════════════════════════════════════════════════════════════
df = st.session_state.df_historical
all_assets = st.session_state.selected_assets

st.markdown(f"""
<div style="background:{BG_CARD}; padding:1.2rem 1.5rem; border-radius:{BORDER_RADIUS};
            border:2px solid {BORDER_COLOR}; margin-bottom:2rem;">
    <div style="display:flex; justify-content:space-between; align-items:center;">
        <div>
            <span style="font-size:0.75rem; color:{TEXT_MUTED}; text-transform:uppercase; 
                         letter-spacing:0.08em; font-weight:600;">Sorgente Dati</span>
            <div style="font-size:1rem; color:{COLOR_HIGHLIGHT}; font-weight:600; margin-top:4px;">
                {st.session_state.data_source}
            </div>
        </div>
        <div style="text-align:right;">
            <span style="font-size:0.75rem; color:{TEXT_MUTED}; text-transform:uppercase; 
                         letter-spacing:0.08em; font-weight:600;">Periodo Dati</span>
            <div style="font-size:0.9rem; color:{TEXT_PRIMARY}; font-weight:600; margin-top:4px;">
                {df.index[0].strftime('%d/%m/%Y')} → {df.index[-1].strftime('%d/%m/%Y')}
            </div>
        </div>
        <div style="text-align:right;">
            <span style="font-size:0.75rem; color:{TEXT_MUTED}; text-transform:uppercase; 
                         letter-spacing:0.08em; font-weight:600;">Asset Totali</span>
            <div style="font-size:1.4rem; color:{COLOR_HIGHLIGHT}; font-weight:700; 
                        font-family:'JetBrains Mono', monospace; margin-top:4px;">
                {len(all_assets)}
            </div>
        </div>
    </div>
</div>
""", unsafe_allow_html=True)


# ══════════════════════════════════════════════════════════════════
#  TAB SYSTEM
# ══════════════════════════════════════════════════════════════════
tab1, tab2, tab3, tab4, tab5, tab6, tab7, tab8 = st.tabs([
    "📊 Serie Storiche",
    "📈 Markowitz",
    "🎲 Montecarlo",
    "🛡️ Antifragile (CVaR & Shrinkage)",
    "📉 Backtest Comparativo",
    "🔮 Proiezione Futura",
    "💹 Mercato Live",
    "ℹ️ Metodologia"
])


# ── TAB 1: SERIE STORICHE (FROM CODICE A) ───────────────────────
with tab1:
    st.markdown('<div class="section-header">Analisi Serie Storiche & Performance</div>',
                unsafe_allow_html=True)
    
    # I dati sono già nel timeframe selezionato in sidebar
    df_final = df
    
    # Determina annualization factor dal timeframe dei dati
    # Controlla la frequenza effettiva dei dati
    if len(df_final) > 1:
        time_diff = (df_final.index[-1] - df_final.index[0]).days / len(df_final)
        if time_diff < 2:  # Giornaliero
            ann_factor = 252
            freq_display = "Giornaliero"
        elif time_diff < 10:  # Settimanale
            ann_factor = 52
            freq_display = "Settimanale"
        else:  # Mensile
            ann_factor = 12
            freq_display = "Mensile"
    else:
        ann_factor = 252
        freq_display = "Giornaliero"
    
    metrics = []
    for col in df_final.columns:
        s = df_final[col]
        if len(s) > 1:
            returns = s.pct_change().dropna()
            tot_ret = ((s.iloc[-1] / s.iloc[0]) - 1) * 100
            vol = returns.std() * np.sqrt(ann_factor) * 100
            roll_max = s.cummax()
            drawdown = (s - roll_max) / roll_max
            max_dd = drawdown.min() * 100
            
            metrics.append({
                "Ticker": col,
                "Prezzo": round(s.iloc[-1], 2),
                "Rend %": round(tot_ret, 2),
                "Volat %": round(vol, 2),
                "Max DD %": round(max_dd, 2)
            })
    
    st.subheader(f"📅 Serie Storiche ({freq_display})")
    st.dataframe(df_final.sort_index(ascending=False).round(2), use_container_width=True, height=400)
    
    st.markdown("<br>", unsafe_allow_html=True)
    
    col1, col2 = st.columns([2, 1])
    with col1:
        st.subheader("📈 Performance (Base 100)")
        if not df_final.empty:
            df_b100 = (df_final / df_final.iloc[0]) * 100
            st.line_chart(df_b100, height=400)
    
    with col2:
        st.subheader("🏆 Metriche Riassuntive")
        st.dataframe(pd.DataFrame(metrics).set_index("Ticker"), use_container_width=True, height=400)
    
    st.markdown("<br>", unsafe_allow_html=True)
    
    st.subheader("🔗 Matrice di Correlazione")
    if len(df_final.columns) > 1:
        corr = df_final.pct_change().corr()
        plt.style.use("default")
        fig, ax = plt.subplots(figsize=(10, 6))
        sns.heatmap(corr, annot=True, cmap="RdYlGn", fmt=".2f", vmin=-1, vmax=1, 
                    ax=ax, cbar_kws={'label': 'Correlazione'},
                    linewidths=0.5, linecolor='white')
        ax.set_title("Matrice di Correlazione tra Asset", fontsize=14, fontweight='bold', pad=20)
        st.pyplot(fig)
    
    st.markdown("<br>", unsafe_allow_html=True)
    st.markdown("### 📥 Download Dati")
    df_final.index.name = "Data"
    csv = df_final.to_csv(sep=";", decimal=",", encoding="utf-8-sig")
    st.download_button(
        label=f"📥 SCARICA CSV ({freq_display.upper()})", 
        data=csv, 
        file_name=f"Serie_Storiche_{freq_display}.csv", 
        mime="text/csv"
    )


# ── OPTIMIZATION TABS (FROM CODICE B) ────────────────────────────
# Preparazione dati per ottimizzazione
with st.spinner("Preparazione dati per ottimizzazione..."):
    mu_strat, sigma_strat, meta = prep_data(df, all_assets, lookback, freq)

if mu_strat is None:
    st.error(f"❌ {meta}")
    st.stop()

_, returns_wf, ann_factor_opt = meta


# ── TAB 2: MARKOWITZ ─────────────────────────────────────────────
with tab2:
    st.markdown('<div class="section-header">Mean-Variance Optimization · SLSQP</div>',
                unsafe_allow_html=True)

    w_mk = get_optimal_weights(mu_strat, sigma_strat, min_weight, max_weight, rf)
    if w_mk is None:
        w_mk = np.array([1.0 / len(all_assets)] * len(all_assets))
        st.warning("⚠️ Ottimizzazione non convergita → Fallback Equal Weight.")

    m_mk = portfolio_metrics(w_mk, mu_strat, sigma_strat, rf)
    kpi_row([
        {"label": "Rendimento Atteso (ann.)", "value": f"{m_mk['return']*100:.2f}%",
         "positive": m_mk['return'] > 0, "sub": "Mean-Variance"},
        {"label": "Volatilità Attesa (ann.)", "value": f"{m_mk['volatility']*100:.2f}%",
         "sub": "Deviazione Standard"},
        {"label": "Sharpe Ratio",            "value": f"{m_mk['sharpe']:.3f}",
         "positive": m_mk['sharpe'] > 1, "sub": f"RF = {rf*100:.1f}%"},
    ])
    st.markdown("<br>", unsafe_allow_html=True)
    col_t, col_p = st.columns(2, gap="large")
    with col_t:
        allocation_table(all_assets, w_mk)
    with col_p:
        st.plotly_chart(pie_chart(all_assets, w_mk, "Allocazione Markowitz"),
                        use_container_width=True)


# ── TAB 3: MONTECARLO ────────────────────────────────────────────
with tab3:
    st.markdown('<div class="section-header">Simulazione Montecarlo · 10,000 Portafogli Casuali</div>',
                unsafe_allow_html=True)

    with st.spinner("Generazione portafogli Montecarlo…"):
        w_mc = get_montecarlo_weights(mu_strat, sigma_strat, min_weight, max_weight,
                                      rf, num_sims=10_000)
    if w_mc is None:
        w_mc = np.array([1.0 / len(all_assets)] * len(all_assets))
        st.warning("⚠️ Simulazione non ha trovato portafogli validi → Fallback Equal Weight.")

    m_mc = portfolio_metrics(w_mc, mu_strat, sigma_strat, rf)
    kpi_row([
        {"label": "Rendimento Atteso (ann.)", "value": f"{m_mc['return']*100:.2f}%",
         "positive": m_mc['return'] > 0, "sub": "Montecarlo Best"},
        {"label": "Volatilità Attesa (ann.)", "value": f"{m_mc['volatility']*100:.2f}%",
         "sub": "Deviazione Standard"},
        {"label": "Sharpe Ratio",            "value": f"{m_mc['sharpe']:.3f}",
         "positive": m_mc['sharpe'] > 1, "sub": f"RF = {rf*100:.1f}%"},
    ])
    st.markdown("<br>", unsafe_allow_html=True)
    col_t2, col_p2 = st.columns(2, gap="large")
    with col_t2:
        allocation_table(all_assets, w_mc)
    with col_p2:
        st.plotly_chart(pie_chart(all_assets, w_mc, "Allocazione Montecarlo"),
                        use_container_width=True)


# ── TAB 4: ANTIFRAGILE ───────────────────────────────────────────
with tab4:
    st.markdown('<div class="section-header">Approccio Antifragile: Min CVaR (Tail Risk) & Ledoit-Wolf Shrinkage</div>',
                unsafe_allow_html=True)
    
    with st.spinner("Calcolo Matrice Covarianza Robusta e Ottimizzazione Code Grasse (CVaR)..."):
        try:
            # 1. GMV con Shrinkage
            lw = LedoitWolf()
            sigma_shrunk = lw.fit(returns_wf).covariance_ * ann_factor_opt
            w_gmv = get_gmv_weights(sigma_shrunk, min_weight, max_weight)
            if w_gmv is None:
                w_gmv = np.array([1.0 / len(all_assets)] * len(all_assets))
            
            # 2. Ottimizzazione CVaR
            w_cvar = get_cvar_weights(returns_wf.values, min_weight, max_weight, alpha=0.05)
            if w_cvar is None:
                w_cvar = np.array([1.0 / len(all_assets)] * len(all_assets))

            m_cvar = portfolio_metrics(w_cvar, mu_strat, sigma_strat, rf)
            
            st.markdown("#### Minimizzazione Rischio di Rovina (CVaR - 95%)")
            kpi_row([
                {"label": "Rendimento Atteso (ann.)", "value": f"{m_cvar['return']*100:.2f}%",
                 "positive": m_cvar['return'] > 0, "sub": "Pesi CVaR"},
                {"label": "Volatilità Attesa (ann.)", "value": f"{m_cvar['volatility']*100:.2f}%",
                 "sub": "Meno rilevante qui"},
                {"label": "Sharpe Ratio",            "value": f"{m_cvar['sharpe']:.3f}",
                 "positive": m_cvar['sharpe'] > 1, "sub": f"RF = {rf*100:.1f}%"},
            ])
            st.markdown("<br>", unsafe_allow_html=True)
            
            col_t3, col_p3 = st.columns(2, gap="large")
            with col_t3:
                st.markdown("##### Allocazione Min-CVaR (Difesa Eventi Estremi)")
                allocation_table(all_assets, w_cvar)
            with col_p3:
                st.plotly_chart(pie_chart(all_assets, w_cvar, "Allocazione Ottimizzata per le Code (CVaR)"),
                                use_container_width=True)
                
            st.divider()

            col_t4, col_p4 = st.columns(2, gap="large")
            with col_t4:
                st.markdown("##### Allocazione GMV con Shrinkage (Ledoit-Wolf)")
                allocation_table(all_assets, w_gmv)
            with col_p4:
                st.plotly_chart(pie_chart(all_assets, w_gmv, "Allocazione Minima Varianza Robusta"),
                                use_container_width=True)

            st.markdown(f"""
            <div style="font-size:0.85rem; color:{TEXT_SECONDARY}; margin-top:1.5rem; 
                        padding:1.2rem; border-left: 4px solid {COLOR_HIGHLIGHT};
                        background:{BG_CARD}; border-radius:8px;">
                <b style="color:{COLOR_HIGHLIGHT};">Analisi Specchio:</b> Markowitz massimizza gli errori; 
                questi due modelli gestiscono i disastri. Il <b>GMV Shrinkage</b> purifica i dati dalle 
                correlazioni fasulle. Il modello <b>CVaR</b> va oltre: ignora la normale fluttuazione 
                giornaliera e ottimizza i pesi per difenderti esclusivamente da quel 5% dei giorni storici 
                in cui il mercato è crollato (Tail Risk).
            </div>
            """, unsafe_allow_html=True)
        except Exception as e:
            st.error(f"❌ Errore nel calcolo Antifragile: {e}")


# ── TAB 5: BACKTEST ──────────────────────────────────────────────
with tab5:
    st.markdown('<div class="section-header">Walk-Forward Backtest · Sfida a 4 Modelli</div>',
                unsafe_allow_html=True)

    window_size = int(lookback * ann_factor_opt)

    if len(returns_wf) <= window_size:
        st.error(
            f"❌ Dati insufficienti per il backtest. "
            f"Richiesto: {window_size} periodi ({lookback} anni). "
            f"Disponibili: {len(returns_wf)}. "
            f"Abbassa lo slider 'Orizzonte Rolling' nella sidebar."
        )
    else:
        wf_ret_mk, wf_ret_mc, wf_ret_gmv, wf_ret_cvar, wf_dates = [], [], [], [], []
        eq_w      = np.array([1.0 / len(all_assets)] * len(all_assets))
        w_opt_mk  = eq_w.copy()
        w_opt_mc  = eq_w.copy()
        w_opt_gmv = eq_w.copy()
        w_opt_cvar = eq_w.copy()

        progress_bar = st.progress(0, text="Calcolo stress-test storici...")
        num_steps = len(returns_wf) - window_size

        for i in range(window_size, len(returns_wf)):
            if i % 10 == 0:
                progress_bar.progress(min((i - window_size) / num_steps, 1.0),
                                       text=f"Ribilanciamento Dinamico: Step {i - window_size} / {num_steps}")

            window_data = returns_wf.iloc[i - window_size: i]
            if window_data.std().sum() > 0:
                mu_win    = window_data.mean() * ann_factor_opt
                sigma_win = window_data.cov()  * ann_factor_opt

                # 1. Markowitz
                w_new_mk = get_optimal_weights(mu_win, sigma_win, min_weight, max_weight, rf)
                if w_new_mk is not None: w_opt_mk = w_new_mk

                # 2. Montecarlo
                w_new_mc = get_montecarlo_weights(mu_win, sigma_win, min_weight, max_weight, rf, num_sims=2000)
                if w_new_mc is not None: w_opt_mc = w_new_mc
                
                # 3. GMV & Shrinkage
                try:
                    lw_win = LedoitWolf()
                    sigma_shrunk_win = lw_win.fit(window_data).covariance_ * ann_factor_opt
                    w_new_gmv = get_gmv_weights(sigma_shrunk_win, min_weight, max_weight)
                    if w_new_gmv is not None: w_opt_gmv = w_new_gmv
                except:
                    pass
                
                # 4. Min CVaR
                w_new_cvar = get_cvar_weights(window_data.values, min_weight, max_weight, alpha=0.05)
                if w_new_cvar is not None: w_opt_cvar = w_new_cvar

            next_ret = returns_wf.iloc[i].values
            wf_ret_mk.append(np.sum(w_opt_mk * next_ret))
            wf_ret_mc.append(np.sum(w_opt_mc * next_ret))
            wf_ret_gmv.append(np.sum(w_opt_gmv * next_ret))
            wf_ret_cvar.append(np.sum(w_opt_cvar * next_ret))
            wf_dates.append(returns_wf.index[i])

        progress_bar.empty()

        df_wf = pd.DataFrame({
            "Markowitz (Avido)": wf_ret_mk, 
            "Montecarlo": wf_ret_mc,
            "GMV Shrinkage (Sicuro)": wf_ret_gmv,
            "Min CVaR (Difesa Code)": wf_ret_cvar
        }, index=wf_dates)
        
        nav = compute_nav(df_wf)

        col_eq, col_dd = st.columns(2, gap="large")
        with col_eq:
            st.plotly_chart(equity_line_chart(nav), use_container_width=True)
        with col_dd:
            st.plotly_chart(drawdown_chart(nav), use_container_width=True)
            
        st.markdown("#### Tabella Comparativa Performance")
        
        results_list = []
        for col in df_wf.columns:
            ret_tot = nav[col].iloc[-1] / 100 - 1
            dd_max = max_drawdown(nav[col])
            vol = df_wf[col].std() * np.sqrt(ann_factor_opt)
            sharpe = (ret_tot - rf) / vol if vol > 0 else 0
            results_list.append({
                "Modello": col,
                "Return Totale": f"{ret_tot*100:.2f}%",
                "Max Drawdown": f"{dd_max*100:.2f}%",
                "Volatilità": f"{vol*100:.2f}%",
                "Sharpe": f"{sharpe:.2f}"
            })
            
        st.table(pd.DataFrame(results_list).set_index("Modello"))


# ── TAB 6: PROIEZIONE FUTURA ─────────────────────────────────────
with tab6:
    st.markdown('<div class="section-header">Simulazione Stocastica · Cono di Probabilità (GBM)</div>', 
                unsafe_allow_html=True)
    
    col_param1, col_param2 = st.columns(2)
    with col_param1:
        anni_futuri = st.slider("Anni di Proiezione", 1, 10, 5, key="anni_proj")
    with col_param2:
        num_simulazioni = st.selectbox("Numero Scenari Paralleli", [1000, 5000, 10000], index=0, key="num_sim")

    with st.spinner("Calcolo traiettorie quantistiche..."):
        w_proj = w_cvar if 'w_cvar' in locals() and w_cvar is not None else np.array([1.0 / len(all_assets)] * len(all_assets))
        cov_proj = sigma_shrunk if 'sigma_shrunk' in locals() else sigma_strat
        
        port_mu = float(np.sum(mu_strat * w_proj))
        port_vol = float(np.sqrt(np.dot(w_proj.T, np.dot(cov_proj, w_proj))))
        
        giorni_trading = 252 * anni_futuri
        dt = 1 / 252 
        
        sim_matrix = np.zeros((giorni_trading + 1, num_simulazioni))
        sim_matrix[0] = 100.0
        
        Z = np.random.standard_normal((giorni_trading, num_simulazioni))
        daily_drift = (port_mu - 0.5 * port_vol**2) * dt
        daily_shock = port_vol * np.sqrt(dt) * Z
        
        sim_matrix[1:] = np.exp(daily_drift + daily_shock)
        sim_matrix = np.cumprod(sim_matrix, axis=0)
        
        percentiles = np.percentile(sim_matrix, [5, 25, 50, 75, 95], axis=1)
        
        last_date = returns_wf.index[-1] if not returns_wf.empty else pd.Timestamp.today()
        future_dates = pd.date_range(start=last_date, periods=giorni_trading + 1, freq='B')

        fig_proj = _base_fig(title=dict(text="Cono di Incertezza (Base 100) - Ancorato a Pesi CVaR", 
                                        font=dict(size=14, color=TEXT_PRIMARY, weight=600), x=0))
        
        fig_proj.add_trace(go.Scatter(x=future_dates, y=percentiles[4], mode='lines', 
                                      line=dict(width=0), showlegend=False, hoverinfo='skip'))
        fig_proj.add_trace(go.Scatter(x=future_dates, y=percentiles[0], mode='lines', 
                                      line=dict(width=0), fill='tonexty', 
                                      fillcolor=_hex_to_rgba(COLOR_HIGHLIGHT, 0.1), 
                                      name='Banda 5%-95%'))
        
        fig_proj.add_trace(go.Scatter(x=future_dates, y=percentiles[3], mode='lines', 
                                      line=dict(width=0), showlegend=False, hoverinfo='skip'))
        fig_proj.add_trace(go.Scatter(x=future_dates, y=percentiles[1], mode='lines', 
                                      line=dict(width=0), fill='tonexty', 
                                      fillcolor=_hex_to_rgba(COLOR_HIGHLIGHT, 0.2), 
                                      name='Banda 25%-75%'))
        
        fig_proj.add_trace(go.Scatter(x=future_dates, y=percentiles[2], mode='lines', 
                                      line=dict(color=COLOR_HIGHLIGHT, width=3), 
                                      name='Mediana (P50)'))
        fig_proj.add_trace(go.Scatter(x=future_dates, y=percentiles[0], mode='lines', 
                                      line=dict(color=COLOR_RED, width=2, dash='dot'), 
                                      name='Pessimo (P05)'))

        fig_proj.update_layout(
            yaxis=dict(**PLOTLY_LAYOUT["yaxis"], title="Capitale Proiettato"),
            legend=dict(orientation="h", yanchor="bottom", y=1.02, xanchor="right", x=1),
            hovermode="x unified"
        )
        
        st.plotly_chart(fig_proj, use_container_width=True)

        ret_p05 = (percentiles[0][-1] / 100.0) - 1
        ret_p50 = (percentiles[2][-1] / 100.0) - 1
        ret_p95 = (percentiles[4][-1] / 100.0) - 1
        
        st.markdown("#### Analisi Scenari a Scadenza (Valore Capitale Base 100)")
        kpi_row([
            {"label": "Valore Mediano (P50)", "value": f"{percentiles[2][-1]:.1f}", 
             "positive": percentiles[2][-1] > 100, "sub": "Il risultato più probabile"},
            {"label": "Worst Case (P05)", "value": f"{percentiles[0][-1]:.1f}", 
             "positive": percentiles[0][-1] > 100, "sub": "Cosa perdi in mercati ostili"},
            {"label": "Best Case (P95)", "value": f"{percentiles[4][-1]:.1f}", 
             "positive": True, "sub": "Ottimismo statistico"},
        ])
        
        st.markdown("<br>", unsafe_allow_html=True)
        df_ret_futuri = pd.DataFrame({
            "Scenario": ["Worst Case (P05)", "Valore Mediano (P50)", "Best Case (P95)"],
            "Rendimento Cumulato (%)": [f"{ret_p05*100:.2f} %", f"{ret_p50*100:.2f} %", f"{ret_p95*100:.2f} %"]
        })
        st.markdown("#### Rendimenti Percentuali Attesi a Scadenza")
        st.table(df_ret_futuri)


# ── TAB 7: MERCATO LIVE ──────────────────────────────────────────
with tab7:
    st.markdown('<div class="section-header">Live Market Widgets</div>', unsafe_allow_html=True)
    c1, c2 = st.columns([2, 1], gap="medium")
    with c1:
        st.markdown("#### 📅 Calendario Economico")
        components.html("""
        <iframe src="https://sslecal2.investing.com?ecoDayBackground=%23FFFFFF&defaultFont=%231A202C&borderColor=%23E2E8F0&columns=exc_flags,exc_currency,exc_importance,exc_actual,exc_forecast,exc_previous&features=datepicker,timezone&countries=25,32,6,37,72,22,17,39,14,10,35,43,56,36,110,11,26,12,4,5&calType=week&timeZone=8&lang=1" width="650" height="467" frameborder="0" allowtransparency="true" marginwidth="0" marginheight="0"></iframe>
        """, height=500)
    with c2:
        st.markdown("#### 📊 Technical Summary")
        components.html("""
        <iframe src="https://ssltsw.investing.com?lang=1&forex=1,2,3,5,7,9,10&commodities=8830,8836,8831,8849,8833,8862,8832&indices=175,166,172,27,179,170,174&stocks=345,346,347,348,349,350,352&tabs=1,2,3,4" width="317" height="467"></iframe>
        """, height=500)


# ── TAB 8: METODOLOGIA ───────────────────────────────────────────
with tab8:
    st.markdown(f"""
    <div class="methodology-box">
        <h3>📖 Metodologia & Approccio Quantitativo</h3>
        
        <h4>1 · La Trappola (Markowitz & Montecarlo)</h4>
        <p>Ottimizzano usando rendimenti attesi derivati dal passato. Generano portafogli instabili, 
        avidi e che tendono a massimizzare le deviazioni statistiche casuali. Ottimi per analisi a posteriori, 
        pessimi per gestire capitali reali nel futuro.</p>
        
        <h4>2 · L'Antidoto Statistico (GMV & Ledoit-Wolf Shrinkage)</h4>
        <p>Smette di tentare di prevedere i ritorni e si concentra solo sulla volatilità. La matrice di 
        covarianza viene "compressa" con algoritmi di Machine Learning (Shrinkage) per eliminare falsi 
        positivi di decorrelazione. Costruisce il portafoglio matematicamente più solido a prescindere 
        dalla direzione del mercato.</p>
        
        <h4>3 · La Difesa Assoluta (Min-CVaR Optimization)</h4>
        <p>La varianza tratta i guadagni esplosivi come un difetto (poiché si discostano dalla media). 
        Il <b>Conditional Value at Risk (Expected Shortfall)</b> no. Questo modello ignora il 95% della 
        vita tranquilla del mercato e calcola i pesi al solo scopo di minimizzare le perdite nel restante 
        5% dei casi (Eventi Estremi/Cigni Neri).</p>
        
        <h4>4 · Verità sui Dati (Total Return)</h4>
        <p>L'intero framework presuppone obbligatoriamente l'utilizzo di dati Total Return (Adjusted Close). 
        Inserire prezzi "puliti" annulla l'impatto critico dell'interesse composto generato dai dividendi 
        nel tempo, penalizzando strutturalmente bond e utility.</p>
        
        <h4>5 · Architettura Applicazione</h4>
        <p>Questa piattaforma integra due moduli distinti: il <b>Data Engine</b> per l'acquisizione di serie 
        storiche (Yahoo Finance, Morningstar, upload manuale) e il <b>Portfolio Optimizer</b> per 
        l'ottimizzazione quantitativa multi-modello. Il flusso dati è completamente automatizzato: i dati 
        scaricati o caricati vengono passati automaticamente agli algoritmi di ottimizzazione.</p>
    </div>
    """, unsafe_allow_html=True)
    
    st.markdown("<br>", unsafe_allow_html=True)
    
    st.markdown(f"""
    <div style="background:{BG_CARD}; padding:1.5rem; border-radius:{BORDER_RADIUS}; 
                border:2px solid {BORDER_COLOR};">
        <h4 style="color:{COLOR_HIGHLIGHT}; margin-top:0;">Codici Ticker Utili</h4>
        <div style="color:{TEXT_SECONDARY}; font-size:0.85rem; line-height:1.8;">
            <b>Indici USA:</b> ^GSPC (S&P 500), ^NDX (NASDAQ 100), ^DJI (Dow Jones), ^VIX (Volatilità)<br>
            <b>Indici Europa:</b> ^GDAXI (DAX), ^FCHI (CAC 40), ^STOXX50E (Euro Stoxx 50), FTSEMIB.MI (FTSE MIB)<br>
            <b>Materie Prime:</b> GC=F (Oro), CL=F (Petrolio), SI=F (Argento)<br>
            <b>Crypto:</b> BTC-USD (Bitcoin), ETH-USD (Ethereum)<br>
            <b>ETF Milano:</b> SWDA.MI (MSCI World), EIMI.MI (MSCI Emerging Markets)
        </div>
    </div>
    """, unsafe_allow_html=True)
