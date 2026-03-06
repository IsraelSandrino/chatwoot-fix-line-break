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

Note the name of the main container (it usually contains the name of your service, without sidekiq, db, or redis).

<br>

### Step 3: Access the container and install the editor


```bash
docker exec -it NOME_DO_CONTAINER sh
apk update && apk add nano
```

<br>

### Step 4: Edit the message.rb file

```ruby
nano /app/app/models/message.rb
```

#### 4.1 Add the callback
Look for existing before_save instances:

```ruby
before_save :ensure_processed_message_content
before_save :ensure_in_reply_to
```

Add logo below:

```ruby
before_save :normalize_content
```

#### 4.2 Add the method
In the _private_ block, add the following before the last _end_:

```ruby
def normalize_content
  return if content.blank?

  self.content = content.gsub("\\\n", "\n")
end
```

#### 4.3 Save and exit
"CTRL + O" → Enter → Ctrl+X <br>
exit

<br>

### Step 5: Persist with the change.
#### 5.1 Create a folder for custom files

```bash
mkdir -p /etc/easypanel/projects/chatwoot/custom
```

#### 5.2 Copy the modified file

```bash
docker cp NOME_DO_CONTAINER:/app/app/models/message.rb /etc/easypanel/projects/chatwoot/custom/message.rb
```

#### 5.3 Identify the new containers

```bash
docker ps | grep chatwoot
```

#### 5.4 Verify

```bash
grep -n "normalize_content" /etc/easypanel/projects/chatwoot/custom/message.rb
```

<br>

Expected result:
```text
69:  before_save :normalize_content
275:  def normalize_content
```

<br>

### Step 6: Configure Volume Mount

```text 
⚠️ Important: Do this in both services: the main app and Sidekiq.
```

<br>

In EasyPanel:
1. Access the Chatwoot service
2. Go to the Mounts tab
3. Click on Add Bind Mount
4. Fill in:


| Campo            | Valor                                                   | 
|------------------|---------------------------------------------------------|
| Host Path        | /etc/easypanel/projects/chatwoot/custom/message.rb      |
| DMount Path      | /app/app/models/message.rb                              |

5. Save and deploy
6. Repeat for the sidekiq service

<br>

### Step 7: Verify installation

```bash
docker ps | grep chatwoot
docker exec NOME_DO_NOVO_CONTAINER grep -n "normalize_content" /app/app/models/message.rb
```

<br>

---

## ✅ Test

Send a message via your integration:

```text
Line 1 'SHIFT+ENTER' 
Line 2 'SHIFT+ENTER'
Line 3
```

It should appear in Chatwoot and WhatsApp:
```text
Line 1
Line 2
Line 3
```

<br>

---

## 📁 File Structure

```text
/etc/easypanel/projects/chatwoot/
└── custom/
    └── message.rb  # Arquivo modificado
```

<br>

---

## 🔄Chatwoot Updates

Ao atualizar o Chatwoot para uma nova versão:

1. Verifique se a estrutura do arquivo message.rb mudou
2. Se necessário, reaplique a modificação no novo arquivo
3. Atualize o arquivo em /etc/easypanel/projects/chatwoot/custom/message.rb

<br>

---

## 🐛 Common Problems

| Problem                | Solution                                                                             | 
|------------------------|--------------------------------------------------------------------------------------|
| Container not found    | Run `docker ps | grep chatwoot` to see the current name                              |
| Change not applied     | Verify that you have successfully deployed the mount after configuring the volume    |
| Error editing file     | Make sure to install nano: apk update && apk add nano                                |

<br>

---

## 🤝 Contribution
Did you find any problems or have a suggestion for improvement? Open an issue or submit a pull request!

<br>

---

## ⭐ Did you like it?
Leave a ⭐ on the repository to help others find this solution!

<br>

---

## 👤 Author
Developed by [Israel Sandrino](https://github.com/IsraelSandrino)

Tested on version: Chatwoot v4.11.1
