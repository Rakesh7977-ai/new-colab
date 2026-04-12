






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
    
