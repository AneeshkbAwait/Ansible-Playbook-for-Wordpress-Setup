This Playbook will fulfil Wordpress installation and environment setup for the given domain on an Amazon Linux-based Operating System.

Ansible Modules used:
1. yum
2. template
3. file
4. service
5. copy
6. shell
7. musql_user
8. mysql_db
9. unarchive
10. get_url

Here, we are using Ansible templates to setup the virtualhost and wordpress wp-config.php configurations.

The Virtualhost configuration is built using the template file virualhost.conf.tmpl and the Wordpress configuration setup is done by using the wp-config.php.tmpl template file.

The 2 Template files is as follows;

virualhost.conf.tmpl
```
<virtualhost *:{{ httpd_port }}>
    
    servername {{ domain_name }}
    documentroot /var/www/html/{{ domain_name }}
    directoryindex index.php index.html
    <directory /var/www/html/{{ domain_name }}>
       allowoverride all
    </directory>
    
</virtualhost>
```

wp-config.php.tmpl
```
<?php
/**
 * The base configuration for WordPress
 *
 * The wp-config.php creation script uses this file during the installation.
 * You don't have to use the web site, you can copy this file to "wp-config.php"
 * and fill in the values.
 *
 * This file contains the following configurations:
 *
 * * Database settings
 * * Secret keys
 * * Database table prefix
 * * ABSPATH
 *
 * @link https://wordpress.org/support/article/editing-wp-config-php/
 *
 * @package WordPress
 */

// ** Database settings - You can get this info from your web host ** //
/** The name of the database for WordPress */
define( 'DB_NAME', '{{ mysql_extra_database }}' );

/** Database username */
define( 'DB_USER', '{{ mysql_extra_user }}' );

/** Database password */
define( 'DB_PASSWORD', '{{ mysql_extra_password }}' );

/** Database hostname */
define( 'DB_HOST', 'localhost' );

/** Database charset to use in creating database tables. */
define( 'DB_CHARSET', 'utf8' );

/** The database collate type. Don't change this if in doubt. */
define( 'DB_COLLATE', '' );

/**#@+
 * Authentication unique keys and salts.
 *
 * Change these to different unique phrases! You can generate these using
 * the {@link https://api.wordpress.org/secret-key/1.1/salt/ WordPress.org secret-key service}.
 *
 * You can change these at any point in time to invalidate all existing cookies.
 * This will force all users to have to log in again.
 *
 * @since 2.6.0
 */
define( 'AUTH_KEY',         'put your unique phrase here' );
define( 'SECURE_AUTH_KEY',  'put your unique phrase here' );
define( 'LOGGED_IN_KEY',    'put your unique phrase here' );
define( 'NONCE_KEY',        'put your unique phrase here' );
define( 'AUTH_SALT',        'put your unique phrase here' );
define( 'SECURE_AUTH_SALT', 'put your unique phrase here' );
define( 'LOGGED_IN_SALT',   'put your unique phrase here' );
define( 'NONCE_SALT',       'put your unique phrase here' );

/**#@-*/

/**
 * WordPress database table prefix.
 *
 * You can have multiple installations in one database if you give each
 * a unique prefix. Only numbers, letters, and underscores please!
 */
$table_prefix = 'wp_';

/**
 * For developers: WordPress debugging mode.
 *
 * Change this to true to enable the display of notices during development.
 * It is strongly recommended that plugin and theme developers use WP_DEBUG
 * in their development environments.
 *
 * For information on other constants that can be used for debugging,
 * visit the documentation.
 *
 * @link https://wordpress.org/support/article/debugging-in-wordpress/
 */
define( 'WP_DEBUG', false );

/* Add any custom values between this line and the "stop editing" line. */

/* That's all, stop editing! Happy publishing. */

/** Absolute path to the WordPress directory. */
if ( ! defined( 'ABSPATH' ) ) {
	define( 'ABSPATH', __DIR__ . '/' );
}

/** Sets up WordPress vars and included files. */
require_once ABSPATH . 'wp-settings.php';
```

As is required by every WordPress installation, the Playbook is divided into three major sections;

1. Apache Installation & Setup.
2. Mariadb Server Installatin & Setup.
3. Wordpress Installation & Setup.
    
```
---
- name: "Installing & configuring mariadb-server"
  hosts: amazon
  become: true
  vars_files:
    - variables.vars
        
    wp_url: "https://wordpress.org/wordpress-5.8.4.tar.gz"
        
  tasks:
    
    - name: "Apache - package Installation"
      yum:
        name: httpd
        state: present
            
    - name: "Apache - Crating VirtualHost /etc/httpd/conf.d/default.conf"
      template:
        src: virualhost.conf.tmpl
        dest: /etc/httpd/conf.d/default.conf
        owner: "{{ httpd_user }}"
        group: "{{ httpd_group }}"
            
            
    - name: "Apache - Creating DocumentRoot /var/www/html/{{ domain_name }}"
      file:
        path: "/var/www/html/{{ domain_name }}"
        state: directory
        owner: "{{ httpd_user }}"
        group: "{{ httpd_group }}"
    
    - name: "Apache - Installing php support"
      shell: amazon-linux-extras install php7.4 -y
    
    - name: "Apache - Restarting/Enabling php-fpm"
      service:
        name: php-fpm
        state: restarted
        enabled: true
    
    - name: "Apache - Restarting/Enabling Service"
      service:
        name: httpd
        state: restarted
        enabled: true
            
    - name: "Mariadb-Server - Package Installation"
      yum:
        name: 
          - mariadb-server
          - MySQL-python
        state: present
            
    - name: "Mariadb-Server - Restarting/Enabling Service"
      service:
        name: mariadb
        state: restarted
        enabled: true
            
    - name: "Mariadb-Server - Setting Root Passsword"
      ignore_errors: true
      mysql_user:
        login_user: "root"
        login_password: ""
        name: "root"
        password: "{{ mysql_root_password }}"
        host_all: true
            
    - name: "Mariadb-Server - Removing Anonymous Users"
      mysql_user:
        login_user: "root"
        login_password: "{{ mysql_root_password }}"
        name: ""
        state: absent
        host_all: true
            
    - name: "Mariadb-Server - Creating Database {{ mysql_extra_database }}"
      mysql_db:
        login_user: "root"
        login_password: "{{ mysql_root_password }}"
        name: "{{ mysql_extra_database }}"
        state: present
            
    - name: "mariadb-Server - Creating User {{ mysql_extra_user }}"
      mysql_user:
        login_user: "root"
        login_password: "{{ mysql_root_password }}"
        name: "{{ mysql_extra_user }}"
        host: "%"
        password: "{{ mysql_extra_password }}"
        priv: "{{ mysql_extra_database }}.*:ALL"
            
    - name: "Wordpress - Downloading Archive File {{ wp_url }}"
      get_url:
        url: "{{ wp_url }}"
        dest: /tmp/wordpress.tar.gz
            
    - name: "Wordpress - Extracting /tmp/wordpress.tar.gz"
      unarchive:
        src: /tmp/wordpress.tar.gz
        dest: /tmp/
        remote_src: true
            
    - name: "Wordpress - Copying Contents from /tmp/wordpress/ to /var/www/html/{{ domain_name }}"
      copy:
        src: /tmp/wordpress/
        dest: "/var/www/html/{{ domain_name }}"
        remote_src: true
        owner: "{{ httpd_user }}"
        group: "{{ httpd_group }}"
            
    - name: "Wordpress - Creating wp-config.php from template"
      template:
        src: wp-config.php.tmpl
        dest: "/var/www/html/{{ domain_name }}/wp-config.php"
        owner: "{{ httpd_user }}"
        group: "{{ httpd_group }}"
            
    - name: "Post-Installation CleanUp"
      file:
        path: "{{ item}}"
        state: absent
      with_items:
        - /tmp/wordpres.tar.gz
        - /tmp/wordpress
```

Once the playbook gets executed it will complete the setup of Wordpress in an Amazon Linux based environment.
