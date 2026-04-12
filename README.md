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
    
