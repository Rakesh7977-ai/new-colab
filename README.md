






import streamlit as st
import ollama
import glob
import os

# --- PAGE CONFIG ---
st.set_page_config(page_title="Local AI Bot", page_icon="🤖")
st.title("🤖 My Private Assistant")

# --- 1. THE FILE LOADER (Your Logic) ---
def load_context():
    context = ""
    files = glob.glob("*.txt")
    for file_name in files:
        with open(file_name, 'r', encoding='utf-8') as f:
            context += f"\n--- Source: {file_name} ---\n{f.read()}\n"
    return context, files

# Load files once and show them in the sidebar
context_data, file_list = load_context()

with st.sidebar:
    st.header("Files Loaded")
    for f in file_list:
        st.write(f"✅ {f}")
    if st.button("Clear Chat"):
        st.session_state.messages = []

# --- 2. CHAT HISTORY ---
if "messages" not in st.session_state:
    st.session_state.messages = []

# Display previous messages
for message in st.session_state.messages:
    with st.chat_message(message["role"]):
        st.markdown(message["content"])

# --- 3. THE CHAT LOGIC ---
if prompt := st.chat_input("Ask me about your files..."):
    # Add user message to chat
    st.session_state.messages.append({"role": "user", "content": prompt})
    with st.chat_message("user"):
        st.markdown(prompt)

    # Generate response
    with st.chat_message("assistant"):
        response_placeholder = st.empty()
        full_response = ""
        
        # We inject the file data as a "System Prompt" so the model knows it
        system_prompt = f"You are a helpful assistant. Use this context to answer: {context_data}"
        
        # Call Ollama (Using your qwen2.5:3b model)
        response = ollama.chat(
            model='qwen2.5:3b',
            messages=[
                {'role': 'system', 'content': system_prompt},
                *st.session_state.messages
            ],
            stream=True,
        )

        for chunk in response:
            content = chunk['message']['content']
            full_response += content
            response_placeholder.markdown(full_response + "▌")
        
        response_placeholder.markdown(full_response)
    
    st.session_state.messages.append({"role": "assistant", "content": full_response})
    













import streamlit as st
import ollama
import yfinance as yf
import plotly.graph_objects as go
import glob

st.set_page_config(layout="wide", page_title="AI Stock Terminal")

# --- 1. DATA LOADERS ---
def get_local_context():
    context = ""
    for file_name in glob.glob("*.txt"):
        with open(file_name, 'r', encoding='utf-8') as f:
            context += f"\n-- {file_name} --\n{f.read()}\n"
    return context

# --- 2. SIDEBAR & SETTINGS ---
with st.sidebar:
    st.header("📈 Controls")
    ticker = st.text_input("Enter Stock Ticker", value="AAPL").upper()
    time_period = st.selectbox("Period", ["1mo", "3mo", "6mo", "1y"])
    analyze_btn = st.button("Run AI Analysis")
    st.divider()
    st.write("Reading files:", glob.glob("*.txt"))

# --- 3. THE DASHBOARD LAYOUT ---
st.title(f"📊 Analysis: {ticker}")

col1, col2 = st.columns([2, 1])

with col1:
    # Fetch Real Stock Data
    data = yf.download(ticker, period=time_period)
    
    if not data.empty:
        # Create a Candlestick Chart
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
        st.error("Ticker not found.")

with col2:
    st.subheader("💡 AI Insights")
    if analyze_btn:
        with st.spinner("Analyzing data and files..."):
            local_files = get_local_context()
            current_price = data['Close'].iloc[-1]
            
            # The Prompt: Combine Live Data + Your Local Rules/Files
            prompt = f"""
            You are a senior stock analyst. 
            LIVE DATA: {ticker} is currently at ${current_price:.2f}.
            LOCAL RESEARCH: {local_files}
            
            Based on the 'answer_style.txt' rules and the data provided, 
            give a brief Technical and Fundamental summary for {ticker}.
            """
            
            response = ollama.generate(model='qwen2.5:0.5b', prompt=prompt)
            st.info(response['response'])
    else:
        st.write("Click 'Run AI Analysis' to combine your local files with market data.")

# --- 4. QUICK STATS ---
if not data.empty:
    m1, m2, m3 = st.columns(3)
    m1.metric("Current Price", f"${data['Close'].iloc[-1]:.2f}")
    m2.metric("High (Period)", f"${data['High'].max():.2f}")
    m3.metric("Low (Period)", f"${data['Low'].min():.2f}")











cd ~/path/to/your/training_models
source venv/bin/activate
streamlit run dashboard.py
