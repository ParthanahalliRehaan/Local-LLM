<div align="center">

# 🦙 RUN LLM LOCAL

**OFFLINE AI · DOCKER + GGUF + SIMPLE CHAT UI · ZERO CLOUD REQUIRED**

![](https://img.shields.io/badge/STACK-DOCKER%20%7C%20WSL2%20%7C%20LLAMA.CPP-F5A623?style=for-the-badge&labelColor=0A0A0A)
![](https://img.shields.io/badge/MODEL-GGUF%20FORMAT-F5A623?style=for-the-badge&labelColor=0A0A0A)
![](https://img.shields.io/badge/STATUS-FULLY%20OFFLINE-4ADE80?style=for-the-badge&labelColor=0A0A0A)
![](https://img.shields.io/badge/PORT-LOCALHOST%3A8080-60A5FA?style=for-the-badge&labelColor=0A0A0A)

</div>

---

## `>` WHAT EACH COMPONENT IS

| COMPONENT | WHAT IT DOES | ROLE |
|:---|:---|:---|
| `DOCKER` | MINI COMPUTER INSIDE YOUR LOCAL MACHINE. RUNS ISOLATED CONTAINERS — NO DIRECT INSTALL ON WINDOWS. | CONTAINER · ISOLATION |
| `WSL2` | WINDOWS SUBSYSTEM FOR LINUX. LETS WINDOWS RUN LINUX PROGRAMS. REQUIRED BACKEND FOR DOCKER. | LINUX LAYER · REQUIRED |
| `LLAMA.CPP` | THE ENGINE THAT LOADS AND RUNS LLAMA MODELS LOCALLY. SUPPORTS CPU/GPU. READS GGUF FILES. | INFERENCE · CPU/GPU |
| `GGUF` | THE ACTUAL AI MODEL FILE. THE **BRAIN**. LLAMA.CPP IS THE **BODY**. | MODEL FILE · THE BRAIN |

---

## `>` SETUP STEPS

### `[01]` ENABLE WSL2 — REQUIRED FOR DOCKER

OPEN **POWERSHELL AS ADMINISTRATOR** AND RUN:

```powershell
# ENABLE WINDOWS SUBSYSTEM FOR LINUX
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
```

RESTART PC. THEN RUN:

```powershell
wsl --install
wsl --set-default-version 2
```

> ⚠ IF YOU GET `403` — USE: `wsl --install --web-download` OR INSTALL WSL FROM MICROSOFT STORE.

---

### `[02]` INSTALL DOCKER DESKTOP

DOWNLOAD FROM **[DOCKER.COM/PRODUCTS/DOCKER-DESKTOP](https://docker.com/products/docker-desktop)**

START DOCKER DESKTOP → ENSURE **DOCKER ENGINE IS RUNNING** BEFORE PROCEEDING.

---

### `[03]` PULL LLAMA.CPP DOCKER IMAGE

```bash
docker pull ghcr.io/ggml-org/llama.cpp:server
```

| INCLUDED IN THIS IMAGE | WHAT IT DOES |
|:---|:---|
| `LLAMA.CPP RUNTIME` | CORE INFERENCE ENGINE |
| `TOKENIZER` | CONVERTS TEXT ↔ TOKENS |
| `SERVER API` | OPENAI-COMPATIBLE HTTP ENDPOINT |
| `CPU BACKENDS` | RUNS ON ANY MACHINE, NO GPU NEEDED |

---

### `[04]` DOWNLOAD A GGUF MODEL

CREATE A FOLDER, THEN DROP YOUR MODEL INSIDE IT:

```
C:\llama-local\models\model.gguf
```

| MODEL | SIZE | STATUS |
|:---|:---|:---|
| `LLAMA 3B` | ~2 GB | ![](https://img.shields.io/badge/WORKS-4ADE80?style=flat-square&labelColor=0A0A0A) |
| `LLAMA 8B` | ~5 GB | ![](https://img.shields.io/badge/WORKS-4ADE80?style=flat-square&labelColor=0A0A0A) |

> ⚠ RENAME YOUR MODEL FILE TO `model.gguf` BEFORE PLACING IT IN THE FOLDER.

---

### `[05]` RUN THE LOCAL LLM SERVER

```bash
docker run --rm -p 8080:8080 \
  -v C:\llama-local\models:/models \
  ghcr.io/ggml-org/llama.cpp:server \
  --model /models/model.gguf
```

WAIT UNTIL YOU SEE:

```
✓ MODEL LOADED
✓ SERVER IS LISTENING ON HTTP://0.0.0.0:8080
```

> THE AI IS NOW ALIVE — FULLY LOCAL, FULLY OFFLINE.

---

### `[06]` TEST VIA POWERSHELL

```powershell
irm "http://localhost:8080/v1/chat/completions" `
  -Method Post `
  -Body '{"model":"local","messages":[{"role":"user","content":"Hello"}]}' `
  -ContentType "application/json" | ConvertTo-Json -Depth 20
```

> ✓ A JSON RESPONSE FROM THE MODEL = YOU ARE LIVE.

---

### `[07]` SIMPLE LOCAL CHAT UI

CREATE `chat.html` — DOUBLE-CLICK TO OPEN IN BROWSER AND CHAT WITH YOUR MODEL:

```html
<!DOCTYPE html>
<html><head><meta charset="UTF-8"></head><body>
<h2>Local Llama Chat</h2>
<textarea id="chat" rows="20" cols="80" readonly></textarea><br><br>
<input id="msg" placeholder="Type your message..." style="width:400px" />
<button onclick="send()">Send</button>
<script>
async function send() {
  const q = document.getElementById("msg").value;
  const res = await fetch("http://localhost:8080/v1/chat/completions", {
    method: "POST",
    headers: {"Content-Type": "application/json"},
    body: JSON.stringify({ model: "local", messages: [{role:"user", content:q}] })
  });
  const data = await res.json();
  document.getElementById("chat").value += "\nAI: " + data.choices[0].message.content;
}
</script></body></html>
```

---

## `>` TROUBLESHOOTING

| ISSUE | CAUSE | FIX |
|:---|:---|:---|
| `403 ON WSL --INSTALL` | NETWORK / POLICY BLOCK | USE `wsl --install --web-download` OR INSTALL WSL FROM MICROSOFT STORE |
| `POWERSHELL CURL ERRORS` | `curl` IS ALIASED IN POWERSHELL | USE `irm` INSTEAD OF `curl` IN POWERSHELL |
| `PORT MISMATCH` | BUILD DEFAULTS VARY | SOME LLAMA BUILDS USE `8080`, NOT `8000` — ALWAYS CHECK SERVER OUTPUT |
| `SLOW INFERENCE` | CPU ONLY, NO GPU | SEE NEXT STEPS → GPU ACCELERATION |
| `STORAGE WORRIES` | MISUNDERSTANDING | ONLY THE GGUF FILE USES DISK. CHAT USES RAM. NO LOGS UNLESS YOU ADD THEM |

---

## `>` NEXT STEPS

| UPGRADE | WHAT YOU GET | HOW |
|:---|:---|:---|
| `💾 CONVERSATION STORAGE` | PERSIST CHAT HISTORY ACROSS SESSIONS | ADD A DATABASE LAYER — SQLITE OR SUPABASE |
| `🖥 OPEN-WEBUI` | MODERN CHATGPT-STYLE INTERFACE FOR YOUR LOCAL MODEL | `docker pull ghcr.io/open-webui/open-webui` |
| `⚡ GPU ACCELERATION` | DRAMATICALLY FASTER INFERENCE | ADD CUDA OR VULKAN FLAGS TO YOUR `docker run` COMMAND |
| `🛠 BUILD ON THE API` | POWER YOUR OWN AI APPS WITH A LOCAL BACKEND | USE THE OPENAI-COMPATIBLE `localhost:8080` ENDPOINT |
| `🔁 SWAP MODELS` | TEST DIFFERENT PERSONALITIES AND CAPABILITIES | DROP A NEW `.gguf` INTO `/models` AND RESTART THE CONTAINER |

---

<div align="center">

![](https://img.shields.io/badge/BUILT%20OFFLINE-F5A623?style=for-the-badge&labelColor=0A0A0A)
![](https://img.shields.io/badge/RUNS%20LOCAL-F5A623?style=for-the-badge&labelColor=0A0A0A)
![](https://img.shields.io/badge/ZERO%20CLOUD-F5A623?style=for-the-badge&labelColor=0A0A0A)
![](https://img.shields.io/badge/SHIPPED%20WITH%20OBSESSION-F5A623?style=for-the-badge&labelColor=0A0A0A)

</div>
