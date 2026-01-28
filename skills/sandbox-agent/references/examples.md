# Sandbox Agent Examples

## Table of Contents
- [Example 1: Simple Script Task](#example-1-simple-script-task)
- [Example 2: Test Generation](#example-2-test-generation)
- [Example 3: Multi-Agent Code Review](#example-3-multi-agent-code-review)
- [Example 4: WordPress Theme Deployment](#example-4-wordpress-theme-deployment)

---

## Example 1: Simple Script Task

**Goal:** Delegate a simple task - list files and save to file. Quick test to verify delegation works.

### Steps

```bash
# 1. Create sandbox
bdy sandbox create -i test-delegate --resources 2x4 --wait-for-configured

# 2. Delegate simple task
bdy sandbox exec command test-delegate "sudo -u claude -i -- claude --dangerously-skip-permissions -p \"List all files in /etc directory and save the output to /tmp/etc-files.txt\""

# 3. Get command ID
bdy sandbox exec list test-delegate
# Note the command ID (e.g., cmd-abc123)

# 4. Monitor progress
bdy sandbox exec status test-delegate <cmd-id>
bdy sandbox exec logs test-delegate <cmd-id>

# 5. Wait for completion
bdy sandbox exec logs test-delegate <cmd-id> --wait

# 6. Verify result file was created
bdy sandbox exec command test-delegate "cat /tmp/etc-files.txt" --wait

# 7. Cleanup
bdy sandbox destroy test-delegate
```

### Expected Result

File `/tmp/etc-files.txt` contains list of files from /etc directory.

---

## Example 2: Test Generation

**Goal:** Delegate test generation for a Node.js application. Demonstrates setup, code copy, and result verification.

### Steps

```bash
# 1. Create sandbox with Node.js
bdy sandbox create -i test-gen --resources 4x8 \
  --install-command "curl -fsSL https://deb.nodesource.com/setup_20.x | bash - && apt-get install -y nodejs" \
  --wait-for-configured

# 2. Create simple test app locally
mkdir -p /tmp/test-app
cat > /tmp/test-app/math.js << 'EOF'
function add(a, b) { return a + b; }
function subtract(a, b) { return a - b; }
function multiply(a, b) { return a * b; }
function divide(a, b) {
  if (b === 0) throw new Error('Division by zero');
  return a / b;
}
module.exports = { add, subtract, multiply, divide };
EOF

cat > /tmp/test-app/package.json << 'EOF'
{"name": "test-app", "version": "1.0.0"}
EOF

# 3. Copy to sandbox
bdy sandbox cp /tmp/test-app test-gen:/app > /dev/null 2>&1

# 4. Delegate test generation
bdy sandbox exec command test-gen "sudo -u claude -i -- claude --dangerously-skip-permissions -p \"Analyze /app/math.js and create comprehensive unit tests using Jest. Install Jest as dev dependency, write tests to /app/math.test.js covering all functions including edge cases, run the tests and ensure they pass.\""

# 5. Get command ID and monitor
bdy sandbox exec list test-gen
bdy sandbox exec logs test-gen <cmd-id> --wait

# 6. Verify tests exist
bdy sandbox exec command test-gen "cat /app/math.test.js" --wait

# 7. Verify tests pass
bdy sandbox exec command test-gen "cd /app && npx jest" --wait

# 8. (Optional) Copy tests back to local
bdy sandbox cp test-gen:/app/math.test.js ./math.test.js > /dev/null 2>&1

# 9. Cleanup
bdy sandbox destroy test-gen
```

### Expected Result

- Tests in `/app/math.test.js` covering add, subtract, multiply, divide
- Edge case test for division by zero
- All tests pass when running `npx jest`

---

## Example 3: Multi-Agent Code Review

**Goal:** Get comprehensive code review from multiple perspectives. Each agent focuses on different aspect, main agent aggregates and prioritizes findings.

### Sample Code with Intentional Issues

```bash
# Create sample code to review (locally)
mkdir -p /tmp/review-app
cat > /tmp/review-app/api.js << 'EOF'
const express = require('express');
const app = express();

// User endpoint - has SQL injection vulnerability
app.get('/user', (req, res) => {
  const userId = req.query.id;
  const query = `SELECT * FROM users WHERE id = ${userId}`;
  db.query(query, (err, result) => {
    res.json(result);
  });
});

// Login endpoint - has hardcoded credentials
app.post('/login', (req, res) => {
  const { username, password } = req.body;
  if (username === 'admin' && password === 'admin123') {
    res.json({ token: 'secret-token' });
  }
});

// Data processing - blocking operation
app.get('/process', (req, res) => {
  const data = [];
  for (let i = 0; i < 10000000; i++) {
    data.push(Math.random());
  }
  res.json({ count: data.length });
});

app.listen(3000);
EOF
```

### Steps

```bash
# 1. Create 3 sandboxes in parallel
bdy sandbox create -i reviewer-security --resources 2x4 --wait-for-configured &
bdy sandbox create -i reviewer-perf --resources 2x4 --wait-for-configured &
bdy sandbox create -i reviewer-quality --resources 2x4 --wait-for-configured &
wait

# 2. Copy code to all sandboxes
bdy sandbox cp /tmp/review-app reviewer-security:/app > /dev/null 2>&1 &
bdy sandbox cp /tmp/review-app reviewer-perf:/app > /dev/null 2>&1 &
bdy sandbox cp /tmp/review-app reviewer-quality:/app > /dev/null 2>&1 &
wait

# 3. Delegate security review
bdy sandbox exec command reviewer-security "sudo -u claude -i -- claude --dangerously-skip-permissions -p \"Review /app/api.js for SECURITY issues only. Look for: SQL injection, XSS, authentication problems, hardcoded secrets, input validation, OWASP top 10. Write findings to /app/security-review.md with severity ratings (CRITICAL/HIGH/MEDIUM/LOW) and remediation suggestions.\""

# 4. Delegate performance review
bdy sandbox exec command reviewer-perf "sudo -u claude -i -- claude --dangerously-skip-permissions -p \"Review /app/api.js for PERFORMANCE issues only. Look for: blocking operations, memory leaks, inefficient algorithms, missing caching, connection pooling, async patterns. Write findings to /app/perf-review.md with severity ratings and optimization suggestions.\""

# 5. Delegate code quality review
bdy sandbox exec command reviewer-quality "sudo -u claude -i -- claude --dangerously-skip-permissions -p \"Review /app/api.js for CODE QUALITY issues only. Look for: error handling, code structure, naming conventions, missing validation, logging, best practices, maintainability. Write findings to /app/quality-review.md with severity ratings and improvement suggestions.\""

# 6. Get command IDs
bdy sandbox exec list reviewer-security
bdy sandbox exec list reviewer-perf
bdy sandbox exec list reviewer-quality

# 7. Wait for all reviews to complete
bdy sandbox exec logs reviewer-security <cmd-id> --wait
bdy sandbox exec logs reviewer-perf <cmd-id> --wait
bdy sandbox exec logs reviewer-quality <cmd-id> --wait

# 8. Collect all reviews
echo "=== SECURITY REVIEW ==="
bdy sandbox exec command reviewer-security "cat /app/security-review.md" --wait

echo "=== PERFORMANCE REVIEW ==="
bdy sandbox exec command reviewer-perf "cat /app/perf-review.md" --wait

echo "=== CODE QUALITY REVIEW ==="
bdy sandbox exec command reviewer-quality "cat /app/quality-review.md" --wait

# 9. Cleanup
bdy sandbox destroy reviewer-security
bdy sandbox destroy reviewer-perf
bdy sandbox destroy reviewer-quality
```

### Expected Results

**Security Review should find:**
- SQL injection vulnerability (CRITICAL) - line with `SELECT * FROM users WHERE id = ${userId}`
- Hardcoded credentials (CRITICAL) - `admin` / `admin123`
- Hardcoded token (HIGH) - `secret-token`
- Missing input validation (MEDIUM)

**Performance Review should find:**
- Blocking synchronous loop (HIGH) - 10M iterations blocking event loop
- Missing async patterns (MEDIUM)
- No connection pooling mentioned (MEDIUM)

**Code Quality Review should find:**
- Missing error handling (HIGH) - no try/catch, no error responses
- Missing input validation (MEDIUM)
- No logging (LOW)
- Missing middleware for body parsing (MEDIUM)

### Main Agent Aggregation

After collecting all reviews, main agent should:
1. Combine all findings into single list
2. De-duplicate overlapping issues
3. Sort by severity (CRITICAL first)
4. Present prioritized report to user with recommended fix order

**Priority order for this example:**
1. SQL Injection (CRITICAL) - immediate fix required
2. Hardcoded credentials (CRITICAL) - immediate fix required
3. Blocking loop (HIGH) - affects all users
4. Missing error handling (HIGH) - can crash app
5. Other MEDIUM/LOW issues

---

## Example 4: WordPress Theme Deployment

**Goal:** Delegate WordPress theme creation and deployment to a sub-agent. The agent will set up WordPress, create a custom theme, and deploy it.

### Important: Prompt Engineering for WordPress Tasks

When delegating WordPress tasks, **include explicit WP-CLI instructions in the prompt**. Sub-agents often struggle with WordPress because they don't know:
- How to install WP-CLI
- That `--allow-root` is required (sandbox runs as root)
- The correct WordPress path

### Prompt Template for WordPress Theme Tasks

Include this context in your delegation prompt:

```
WORDPRESS SETUP CONTEXT:
- WordPress is at: /var/www/html/wordpress
- Database: wordpress, User: wpuser, Password: wppass123
- Public URL: https://wordpress-<sandbox>-<project>-<workspace>.us-1.buddy.app

WP-CLI INSTRUCTIONS (CRITICAL):
1. Install WP-CLI:
   curl -O https://raw.githubusercontent.com/wp-cli/builds/gh-pages/phar/wp-cli.phar
   chmod +x wp-cli.phar && mv wp-cli.phar /usr/local/bin/wp

2. All wp commands require --allow-root flag (running as root)

3. Complete WP installation:
   cd /var/www/html/wordpress
   wp core install --url='<PUBLIC_URL>' --title='Site' --admin_user=admin --admin_password=admin123 --admin_email=admin@example.com --allow-root

4. Activate theme:
   wp theme activate <theme-name> --allow-root

5. Create sample content:
   wp post create --post_title='Hello' --post_content='Content' --post_status=publish --allow-root
```

### Steps

```bash
# 1. Create sandbox with LAMP stack + curl (required for WP-CLI download)
bdy sandbox create -i wp-agent --resources 4x8 \
  --install-command "apt-get update && DEBIAN_FRONTEND=noninteractive apt-get install -y apache2 mariadb-server php php-mysql php-curl php-gd php-mbstring php-xml php-xmlrpc php-soap php-intl php-zip libapache2-mod-php unzip wget curl" \
  --wait-for-configured

# 2. Setup database
bdy sandbox exec command wp-agent "service mariadb start" --wait
bdy sandbox exec command wp-agent "mysql -e \"CREATE DATABASE wordpress; CREATE USER 'wpuser'@'localhost' IDENTIFIED BY 'wppass123'; GRANT ALL PRIVILEGES ON wordpress.* TO 'wpuser'@'localhost'; FLUSH PRIVILEGES;\"" --wait

# 3. Download WordPress
bdy sandbox exec command wp-agent "cd /var/www/html && wget -q https://wordpress.org/latest.tar.gz && tar -xzf latest.tar.gz && rm latest.tar.gz && chown -R www-data:www-data wordpress" --wait

# 4. Configure wp-config.php (create locally, copy to sandbox)
cat > /tmp/wp-config.php << 'EOF'
<?php
define('DB_NAME', 'wordpress');
define('DB_USER', 'wpuser');
define('DB_PASSWORD', 'wppass123');
define('DB_HOST', 'localhost');
define('DB_CHARSET', 'utf8mb4');
define('DB_COLLATE', '');
define('AUTH_KEY',         'unique-1');
define('SECURE_AUTH_KEY',  'unique-2');
define('LOGGED_IN_KEY',    'unique-3');
define('NONCE_KEY',        'unique-4');
define('AUTH_SALT',        'unique-5');
define('SECURE_AUTH_SALT', 'unique-6');
define('LOGGED_IN_SALT',   'unique-7');
define('NONCE_SALT',       'unique-8');
$table_prefix = 'wp_';
define('WP_DEBUG', false);
define('WP_HOME', 'https://wordpress-wp-agent-<PROJECT>-<WORKSPACE>.us-1.buddy.app');
define('WP_SITEURL', 'https://wordpress-wp-agent-<PROJECT>-<WORKSPACE>.us-1.buddy.app');
define('FORCE_SSL_ADMIN', true);
if (isset($_SERVER['HTTP_X_FORWARDED_PROTO']) && $_SERVER['HTTP_X_FORWARDED_PROTO'] === 'https') {
    $_SERVER['HTTPS'] = 'on';
}
if (!defined('ABSPATH')) {
    define('ABSPATH', __DIR__ . '/');
}
require_once ABSPATH . 'wp-settings.php';
EOF
bdy sandbox cp /tmp/wp-config.php wp-agent:/var/www/html/wordpress/ > /dev/null 2>&1

# 5. Configure Apache
cat > /tmp/wordpress.conf << 'EOF'
<VirtualHost *:80>
    DocumentRoot /var/www/html/wordpress
    <Directory /var/www/html/wordpress>
        AllowOverride All
        Require all granted
    </Directory>
</VirtualHost>
EOF
bdy sandbox cp /tmp/wordpress.conf wp-agent:/etc/apache2/sites-available/ > /dev/null 2>&1
bdy sandbox exec command wp-agent "rm -f /var/www/html/index.html && a2dissite 000-default && a2ensite wordpress && a2enmod rewrite && service apache2 start" --wait

# 6. Expose endpoint
bdy sandbox endpoint add wp-agent -n wordpress -e 80
# Note the URL - use it in wp-config.php and delegation prompt

# 7. Delegate theme creation to Claude sub-agent
bdy sandbox exec command wp-agent "sudo -u claude -i -- claude --dangerously-skip-permissions -p \"
Create a unique WordPress theme called 'agency-starter' with these requirements:
- Modern, minimal design with hero section
- Blue (#2563eb) primary color scheme
- Mobile responsive
- Custom header, footer, index.php, single.php, page.php

WORDPRESS SETUP CONTEXT:
- WordPress is at: /var/www/html/wordpress
- Database credentials: wordpress/wpuser/wppass123
- Public URL: https://wordpress-wp-agent-<PROJECT>-<WORKSPACE>.us-1.buddy.app

WP-CLI INSTRUCTIONS (CRITICAL - follow exactly):
1. First install WP-CLI:
   curl -O https://raw.githubusercontent.com/wp-cli/builds/gh-pages/phar/wp-cli.phar
   chmod +x wp-cli.phar && mv wp-cli.phar /usr/local/bin/wp

2. Complete WordPress installation:
   cd /var/www/html/wordpress
   wp core install --url='https://wordpress-wp-agent-<PROJECT>-<WORKSPACE>.us-1.buddy.app' --title='Agency Site' --admin_user=admin --admin_password=admin123 --admin_email=admin@example.com --allow-root

3. Create theme files in: /var/www/html/wordpress/wp-content/themes/agency-starter/
   Required files: style.css (with theme header), functions.php, index.php, header.php, footer.php, single.php, page.php

4. Set permissions:
   chown -R www-data:www-data /var/www/html/wordpress/wp-content/themes/agency-starter

5. Activate theme:
   cd /var/www/html/wordpress && wp theme activate agency-starter --allow-root

6. Create sample content:
   wp post create --post_title='Welcome to Agency' --post_content='Our services include...' --post_status=publish --allow-root
   wp post create --post_type=page --post_title='About Us' --post_content='About page content' --post_status=publish --allow-root

7. Verify theme is active:
   wp theme status --allow-root

Report what you created and any issues encountered.
\""

# 8. Monitor progress
bdy sandbox exec list wp-agent
bdy sandbox exec logs wp-agent <cmd-id> --wait

# 9. Verify result
bdy sandbox exec command wp-agent "cd /var/www/html/wordpress && wp theme status --allow-root" --wait
bdy sandbox exec command wp-agent "ls -la /var/www/html/wordpress/wp-content/themes/agency-starter/" --wait

# 10. Copy theme back locally (optional)
bdy sandbox cp wp-agent:/var/www/html/wordpress/wp-content/themes/agency-starter ./ > /dev/null 2>&1

# 11. Cleanup
bdy sandbox destroy wp-agent
```

### Expected Result

- Custom theme "agency-starter" created with all required files
- Theme activated and visible at WordPress URL
- Sample posts and pages created
- Theme can be copied back for reuse

### Common Issues and Solutions

**Issue: "wp: command not found"**
- WP-CLI not installed. Include installation step in prompt.

**Issue: "Error: YIKES! It looks like you're running this as root"**
- Missing `--allow-root` flag. Include in prompt explicitly.

**Issue: "Error: This does not seem to be a WordPress install"**
- Wrong directory. Always use `cd /var/www/html/wordpress` before wp commands.

**Issue: "curl: command not found"**
- curl not in install-command. Add `curl` to the apt-get install list.

**Issue: Apache shows default page instead of WordPress**
- Default index.html not removed. Include `rm -f /var/www/html/index.html` in setup.

**Issue: Theme created but not visible**
- Permissions wrong. Include `chown -R www-data:www-data` after creating theme files.
