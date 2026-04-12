import streamlit as st
import ollama
import yfinance as yf
import plotly.graph_objects as go
import glob

# --- 1. PAGE SETUP ---
st.set_page_config(layout="wide", page_title="AI Stock Terminal")

# --- 2. FILE LOADER ---
def get_local_context():
    context = ""
    # Looks for all .txt files seen in your explorer
    for file_name in glob.glob("*.txt"):
        try:
            with open(file_name, 'r', encoding='utf-8') as f:
                context += f"\n-- {file_name} --\n{f.read()}\n"
        except Exception:
            continue
    return context

# --- 3. SIDEBAR ---
with st.sidebar:
    st.header("📈 Controls")
    ticker = st.text_input("Enter Stock Ticker", value="NVDA").upper()
    time_period = st.selectbox("Period", ["1mo", "3mo", "6mo", "1y"])
    analyze_btn = st.button("Run AI Analysis")
    st.divider()
    st.write("Files found:", glob.glob("*.txt"))

# --- 4. DATA FETCHING ---
st.title(f"📊 Analysis: {ticker}")
col1, col2 = st.columns([2, 1])

# Fetch data and clean it immediately to avoid the 'Series' error
data = yf.download(ticker, period=time_period, interval="1d")

with col1:
    if not data.empty:
        fig = go.Figure(data=[go.Candlestick(
            x=data.index,
            open=data['Open'],
            high=data['High'],
            low=data['Low'],
            close=data['Close']
        )])
        fig.update_layout(template="plotly_dark", title=f"{ticker} Price Action")
        st.plotly_chart(fig, use_container_width=True)
    else:
        st.error("Ticker not found or no internet connection.")

with col2:
    st.subheader("💡 AI Insights")
    if analyze_btn and not data.empty:
        with st.spinner("Reading files and analyzing..."):
            try:
                # FIX: We grab the last price and convert it to a plain number (.item())
                # This prevents the "unsupported format string" error
                last_price_raw = data['Close'].iloc[-1]
                current_price = float(last_price_raw.item()) if hasattr(last_price_raw, 'item') else float(last_price_raw)
                
                local_files = get_local_context()
                
                # The Prompt uses your specific answer_style.txt rules
                prompt = f"""
                You are a senior stock analyst. 
                LIVE DATA: {ticker} is currently at ${current_price:.2f}.
                LOCAL RESEARCH: {local_files}
                
                Using the 'Style rules' from answer_style.txt:
                1. Summarize the technical and fundamental view.
                2. Give a final conclusion (BUY, HOLD, WAIT, or AVOID) based strictly on my rules.
                """
                
                # Using the lightweight model for speed
                response = ollama.generate(model='qwen2.5:0.5b', prompt=prompt)
                st.info(response['response'])
                
            except Exception as e:
                st.error(f"Analysis Error: {e}")
    elif analyze_btn:
        st.warning("Please enter a valid ticker first.")

# --- 5. BOTTOM METRICS ---
if not data.empty:
    st.divider()
    m1, m2, m3 = st.columns(3)
    latest = float(data['Close'].iloc[-1].item())
    high = float(data['High'].max().item())
    low = float(data['Low'].min().item())
    
    m1.metric("Current Price", f"${latest:.2f}")
    m2.metric("Period High", f"${high:.2f}")
    m3.metric("Period Low", f"${low:.2f}")
    






















import streamlit as st
import ollama
import yfinance as yf
import plotly.graph_objects as go
import glob
import pandas as pd

# --- CONFIG ---
st.set_page_config(layout="wide", page_title="Screener AI Pro")

# --- DATA ENGINE ---
def get_clean_context(symbol):
    context = ""
    for f_name in glob.glob("*.txt"):
        try:
            with open(f_name, 'r', encoding='utf-8') as f:
                content = f.read()
                # Only include file if it's a rule file or mentions the ticker
                if symbol.lower() in content.lower() or "rules" in f_name or "style" in f_name:
                    context += f"\n-- SOURCE: {f_name} --\n{content}\n"
        except: continue
    return context

# --- UI SIDEBAR ---
with st.sidebar:
    st.header("🔍 Stock Screener")
    ticker = st.text_input("Ticker Symbol", value="NVDA").upper()
    period = st.selectbox("Timeline", ["1mo", "3mo", "6mo", "1y", "5y"], index=3)
    run_audit = st.button("🚀 Run AI Audit", use_container_width=True)
    st.divider()
    st.caption("Connected to: qwen2.5:0.5b")

# --- DATA FETCH ---
stock = yf.Ticker(ticker)
hist = stock.history(period=period)
info = stock.info

# --- HEADER METRICS ---
st.title(f"{info.get('longName', ticker)} ({ticker})")
m1, m2, m3, m4, m5 = st.columns(5)
m1.metric("Current Price", f"${hist['Close'].iloc[-1]:.2f}" if not hist.empty else "N/A")
m2.metric("Market Cap", f"{info.get('marketCap', 0)/1e9:.2f}B" if info.get('marketCap') else "N/A")
m3.metric("Stock P/E", f"{info.get('trailingPE', 0):.2f}" if info.get('trailingPE') else "N/A")
m4.metric("ROCE", f"{info.get('returnOnCapitalEmployed', 0)*100:.1f}%" if info.get('returnOnCapitalEmployed') else "N/A")
m5.metric("Debt to Equity", f"{info.get('debtToEquity', 0)/100:.2f}" if info.get('debtToEquity') else "N/A")

st.divider()

# --- MAIN LAYOUT ---
tab1, tab2 = st.tabs(["📊 Technical Chart", "📋 Financial Health"])

with tab1:
    col_chart, col_ai = st.columns([2, 1])
    
    with col_chart:
        if not hist.empty:
            fig = go.Figure(data=[go.Candlestick(
                x=hist.index, open=hist['Open'], high=hist['High'],
                low=hist['Low'], close=hist['Close']
            )])
            fig.update_layout(template="plotly_dark", height=500, xaxis_rangeslider_visible=False, margin=dict(l=0,r=0,t=0,b=0))
            st.plotly_chart(fig, use_container_width=True)
        else:
            st.error("No data found. Check ticker or internet connection.")

    with col_ai:
        st.subheader("🤖 AI Analysis")
        if run_audit:
            with st.spinner("Analyzing..."):
                local_txt = get_clean_context(ticker)
                prompt = f"""
                Analyze {ticker}. Current Price: ${hist['Close'].iloc[-1]:.2f}.
                
                METRICS:
                - P/E: {info.get('trailingPE')}
                - Market Cap: {info.get('marketCap')}
                - Business: {info.get('longBusinessSummary', 'N/A')[:500]}...
                
                LOCAL CONTEXT:
                {local_txt}
                
                TASK:
                1. Provide a Summary. 
                2. Technical & Fundamental View.
                3. Final Verdict (BUY/HOLD/WAIT/AVOID) based on 'answer_style.txt' rules.
                """
                res = ollama.generate(model='qwen2.5:0.5b', prompt=prompt)
                st.info(res['response'])
        else:
            st.write("Click 'Run AI Audit' to see insights.")

with tab2:
    st.subheader("Key Statistics")
    # Clean up info dictionary for display
    display_keys = ['profitMargins', 'forwardEps', 'dividendYield', 'totalRevenue', 'grossMargins', 'operatingMargins']
    stats_data = {k: info.get(k, 'N/A') for k in display_keys}
    st.table(pd.DataFrame([stats_data]))
    
