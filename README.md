<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>TeachBek</title>
<style>
  * { margin: 0; padding: 0; box-sizing: border-box; }
  body { background: #0F1117; color: #D4D8E8; font-family: Georgia, serif; height: 100vh; display: flex; flex-direction: column; }

  #header { background: #1A1D2E; padding: 16px 24px; display: flex; align-items: center; gap: 14px; border-bottom: 1px solid #252840; }
  #header .avatar { font-size: 32px; }
  #header h1 { font-size: 20px; color: #E8D5A3; }
  #header p { font-size: 12px; color: #7A8099; margin-top: 2px; }

  #chat { flex: 1; overflow-y: auto; padding: 20px 24px; display: flex; flex-direction: column; gap: 16px; }

  .msg { display: flex; flex-direction: column; gap: 4px; max-width: 80%; }
  .msg.professor { align-self: flex-start; }
  .msg.user { align-self: flex-end; }

  .msg-label { font-size: 12px; color: #7A8099; }
  .msg.user .msg-label { text-align: right; color: #4ECDC4; }

  .msg-bubble { padding: 12px 16px; border-radius: 12px; font-size: 14px; line-height: 1.6; white-space: pre-wrap; }
  .msg.professor .msg-bubble { background: #1A1D2E; border: 1px solid #252840; color: #D4D8E8; border-radius: 4px 12px 12px 12px; }
  .msg.user .msg-bubble { background: #252840; color: #B8E0DC; border-radius: 12px 4px 12px 12px; }

  .correction { color: #FFB347; }
  .typing { color: #5A6080; font-style: italic; font-size: 13px; padding: 8px 0; }

  #bottom { background: #1A1D2E; padding: 16px 24px; border-top: 1px solid #252840; display: flex; gap: 10px; align-items: flex-end; }
  #input { flex: 1; background: #252840; border: 1px solid #3A3D52; color: #E8EAF6; padding: 10px 14px; border-radius: 8px; font-size: 14px; font-family: Georgia, serif; resize: none; min-height: 44px; max-height: 120px; outline: none; }
  #input:focus { border-color: #4ECDC4; }
  #sendbtn { background: #4ECDC4; color: #0F1117; border: none; padding: 10px 20px; border-radius: 8px; cursor: pointer; font-size: 14px; font-weight: bold; font-family: Georgia, serif; height: 44px; }
  #sendbtn:hover { background: #3ABAB2; }
  #sendbtn:disabled { background: #3A3D52; color: #7A8099; cursor: not-allowed; }
  #hint { text-align: center; font-size: 11px; color: #3A3D52; padding: 6px; background: #0F1117; }
</style>
</head>
<body>

<div id="header">
  <div class="avatar">🎓</div>
  <div>
    <h1>TeachBek</h1>
    <p>Your Personal AI English Teacher</p>
  </div>
</div>

<div id="chat"></div>

<div id="bottom">
  <textarea id="input" placeholder="Write in English or Russian..." rows="1" onkeydown="handleKey(event)" oninput="autoResize(this)"></textarea>
  <button id="sendbtn" onclick="sendMessage()">Send ➤</button>
</div>
<div id="hint">Enter — отправить &nbsp;•&nbsp; Shift+Enter — новая строка</div>

<script>
const API_KEY = "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";
let history = [];
let thinking = false;

const SYSTEM = `You are TeachBek, a warm and encouraging English teacher. The student speaks Russian and is learning English.

YOUR ROLE:
- Chat naturally in English
- If they write in Russian, respond in English and gently encourage them to try in English
- ALWAYS correct grammar, spelling, vocabulary mistakes politely
- Use this format for corrections: ✏️ Correction: "wrong" → "correct" — brief explanation
- Praise good English with 🌟
- Keep responses short (2-4 sentences + corrections)
- Ask follow-up questions to keep conversation going

Start by greeting the student warmly and asking about their English level.`;

window.onload = () => getReply();

function handleKey(e) {
  if (e.key === "Enter" && !e.shiftKey) { e.preventDefault(); sendMessage(); }
}

function autoResize(el) {
  el.style.height = "auto";
  el.style.height = Math.min(el.scrollHeight, 120) + "px";
}

function addMessage(role, text) {
  const chat = document.getElementById("chat");
  const div = document.createElement("div");
  div.className = "msg " + role;

  const label = document.createElement("div");
  label.className = "msg-label";
  label.textContent = role === "professor" ? "🎓 TeachBek" : "👤 You";

  const bubble = document.createElement("div");
  bubble.className = "msg-bubble";

  text.split("\n").forEach(line => {
    const span = document.createElement("span");
    if (line.trim().startsWith("✏️")) span.className = "correction";
    span.textContent = line + "\n";
    bubble.appendChild(span);
  });

  div.appendChild(label);
  div.appendChild(bubble);
  chat.appendChild(div);
  chat.scrollTop = chat.scrollHeight;
}

function showTyping() {
  const chat = document.getElementById("chat");
  const div = document.createElement("div");
  div.id = "typing";
  div.className = "typing";
  div.textContent = "🎓 TeachBek is typing...";
  chat.appendChild(div);
  chat.scrollTop = chat.scrollHeight;
}

function removeTyping() {
  const t = document.getElementById("typing");
  if (t) t.remove();
}

async function sendMessage() {
  if (thinking) return;
  const input = document.getElementById("input");
  const text = input.value.trim();
  if (!text) return;
  input.value = "";
  input.style.height = "auto";
  addMessage("user", text);
  history.push({ role: "user", content: text });
  getReply();
}

async function getReply() {
  thinking = true;
  document.getElementById("sendbtn").disabled = true;
  showTyping();

  const messages = [{ role: "system", content: SYSTEM }, ...history];

  try {
    const res = await fetch("https://api.groq.com/openai/v1/chat/completions", {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
        "Authorization": "Bearer " + API_KEY
      },
      body: JSON.stringify({ model: "llama-3.3-70b-versatile", max_tokens: 1024, messages })
    });
    const data = await res.json();
    if (data.error) throw new Error(data.error.message);
    const reply = data.choices[0].message.content;
    history.push({ role: "assistant", content: reply });
    removeTyping();
    addMessage("professor", reply);
  } catch (e) {
    removeTyping();
    addMessage("professor", "⚠️ Error: " + e.message);
  }

  thinking = false;
  document.getElementById("sendbtn").disabled = false;
  document.getElementById("input").focus();
}
</script>
</body>
</html>
