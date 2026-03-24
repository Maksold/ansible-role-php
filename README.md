# Ansible Role: PHP

Installs and configures PHP on RedHat/CentOS and Debian/Ubuntu servers.

> Fork of [geerlingguy.php](https://github.com/geerlingguy/ansible-role-php) with PHP 8.4 modernization, production security hardening, and Magento 2 optimizations.

## Requirements

A maintained PHP version repository (e.g. Remi for RHEL/CentOS, Ondrej PPA for Ubuntu). This role works with [currently supported PHP versions](http://php.net/supported-versions.php).

## Role Variables

Available variables are listed below, along with default values (see `defaults/main.yml`):

    php_packages: []

A list of the PHP packages to install (OS-specific by default).

    php_packages_extra: []

A list of extra PHP packages to install without overriding the default list.

    php_enable_webserver: true

Set to `false` if not using PHP with a web server.

    php_webserver_daemon: "httpd"

The web server daemon name (`httpd` for RedHat, `apache2` for Debian, `nginx` for Nginx).

    php_enablerepo: ""

(RedHat/CentOS only) Additional repositories to enable during PHP package installation.

    php_packages_state: "present"

Set to `"latest"` to upgrade PHP packages.

    php_executable: "php"

The PHP CLI executable path.

### PHP-FPM

    php_enable_php_fpm: false

Set to `true` to enable PHP-FPM configuration.

    php_fpm_state: started
    php_fpm_enabled_on_boot: true
    php_fpm_handler_state: restarted

FPM service state. Use `reloaded` for handler_state to reload instead of restart.

    php_fpm_pools:
      - pool_name: www
        pool_template: www.conf.j2
        pool_listen: "127.0.0.1:9000"
        pool_listen_allowed_clients: "127.0.0.1"
        pool_pm: dynamic
        pool_pm_max_children: 50
        pool_pm_start_servers: 5
        pool_pm_min_spare_servers: 5
        pool_pm_max_spare_servers: 5
        pool_pm_max_requests: 500
        pool_pm_status_path: ""

PHP-FPM pool configuration. Add items to the list for additional pools.

    php_fpm_pool_user: "[apache|www-data]"  # OS-specific default
    php_fpm_pool_group: "[apache|www-data]"

FPM pool process user/group.

    php_fpm_php_admin_values: []

List of `php_admin_value` directives for the pool. Each item is a single-key dict:

```yaml
php_fpm_php_admin_values:
  - memory_limit: 256M
  - error_log: "/var/log/php-fpm/www-error.log"
```

    php_fpm_php_admin_flags: []

List of `php_admin_flag` directives for the pool:

```yaml
php_fpm_php_admin_flags:
  - log_errors: "on"
  - display_errors: "off"
```

#### FPM Timeouts and Slowlog

    php_fpm_request_terminate_timeout: "120s"

Kills worker processes after this wall-clock time. Unlike `max_execution_time` (CPU time only), this covers all delays including sleep, HTTP requests, and I/O. Set higher than `max_execution_time`.

    php_fpm_slowlog: "/var/log/php-fpm/www-slow.log"
    php_fpm_request_slowlog_timeout: "5s"
    php_fpm_request_slowlog_trace_depth: "20"

Slow request logging with full PHP stack trace. Set timeout to `0` to disable.

#### FPM Security and Logging

    php_fpm_catch_workers_output: "yes"

Redirect worker stdout/stderr to error log instead of /dev/null.

    php_fpm_decorate_workers_output: "yes"

Add timestamp and pool name to worker output (PHP 7.3.15+).

    php_fpm_clear_env: "yes"

Clear environment variables in workers. Set to `"no"` only if application needs OS env vars.

    php_fpm_security_limit_extensions: ".php"

Restrict which file extensions FPM will execute. Prevents malicious execution of non-PHP files.

#### FPM Access Log (optional)

    php_fpm_access_log: ""
    php_fpm_access_format: ""

Per-pool access log. Leave empty to disable. Example format:

```yaml
php_fpm_access_log: "/var/log/php-fpm/www-access.log"
php_fpm_access_format: "%t %m %s %{mili}d ms %{kilo}M kB %C%% %{REQUEST_URI}e"
```

### php.ini Settings

    php_use_managed_ini: true

Set to `false` to self-manage php.ini.

#### Core Settings

    php_expose_php: "Off"
    php_memory_limit: "256M"
    php_max_execution_time: "60"
    php_max_input_time: "60"
    php_max_input_vars: "1000"
    php_sys_temp_dir: "/tmp"
    php_realpath_cache_size: "32K"
    php_realpath_cache_ttl: "120"
    php_file_uploads: "On"
    php_upload_tmp_dir: "/tmp"
    php_upload_max_filesize: "64M"
    php_max_file_uploads: "20"
    php_post_max_size: "32M"
    php_date_timezone: "America/Chicago"
    php_allow_url_fopen: "On"
    php_sendmail_path: "/usr/sbin/sendmail -t -i"
    php_output_buffering: "4096"
    php_short_open_tag: "Off"
    php_disable_functions: []
    php_precision: 14
    php_serialize_precision: "-1"
    php_default_charset: "UTF-8"
    php_html_errors: "On"

#### Error Reporting

    php_error_reporting: "E_ALL & ~E_DEPRECATED & ~E_STRICT"
    php_display_errors: "Off"
    php_display_startup_errors: "Off"
    php_error_log: "/var/log/php-errors.log"

#### Production Security Hardening

    php_zend_exception_ignore_args: "On"

Hide function arguments in stack traces (prevents leaking sensitive data).

    php_zend_exception_string_param_max_len: 0

Maximum length of string arguments in stack traces. `0` = don't show.

    php_zend_assertions: -1

Compile-time assertion control. `-1` = don't compile assertions (production performance).

    php_mail_add_x_header: "Off"

Disable X-PHP-Originating-Script header in emails (hides script paths).

#### Session Security

    php_session_cookie_lifetime: 0
    php_session_gc_probability: 1
    php_session_gc_divisor: 1000
    php_session_gc_maxlifetime: 1440
    php_session_save_handler: files
    php_session_save_path: ''
    php_session_cookie_httponly: "1"

Prevent JavaScript access to session cookies (XSS protection).

    php_session_cookie_secure: "0"

Send cookies only over HTTPS. Set to `"1"` for HTTPS-only environments.

    php_session_cookie_samesite: "Lax"

CSRF protection. Values: `"Strict"`, `"Lax"`, `"None"`.

    php_session_use_strict_mode: "1"

Reject uninitialized session IDs (session fixation protection).

### OpCache Variables

    php_opcache_enable: "1"
    php_opcache_enable_cli: "0"
    php_opcache_memory_consumption: "256"
    php_opcache_interned_strings_buffer: "16"
    php_opcache_max_accelerated_files: "10000"
    php_opcache_max_wasted_percentage: "5"
    php_opcache_validate_timestamps: "1"
    php_opcache_revalidate_path: "0"
    php_opcache_revalidate_freq: "2"
    php_opcache_max_file_size: "0"

Core OpCache settings. Ensure `memory_consumption` and `max_accelerated_files` are sufficient for your codebase. Calculate files with: `find . -type f -name '*.php' | wc -l` and add 25% margin.

    php_opcache_save_comments: "1"

Preserve PHPDoc comments. **Required by Magento 2** and other frameworks using reflection.

    php_opcache_enable_file_override: "0"

Accelerate `file_exists()`/`is_file()` via OPcache. Set to `"1"` for production.

    php_opcache_huge_code_pages: "0"

Use huge memory pages to reduce TLB misses. Set to `"1"` on Linux with huge pages configured.

    php_opcache_consistency_checks: "0"

Checksum validation on each request. `"0"` for production (performance).

    php_opcache_record_warnings: "0"

Record compilation warnings per cached file (PHP 8.1+).

    php_opcache_file_update_protection: "2"

Seconds to wait before caching a new file (prevents caching partially-written files).

    php_opcache_exclude_list_filename: ""

Path to file listing scripts to exclude from caching. Renamed from `blacklist_filename` in PHP 8.0.

#### OpCache JIT (PHP 8.0+)

    php_opcache_jit: "disable"

JIT compilation mode. Values: `"disable"`, `"tracing"`, `"function"`. **Warning**: JIT causes segfaults with Magento 2 — keep disabled for Magento deployments.

    php_opcache_jit_buffer_size: "0"

Memory for JIT compiled code. Set to `"0"` when JIT is disabled.

### APCu Variables

    php_enable_apc: true
    php_apc_shm_size: "96M"
    php_apc_enable_cli: "0"

APCu user cache settings. Ensure `php-pecl-apcu` (RHEL) or `php-apcu` (Debian) is installed.

## PHP 8.4 Compatibility

This role has been updated for PHP 8.4. The following deprecated/removed directives have been cleaned up:

**Removed from php.ini template:**
- `track_errors` (removed PHP 8.0)
- `disable_classes` (removed PHP 8.5)
- `sql.safe_mode` (removed PHP 7.2)
- `[MySQL]` section (removed PHP 7.0)
- `[MSSQL]` section (removed PHP 7.0)
- `session.hash_function`, `session.hash_bits_per_character` (removed PHP 7.1)
- `mysqli.reconnect` (removed PHP 8.2)
- `pdo_mysql.cache_size`, `mysqli.cache_size` (removed PHP 5.3)

**Removed from opcache template:**
- `zend_extension=opcache.so` — OPcache is built-in since PHP 8.0, loaded automatically
- `opcache.blacklist_filename` — renamed to `opcache.exclude_list_filename`

**Deprecated in PHP 8.4 (kept for now, will be removed in PHP 9.0):**
- `session.use_only_cookies`
- `session.use_trans_sid`
- `session.referer_check`

## Dependencies

None.

## Example Playbook

```yaml
- hosts: webservers
  vars:
    php_enable_php_fpm: true
    php_memory_limit: "512M"
    php_max_execution_time: "120"
    php_opcache_memory_consumption: "512"
    php_opcache_max_accelerated_files: "60000"
    php_fpm_php_admin_values:
      - error_log: "/var/log/php-fpm/www-error.log"
    php_fpm_php_admin_flags:
      - log_errors: "on"
  roles:
    - maksold.php
```

## License

MIT / BSD

## Author Information

Fork maintained by [Maksold](https://github.com/Maksold).
Original role by [Jeff Geerling](https://www.jeffgeerling.com/).
