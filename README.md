  }x_inches='tight', dpi=300)

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
    @page


























import streamlit as st
import yfinance as yf
import matplotlib.pyplot as plt
import ollama
from weasyprint import HTML
from jinja2 import Template

# --- PAGE CONFIG ---
st.set_page_config(page_title="Equity Research Dashboard", layout="wide")
st.title("📈 Professional Equity Research UI")

# --- SIDEBAR INPUTS ---
st.sidebar.header("Report Settings")
ticker = st.sidebar.text_input("Enter Ticker (BSE/NSE)", value="543965.BO")
target_price = st.sidebar.number_input("Target Price (INR)", value=285)

if st.sidebar.button("Generate Full Analysis"):
    with st.spinner("Fetching data and running AI analysis..."):
        # 1. Fetch Stock Data & Chart
        df = yf.download(ticker, period="1y")
        fig, ax = plt.subplots(figsize=(10, 4))
        ax.plot(df['Close'], color='#004080')
        ax.set_title(f"{ticker} Performance")
        plt.savefig("ui_chart.png")
        
        # 2. Display Chart in UI
        st.subheader(f"Market Analysis: {ticker}")
        st.pyplot(fig)

        # 3. Run AI Analysis
        with open("data.txt", "r") as f:
            raw_data = f.read()
        
        res = ollama.generate(model='qwen2.5:0.5b', prompt=f"Analyze this: {raw_data}")
        ai_summary = res['response']
        
        # 4. Display AI Summary in UI
        st.subheader("AI Investment Summary")
        st.info(ai_summary)

        # 5. PDF Download Button
        st.success("Analysis Complete! You can now download the PDF.")
        # (PDF generation logic would go here, same as your auto_report.py)




























        import streamlit as st
import yfinance as yf
import pandas as pd
import matplotlib.pyplot as plt
import ollama
from weasyprint import HTML
from jinja2 import Template
import os

# --- PAGE CONFIG ---
st.set_page_config(page_title="EFC Pro Screener", layout="wide")

# Custom CSS for Professional UI
st.markdown("""
<style>
    .stApp { background-color: #f8f9fa; }
    .stButton>button { width: 100%; border-radius: 5px; height: 3em; background-color: #004080; color: white; font-weight: bold; }
    .metric-card { background-color: white; padding: 15px; border-radius: 10px; border-left: 5px solid #004080; box-shadow: 2px 2px 5px rgba(0,0,0,0.1); }
</style>
""", unsafe_allow_html=True)

# --- APP HEADER ---
st.title("🏛️ EFC (I) Limited: Institutional Research Screener")
ticker = "543965.BO"

# --- SIDEBAR CONTROLS ---
st.sidebar.header("Control Panel")
target_price = st.sidebar.number_input("Target Price (INR)", value=285)
run_audit = st.sidebar.button("🔍 Run AI Research Audit")

# --- DATA ENGINE ---
@st.cache_data
def fetch_live_data(t):
    data = yf.download(t, period="1y")
    return data

stock_df = fetch_live_data(ticker)
current_price = stock_df['Close'].iloc[-1]
upside = ((target_price / current_price) - 1) * 100

# --- UI TABS ---
tab1, tab2, tab3 = st.tabs(["📊 Screener Overview", "📈 Technical View", "📥 Download Report"])

with tab1:
    # Key Metrics Row
    c1, c2, c3 = st.columns(3)
    with c1:
        st.markdown(f"<div class='metric-card'><b>Current Price</b><br><h2>₹{current_price:.2f}</h2></div>", unsafe_allow_html=True)
    with c2:
        st.markdown(f"<div class='metric-card'><b>Target Price</b><br><h2>₹{target_price}</h2></div>", unsafe_allow_html=True)
    with c3:
        st.markdown(f"<div class='metric-card'><b>Potential Upside</b><br><h2 style='color:green;'>+{upside:.1f}%</h2></div>", unsafe_allow_html=True)

    st.divider()
    
    # AI Analysis Section
    st.subheader("Deep-Dive AI Audit (Reading from data.txt)")
    if run_audit:
        if os.path.exists("data.txt"):
            with open("data.txt", "r") as f:
                raw_data = f.read()
            
            with st.spinner("Qwen AI is auditing research files..."):
                prompt = f"Analyze this EFC research data: {raw_data}. Provide a 'Verdict', 'Structural Moat', and 'Key Risks'."
                response = ollama.generate(model='qwen2.5:0.5b', prompt=prompt)
                st.info(response['response'])
                # Store in session state for PDF generation
                st.session_state['ai_res'] = response['response']
        else:
            st.error("Error: 'data.txt' not found in your folder. Please create it.")

with tab2:
    st.subheader("BSE Technical Price Action")
    fig, ax = plt.subplots(figsize=(12, 4))
    ax.plot(stock_df['Close'], color='#004080', linewidth=2)
    ax.fill_between(stock_df.index, stock_df['Close'], color='#e6f2ff', alpha=0.5)
    ax.set_ylabel("Price (INR)")
    st.pyplot(fig)
    
    st.subheader("Live Market Screener Data")
    st.dataframe(stock_df.tail(10), use_container_width=True)

with tab3:
    st.subheader("Generate Institutional PDF")
    st.write("This function converts your screen data into the Meridian-style PDF report.")
    
    if st.button("🚀 Build PDF Report"):
        with st.spinner("Rendering professional PDF..."):
            # 1. Save chart for PDF
            plt.savefig("temp_chart.png", bbox_inches='tight')
            
            # 2. PDF HTML Template
            html_content = f"""
            <html><body style="font-family: Helvetica; margin: 40px; color: #333;">
                <div style="border-bottom: 4px solid #004080; padding-bottom: 10px;">
                    <h1 style="color: #004080; margin:0;">EFC (I) LIMITED RESEARCH</h1>
                    <p style="margin:0;">Institutional Grade Equity Report</p>
                </div>
                <div style="background: #e6f2ff; padding: 20px; border-left: 10px solid #004080; margin: 20px 0;">
                    <h2 style="margin:0;">BUY | Target: INR {target_price}</h2>
                    <p>Potential Upside: {upside:.1f}%</p>
                </div>
                <h3>AI Investment Summary</h3>
                <p>{st.session_state.get('ai_res', 'Audit not run yet.')}</p>
                <h3>Price Action</h3>
                <img src="temp_chart.png" style="width: 100%;">
            </body></html>
            """
            HTML(string=html_content, base_url=".").write_pdf("EFC_Final_Report.pdf")
            st.success("✅ PDF Built Successfully!")

    # 3. Functional Download Button
    if os.path.exists("EFC_Final_Report.pdf"):
        with open("EFC_Final_Report.pdf", "rb") as f:
            st.download_button(
                label="📥 Download Professional PDF",
                data=f,
                file_name="EFC_Equity_Research.pdf",
                mime="application/pdf"
            )
            

