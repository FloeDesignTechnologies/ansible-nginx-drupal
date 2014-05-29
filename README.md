Nginx Drupal
============

Ansible role to configure Nginx for running Drupal using [perusio's configuration](https://github.com/perusio/drupal-with-nginx).

This role only configure Nginx to run Drupal sites, it will not install PHP,
Nginx, Drupal, MySQL, etc. It will however, override the entire content of the
Nginx configuration directory. You can still add file to the Nginx configuration
directory after this role.

Requirements
------------

- Git
- A `reload nginx` handler is used to reload Nginx after configuration changes
  and must be defined in your playbook.

Role Variables
--------------

The following variables are available to configure the role:

- **nginx_drupal_git**
    - **repo**: The URL of the Git repository to checkout the base
    configuration from, defaults to https://github.com/perusio/drupal-with-nginx.git
    - **version** The version of the version of the repository to
    check out. This can be the full 40-character SHA-1 hash, the literal string
    HEAD, a branch name, or a tag name. Defaults to 'D7'.
- **nginx_drupal_config_path**: The path to Nginx configuration folder,
  defaults to "/etc/nginx".
- **nginx_drupal_log_path**: The path to Nginx log files, defaults to
  "/var/log/nginx"
- **nginx_drupal_php_handling**: The PHP handling method, one of "php-fpm",
  "php-cgi" or "proxy", defaults to "php-fpm".
- **nginx_drupal_escape_uri**: Whether or not to escaped URIs, defaults to
  false. **No implemented**
- **nginx_drupal_use_boost**: Whether or not [Boost](http://drupal.org/project/boost)
  is used, defaults to false. **No implemented**
- **nginx_drupal_use_drush**: Whether or not [Drush](https://github.com/drush-ops/drush)
  is used, defaults to true.
- **nginx_drupal_allow_install**: Whether or not to allow access to the
  ```install.php``` file, defaults to false.
- **nginx_drupal_use_spdy**: Whether or not to use SPDY, defaults to false.
- **nginx_drupal_nginx_status_allowed_hosts**: The list of host allowed to
  access Nginx status page, defaults to ```["127.0.0.1", "192.168.1.0/24"]```.
- **nginx_drupal_php_fpm_status_allowed_hosts**: The list of host allowed to
  access PHP-FPM status page, defaults to ```["127.0.0.1", "192.168.1.0/24"]```.
- **nginx_drupal_hotlinking_protection**: Whether or not to prevent image
  hotlinking, defaults to false.
- **nginx_drupal_admin_basic_auth**: Whether or not to protect access to admin
  pages (```/admin/*```) using HTTP auth, defaults to false.
- **nginx_drupal_microcache**: Whether or not to use microcaching, defaults to
  true.
- **nginx_drupal_microcache_auth**: Whether or not to use microcaching for
  authenticated users, defaults to false.
- **nginx_drupal_upload_progress**: Whether or not to use upload progress (this
  require the
- **nginx_drupal_use_aio**: Whether or not to use AIO to server video and audio
   file, defaults to true.
- **nginx_drupal_flv_streaming**: Whether or not to use FLV pseudo streaming
  (cf. http://wiki.nginx.org/HttpFlvStreamModule), defaults to false.
- **nginx_drupal_mp4_streaming**: Whether or not to use MP4 streaming, (cf.
  http://nginx.org/en/docs/http/ngx_http_mp4_module.html) defaults to false.
- **nginx_drupal_http_pre_includes**: A list of file to include in the ```http```
  context (in ```nginx.conf```), before any other directives.
- **nginx_drupal_http_post_includes**: A list of file to include in the ```http```
  context (in ```nginx.conf```), after any other directives except the enabled
  site configuration files.
- **nginx_drupal_upstream_servers**: The list of PHP upstream servers, each item
  is a server address (and parameters, see
  http://nginx.org/en/docs/http/ngx_http_upstream_module.html#server), defaults
  to ```["unix:/var/run/php-fpm.sock", "php-fpm-zwei.sock"]```.
- **nginx_drupal_upstream_backup_servers**: The list of PHP upstream backup
  servers, defaults to ```["unix:/var/run/php-fpm-bkp.sock"]```.
- **nginx_drupal_language_path_prefixes**: (optional) The list of enabled
  language path prefixes used on the site.
- **nginx_drupal_sites**: The list of available sites.
    Each site uses the following structure:
    - **file_name**: The name of the site configuration file.
    - **http**: HTTP server configuration (leave empty to disable HTTP)
        - **port**: The port to listen on
    - **https**: HTTPS server configuration  (leave empty to disable HTTPS)
        - **port**: The port to listen on.
        - **certificate**: Path to the SSL certificate of the server (in the PEM
          format).
        - **certificate_key**: Path to the SSL secret key of the server (in the
          PEM format).
    - **server_name**: The (primary) server name.
    - **ipv6**: (optional) IPv6 address of the server
    - **alternate_server_name**: (optional) Alternate server name, configured as
      redirect to the primary server site. This can be used to remove the
      ```www.``` prefix.
    - **root**: Path to the root directory for the site.
    - **limit_conn**: (optional) The limit_conn for the site (defaults to
      ```arbeit 32```).
    - **enabled**: Whether or not the site should be enabled (defaults to true).
    * **rewrites**: A list of rewrites directives, using the following structure:
        - **regex**: The regular expression used to match the URI.
        - **replacement**: The replacement pattern used for the rewrite.
        - **flags**: (optiona) The flag parameter for the rewrite.


Examples
--------

Two Drupal 7 sites, one available in HTTP and HTTPS. The other only available in
HTTPS but disabled.


    - hosts: all
      roles:
      - role: nginx-drupal
        nginx_drupal_sites:
          - file_name: foo
            server_name: foo.org
            alternate_server_name: www.foo.org
            root: /var/www/foo
            http:
              port: 80
            https:
              port: 443
              certificate: /etc/nginx/ssl/foo.cert
              certificate_key: /etc/nginx/ssl/foo.key
          - file_name: bar
            server_name: bar.org
            alternate_server_name: www.bar.org
            root: /var/www/bar
            enabled: false
            https:
              port: 443
              certificate: /etc/nginx/ssl/bar.cert
              certificate_key: /etc/nginx/ssl/bar.key

Nginx as a Reverse Proxy for a single Drupal 6 sites, without microcaching,
with image hot linking protection and a rewrite directive.


    - hosts: all
      roles:
      - role: nginx-drupal
        nginx_drupal_git:
          version: D6
        nginx_drupal_hotlinking_protection: true
        nginx_drupal_php_handling: proxy
        nginx_drupal_microcache: false
        nginx_drupal_sites:
          - file_name: foo
            server_name: foo.org
            alternate_server_name: www.foo.org
            root: /var/www/foo
            http:
              port: 80
            rewrites:
             - regex: '^/foo-bar.htm$'
               replacement: '/foo/bar'
               flags: 'permanent'

License
-------

Apache v2

Author Information
------------------

Pierre Buyle <buyle@pheromone.ca>