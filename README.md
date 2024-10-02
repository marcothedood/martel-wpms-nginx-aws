Wordpress Multisite Box
--------
> Martel's WP Multisite stack on just one Ubuntu machine +  AWS RDS MariaDB Instance

So we've migrated our WP Multisite service away from cPanel to a dedicated
Ubuntu server. In the process, we've developed quite a bit of functionality which resulted
in optimizing hardware costs while improving reliability and performance at the same time.

Below is an UML-ish deployment diagram followed by an overview of 
the main features:

![AWS - WPMS - NGINX](https://github.com/user-attachments/assets/fd94bac6-809e-49a1-8f9c-8cfd10bcd02f)

## Features

### WP Multisite Service

An AMI `(ami-07c39ac3d711ea0f2)` ARM64-based to run WP Multisite + NGINX:
- Wordpress installation configured to run as a multisite environment
- Pre-configured `ca-bundle` to allow SSL Encryption thru srv and RDS.
- PHP 8.2 w/ + fpm + deps. for mysql and imagick modules.
- Nginx TLS reverse proxy w/ several vhosts to serve multiple websites,
  exposing them to the internet.
- Minimal Ubuntu OS base system with SSH and a dedicated volume for WP data
- Certbot for Let's Encrypt certs thru ACME.

As for security, we stick to Least Privilege and Zero
Trust principles.


### Database

The DB Instance is hosted on AWS using RDS MariaDB.
We perform daily DB snapshot for db consistency - backup window is configured on AWS.
Check the configs directly on AWS Console.
Traffic btween RDS and our WP Server is encrypted thank to the ca-bundle AWS provides.
The db user `(devngieu_current_multisite)` is configured only to use encrypted connections to connect
and it has privileges only on its db.
Automatic minor updates are enabled while major are disabled by default.

### System & others

Also, we've set up a few things to make the sys admin's life a
bit easier:
- Login. The only users configured and accessible thru SSH are `vito` and `marco`.
  These can only log in over SSH with their respective
  - SSH identities. `PasswordAuthentication` is set to `no`.
  - SSH has been disabled for `ubuntu` and the variable `PermitRootLogin`
    is set to `prohibit-password` in `/etc/ssh/sshd_config`.  
- TLS certificates. We use Let's Encrypt as a CA in prod.
  Automatic renewal of domains' TLS certs is set-up.
- Backups. Automatic backup is performed thru AWS Lifecycle rules.
  We backup both the EBS volumes (/) and (/var/public_html) daily,
  schedules occurs at 01:OO UTC Time.
- Git-versioned /etc. We have set-up [a secondary pvt repo](https://github.com/marcothedood/wpms-etc) to manage
  every tweak and update made on the main configs (hosted on /etc).
  On the machine, etckeeper rules and performs a daily commit of whatever
  changes under /etc.
  We won't install anyother tool and the only ops would consist of creating
  new vhosts in case of new sites.
