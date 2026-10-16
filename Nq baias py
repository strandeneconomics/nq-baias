import datetime as dt
import pandas as pd
import yfinance as yf

def fred(i):
    s = pd.read_csv(f"https://fred.stlouisfed.org/graph/fredgraph.csv?id={i}",
                    index_col=0, parse_dates=True).iloc[:, 0]
    return pd.to_numeric(s, errors="coerce").dropna()

def sc(x, c1, c2, invert=False):
    s = 2 if x >= c2 else 1 if x >= c1 else -2 if x <= -c2 else -1 if x <= -c1 else 0
    return -s if invert else s

T = ["NQ=F", "^VIX", "DX-Y.NYB", "SMH", "QQQ", "QQEW", "AAPL", "MSFT", "NVDA", "GOOGL", "AMZN", "META", "TSLA"]
px = yf.download(T, period="1y", auto_adjust=True, progress=False)["Close"].ffill()
r = lambda t, n=21: px[t].iloc[-1] / px[t].iloc[-1 - n] - 1

def f_fed():
    d = fred("DGS2"); ch = d.iloc[-1] - d.iloc[-21]
    return sc(ch, 0.10, 0.25, True), f"2Y {d.iloc[-1]:.2f}% ({ch:+.2f} pp siste 20d)"
def f_cpi():
    y = fred("CPIAUCSL").pct_change(12).dropna() * 100; ch = y.iloc[-1] - y.iloc[-4]
    return sc(ch, 0.1, 0.3, True), f"CPI {y.iloc[-1]:.1f}% å/å ({ch:+.1f} pp siste 3 mnd)"
def f_10y():
    d = fred("DGS10"); ch = d.iloc[-1] - d.iloc[-21]
    return sc(ch, 0.10, 0.25, True), f"10Y {d.iloc[-1]:.2f}% ({ch:+.2f} pp siste 20d)"
def f_dxy():
    x = r("DX-Y.NYB"); return sc(x, 0.007, 0.02, True), f"DXY {x:+.1%} siste måned"
def f_jobs():
    u = fred("UNRATE"); ch = u.iloc[-1] - u.iloc[-4]
    s = -2 if ch >= 0.5 else -1 if ch >= 0.3 else 1 if ch <= -0.2 else 0
    return s, f"Arbeidsledighet {u.iloc[-1]:.1f}% ({ch:+.1f} pp siste 3 mnd)"
def f_vix():
    v = px["^VIX"].iloc[-1]
    return (-2 if v > 30 else -1 if v > 22 else 2 if v < 14 else 1 if v < 18 else 0), f"VIX {v:.1f}"
def f_liq():
    w = fred("WALCL"); ch = w.iloc[-1] / w.iloc[-5] - 1
    return sc(ch, 0.003, 0.01), f"Fed-balanse {ch:+.2%} siste 4 uker"
def f_mega():
    a = sum(r(t) for t in ["AAPL", "MSFT", "NVDA", "GOOGL", "AMZN", "META", "TSLA"]) / 7
    return sc(a, 0.02, 0.05), f"Snitt mega-cap {a:+.1%} siste måned (momentum som proxy for inntjening)"
def f_smh():
    x = px["SMH"].iloc[-1] / px["SMH"].rolling(50).mean().iloc[-1] - 1
    return sc(x, 0.015, 0.05), f"SMH {x:+.1%} vs 50-dagers"
def f_breadth():
    x = r("QQEW") - r("QQQ"); return sc(x, 0.003, 0.01), f"QQEW – QQQ {x:+.1%} siste måned (bredde-proxy)"
def f_val():
    pe = yf.Ticker("QQQ").info.get("trailingPE")
    return (-2 if pe > 36 else -1 if pe > 32 else 2 if pe < 20 else 1 if pe < 24 else 0), f"QQQ trailing P/E {pe:.1f}"
def f_sent():
    d = px["NQ=F"].diff(); up = d.clip(lower=0).rolling(14).mean(); dn = (-d.clip(upper=0)).rolling(14).mean()
    rsi = (100 - 100 / (1 + up / dn)).iloc[-1]
    return (-2 if rsi > 75 else -1 if rsi > 70 else 2 if rsi < 25 else 1 if rsi < 30 else 0), f"NQ RSI(14) {rsi:.0f} (kontrær)"

FACT = [("Makro", "Fed / rentebane (2Y)", 3, f_fed), ("Makro", "Inflasjon (CPI)", 2, f_cpi),
        ("Makro", "10-års rente", 3, f_10y), ("Makro", "Dollar (DXY)", 1, f_dxy),
        ("Makro", "Vekst / arbeidsmarked", 2, f_jobs), ("Makro", "Risikoappetitt (VIX)", 2, f_vix),
        ("Makro", "Likviditet", 1, f_liq), ("Mikro", "Mega-cap momentum", 3, f_mega),
        ("Mikro", "AI / halvledere", 2, f_smh), ("Mikro", "Markedsbredde", 1, f_breadth),
        ("Mikro", "Verdsettelse", 1, f_val), ("Mikro", "Sentiment (kontrær)", 1, f_sent)]

rows = []
for g, n, w, fn in FACT:
    try: s, d = fn()
    except Exception as e: s, d = 0, f"mangler data ({type(e).__name__})"
    rows.append((g, n, w, s, d))

def score(grp=None):
    a = sum(s * w for g, _, w, s, _ in rows if grp in (None, g))
    b = sum(2 * w for g, _, w, _, _ in rows if grp in (None, g))
    return round(a / b * 100)

def verdict(x):
    return ("Sterkt bullish" if x >= 40 else "Bullish bias" if x >= 15 else "Nøytral / choppy" if x > -15
            else "Bearish bias" if x > -40 else "Sterkt bearish")

tot, mac, mik = score(), score("Makro"), score("Mikro")
nq = px["NQ=F"]; trend = f"NQ {'over' if nq.iloc[-1] > nq.rolling(50).mean().iloc[-1] else 'under'} 50-dagers, " \
     f"{'over' if nq.iloc[-1] > nq.rolling(200).mean().iloc[-1] else 'under'} 200-dagers"
conflict = "⚠ Makro og mikro peker motsatt vei" if mac * mik < 0 and abs(mac) >= 20 and abs(mik) >= 20 else ""

print(f"NQ BIAS: {verdict(tot)} (score {tot:+d}) | makro {mac:+d} mikro {mik:+d} | {trend} {conflict}")
for g, n, w, s, d in rows: print(f"{g:5} {n:24} v{w} {s:+d}  {d}")

import json, zoneinfo
json.dump({
    "ts": dt.datetime.now(dt.timezone.utc).isoformat(),
    "updated": dt.datetime.now(zoneinfo.ZoneInfo("America/New_York")).strftime("%Y-%m-%d %H:%M"),
    "total": tot, "macro": mac, "micro": mik, "verdict": verdict(tot),
    "trend": trend, "conflict": conflict,
    "rows": [dict(g=g, n=n, w=w, s=s, d=d) for g, n, w, s, d in rows],
}, open("data.json", "w"), ensure_ascii=False, default=str)
