# Python Flask from Git

Deploy a Python Flask application from a Git repository.

```bash
# Create sandbox
bdy sandbox create -i flask-app --resources 1x2 \
  --install-command "apt-get update && apt-get install -y python3 python3-pip python3-venv" \
  --wait-for-configured

# Deploy
bdy sandbox exec command flask-app "git clone https://github.com/user/flask-app.git /app" --wait
bdy sandbox exec command flask-app "cd /app && python3 -m venv venv && . venv/bin/activate && pip install -r requirements.txt" --wait
bdy sandbox exec command flask-app "cd /app && . venv/bin/activate && python3 app.py"

# Expose
bdy sandbox endpoint add flask-app -n api -e 5000
```

**Note:** Ubuntu 24.04 requires venv for pip packages (PEP 668).
