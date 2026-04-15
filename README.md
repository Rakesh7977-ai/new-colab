pside:.1f}%</p>
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
            





















import streamlit as st
import yfinance as yf
import pandas as pd
import matplotlib.pyplot as plt
import ollama
from weasyprint import HTML
import os

# --- PAGE CONFIG ---
st.set_page_config(page_title="EFC Pro Screener", layout="wide")

st.markdown("""
<style>
    .stApp { background-color: #f8f9fa; }
    .stButton>button { width: 100%; border-radius: 5px; height: 3em; background-color: #004080; color: white; font-weight: bold; }
    .metric-card { background-color: white; padding: 15px; border-radius: 10px; border-left: 5px solid #004080; box-shadow: 2px 2px 5px rgba(0,0,0,0.1); }
</style>
""", unsafe_allow_html=True)

st.title("🏛️ EFC (I) Limited: Institutional Research Screener")
ticker = "543965.BO"

# --- DATA ENGINE WITH ERROR HANDLING ---
@st.cache_data
def fetch_live_data(t):
    try:
        data = yf.download(t, period="1y", progress=False)
        return data
    except Exception:
        return pd.DataFrame()

stock_df = fetch_live_data(ticker)

# Check if data is empty to prevent the Crash
if stock_df.empty:
    st.warning("⚠️ Market data currently unavailable. Using last known CMP for UI.")
    current_price = 185.0  # Fallback price from your report
else:
    current_price = stock_df['Close'].iloc[-1]

# --- SIDEBAR ---
st.sidebar.header("Control Panel")
target_price = st.sidebar.number_input("Target Price (INR)", value=285)
run_audit = st.sidebar.button("🔍 Run AI Research Audit")

upside = ((target_price / current_price) - 1) * 100

# --- UI TABS ---
tab1, tab2, tab3 = st.tabs(["📊 Screener Overview", "📈 Technical View", "📥 Download Report"])

with tab1:
    c1, c2, c3 = st.columns(3)
    with c1:
        st.markdown(f"<div class='metric-card'><b>Current Price</b><br><h2>₹{current_price:.2f}</h2></div>", unsafe_allow_html=True)
    with c2:
        st.markdown(f"<div class='metric-card'><b>Target Price</b><br><h2>₹{target_price}</h2></div>", unsafe_allow_html=True)
    with c3:
        st.markdown(f"<div class='metric-card'><b>Potential Upside</b><br><h2 style='color:green;'>+{upside:.1f}%</h2></div>", unsafe_allow_html=True)

    st.divider()
    st.subheader("Deep-Dive AI Audit (Reading from data.txt)")
    
    if run_audit:
        # Check for any of your data files
        data_file = "data.txt" if os.path.exists("data.txt") else "equity_report_data.txt"
        
        if os.path.exists(data_file):
            with open(data_file, "r") as f:
                raw_data = f.read()
            with st.spinner("Qwen AI is auditing research files..."):
                prompt = f"Analyze this EFC research data: {raw_data}. Provide a 'Verdict', 'Structural Moat', and 'Key Risks'."
                response = ollama.generate(model='qwen2.5:0.5b', prompt=prompt)
                st.session_state['ai_res'] = response['response']
                st.info(st.session_state['ai_res'])
        else:
            st.error(f"Error: No data file found. Please ensure {data_file} exists in your folder.")

with tab2:
    if not stock_df.empty:
        st.subheader("BSE Technical Price Action")
        fig, ax = plt.subplots(figsize=(12, 4))
        ax.plot(stock_df['Close'], color='#004080', linewidth=2)
        ax.fill_between(stock_df.index, stock_df['Close'], color='#e6f2ff', alpha=0.5)
        st.pyplot(fig)
        st.dataframe(stock_df.tail(10), use_container_width=True)
    else:
        st.error("Cannot render charts: No market data found.")

with tab3:
    st.subheader("Generate Institutional PDF")
    if st.button("🚀 Build PDF Report"):
        with st.spinner("Rendering professional PDF..."):
            # Ensure we have a chart image even if data is missing
            if not stock_df.empty:
                plt.savefig("temp_chart.png", bbox_inches='tight')
            
            summary = st.session_state.get('ai_res', "Audit not run yet.")
            
            html_content = f"""
            <html><body style="font-family: sans-serif; margin: 40px;">
                <h1 style="color: #004080;">EFC (I) LIMITED RESEARCH</h1>
                <div style="background: #e6f2ff; padding: 15px; border-left: 5px solid #004080;">
                    <h2>BUY | Target: INR {target_price}</h2>
                </div>
                <h3>AI Investment Summary</h3>
                <p>{summary}</p>
            </body></html>
            """
            HTML(string=html_content).write_pdf("EFC_Final_Report.pdf")
            st.success("✅ PDF Built! You can n"ow download it below.")

    if os.path.exists("EFC_Final_Report.pdf"):
        with open("EFC_Final_Report.pdf", "rb") as f:
            st.download_button("📥 Download PDF", f, "EFC_Equity_Research.pdf", "application/pdf)
            























import streamlit as st
import yfinance as yf
import pandas as pd
import matplotlib.pyplot as plt
import ollama
from weasyprint import HTML
import os
import base64

# --- PAGE SETUP ---
st.set_page_config(page_title="EFC Institutional Screener", layout="wide")

# CSS for a minimalist, professional look
st.markdown("""
<style>
    .stApp { background-color: white; }
    .stButton>button { background-color: #004080; color: white; border-radius: 0px; height: 3em; font-weight: bold; }
    .report-card { border: 1px solid #e0e0e0; padding: 20px; border-radius: 4px; margin-bottom: 20px; }
    .metric-value { font-size: 24px; font-weight: bold; color: #004080; }
</style>
""", unsafe_allow_html=True)

# --- CONFIG & LIVE DATA ---
TICKER = "543965.BO"
DATA_FILE = "data.txt" # Ensure this file has your EFC analysis

@st.cache_data(ttl=3600)
def get_clean_data(t):
    try:
        df = yf.download(t, period="1y", progress=False)
        if df.empty: return None
        return df
    except: return None

data = get_clean_data(TICKER)
cmp = data['Close'].iloc[-1] if data is not None else 199.04  # Latest 2026 Price

# --- MAIN UI ---
st.title("🏛️ EFC (I) Limited: Equity Audit Dashboard")

tab1, tab2, tab3 = st.tabs(["🔍 Audit Screener", "📉 Technicals", "📄 PDF Generator"])

with tab1:
    c1, c2, c3 = st.columns(3)
    c1.markdown(f"<div class='report-card'>CMP<br><span class='metric-value'>₹{cmp:.2f}</span></div>", unsafe_allow_html=True)
    c2.markdown(f"<div class='report-card'>Rating<br><span class='metric-value' style='color:green;'>BUY</span></div>", unsafe_allow_html=True)
    c3.markdown(f"<div class='report-card'>Upside<br><span class='metric-value'>+43.2%</span></div>", unsafe_allow_html=True)

    st.subheader("AI Analysis (Reading from your files)")
    if st.button("🚀 Run Deep Audit"):
        if os.path.exists(DATA_FILE):
            with open(DATA_FILE, "r") as f:
                content = f.read()
            with st.spinner("Analyzing company reports..."):
                # Using Qwen to summarize the EFC structural moat & April 2026 Rights Issue
                prompt = f"Summarize the EFC structural moat and the impact of the 160cr rights issue from this: {content}"
                res = ollama.generate(model='qwen2.5:0.5b', prompt=prompt)
                st.session_state['summary'] = res['response']
                st.write(st.session_state['summary'])
        else:
            st.error(f"Please place your analysis in {DATA_FILE}")

with tab2:
    if data is not None:
        fig, ax = plt.subplots(figsize=(10, 3))
        ax.plot(data['Close'], color='#004080', linewidth=1.5)
        ax.fill_between(data.index, data['Close'], color='#e6f2ff', alpha=0.5)
        ax.axis('off') # Minimalist look
        st.pyplot(fig)
    else:
        st.write("Market data currently offline.")

with tab3:
    st.subheader("Generate Meridian-Style Report")
    if st.button("🛠️ Build PDF"):
        if 'summary' not in st.session_state:
            st.warning("Please run the 'Audit' in Tab 1 first.")
        else:
            # Save chart for the PDF
            plt.savefig("report_chart.png", bbox_inches='tight', transparent=True)
            
            # HTML Template based on your uploaded Meridian Report
            html_template = f"""
            <style>
                body {{ font-family: 'Helvetica'; margin: 50px; }}
                .header {{ border-bottom: 5px solid #004080; padding-bottom: 10px; }}
                .rating {{ background: #e6f2ff; padding: 20px; border-left: 10px solid #004080; }}
            </style>
            <div class="header">
                <h1 style="color:#004080; margin:0;">MERIDIAN EQUITY RESEARCH</h1>
                <p>INITIATING COVERAGE | APRIL 2026</p>
            </div>
            <div class="rating">
                <h2>BUY | Target: INR 285</h2>
                <p><b>Company:</b> EFC (I) Limited (543965.BO)</p>
            </div>
            <h3>Investment Thesis</h3>
            <p>{st.session_state['summary']}</p>
            <h3>Price Action</h3>
            <img src="report_chart.png" style="width:100%;">
            """
            HTML(string=html_template, base_url=".").write_pdf("EFC_Research.pdf")
            st.success("PDF Ready!")

    if os.path.exists("EFC_Research.pdf"):
        with open("EFC_Research.pdf", "rb") as f:
            st.download_button("📥 Download Final PDF", f, "EFC_Equity_Report.pdf")
            
