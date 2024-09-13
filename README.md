# Automated-Apache-Web-Server-Setup-Secure-Access-Monitoring
Automated Apache Web Server Setup: Secure Access &amp; Monitoring

#!/bin/bash

# Variables
APACHE_CONFIG="/etc/httpd/conf/httpd.conf"   # Path to Apache main config file
APACHE_STATUS_CONF="/etc/httpd/conf.d/status.conf"  # Path to status config file
HTPASSWD_FILE="/etc/httpd/.htpasswd"  # Path to store the .htpasswd file
WEB_ROOT="/var/www/html"  # Web root directory

# Update and install Apache
echo "Installing Apache..."
yum update -y
yum install httpd -y

# Start and enable Apache service
echo "Starting and enabling Apache..."
systemctl start httpd
systemctl enable httpd

# Create the web root directory if it doesn't exist
if [ ! -d "$WEB_ROOT" ]; then
  mkdir -p "$WEB_ROOT"
fi

# Create a basic HTML file
echo "<h1>Welcome to the Apache Web Server</h1>" > "$WEB_ROOT/index.html"

# Configuring Apache to use Basic Authentication
echo "Setting up basic authentication..."
htpasswd -cb $HTPASSWD_FILE admin password123  # Change 'admin' and 'password123' as needed

# Add basic authentication config to Apache config
cat <<EOL >> $APACHE_CONFIG

<Directory "$WEB_ROOT">
    AuthType Basic
    AuthName "Restricted Access"
    AuthUserFile $HTPASSWD_FILE
    Require valid-user
</Directory>

EOL

# Enable mod_status for monitoring
echo "Enabling mod_status for monitoring..."
cat <<EOL > $APACHE_STATUS_CONF
<Location "/server-status">
    SetHandler server-status
    Require ip 127.0.0.1
    Require ip ::1
</Location>

ExtendedStatus On
EOL

# Restart Apache to apply changes
echo "Restarting Apache to apply changes..."
systemctl restart httpd

echo "Apache setup completed with basic authentication and monitoring enabled."
