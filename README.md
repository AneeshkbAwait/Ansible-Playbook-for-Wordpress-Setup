## Ansible-Playbook-for-Wordpress-Setup
This Playbook will fulfil Wordpress installation and environment setup for the given domain on an Amazon Linux-based Operating System.

The Virtualhost configuration is built using the template file virualhost.conf.tmpl and the Wordpress configuration setup is done by using the wp-config.php.tmpl template file.

As is required by every WordPress installation, the Playbook is divided into three major sections;
- Apache Installation & Setup.
- Mariadb Server Installatin & Setup.
- Wordpress Installation & Setup.
