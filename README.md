<!DOCTYPE html>
<html lang="en" dir="auto">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>AI Chatbot</title>
<style>
  :root {
    --bg: #1a1d24;
    --panel: #22262f;
    --accent: #e8a33d;
    --accent-dim: #5a4326;
    --text: #ecedf0;
    --text-dim: #9a9fab;
    --bubble-user: #2d3340;
    --bubble-bot: #2a2f38;
    --radius: 16px;
  }

  * { box-sizing: border-box; }

  body {
    margin: 0;
    font-family: -apple-system, "Segoe UI", "Noto Nastaliq Urdu", Roboto, sans-serif;
    background: var(--bg);
    color: var(--text);
    height: 100vh;
    display: flex;
    align-items: center;
    justify-content: center;
  }

  .chat-shell {
    width: 100%;
    max-width: 480px;
    height: 100vh;
    max-height: 760px;
    background: var(--panel);
    border-radius: var(--radius);
    display: flex;
    flex-direction: column;
    overflow: hidden;
    box-shadow: 0 20px 60px rgba(0,0,0,0.4);
  }

  .chat-header {
    padding: 18px 20px;
    border-bottom: 1px solid rgba(255,255,255,0.06);
    display: flex;
    align-items: center;
    gap: 10px;
  }

  .dot {
    width: 9px;
    height: 9px;
    border-radius: 50%;
    background: var(--accent);
    box-shadow: 0 0 8px var(--accent);
  }

  .chat-header h1 {
    font-size: 15px;
    font-weight: 600;
    margin: 0;
    letter-spacing: 0.2px;
  }

  .chat-header span {
    font-size: 12px;
    color: var(--text-dim);
    margin-right: auto;
  }

  .messages {
    flex: 1;
    overflow-y: auto;
    padding: 20px;
    display: flex;
    flex-direction: column;
    gap: 14px;
  }

  .messages::-webkit-scrollbar { width: 6px; }
  .messages::-webkit-scrollbar-thumb { background: rgba(255,255,255,0.08); border-radius: 4px; }

  .msg {
    max-width: 82%;
    padding: 11px 15px;
    border-radius: 14px;
    font-size: 14.5px;
    line-height: 1.5;
    white-space: pre-wrap;
    word-wrap: break-word;
    animation: rise 0.25s ease-out;
  }

  @keyframes rise {
    from { opacity: 0; transform: translateY(6px); }
    to { opacity: 1; transform: translateY(0); }
  }

  .msg.user {
    background: var(--bubble-user);
    align-self: flex-end;
    border-bottom-right-radius: 4px;
  }

  .msg.bot {
    background: var(--bubble-bot);
    align-self: flex-start;
    border-bottom-left-radius: 4px;
    border: 1px solid rgba(232,163,61,0.1);
  }

  .msg.bot.thinking {
    color: var(--text-dim);
    font-style: italic;
  }

  .typing-dots {
    display: inline-flex;
    gap: 4px;
    align-items: center;
  }

  .typing-dots span {
    width: 6px;
    height: 6px;
    border-radius: 50%;
    background: var(--accent);
    opacity: 0.4;
    animation: pulse 1.2s infinite ease-in-out;
  }
  .typing-dots span:nth-child(2) { animation-delay: 0.2s; }
  .typing-dots span:nth-child(3) { animation-delay: 0.4s; }

  @keyframes pulse {
    0%, 60%, 100% { opacity: 0.3; transform: scale(0.85); }
    30% { opacity: 1; transform: scale(1); }
  }

  .input-row {
    padding: 14px 16px;
    border-top: 1px solid rgba(255,255,255,0.06);
    display: flex;
    gap: 10px;
    align-items: flex-end;
  }

  textarea#userInput {
    flex: 1;
    resize: none;
    background: var(--bg);
    color: var(--text);
    border: 1px solid rgba(255,255,255,0.08);
    border-radius: 12px;
    padding: 11px 14px;
    font-size: 14.5px;
    font-family: inherit;
    max-height: 100px;
    outline: none;
    transition: border-color 0.15s;
  }

  textarea#userInput:focus {
    border-color: var(--accent-dim);
  }

  button#sendBtn {
    background: var(--accent);
    color: #1a1d24;
    border: none;
    border-radius: 12px;
    width: 42px;
    height: 42px;
    flex-shrink: 0;
    cursor: pointer;
    display: flex;
    align-items: center;
    justify-content: center;
    transition: transform 0.15s, opacity 0.15s;
  }

  button#sendBtn:hover { transform: scale(1.05); }
  button#sendBtn:disabled { opacity: 0.4; cursor: not-allowed; transform: none; }

  .empty-state {
    flex: 1;
    display: flex;
    align-items: center;
    justify-content: center;
    color: var(--text-dim);
    font-size: 13.5px;
    text-align: center;
    padding: 30px;
  }
</style>
</head>
<body>

<div class="chat-shell">
  <div class="chat-header">
    <div class="dot"></div>
    <h1>AI Assistant</h1>
    <span>Online</span>
  </div>

  <div class="messages" id="messages">
    <div class="empty-state" id="emptyState">Ask me anything — I'm here to help.</div>
  </div>

  <div class="input-row">
    <textarea id="userInput" placeholder="Type your message here..." rows="1"></textarea>
    <button id="sendBtn" aria-label="Send">
      <svg width="18" height="18" viewBox="0 0 24 24" fill="none">
        <path d="M3 11L21 3L13 21L11 13L3 11Z" stroke="currentColor" stroke-width="2" stroke-linejoin="round" stroke-linecap="round"/>
      </svg>
    </button>
  </div>
</div>

<script>
  const messagesEl = document.getElementById('messages');
  const inputEl = document.getElementById('userInput');
  const sendBtn = document.getElementById('sendBtn');
  const emptyState = document.getElementById('emptyState');

  let history = [];

  // Auto-grow textarea
  inputEl.addEventListener('input', () => {
    inputEl.style.height = 'auto';
    inputEl.style.height = Math.min(inputEl.scrollHeight, 100) + 'px';
  });

  inputEl.addEventListener('keydown', (e) => {
    if (e.key === 'Enter' && !e.shiftKey) {
      e.preventDefault();
      sendMessage();
    }
  });

  sendBtn.addEventListener('click', sendMessage);

  function addBubble(text, role) {
    if (emptyState) emptyState.remove();
    const div = document.createElement('div');
    div.className = 'msg ' + (role === 'user' ? 'user' : 'bot');
    div.textContent = text;
    messagesEl.appendChild(div);
    messagesEl.scrollTop = messagesEl.scrollHeight;
    return div;
  }

  function addThinkingBubble() {
    if (emptyState) emptyState.remove();
    const div = document.createElement('div');
    div.className = 'msg bot thinking';
    div.innerHTML = '<span class="typing-dots"><span></span><span></span><span></span></span>';
    messagesEl.appendChild(div);
    messagesEl.scrollTop = messagesEl.scrollHeight;
    return div;
  }

  async function sendMessage() {
    const text = inputEl.value.trim();
    if (!text) return;

    addBubble(text, 'user');
    history.push({ role: 'user', content: text });
    inputEl.value = '';
    inputEl.style.height = 'auto';
    sendBtn.disabled = true;

    const thinkingBubble = addThinkingBubble();

    try {
      const response = await fetch("https://api.anthropic.com/v1/messages", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({
          model: "claude-sonnet-4-6",
          max_tokens: 1000,
          messages: history
        })
      });

      const data = await response.json();
      const replyText = (data.content || [])
        .map(block => block.type === 'text' ? block.text : '')
        .filter(Boolean)
        .join('\n');

      thinkingBubble.remove();
      addBubble(replyText || "Sorry, no response received. Please try again.", 'bot');
      history.push({ role: 'assistant', content: replyText });

    } catch (err) {
      thinkingBubble.remove();
      addBubble("Connection error. Please try again.", 'bot');
      console.error(err);
    } finally {
      sendBtn.disabled = false;
      inputEl.focu