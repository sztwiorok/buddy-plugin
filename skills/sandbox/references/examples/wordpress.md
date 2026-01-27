# WordPress Deployment

Deploy WordPress on Buddy Sandbox with WP-CLI for automation.

## Quick Reference

| Item | Value |
|------|-------|
| WordPress path | `/var/www/html/wordpress` |
| Database | `wordpress` / `wpuser` / `wppass123` |
| WP-CLI flag | `--allow-root` (required) |

## Step 1: Create Sandbox

```bash
bdy sandbox create -i wordpress --resources 4x8 \
  --install-command "apt-get update && DEBIAN_FRONTEND=noninteractive apt-get install -y apache2 mariadb-server php php-mysql php-curl php-gd php-mbstring php-xml php-xmlrpc php-soap php-intl php-zip libapache2-mod-php wget curl" \
  --wait-for-configured
```

## Step 2: Setup Database

```bash
bdy sandbox exec command wordpress "service mariadb start" --wait

bdy sandbox exec command wordpress "mysql -e \"CREATE DATABASE wordpress; CREATE USER 'wpuser'@'localhost' IDENTIFIED BY 'wppass123'; GRANT ALL PRIVILEGES ON wordpress.* TO 'wpuser'@'localhost'; FLUSH PRIVILEGES;\"" --wait
```

## Step 3: Download WordPress & Install WP-CLI

```bash
bdy sandbox exec command wordpress "cd /var/www/html && wget -q https://wordpress.org/latest.tar.gz && tar -xzf latest.tar.gz && rm latest.tar.gz && chown -R www-data:www-data wordpress" --wait

bdy sandbox exec command wordpress "curl -sO https://raw.githubusercontent.com/wp-cli/builds/gh-pages/phar/wp-cli.phar && chmod +x wp-cli.phar && mv wp-cli.phar /usr/local/bin/wp" --wait
```

## Step 4: Configure Apache

```bash
cat > /tmp/wordpress.conf << 'EOF'
<VirtualHost *:80>
    DocumentRoot /var/www/html/wordpress
    <Directory /var/www/html/wordpress>
        AllowOverride All
        Require all granted
    </Directory>
</VirtualHost>
EOF

bdy sandbox cp /tmp/wordpress.conf wordpress:/etc/apache2/sites-available/ > /dev/null 2>&1

bdy sandbox exec command wordpress "rm -f /var/www/html/index.html && a2dissite 000-default && a2ensite wordpress && a2enmod rewrite && service apache2 restart" --wait
```

## Step 5: Expose Endpoint

```bash
bdy sandbox endpoint add wordpress -n wordpress -e 80
# Output example: https://wordpress-wordpress-1-skills-tests-myplayground.us-1.buddy.app
# Save this URL for the next steps!
```

## Step 6: Configure wp-config.php

**Use the actual URL from Step 5** in WP_HOME and WP_SITEURL:

```bash
cat > /tmp/wp-config.php << 'EOF'
<?php
define('DB_NAME', 'wordpress');
define('DB_USER', 'wpuser');
define('DB_PASSWORD', 'wppass123');
define('DB_HOST', 'localhost');
define('DB_CHARSET', 'utf8mb4');
define('DB_COLLATE', '');

define('AUTH_KEY',         'put-unique-phrase-here-1');
define('SECURE_AUTH_KEY',  'put-unique-phrase-here-2');
define('LOGGED_IN_KEY',    'put-unique-phrase-here-3');
define('NONCE_KEY',        'put-unique-phrase-here-4');
define('AUTH_SALT',        'put-unique-phrase-here-5');
define('SECURE_AUTH_SALT', 'put-unique-phrase-here-6');
define('LOGGED_IN_SALT',   'put-unique-phrase-here-7');
define('NONCE_SALT',       'put-unique-phrase-here-8');

$table_prefix = 'wp_';
define('WP_DEBUG', false);

// Use the actual URL from Step 5!
define('WP_HOME', 'https://wordpress-wordpress-1-skills-tests-myplayground.us-1.buddy.app');
define('WP_SITEURL', 'https://wordpress-wordpress-1-skills-tests-myplayground.us-1.buddy.app');
define('FORCE_SSL_ADMIN', true);

if (isset($_SERVER['HTTP_X_FORWARDED_PROTO']) && $_SERVER['HTTP_X_FORWARDED_PROTO'] === 'https') {
    $_SERVER['HTTPS'] = 'on';
}

if (!defined('ABSPATH')) {
    define('ABSPATH', __DIR__ . '/');
}
require_once ABSPATH . 'wp-settings.php';
EOF

bdy sandbox cp /tmp/wp-config.php wordpress:/var/www/html/wordpress/ > /dev/null 2>&1
```

## Step 7: Complete Installation

Use the same URL from Step 5:

```bash
bdy sandbox exec command wordpress "cd /var/www/html/wordpress && wp core install \
  --url='https://wordpress-wordpress-1-skills-tests-myplayground.us-1.buddy.app' \
  --title='My Site' \
  --admin_user=admin \
  --admin_password=admin123 \
  --admin_email=admin@example.com \
  --allow-root" --wait
```

Done! Visit the URL to see your WordPress site.

---

## Deploy Custom Theme (Optional)

```bash
# Copy theme to sandbox
bdy sandbox cp /path/to/my-theme wordpress:/var/www/html/wordpress/wp-content/themes/ > /dev/null 2>&1

# Fix permissions
bdy sandbox exec command wordpress "chown -R www-data:www-data /var/www/html/wordpress/wp-content/themes/my-theme" --wait

# Activate theme
bdy sandbox exec command wordpress "cd /var/www/html/wordpress && wp theme activate my-theme --allow-root" --wait
```

## Create Sample Content (Optional)

```bash
bdy sandbox exec command wordpress "cd /var/www/html/wordpress && wp post create \
  --post_title='Welcome' \
  --post_content='Hello from WordPress!' \
  --post_status=publish \
  --allow-root" --wait
```

---

## WP-CLI Essentials

All WP-CLI commands require `--allow-root` because sandbox runs as root.

```bash
# List themes
wp theme list --allow-root

# Install theme from WordPress.org
wp theme install flavor --activate --allow-root

# Install plugin
wp plugin install woocommerce --activate --allow-root
```

---

## Troubleshooting

### WordPress redirects to localhost

WP_HOME/WP_SITEURL not set correctly. Ensure wp-config.php has:
- The actual URL from `bdy sandbox endpoint add` output
- These lines BEFORE `require_once ABSPATH . 'wp-settings.php'`

### Services not running after restart

```bash
bdy sandbox exec command wordpress "service mariadb start && service apache2 start" --wait
```
