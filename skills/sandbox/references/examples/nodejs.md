# Node.js from Git

Deploy a Node.js application from a Git repository.

```bash
# Create sandbox with Node.js
bdy sandbox create -i nodejs-app --resources 2x4 \
  --install-command "curl -fsSL https://deb.nodesource.com/setup_20.x | bash -" \
  --install-command "apt-get install -y nodejs" \
  --wait-for-configured

# Clone and setup
bdy sandbox exec command nodejs-app "git clone https://github.com/user/app.git /app" --wait
bdy sandbox exec command nodejs-app "cd /app && npm install" --wait

# Start server (runs in background by default)
bdy sandbox exec command nodejs-app "cd /app && npm start"

# Expose port 3000
bdy sandbox endpoint add nodejs-app -n web -e 3000
```
