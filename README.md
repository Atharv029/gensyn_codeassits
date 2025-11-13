# Run Gensyn `codeassist` on a VPS (with Port Forwarding)

This guide explains how to install and run **Gensyn CodeAssist** on a VPS, especially when you’re running both a **Gensyn Node** and **CodeAssist** on the same server.  
It also covers how to access the CodeAssist web UI from your local machine (WSL) using **SSH port forwarding**.

---

##  Steps to Setup

### **Step 1  Clone the Repository**
```bash
git clone https://github.com/gensyn-ai/codeassist
```

---

### **Step 2  Install UV (Astral Runtime)**
```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

---

### **Step 3  Reload Your Shell**
```bash
source ~/.bashrc
```
> If you’re using Zsh, run `source ~/.zshrc` instead.

---

### **Step 4  Enter the Project Directory**
```bash
cd codeassist
```

---

### **Step 5  Change Ports (For VPS Users Running Both Gensyn Node & CodeAssist)**

If your VPS already has a **Gensyn Node** running on port `3000`, you need to change CodeAssist’s external port to avoid conflicts.

#### **Edit the Docker Compose file**
```bash
nano compose.yml
```

Find this line:
```yaml
ports:
  - "3000:3000"
```

Change it to:
```yaml
ports:
  - "3001:3000"
```
<br>

after last update both rl-swarm and code assits using the ollama and it running on same port so some faced to run this two node on vps
so chnage this port number also to 11500 

<img width="684" height="350" alt="image" src="https://github.com/user-attachments/assets/09a0c053-0502-427f-81a6-b6d15ddc30e9" />


like this 

<img width="746" height="368" alt="image" src="https://github.com/user-attachments/assets/24f2ad98-fb89-477b-a366-127e41f5758f" />


Save and exit (`Ctrl + X`, `press y and Enter`, ).

---

#### **Edit the `run.py` File**

```
nano run.py
```

**after last update both rl-swarm and code assits using the ollama and it running on same port so some faced to run this two node on vps**

so here is the solution chnage the port for ollama also in run.py 

you will see red comment like this and there will be port number 11434 so change that port number to 11500  


<img width="1149" height="392" alt="image" src="https://github.com/user-attachments/assets/03f1bdc0-8386-4d45-bbcb-0c15b92e6434" />

<br>
after this scroll down more and you will see this same here change port number to 11500


<img width="1205" height="301" alt="image" src="https://github.com/user-attachments/assets/8fef012a-9dea-4acd-97de-789f49ed180f" />


```

# Changed from:
if not requests.get("http://localhost:11435").ok:
    raise Exception("Ollama API is not reachable at http://localhost:11435")

# To:
if not requests.get("http://localhost:11500").ok:
    raise Exception("Ollama API is not reachable at http://localhost:11500")

```
    
Save and exit.

---

then 

```
cd web-ui/src/simulation/simulators && nano OllamaCodeSimulator.ts

```


scroll down and do change here 


<img width="1166" height="255" alt="image" src="https://github.com/user-attachments/assets/21e649e3-9824-4540-94ad-7d43563c8029" />


Save and exit (`Ctrl + X`, `press y and Enter`, )



go to codeassist directory 
```
cd ~/codeassist
```


---

### **Step 6 Run the Application**


```

echo 'export OLLAMA_BASE_URL="http://localhost:11500"' >> ~/.bashrc
echo 'export OLLAMA_HOST="http://localhost:11500"' >> ~/.bashrc
source ~/.bashrc

```

then

```
uv run run.py --port 3001
```

```

When prompted for your **Hugging Face token**:
- Paste it using **Ctrl + V** (it won’t appear on screen).
- Press **Enter**.

---

### **Step 7  Forward Ports from VPS to Your Local Machine (WSL)**

Since you can’t directly open `localhost` on the VPS, forward the required ports to your local machine.

Run this **on your local WSL terminal** (replace with your VPS username and IP):

```bash
ssh -L 3001:localhost:3001 \
    -L 8000:localhost:8000 \
    -L 8001:localhost:8001 \
    -L 8008:localhost:8008 \
    username@your_vps_ip
```

- `3001` is for CodeAssist’s web UI.  
- `8000`, `8001`, `8008` are optional — useful for other services like Gensyn Node.

If your VPS is linked with a public SSH key, it may simply ask for confirmation (`yes`) and your password.

---

### **Alternative Method — Setup SSH Key Authentication (Recommended)**

If you want to avoid entering your VPS password each time or enable SSH forwarding securely, follow these steps to set up an SSH key:

#### **Step 1  Generate an SSH key on your local machine (WSL)**
```bash
ssh-keygen -t ed25519 -C "batman"
```

> You can replace `"batman"` with any name or comment you like.  
> When asked for a passphrase, just press **Enter** three times (no need to set one).

---

#### **Step 2 Display your public key**
```bash
cat ~/.ssh/id_ed25519.pub
```

Copy the entire output  this is your **public SSH key**.

---

#### **Step 3 Add your SSH key to the VPS**

Now, you need to paste this key into your VPS so it recognizes your local machine.

If your VPS hosting provider **doesn’t allow adding SSH keys after setup**, you can still do it manually using the VPS console (VNC portal).  

In your VPS terminal, run:
```bash
nano ~/.ssh/authorized_keys
```

Paste your copied key inside this file.  
To save it:
```
Ctrl + X → Y and Enter 
```

---

> ⚠️ Some VPS providers like **Serverica** don’t allow SSH keys to be added later through the dashboard, so you must use the **VNC console** method to add the key manually as shown above.  
> Once added, you can connect directly without typing your password each time.

---

#### **Step 4  Now connect using your SSH key**
Once your key is added to the VPS, use the same SSH forwarding command:

```bash
ssh -L 3001:localhost:3001 \
    -L 8000:localhost:8000 \
    -L 8001:localhost:8001 \
    -L 8008:localhost:8008 \
    username@your_vps_ip
```

Now it will connect automatically (no password needed).

---

### **Step 8  Access CodeAssist in Your Browser**

Once the tunnel is active, open this in your local browser:

```
http://localhost:3001
```

A link will also appear in the terminal  click it.  
Log in with your email when prompted, and you’re ready to use CodeAssist!

---

##  Example Summary

**Final `compose.yml` snippet:**
```yaml
services:
  codeassist:
    image: gensyn/codeassist:latest
    ports:
      - "3001:3000"
    environment:
      - HF_TOKEN=${HF_TOKEN}
```

**Example SSH Command (single line):**
```bash
ssh -L 3001:localhost:3001 -L 8000:localhost:8000 -L 8001:localhost:8001 -L 8008:localhost:8008 username@your_vps_ip
```

---

##  Notes & Tips

- When pasting the Hugging Face token, it will not appear  that’s normal.
- Keep the SSH terminal open while using CodeAssist (the tunnel closes if you close it).
- If port `3001` is already in use, choose another (e.g., `3002`) and update both files.
- Local users (non-VPS): skip SSH forwarding and open `http://localhost:3000` directly.
- For safety, do **not** expose CodeAssist ports publicly use SSH tunneling.
- Using SSH keys saves time and increases security for future connections.

---

✅ **You’re all set!**  
Once you log in via your email, you’re ready to use Gensyn CodeAssist to run code, solve questions, and explore features!
