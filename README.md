# 🔧 Fix: Line Break in Chatwoot (\n literal)

## 📌 Problem

Messages sent via API/integration (n8n, Evolution API, etc.) display a literal `\n` instead of a line break.

| Before (❌) | After (✅) |

|-----------|-------------|

| `Hello!\nWelcome!\nHow can I help?` | Hello!<br>Welcome!<br>How can I help? |

---

## 🎯 Solution

Modify the Chatwoot `message.rb` file to convert the literal `\n` into a real line break before saving the message.

---

## 📋 Prerequisites

- SSH access to the server
- Chatwoot running with Docker
- EasyPanel (or another container manager)

---

## 🚀 Installation

### Step 1: Access the server via SSH

```bash
ssh root@YOUR_SERVER_IP
```

<br>

### Step 2: Identify the Chatwoot containers

```bash
docker ps | grep chatwoot
```

Anote o nome do container principal (geralmente contém o nome do seu serviço, sem sidekiq, db ou redis).

<br>

### Step 3: Access the container and install the editor


```bash
docker exec -it NOME_DO_CONTAINER sh
apk update && apk add nano
```

<br>

### Step 4: Edit the message.rb file

```bash
nano /app/app/models/message.rb
```

#### 4.1 Add the callback
Procure pelos before_save existentes:

```ruby
before_save :ensure_processed_message_content
before_save :ensure_in_reply_to
```
