  }

.subbrand {
  font-size: 10px;
  opacity: 0.95;
}

.badge {
  background: #d9a441;
  color: #111;
  font-weight: 700;
  padding: 8px 14px;
  border-radius: 4px;
  text-align: center;
  min-width: 120px;
}

.container {
  padding: 18px;
}

.title {
  font-size: 28px;
  font-weight: 800;
  margin: 10px 0 2px 0;
}

.subtitle {
  color: #555;
  font-size: 13px;
  margin-bottom: 18px;
}

.grid {
  display: grid;
  grid-template-columns: repeat(5, 1fr);
  gap: 10px;
  margin: 16px 0 20px 0;
}

.card {
  border: 1px solid #d9e1ee;
  background: #f7f9fc;
  border-radius: 8px;
  padding: 10px;
  min-height: 82px;
}

.card .label {
  font-size: 9px;
  color: #666;
  text-transform: uppercase;
  letter-spacing: 0.4px;
}

.card .value {
  font-size: 15px;
  font-weight: 700;
  margin-top: 6px;
}

.section {
  margin: 16px 0 18px 0;
}

.section h2 {
  font-size: 18px;
  margin: 0 0 8px 0;
  color: #0b2d5b;
}

.section h3 {
  font-size: 14px;
  margin: 12px 0 6px 0;
  color: #1a1a1a;
}

p {
  margin: 0 0 10px 0;
  font-size: 11px;
}

ul, ol {
  margin: 0 0 10px 18px;
  padding: 0;
  font-size: 11px;
}

li {
  margin-bottom: 5px;
}

.table {
  width: 100%;
  border-collapse: collapse;
  margin: 10px 0 14px 0;
  font-size: 10px;
}

.table th, .table td {
  border: 1px solid #cfd8e6;
  padding: 7px 8px;
  vertical-align: top;
}

.table th {
  background: #0b2d5b;
  color: white;
  text-align: left;
}

.table tr:nth-child(even) td {
  background: #f7f9fc;
}

.two-col {
  display: grid;
  grid-template-columns: 1.2fr 0.8fr;
  gap: 16px;
}

.note {
  background: #f4f7fb;
  border-left: 4px solid #0b2d5b;
  padding: 10px 12px;
  font-size: 11px;
}

.footer {
  font-size: 9px;
  color: #666;
  border-top: 1px solid #ddd;
  padding-top: 8px;
  margin-top: 12px;
}

.disclaimer {
  font-size: 9px;
  color: #444;
}

.chart {
  width: 100%;
  margin-top: 8px;
  border: 1px solid #e2e8f0;
  border-radius: 6px;
}

.small {
  font-size: 10px;
}

.risk-high { color: #b00020; font-weight: 700; }
.risk-medium { color: #b36b00; font-weight: 700; }
.risk-low { color: #0a6b2b; font-weight: 700; }
</style>
</head>
<body>

<div class="header-bar">
  <div>
    <div class="brand">MERIDIAN EQUITY RESEARCH</div>
    <div class="subbrand">INITIATING COVERAGE | {{ report.sector | default("SECTOR") }}</div>
  </div>
  <div class="badge">
    {{ report.rating | default("BUY") }}<br>
    Target: {{ report.target_price | default("N/A") }}
  </div>
</div>

<div class="container">
  <div class="title">{{ report.company_name | default(ticker) }}</div>
  <div class="subtitle">
    {{ report.subtitle | default("Equity Research Report") }} | Date: {{ report.date | default("April 2025") }}
  </div>

  <div class="grid">
    <div class="card"><div class="label">Rating</div><div class="value">{{ report.rating | default("BUY") }}</div></div>
    <div class="card"><div class="label">CMP</div><div class="value">{{ report.cmp | default("N/A") }}</div></div>
    <div class="card"><div class="label">Target Price</div><div class="value">{{ report.target_price | default("N/A") }}</div></div>
    <div class="card"><div class="label">Market Cap</div><div class="value">{{ report.market_cap | default("N/A") }}</div></div>
    <div class="card"><div class="label">Exchange</div><div class="value">{{ report.exchange | default("N/A") }}</div></div>
  </div>

  <div class="section">
    <h2>Investment Summary</h2>
    <p>{{ report.investment_summary | default("No summary available.") }}</p>
  </div>

  {% if chart_path %}
  <div class="section">
    <h2>Price Action Chart</h2>
    <img class="chart" src="{{ chart_path }}" alt="Price chart">
  </div>
  {% endif %}

  <div class="page-break"></div>

  <div class="section">
    <h2>1. Sector Overview</h2>
    <p>{{ report.sector_overview | default("No sector overview available.") }}</p>
    <h3>Growth Drivers</h3>
    <ul>
      {% for item in report.growth_drivers | default([]) %}
      <li>{{ item }}</li>
      {% endfor %}
    </ul>
  </div>

  <div class="section">
    <h2>2. Managed Office / Value Chain Analysis</h2>
    <table class="table">
      <thead>
        <tr>
          <th>Stage</th>
          <th>Activity</th>
          <th>Key Metric</th>
          <th>Capability</th>
        </tr>
      </thead>
      <tbody>
      {% for row in report.value_chain | default([]) %}
        <tr>
          <td>{{ row.stage }}</td>
          <td>{{ row.activity }}</td>
          <td>{{ row.key_metric }}</td>
          <td>{{ row.capability }}</td>
        </tr>
      {% endfor %}
      </tbody>
    </table>
  </div>

  <div class="section">
    <h2>3. Sector Beneficiaries</h2>
    <table class="table">
      <thead>
        <tr>
          <th>Company</th>
          <th>Model</th>
          <th>Status</th>
        </tr>
      </thead>
      <tbody>
      {% for row in report.beneficiaries | default([]) %}
        <tr>
          <td>{{ row.company }}</td>
          <td>{{ row.model }}</td>
          <td>{{ row.status }}</td>
        </tr>
      {% endfor %}
      </tbody>
    </table>
  </div>

  <div class="section">
    <h2>4. Deep-Dive: Company Overview</h2>
    <p>{{ report.company_overview | default("No company overview available.") }}</p>

    <h3>Revenue Mix</h3>
    <table class="table">
      <thead>
        <tr>
          <th>Segment</th>
          <th>Contribution</th>
          <th>Role</th>
          <th>Target</th>
        </tr>
      </thead>
      <tbody>
      {% for row in report.revenue_mix | default([]) %}
        <tr>
          <td>{{ row.segment }}</td>
          <td>{{ row.contribution }}</td>
          <td>{{ row.role }}</td>
          <td>{{ row.target }}</td>
        </tr>
      {% endfor %}
      </tbody>
    </table>

    <h3>Competitive Advantages</h3>
    <ul>
      {% for item in report.competitive_advantages | default([]) %}
      <li>{{ item }}</li>
      {% endfor %}
    </ul>

    <h3>Growth Drivers</h3>
    <ul>
      {% for item in report.company_growth_drivers | default([]) %}
      <li>{{ item }}</li>
      {% endfor %}
    </ul>
  </div>

  <div class="section">
    <h2>5. Risk Analysis</h2>
    <table class="table">
      <thead>
        <tr>
          <th>Factor</th>
          <th>Severity</th>
          <th>Description</th>
        </tr>
      </thead>
      <tbody>
      {% for row in report.risks | default([]) %}
        <tr>
          <td>{{ row.factor }}</td>
          <td>{{ row.severity }}</td>
          <td>{{ row.description }}</td>
        </tr>
      {% endfor %}
      </tbody>
    </table>
  </div>

  <div class="section">
    <h2>6. Investment Thesis & Valuation</h2>
    <div class="note">
      <div><b>Base Case:</b> {{ report.valuation.base_case | default("N/A") }}</div>
      <div><b>Bear Case:</b> {{ report.valuation.bear_case | default("N/A") }}</div>
      <div><b>Bull Case:</b> {{ report.valuation.bull_case | default("N/A") }}</div>
      <div><b>Blended Target:</b> {{ report.valuation.blended_target | default("N/A") }}</div>
      <div><b>Rating:</b> {{ report.valuation.rating | default(report.rating | default("BUY")) }}</div>
      <div><b>Upside:</b> {{ report.valuation.upside | default("N/A") }}</div>
    </div>
  </div>

  <div class="section">
    <h2>Disclaimer</h2>
    <p class="disclaimer">{{ report.disclaimer | default("This report is for informational purposes only.") }}</p>
  </div>

  <div class="footer">
    {{ report.company_name | default(ticker) }} | Generated by your local AI pipeline
  </div>
</div>

</body>
</html>
        """
    )

    return template.render(
        report=report,
        ticker=ticker,
        chart_path=chart_rel if chart_rel else None,
    )


def main():
    OUTPUT_DIR.mkdir(exist_ok=True)
    CHART_DIR.mkdir(exist_ok=True)

    ticker = input("Enter stock ticker (example: EFC.NS or NVDA): ").strip()
    raw_file = input("Raw text file [input/raw.txt]: ").strip() or "input/raw.txt"

    raw_path = BASE_DIR / raw_file
    raw_text = read_text_file(raw_path)

    chart_path = fetch_price_chart(ticker)

    report = ask_ollama_for_report(raw_text, ticker)

    # Basic safety defaults
    report.setdefault("company_name", ticker)
    report.setdefault("sector", "Equity Research")
    report.setdefault("date", "April 2025")
    report.setdefault("rating", "BUY")
    report.setdefault("cmp", "N/A")
    report.setdefault("target_price", "N/A")
    report.setdefault("market_cap", "N/A")
    report.setdefault("exchange", "N/A")
    report.setdefault("investment_summary", "")
    report.setdefault("sector_overview", "")
    report.setdefault("growth_drivers", [])
    report.setdefault("value_chain", [])
    report.setdefault("beneficiaries", [])
    report.setdefault("company_overview", "")
    report.setdefault("revenue_mix", [])
    report.setdefault("competitive_advantages", [])
    report.setdefault("company_growth_drivers", [])
    report.setdefault("risks", [])
    report.setdefault("valuation", {})
    report.setdefault("disclaimer", "This report is generated for informational purposes only.")

    html = render_html(report, ticker, chart_path)

    html_path = OUTPUT_DIR / f"{ticker}_equity_report.html"
    pdf_path = OUTPUT_DIR / f"{ticker}_equity_report.pdf"

    html_path.write_text(html, encoding="utf-8")
    HTML(string=html, base_url=str(BASE_DIR)).write_pdf(str(pdf_path))

    print(f"\nDone.\nHTML: {html_path}\nPDF:  {pdf_path}\n")


if __name__ == "__main__":
    main()
                





























import yfinance as yf
import matplotlib.pyplot as plt
import ollama
from jinja2 import Template
from weasyprint import HTML

# 1. FETCH & PLOT STOCK DATA (The "Chart Reading" part)
ticker = "543965.BO" # EFC (I) Limited
df = yf.download(ticker, period="1y")
plt.figure(figsize=(10, 4))
plt.plot(df['Close'], color='#004080', linewidth=2) # Professional Blue
plt.fill_between(df.index, df['Close'], color='#e6f2ff')
plt.title("Price Action - 12 Months", fontsize=12, fontweight='bold')
plt.savefig("stock_plot.png", bbox_inches='tight', dpi=300)

# 2. AI ANALYSIS (The "Model Summary" part)
with open("data.txt", "r") as f:
    raw_data = f.read()

prompt = f"Act as a Senior Equity Analyst. Based on this data: {raw_data}. Provide a 3-sentence Investment Verdict (BUY/HOLD) and 3 key Growth Drivers."
res = ollama.generate(model='qwen2.5:0.5b', prompt=prompt)
ai_summary = res['response']

# 3. PROFESSIONAL PDF GENERATION (Margins, Colors, Fonts)
html_template = """
<html>
<head><style>
    @page { size: A4; margin: 2cm; }
    body { font-family: 'Helvetica', 'Arial', sans-serif; color: #333; line-height: 1.5; }
    .header { border-bottom: 4px solid #004080; padding-bottom: 10px; margin-bottom: 30px; }
    .title { color: #004080; font-size: 28px; font-weight: bold; text-transform: uppercase; }
    .rating-badge { background: #004080; color: white; padding: 10px 20px; display: inline-block; font-weight: bold; margin-top: 10px; }
    .section-title { color: #004080; border-bottom: 1px solid #ddd; padding-bottom: 5px; margin-top: 30px; text-transform: uppercase; font-size: 16px; }
    .chart-img { width: 100%; margin-top: 20px; border: 1px solid #eee; }
</style></head>
<body>
    <div class="header">
        <div class="title">Equity Research Report</div>
        <div class="rating-badge">BUY | TARGET: INR 285</div>
    </div>
    
    <div class="section-title">AI Analyst Summary & Verdict</div>
    <p>{{ summary }}</p>

    <div class="section-title">Technical Performance (BSE)</div>
    <img src="stock_plot.png" class="chart-img">

    <div class="section-title">Core Business Analysis</div>
    <p style="font-size: 12px; color: #666;">{{ raw_text }}</p>
</body>
</html>
"""

# Render and Export
t = Template(html_template)
rendered_html = t.render(summary=ai_summary, raw_text=raw_data[:800])
HTML(string=rendered_html, base_url=".").write_pdf("Professional_Stock_Report.pdf")
print("✅ Report Generated: Professional_Stock_Report.pdf")























import yfinance as yf
import matplotlib.pyplot as plt
import ollama
from jinja2 import Template
from weasyprint import HTML
import os

# --- 1. CONFIGURATION & LIVE DATA ---
TICKER = "543965.BO"  # EFC (I) Limited on BSE
REPORT_TITLE = "Meridian Equity Research"
PRIMARY_COLOR = "#004080"  # Professional Blue from sample

# Fetch Live Stock Data and Generate Chart
print("Fetching live market data...")
df = yf.download(TICKER, period="1y")
plt.figure(figsize=(10, 4))
plt.plot(df['Close'], color=PRIMARY_COLOR, linewidth=2)
plt.fill_between(df.index, df['Close'], color='#e6f2ff', alpha=0.5)
plt.title(f"{TICKER} - 12 Month Performance", fontsize=12, fontweight='bold', color=PRIMARY_COLOR)
plt.grid(axis='y', linestyle='--', alpha=0.7)
plt.savefig("chart.png", bbox_inches='tight', dpi=300)

# --- 2. AI ANALYSIS ---
print("Running AI analysis with Ollama...")
with open("data.txt", "r") as f:
    research_content = f.read()

prompt = f"Act as a Senior Equity Research Analyst. Based on this data: {research_content}. Summarize into: 1. Investment Verdict, 2. Three Key Growth Drivers, and 3. Risk Warning."
res = ollama.generate(model='qwen2.5:0.5b', prompt=prompt)
ai_summary = res['response']

# --- 3. PROFESSIONAL PDF GENERATION ---
html_template = """
<html>
<head><style>
    @page { size: A4; margin: 2cm; }
    body { font-family: 'Helvetica', sans-serif; color: #333; line-height: 1.6; }
    .header { border-bottom: 4px solid #004080; padding-bottom: 10px; margin-bottom: 20px; }
    .title { color: #004080; font-size: 26px; font-weight: bold; text-transform: uppercase; margin: 0; }
    .buy-box { background: #e6f2ff; border-left: 6px solid #004080; padding: 15px; margin: 20px 0; }
    .section-title { color: #004080; border-bottom: 1px solid #ddd; padding-bottom: 5px; margin-top: 30px; text-transform: uppercase; font-size: 16px; font-weight: bold; }
    .chart-container { text-align: center; margin: 20px 0; }
    .footer { position: fixed; bottom: 0; width: 100%; text-align: center; font-size: 10px; color: #999; border-top: 1px solid #eee; padding-top: 5px; }
</style></head>
<body>
    <div class="header">
        <p style="margin:0; font-weight:bold; color:#666;">INITIATING COVERAGE | INDIA REAL ESTATE</p>
        <h1 class="title">{{ title }}</h1>
    </div>

    <div class="buy-box">
        <h2 style="margin:0; color:#004080;">BUY | Target: INR 285</h2>
        <p style="margin:5px 0 0 0;"><strong>Company:</strong> EFC (I) Limited (BSE: 543965)</p>
    </div>

    <div class="section-title">AI Analyst Verdict & Summary</div>
    <div style="white-space: pre-line;">{{ summary }}</div>

    <div class="section-title">Technical Performance (BSE Chart)</div>
    <div class="chart-container">
        <img src="chart.png" style="width: 100%;">
    </div>

    <div class="section-title">Investment Thesis Highlights</div>
    <p style="font-size: 12px; color: #444;">{{ raw_data[:1200] }}...</p>

    <div class="footer">For institutional use only. Prepared via Meridian AI Engine.</div>
</body>
</html>
"""

print("Finalizing professional PDF...")
t = Template(html_template)
rendered_html = t.render(title=REPORT_TITLE, summary=ai_summary, raw_data=research_content)
HTML(string=rendered_html, base_url=".").write_pdf("EFC_Professional_Report.pdf")

print("✅ DONE! Open 'EFC_Professional_Report.pdf' to see the result.")

