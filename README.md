# VPS OPENCLAW SETUP 
Kali VPS OpenClaw Cybersecurity Setup Walkthrough

This guide walks you through installing OpenClaw on a Kali Linux VPS, configuring DeepSeek as your AI model, and connecting Telegram so you can control your cybersecurity AI agent from anywhere.

What You Are Building

You are setting up an autonomous AI agent that runs 24/7 on your VPS. This agent can:

· Search the web for security research and CVE information
· Write and execute code (with your approval)
· Connect to GitHub to read and write repositories
· Interact with you through Telegram from anywhere
· Be programmed with a "soul" file that defines its cybersecurity role

OpenClaw is an agent harness that works 24/7 to automate tasks. DeepSeek-V4 is now the default model, providing enhanced reasoning and agent execution capabilities.

Prerequisites

Before starting, you need two things:

1. DeepSeek API Key – Get it from https://platform.deepseek.com/api_keys. It starts with sk-.
2. Telegram Bot Token – Message @BotFather on Telegram, send /newbot, follow the prompts, and copy the token. It looks like 1234567890:ABCdefGHIjklMNOpqrsTUVwxyz.

Step 1: Prepare Your Kali VPS Environment

SSH into your Kali VPS as root (Kali typically uses root by default, but if you're using a non-root user with sudo, adjust commands accordingly).

Update the system:

```bash
apt update && apt upgrade -y
```

Install essential build tools:

```bash
apt install -y build-essential git curl wget
```

Step 2: Install Node.js 22

OpenClaw requires Node.js version 22 or higher. Use NodeSource for the official binary:

```bash
curl -fsSL https://deb.nodesource.com/setup_22.x | bash -
apt install -y nodejs
```

Verify the installation:

```bash
node --version   # Should show v22.x.x
npm --version    # Should show 10.x.x or higher
```

Step 3: Install OpenClaw

The official install script handles everything automatically:

```bash
curl -fsSL https://openclaw.ai/install.sh | bash
```

The script will:

· Clone the OpenClaw repository
· Install dependencies
· Build the project
· Make the openclaw command available

After the script finishes, reload your shell:

```bash
source ~/.bashrc
```

Verify the installation:

```bash
openclaw --version
```

If you get "command not found", add the npm global bin directory to your PATH:

```bash
echo 'export PATH="/root/.npm-global/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

Step 4: Run the Setup Wizard

Start the configuration wizard:

```bash
openclaw onboard
```

The wizard will guide you through:

1. Security disclaimer – select Yes to continue
2. Setup mode – choose QuickStart
3. Gateway configuration – accept defaults (port 18789, loopback binding)
4. Model provider – select DeepSeek
5. DeepSeek API key – paste your key
6. Model selection – choose deepseek/deepseek-chat (or the default V4 flash version)
7. Channel selection – choose Telegram (Bot API)
8. Telegram plugin – select Use local plugin path
9. Telegram bot token – paste your token from BotFather
10. DM policy – select pairing (secure default requiring approval)

When the wizard finishes, it will display a pairing code. Copy this code.

Step 5: Pair Your Telegram Account

Open Telegram on your phone or computer. Find your bot by searching for its username.

Send the pairing code as a message. Just the code, nothing else.

The bot will confirm that you are now paired.

If you need to approve a pairing code manually from the command line:

```bash
openclaw pairing approve telegram YOUR_CODE_HERE
```

Step 6: Create Your Cybersecurity Agent Soul (AGENTS.md)

This is where you define your AI agent's role. OpenClaw agents are specialized through a "soul" file that contains system prompts and behavioral protocols. Create a file called AGENTS.md in the workspace:

```bash
mkdir -p /root/clawd
nano /root/clawd/AGENTS.md
```

Paste this configuration for a cybersecurity penetration testing agent:

```markdown
# Agent: Cybersecurity Penetration Testing Specialist

_You are an autonomous security researcher and penetration tester. Your environment is Kali Linux, equipped with standard security tools._

## Core Truths

**Find Vulnerabilities.** Your purpose is to identify security weaknesses in code and infrastructure through authorized testing.
**Do No Harm.** Never execute attacks on systems without explicit permission from the user.
**Document Everything.** Every finding requires evidence and remediation steps.

## Operational Protocols

- Use the cycle: `THOUGHT:` -> `ACTION:` -> `OBSERVATION:` for every step
- Research known vulnerabilities using web_search
- Write and test exploit proofs-of-concept only when ethically authorized
- Generate secure code following OWASP guidelines

## Available Tools

- exec – Run commands with user approval
- web_search – Research CVEs and exploits
- read/write – Work with files
- github – Clone and manage repositories

## Constraints

- Never execute commands that modify system files without confirmation
- Always ask for explicit permission before scanning external targets
- Report findings with severity ratings (Critical/High/Medium/Low)
- Include remediation steps for every vulnerability found
```

Save the file (Ctrl+O, Enter, Ctrl+X).

Load this soul into OpenClaw:

```bash
openclaw agent load --file /root/clawd/AGENTS.md
```

Step 7: Start the Gateway

Start the OpenClaw gateway:

```bash
openclaw gateway
```

You will see log output showing the gateway is running. Keep this terminal open.

To verify the gateway is running properly:

```bash
openclaw gateway status
```

If you need to restart the gateway later:

```bash
openclaw gateway restart
```

Step 8: Test Your Agent

Send test messages to your Telegram bot to verify each capability.

Test 1 – Basic conversation:
Send: Who are you and what is your purpose?

The bot should respond based on your AGENTS.md file.

Test 2 – Web search:
Send: Search the web for the latest critical CVEs released this month.

The bot will use the web_search tool to find and summarize recent vulnerabilities.

Test 3 – Code execution with approval:
Send: Write a Python script that scans 127.0.0.1 for open ports 22, 80, and 443. Then run it.

The bot will generate the code and ask for your approval. Type yes or approve to let it run.

Test 4 – Security research:
Send: Research Log4Shell and explain the exploit, impact, and mitigation.

The bot will search the web and provide a detailed analysis.

Step 9: Configure GitHub Access (Optional)

If you want your agent to interact with GitHub repositories, generate a personal access token:

1. Go to GitHub.com -> Settings -> Developer settings -> Personal access tokens -> Tokens (classic)
2. Generate new token
3. Name it "OpenClaw Agent"
4. Expiration: No expiration
5. Scopes: check repo and workflow
6. Copy the token

Set the token in OpenClaw:

```bash
openclaw config set integrations.github.token YOUR_GITHUB_TOKEN
```

Restrict which repositories the agent can access:

```bash
openclaw config set integrations.github.repos "YOUR_USERNAME/REPO_NAME"
```

Enable GitHub in the configuration file:

```bash
nano ~/.openclaw/openclaw.json
```

Find the github section and change "enabled": false to "enabled": true. Save and exit.

Restart the gateway:

```bash
openclaw gateway restart
```

Step 10: Install Additional Skills

OpenClaw's capabilities can be extended with skills that add specific tools. Install useful skills for cybersecurity work:

```bash
cd ~/openclaw
npx clawhub install desearch-web-search --force
npx clawhub install proactive-agent
npx clawhub install skill-vetter
```

After installing skills, restart the gateway:

```bash
openclaw gateway restart
```

Step 11: Keep the Gateway Running Permanently

Since you are on a VPS, you want the agent to keep running even after you disconnect. Use tmux:

```bash
apt install -y tmux
tmux new -s openclaw
openclaw gateway
```

Press Ctrl+B, then D to detach. The gateway continues running.

To reattach later:

```bash
tmux attach -t openclaw
```

Alternative: Install as a systemd service:

```bash
openclaw onboard --install-daemon
systemctl enable openclaw-gateway
systemctl start openclaw-gateway
```

Security Hardening

OpenClaw can execute commands on your system, which requires careful security considerations.

Approval Required: Your configuration already has "confirm": true for the exec tool. OpenClaw will never run a terminal command without your explicit yes or approve in Telegram.

Limited Workspace: The workspace is set to /root/clawd. The agent cannot read or write outside this directory by default. Do not change this to a broader path.

Pairing Protection: DM policy is set to pairing, which means unknown users who message your bot receive a pairing code. You must approve them before they can interact.

Protect the Gateway Token: The token in ~/.openclaw/openclaw.json controls access to the web UI. Keep it secret.

For multi-user scenarios, run openclaw security audit --deep regularly.

Useful Maintenance Commands

Command Description
openclaw gateway status Check if gateway is running
openclaw gateway restart Restart the gateway
openclaw gateway stop Stop the gateway
openclaw pairing list List approved Telegram users
openclaw logs View agent logs
openclaw config get models.providers.deepseek.apiKey Verify DeepSeek API key
openclaw doctor --fix Diagnose and fix configuration issues
journalctl -u openclaw-gateway -f View systemd service logs (if using daemon)

Troubleshooting

Gateway already running error: Run openclaw gateway stop first, then openclaw gateway.

Pairing does not persist in Docker/container: If you see repeated pairing requests despite approving, check that credentials are being saved to persistent storage. The approval should be recorded in /data/.openclaw/credentials/telegram-pairing.json.

Bot does not respond: Check the gateway logs for errors. Most often this is an invalid API key or bot token.

"command not found" for openclaw: Add the npm global bin directory to PATH: export PATH="/root/.npm-global/bin:$PATH"

Port 18789 already in use: Stop the existing gateway with openclaw gateway stop or use a different port in the config file.

Web interface shows unauthorized: The token in your URL must match the token in ~/.openclaw/openclaw.json under gateway.auth.token.

What's Next

Your OpenClaw agent is now running on your Kali VPS with DeepSeek as the AI engine. You can control it entirely through Telegram.

To explore more advanced configurations:

· Create multiple agent souls for different roles (Red Team, Blue Team)
· Connect additional channels like Discord, Slack, or WhatsApp
· Integrate with Composio for access to 1000+ tools across different apps
· Build automated workflows that run on schedules

For complete documentation, visit docs.openclaw.ai.

---

# zima-board-openclaw-

Zima/openCLAW

You are building a penetration testing and offensive coding master.


Step 1: Prepare Your Zima Board's Operating System

SSH into your Zima Board or open a terminal directly on it.

First, update everything.


sudo apt update && sudo apt upgrade -y

Now install Docker. This is the container system that will run OpenClaw in isolation.

sudo apt install -y docker.io docker-compose

Enable Docker to start automatically when your Zima Board reboots.

sudo systemctl enable docker --now

Add your user to the Docker group so you can run commands without sudo every time.

sudo usermod -aG docker $USER

Log out of your SSH session and log back in for this change to take effect. You can test it by running docker ps. It should show no errors.

Step 2: Create the OpenClaw Workspace Directory

Create a folder on your Zima Board where OpenClaw will store its configuration and workspace files.

mkdir -p ~/openclaw/data ~/openclaw/workspace

The data folder stores your config and chat history. The workspace folder is where OpenClaw will read and write files and execute code. This is its "desktop."

Step 3: Get Your API Keys (Only Two Things)

You only need two things for this setup.

First, go to platform.deepseek.com/api_keys. Log in or sign up. Click "Create API key." Give it a name. Copy the key that starts with sk. Save this somewhere safe.

Second, open Telegram. Search for @BotFather. Start a chat. Send the command /newbot. Follow the prompts. Give it a name like "My Security Claw." Give it a username that ends in bot, like mysecurityclaw_bot. When it finishes, BotFather will give you a token that looks like 1234567890:ABCdefGHIjklMNOpqrsTUVwxyz. Copy this token.

Step 4: Run the OpenClaw Docker Container

We will use the official OpenClaw Docker image. This command creates the container and connects it to your folders.

docker run -d 
--name openclaw 
--restart unless-stopped 
-p 18789:18789 
-p 18791:18791 
-v ~/openclaw/data:/root/.openclaw 
-v ~/openclaw/workspace:/root/clawd 
khal3d/openclaw:latest

Here is what this command does. It names the container openclaw. It makes it restart automatically if it crashes or if your Zima board reboots. It opens port 18789 for the web interface. It connects your data folder to the container's config folder. It connects your workspace folder to the container's working directory.

Wait about 30 seconds for the container to fully start.

Step 5: Enter the Container to Configure OpenClaw

You need to get inside the running container to run setup commands.

docker exec -it openclaw bash

Your prompt will change to something like root@container-id. You are now inside the container.

Step 6: Configure the Gateway Token for Web Access

Inside the container, run the setup command.

openclaw setup

It will ask you questions. When it asks for a gateway token, generate a random one like mysecretclawtoken2025. You will use this to log into the web interface later. Just type it in and press enter.

When it asks if you want to set up a channel, say no for now. We will do Telegram manually.

Step 7: Configure DeepSeek as Your AI Model

OpenClaw needs to know how to talk to DeepSeek. Run this command to open the config file.

nano /root/.openclaw/openclaw.json

This file is currently empty or has default settings. Delete everything in it and paste the exact configuration below. Replace YOUR_DEEPSEEK_API_KEY with your actual key from Step 3.

{
"gateway": {
"port": 18789,
"bind": "0.0.0.0",
"auth": {
"mode": "token",
"token": "mysecretclawtoken2025"
}
},
"models": {
"providers": {
"deepseek": {
"apiKey": "YOUR_DEEPSEEK_API_KEY",
"baseUrl": "https://api.deepseek.com/v1",
"api": "openai-completions"
}
},
"defaults": {
"model": "deepseek/deepseek-chat"
}
},
"agents": {
"defaults": {
"workspace": "/root/clawd",
"maxIterations": 25,
"model": "deepseek/deepseek-chat"
}
},
"tools": {
"exec": {
"enabled": true,
"confirm": true,
"sandbox": true
},
"web_search": {
"enabled": true
},
"github": {
"enabled": false
}
}
}

Save the file. In nano, press Ctrl+X, then Y, then Enter.

This configuration does several things. It tells OpenClaw to use DeepSeek as its brain. It enables the exec tool for running code with your approval. It enables the web_search tool for internet access. It sets the workspace to your mounted folder.

Step 8: Configure the Telegram Channel

Still inside the container, run the command to configure Telegram.

openclaw channels add telegram

It will ask for your bot token. Paste the token you got from BotFather in Step 3.

It will ask about DM policy. Select pairing. This is secure. It requires you to approve yourself before anyone can talk to your bot.

It will ask if you want to configure now. Say yes.

Step 9: Restart OpenClaw to Load the Config

Exit the container.

exit

Now restart the container to load the new configuration.

docker restart openclaw

Wait about 10 seconds for it to come back up.

Step 10: Get Your Pairing Code and Approve Yourself

Check the logs to get your pairing code.

docker logs openclaw 2>&1 | grep -i "pairing code"

It will show a line that says something like Pairing code: 123ABC. Copy that code.

Now open Telegram on your phone or computer. Find your bot by searching for the username you created. Send a message to the bot. Just send the pairing code exactly as it appears, nothing else.

The bot will confirm that you are now paired. You can now send commands to your bot and it will respond.

Step 11: Access the Web Interface

Open a web browser on any computer on your same home network. Type your Zima Board's IP address followed by port 18789 and your token.

http://YOUR_ZIMA_IP:18789/?token=mysecretclawtoken2025

Replace YOUR_ZIMA_IP with the actual IP address of your Zima board. This dashboard lets you see what your agent is doing, view logs, and manage settings.

Step 12: Give OpenClaw Its Security and Coding Powers

Your OpenClaw is now running with DeepSeek and basic tools. But you want a penetration testing master. You need to install the specific skills that give it offensive security knowledge.

OpenClaw's power comes from Skills. These are like apps you install that teach it how to do specific things. The base tools (exec, web_search, read, write) are the muscles. Skills are the training.

Go back inside the container.

docker exec -it openclaw bash

Now install the Self-Improving Agent skill. This allows OpenClaw to learn from its mistakes and remember your preferences.

npx clawhub install self-improving-agent

Install the Proactive Agent skill. This makes it actively work on problems instead of just waiting for your next command.

npx clawhub install proactive-agent

Install the GitHub skill so it can clone, read, and write code to repositories.

npx clawhub install github

Install the Agent Browser skill. This gives it a full web browser it can control for advanced research and logging into websites.

npx clawhub install agent-browser

Install the Skill Vetter skill. This scans any new skill you install for security risks.

npx clawhub install skill-vetter

Exit the container.

exit

Restart OpenClaw one more time to load all the new skills.

docker restart openclaw

Step 13: Configure GitHub Access (For Cloning and Writing Code)

OpenClaw can work with GitHub using the native github tool. You need to give it a personal access token.

Go to GitHub. Click your profile picture in the top right. Click Settings. Scroll to the bottom of the left sidebar and click Developer settings. Click Personal access tokens. Click Tokens (classic). Click Generate new token (classic).

Give it a name like "OpenClaw Agent." Under Expiration, select No expiration for a permanent token. Under scopes, check the following boxes. repo (full control of private repositories). workflow (updates GitHub Action workflows). write:packages (for publishing code). admin:org (if you manage team repos). Delete:repo (if you want it to clean up after itself).

Click Generate token. Copy the token immediately. You cannot see it again.

Now enter the container.

docker exec -it openclaw bash

Run the command to set the GitHub token.

openclaw config set integrations.github.token YOUR_GITHUB_TOKEN

Replace YOUR_GITHUB_TOKEN with the token you just copied.

Now tell OpenClaw which repositories it is allowed to access. Use your GitHub username and repository name.

openclaw config set integrations.github.repos "YOUR_USERNAME/REPO_NAME"

You can add multiple repos separated by commas.

Exit and restart.

exit
docker restart openclaw

Step 14: Create Your Penetration Testing Agent Definition

This is where you transform OpenClaw from a general assistant into an offensive security master. You will create an AGENTS.md file. This is a structured document that defines exactly who OpenClaw is, what skills it has, and how it should behave.

Create the file in your workspace folder.

nano ~/openclaw/workspace/AGENTS.md

Paste the following content. This defines your agent as a penetration testing and offensive coding specialist.

```
# Agent: Penetration Testing Master
description: Offensive security specialist and advanced code developer
goals:
  - Identify security vulnerabilities in code and infrastructure
  - Write exploit proofs-of-concept when ethically authorized
  - Generate secure code following OWASP and NIST guidelines
  - Explain attack vectors and mitigation strategies in detail
  - Never execute attacks on systems without explicit permission from the user
skills:
  - self-improving-agent
  - proactive-agent
  - github
  - agent-browser
  - code-review
  - web-search
tools:
  - exec
  - read
  - write
  - web_search
  - github
  - browser
workflow:
  - Receive the target or code to analyze
  - Ask clarifying questions if the scope is unclear
  - Research known vulnerabilities using web_search
  - Analyze code or infrastructure for weaknesses
  - Generate a detailed report with findings
  - Write secure code or exploit examples as requested
constraints:
  - Never execute commands that modify system files without user confirmation
  - Always ask for explicit permission before scanning external targets
  - Report all findings with severity ratings
  - Include remediation steps for every vulnerability found
output:
  format: markdown
  language: en-US
  include_code_blocks: true
```

Save the file. In nano, press Ctrl+X, then Y, then Enter.

Now load this agent definition into OpenClaw. Enter the container.

docker exec -it openclaw bash

Run the command to load the agent.

openclaw agent load --file /root/clawd/AGENTS.md

You should see a confirmation that the agent was loaded.

Step 15: Verify Everything Is Working

Send these test messages to your Telegram bot to confirm each capability.

First, test basic reasoning. Send "Who are you and what is your purpose?" It should respond based on the AGENTS.md file you created.

Second, test internet search. Send "Search the web for the latest CVEs released this month." The bot will use the web_search tool to find and summarize recent vulnerabilities.

Third, test code execution. Send "Write a Python script that scans a given IP address for open ports 22, 80, and 443." The bot will generate the code. It will ask for your approval before running it. This is the confirm flag in action. Type "yes" or "approve" to let it run.

Fourth, test GitHub access. Send "List my repositories on GitHub." It will use the github tool with your token to fetch and display your repos.

Fifth, test browser automation. Send "Go to example.com and take a screenshot." It will use the agent-browser skill to launch a headless browser, navigate, and capture a screenshot.

Step 16: Security Hardening (Read This Carefully)

You are giving your AI agent the ability to run code and search the internet. This is powerful but requires safety measures.

The exec tool requires your confirmation before running any command. This is already set in your config with "confirm": true. Every time OpenClaw wants to run a terminal command, it will send you a message asking for approval. You must type "yes" or "approve" for it to proceed.

The workspace folder is the only place OpenClaw can read and write files. It cannot access your Zima board's system files or your home directory. This is enforced by the Docker volume mount. The container only sees /root/clawd which maps to your ~/openclaw/workspace folder.

Do not mount your entire home directory or system root into the container. If you ever change the docker run command, never add -v /:/mnt or -v ~/.ssh:/root/.ssh. This would give the AI access to your SSH keys.

For remote access to the web interface from outside your home network, do not expose port 18789 directly to the internet. Use a VPN or SSH tunnel instead. The web interface uses a token for authentication, but tokens can be intercepted over plain HTTP.

Step 17: Useful Commands for Daily Operation

View the live logs of your agent.

docker logs -f openclaw

Restart the agent if it becomes unresponsive.

docker restart openclaw

Enter the container to run manual commands.

docker exec -it openclaw bash

Update all installed skills to their latest versions.

docker exec -it openclaw bash -c "npx clawhub update --all"

Backup your entire OpenClaw configuration and workspace.

tar -czvf openclaw-backup.tar.gz ~/openclaw/

Step 18: Troubleshooting Common Issues

If the bot does not respond to your messages, check the logs with docker logs openclaw. Look for any lines containing error or failed. The most common issue is an invalid API key or Telegram token.

If you see "pairing required" in Telegram, you need to re-pair. Get a new pairing code from docker logs openclaw 2>&1 | grep -i "pairing code" and send it to the bot again.

If OpenClaw says "command not found" when trying to use a skill, the skill did not install correctly. Enter the container with docker exec -it openclaw bash and run npx clawhub list to see installed skills. Reinstall any missing ones.

If the web interface shows "unauthorized," your token is wrong. Check the token in your ~/openclaw/data/openclaw.json file and make sure you are using it correctly in the URL: http://YOUR_IP:18789/?token=your_token_here.

If OpenClaw refuses to execute code even after you approve, the exec tool might be disabled. Check the config file and make sure "enabled": true and "confirm": true are both set correctly.

You now have a fully functional OpenClaw agent running on your Zima Board inside Docker. It has DeepSeek as its brain, native internet search, GitHub integration, browser automation, and code execution capabilities. It is defined as a penetration testing master with specific goals and constraints.

Your agent can write exploit code, research vulnerabilities, scan networks (with your permission), manage GitHub repositories, and browse the web. All of this runs inside a container that cannot touch your host system files unless you explicitly mount them.

To make it do something specific, just ask it in Telegram. Tell it to audit a GitHub repository for security issues. Tell it to write a buffer overflow exploit for a training challenge. Tell it to research and explain the latest attack techniques. The agent will use its tools and skills to accomplish whatever you ask, within the constraints you defined in AGENTS.md.

## only use on Networks you have permission to test on
