import streamlit as st
import yfinance as yf
import pandas as pd
import ta
import openai
import datetime

# --- SETTINGS ---
openai.api_key = st.secrets["OPENAI_API_KEY"]
TICKERS = ["AAPL", "TSLA", "NVDA", "MSFT", "AMD"]

st.set_page_config(page_title="AI Day Trader", layout="wide")

st.title("📈 AI-Powered Day Trading Signal App")

def get_data(ticker):
    data = yf.download(ticker, period="1d", interval="5m", progress=False)
    data.dropna(inplace=True)
    return data

def generate_signals(df):
    df['rsi'] = ta.momentum.RSIIndicator(df['Close']).rsi()
    df['ema20'] = ta.trend.EMAIndicator(df['Close'], window=20).ema_indicator()
    df['ema50'] = ta.trend.EMAIndicator(df['Close'], window=50).ema_indicator()

    last = df.iloc[-1]
    if last['rsi'] < 30 and last['ema20'] > last['ema50']:
        return "BUY", "RSI < 30 & EMA20 > EMA50 (Bullish)"
    elif last['rsi'] > 70 and last['ema20'] < last['ema50']:
        return "SELL", "RSI > 70 & EMA20 < EMA50 (Bearish)"
    else:
        return "HOLD", "Neutral indicators"

def explain_trade(ticker, signal, reason):
    prompt = f"""Generate a short explanation for a trade on {ticker}:
Signal: {signal}
Reason: {reason}
Make it sound like advice from a financial analyst."""
    
    try:
        response = openai.ChatCompletion.create(
            model="gpt-4",
            messages=[{"role": "user", "content": prompt}]
        )
        return response.choices[0].message["content"].strip()
    except Exception as e:
        return f"Error generating explanation: {e}"

# --- UI ---
with st.sidebar:
    st.header("Settings")
    selected_tickers = st.multiselect("Select tickers:", TICKERS, default=TICKERS)
    refresh_button = st.button("🔄 Refresh Signals")

if refresh_button:
    st.info("Fetching data and generating signals...")

    for ticker in selected_tickers:
        df = get_data(ticker)
        signal, reason = generate_signals(df)
        explanation = explain_trade(ticker, signal, reason)

        st.subheader(f"{ticker} - Signal: {signal}")
        st.write(f"📌 Reason: {reason}")
        st.write(f"💬 AI Explanation: {explanation}")
        st.line_chart(df["Close"])

        st.markdown("---")
