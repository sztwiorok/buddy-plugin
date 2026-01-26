# phpMyAdmin

Add phpMyAdmin to an existing WordPress sandbox for database management.

## Prerequisites

- Existing sandbox with MariaDB (e.g., from [wordpress.md](wordpress.md))

## Step 1: Download phpMyAdmin

```bash
bdy sandbox exec command wordpress "cd /var/www/html && wget -q https://files.phpmyadmin.net/phpMyAdmin/5.2.1/phpMyAdmin-5.2.1-all-languages.tar.gz && tar -xzf phpMyAdmin-5.2.1-all-languages.tar.gz && mv phpMyAdmin-5.2.1-all-languages phpmyadmin && rm phpMyAdmin-5.2.1-all-languages.tar.gz && chown -R www-data:www-data phpmyadmin" --wait
```

## Step 2: Configure phpMyAdmin

```bash
cat > /tmp/phpmyadmin-config.php << 'EOF'
<?php
$cfg['blowfish_secret'] = 'abcdefghijklmnopqrstuvwxyz123456';
$i = 0;
$i++;
$cfg['Servers'][$i]['host'] = 'localhost';
$cfg['Servers'][$i]['compress'] = false;
$cfg['Servers'][$i]['AllowNoPassword'] = false;
EOF

bdy sandbox cp --silent /tmp/phpmyadmin-config.php wordpress:/var/www/html/phpmyadmin/config.inc.php
```

## Step 3: Configure Apache (port 8080)

```bash
cat > /tmp/phpmyadmin.conf << 'EOF'
Listen 8080
<VirtualHost *:8080>
    DocumentRoot /var/www/html/phpmyadmin
    <Directory /var/www/html/phpmyadmin>
        AllowOverride All
        Require all granted
    </Directory>
</VirtualHost>
EOF

bdy sandbox cp --silent /tmp/phpmyadmin.conf wordpress:/etc/apache2/sites-available/phpmyadmin.conf

bdy sandbox exec command wordpress "a2ensite phpmyadmin && service apache2 restart" --wait
```

## Step 4: Expose Endpoint

```bash
bdy sandbox endpoint add wordpress -n phpmyadmin -e 8080
```

## Credentials

| Username | Password |
|----------|----------|
| wpuser | wppass123 |
| root | (no password) |
