# Building a Highly Portable, Cross-Play Minecraft Server with Docker

This guide will walk you through setting up a modern Minecraft server. By the end of this tutorial, you will have a server that:
* **Runs in Docker:** Making it incredibly easy to start, stop, and move between computers.
* **Supports Cross-Play:** Both Java Edition (PC) and Bedrock Edition (Console, Mobile, Windows) players can play together on the same server.
* **Is Version Flexible:** Bedrock players are forced to use the latest version of Minecraft, but this server uses plugins to translate network packets so they can join even if the core server hasn't been updated yet.
* **Is Git Ready:** We use a specific `.gitignore` configuration so you can back up your server's configurations to Git without uploading gigabytes of heavy, constantly changing world data.

### Step 1: Create your project folder
Create a new directory for your Minecraft server and navigate into it:
```bash
mkdir Minecraft
cd Minecraft
```

### Step 2: The Docker Compose File
We will use the `itzg/minecraft-server` Docker image to handle the heavy lifting. Create a file named `docker-compose.yaml` in your folder and paste the following configuration:

```yaml
version: "3.8"

services:
  mc-crossplay:
    image: itzg/minecraft-server:latest
    container_name: mc-crossplay
    environment:
      EULA: "TRUE"
      TYPE: "PAPER"
      # Pin to your exact Java client version to avoid mismatches (e.g., "1.21.4")
      VERSION: "1.21.4"
      # Increase memory allocation; default 1GB is usually insufficient for proxy plugins
      MEMORY: "4G"
      # Disabling secure profiles is required for Bedrock players to join via Floodgate
      ENFORCE_SECURE_PROFILE: "false" 
      # Automatically downloads plugins to allow newer clients to join an older server
      MODRINTH_PROJECTS: "viaversion,viabackwards"
      # Automatically downloads the latest Geyser (Bedrock bridge) and Floodgate (Authentication) plugins
      PLUGINS: |
        https://download.geysermc.org/v2/projects/geyser/versions/latest/builds/latest/downloads/spigot
        https://download.geysermc.org/v2/projects/floodgate/versions/latest/builds/latest/downloads/spigot
    ports:
      - "25565:25565/tcp"  # Java Edition Port
      - "19132:19132/udp"  # Bedrock Edition Port
    volumes:
      - ./server-data:/data
    restart: unless-stopped
```

### Step 3: Secure Server Management (RCON)
RCON allows you to send administrative commands to your server remotely. We need to set a secure password for this, but we **do not** want to put it directly in our configuration files where it might accidentally get uploaded to GitHub.

Create a folder called `server-data`, and inside it, create a file named `.rcon-cli.env`:
```bash
mkdir server-data
touch server-data/.rcon-cli.env
```

Open `server-data/.rcon-cli.env` and add your secure password:
```env
password=your_super_secret_password_here
```

### Step 4: Configure Git for Easy Backups
We want to back up our server settings using Git, but Minecraft generates huge files (like world saves and `.jar` files) that don't belong in version control. Furthermore, we must ensure our `.rcon-cli.env` password file is never tracked!

In the root of your `Minecraft` folder, create a `.gitignore` file and paste the following:

```gitignore
# Ignore environment variables (Secures your RCON password!)
.env
server-data/*.env

# Ignore generated world data
server-data/world/
server-data/world_nether/
server-data/world_the_end/

# Ignore logs and crash reports
server-data/logs/
server-data/crash-reports/

# Ignore server binaries and cache
server-data/*.jar
server-data/cache/
server-data/libraries/
server-data/versions/

# Ignore temporary/runtime plugin data and databases
server-data/plugins/spark/tmp/
server-data/plugins/spark/tmp-client/
*.db
*.sqlite
*.sqlite-journal
```

### Step 5: Start the Server
Now that everything is securely configured, you can launch the server. Run the following command in your terminal:

```bash
docker-compose up -d
```

**What happens now?**
1. Docker will download the Java environment and PaperMC server files.
2. It will dynamically download Geyser, Floodgate, ViaVersion, and ViaBackwards.
3. It will generate your world, apply your memory limits, and open the ports for both Java and Bedrock players.

### Step 6: Connecting from a Nintendo Switch (or other Consoles)
By default, console versions of Minecraft do not allow you to add custom server IPs. To join your new server from a Nintendo Switch, you will need to use a DNS workaround (like BedrockConnect):

1. On your Switch, go to **System Settings** -> **Internet** -> **Internet Settings**.
2. Select your current Wi-Fi network and choose **Change Settings**.
3. Scroll down to **DNS Settings** and change it from **Automatic** to **Manual**.
4. Set the **Primary DNS** to `104.238.130.180`.
5. Set the **Secondary DNS** to `8.8.8.8` (Google) or `1.1.1.1` (Cloudflare).
6. Save the settings and connect to the network.
7. Open Minecraft and go to the **Servers** tab.
8. Try to join any of the pre-approved "Featured Servers" (like *The Hive* or *Mineville*).
9. Instead of joining that server, a custom menu will appear! From here, choose **Connect to a Server**, and enter your server's IP address and the Bedrock port (`19132`).

*Note: When you are done playing on your custom server, you may want to revert your DNS Settings back to Automatic to ensure normal internet functionality for other games.*
