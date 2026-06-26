import os
import smtplib
import requests
from email.mime.multipart import MIMEMultipart
from email.mime.text import MIMEText
from datetime import datetime
from bs4 import BeautifulSoup
import urllib.parse

# ── Configurazione ──────────────────────────────────────────────
GMAIL_USER     = os.environ["GMAIL_USER"]
GMAIL_PASSWORD = os.environ["GMAIL_APP_PW"]
EMAIL_TO       = os.environ["EMAIL_TO"]

HEADERS = {
    "User-Agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) "
                  "AppleWebKit/537.36 (KHTML, like Gecko) "
                  "Chrome/120.0.0.0 Safari/537.36"
}

# ── Query per ogni categoria ────────────────────────────────────
QUERIES = [
    {
        "titolo": "🎬 Bandi cinema e documentario — Italia",
        "colore": "#c0392b",
        "query": "bando finanziamento cinema documentario produzione audiovisiva Italia 2026",
    },
    {
        "titolo": "🎞️ MIC — Ministero della Cultura (contributi selettivi)",
        "colore": "#e74c3c",
        "query": "MIC ministero cultura contributi selettivi cinema documentario 2026 bando",
    },
    {
        "titolo": "📺 Bandi RAI, Rai Cinema, Luce Cinecittà",
        "colore": "#e67e22",
        "query": "bando RAI Cinema Cinecittà Luce documentario coproduzione 2026",
    },
    {
        "titolo": "🏛️ Bandi regionali — produzione audiovisiva",
        "colore": "#d35400",
        "query": "bando regionale film commission produzione audiovisiva cinema documentario 2026",
    },
    {
        "titolo": "🇪🇺 Creative Europe MEDIA — sviluppo e produzione",
        "colore": "#2980b9",
        "query": "Creative Europe MEDIA development production documentary 2026 call",
    },
    {
        "titolo": "🌍 EURIMAGES — coproduzione europea",
        "colore": "#1a5276",
        "query": "Eurimages coproduction fund documentary feature film 2026 call",
    },
    {
        "titolo": "🌐 Bandi internazionali — documentario e cinema",
        "colore": "#27ae60",
        "query": "international fund documentary production grant 2026 IDFA Sundance Tribeca",
    },
    {
        "titolo": "💰 Tax credit e incentivi fiscali audiovisivo",
        "colore": "#8e44ad",
        "query": "tax credit cinema audiovisivo produzione Italia 2026 incentivi",
    },
]

# ── Fetch Google News RSS ───────────────────────────────────────
def cerca_news(query, max_results=6):
    q = urllib.parse.quote_plus(query)
    url = f"https://news.google.com/rss/search?q={q}&hl=it&gl=IT&ceid=IT:it"
    try:
        r = requests.get(url, headers=HEADERS, timeout=10)
        soup = BeautifulSoup(r.content, "xml")
        items = soup.find_all("item")[:max_results]
        results = []
        for item in items:
            title = item.find("title").get_text(strip=True) if item.find("title") else "—"
            link  = item.find("link").get_text(strip=True) if item.find("link") else "#"
            pub   = item.find("pubDate")
            pub_str = pub.get_text(strip=True)[:16] if pub else ""
            # rimuovi fonte dal titolo (Google News aggiunge " - Fonte")
            if " - " in title:
                title = title.rsplit(" - ", 1)[0]
            results.append({"title": title, "url": link, "date": pub_str})
        return results
    except Exception as e:
        print(f"Errore RSS '{query}': {e}")
        return []

# ── Sezione HTML ────────────────────────────────────────────────
def build_section(titolo, colore, query):
    risultati = cerca_news(query)
    rows = ""
    if risultati:
        for r in risultati:
            rows += f"""
            <tr>
              <td style="padding:7px 8px;border-bottom:1px solid #f0f0f0;">
                <a href="{r['url']}" style="color:#1a73e8;text-decoration:none;font-weight:500;line-height:1.4;">{r['title']}</a>
              </td>
              <td style="padding:7px 8px;border-bottom:1px solid #f0f0f0;color:#999;font-size:12px;white-space:nowrap;vertical-align:top;">{r.get('date','')}</td>
            </tr>"""
    else:
        rows = '<tr><td colspan="2" style="padding:8px;color:#bbb;font-size:13px;">Nessun risultato questa settimana</td></tr>'

    return f"""
    <div style="margin-bottom:28px;">
      <h2 style="font-size:15px;color:#222;border-left:4px solid {colore};padding-left:10px;margin-bottom:10px;line-height:1.3;">{titolo}</h2>
      <table style="width:100%;border-collapse:collapse;font-size:14px;">{rows}</table>
    </div>"""

# ── Link utili fissi ────────────────────────────────────────────
LINK_UTILI = [
    ("MIC — Direzione Cinema e Audiovisivo", "https://cinema.cultura.gov.it"),
    ("Creative Europe MEDIA", "https://media.cultureandcreativity.eu/funding-opportunities"),
    ("Eurimages", "https://www.coe.int/en/web/eurimages/calls"),
    ("Rai Cinema — bandi", "https://www.raicinema.it"),
    ("Italian Film Commissions", "https://www.italianfilmcommissions.it"),
    ("IDFA Bertha Fund", "https://www.idfa.nl/en/info/idfa-bertha-fund"),
    ("Sundance Documentary Fund", "https://www.sundance.org/programs/documentary-fund"),
    ("Hot Docs Forum", "https://hotdocs.ca/programs/hot-docs-forum"),
]

def build_link_utili():
    righe = "".join(
        f'<li style="margin-bottom:6px;"><a href="{url}" style="color:#1a73e8;text-decoration:none;">{nome}</a></li>'
        for nome, url in LINK_UTILI
    )
    return f"""
    <div style="margin-bottom:28px;">
      <h2 style="font-size:15px;color:#222;border-left:4px solid #555;padding-left:10px;margin-bottom:10px;">🔗 Fonti da controllare sempre</h2>
      <ul style="margin:0;padding-left:16px;font-size:14px;color:#333;">{righe}</ul>
    </div>"""

# ── Email completa ──────────────────────────────────────────────
def build_email():
    oggi = datetime.now().strftime("%d %B %Y")
    sezioni = "".join(build_section(**q) for q in QUERIES)
    link_utili = build_link_utili()

    return f"""<!DOCTYPE html>
    <html>
    <head><meta charset="utf-8"></head>
    <body style="font-family:-apple-system,BlinkMacSystemFont,'Segoe UI',sans-serif;max-width:660px;margin:0 auto;padding:20px;color:#222;">
      <div style="background:linear-gradient(135deg,#1a1a2e,#c0392b);color:white;padding:22px 24px;border-radius:12px;margin-bottom:28px;">
        <h1 style="margin:0;font-size:22px;">🎬 Bandi Cinema & Documentario</h1>
        <p style="margin:6px 0 0;opacity:0.85;font-size:14px;">{oggi} · Report settimanale per case di produzione</p>
      </div>
      <p style="font-size:14px;color:#555;margin-bottom:24px;line-height:1.6;">
        Riepilogo settimanale di bandi, fondi e opportunità di finanziamento per la produzione di cinema, documentari e contenuti audiovisivi — nazionali, europei e internazionali.
      </p>
      {sezioni}
      {link_utili}
      <hr style="border:none;border-top:1px solid #eee;margin:24px 0;">
      <p style="font-size:12px;color:#bbb;text-align:center;">
        Report automatico · GitHub Actions · ogni lunedì alle 8:00<br>
        I risultati provengono da Google News RSS — verifica sempre le fonti ufficiali.
      </p>
    </body>
    </html>"""

# ── Invio ───────────────────────────────────────────────────────
def send_email(html_body):
    msg = MIMEMultipart("alternative")
    msg["Subject"] = f"🎬 Bandi Cinema & Documentario — {datetime.now().strftime('%d/%m/%Y')}"
    msg["From"]    = GMAIL_USER
    msg["To"]      = EMAIL_TO
    msg.attach(MIMEText(html_body, "html"))
    with smtplib.SMTP_SSL("smtp.gmail.com", 465) as server:
        server.login(GMAIL_USER, GMAIL_PASSWORD)
        server.sendmail(GMAIL_USER, EMAIL_TO, msg.as_string())
    print(f"✅ Email inviata a {EMAIL_TO}")

if __name__ == "__main__":
    print("🔍 Raccolta bandi in corso...")
    html = build_email()
    send_email(html)
