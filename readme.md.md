# UAS SISTEM TERDISTRIBUSI

**Kelompok 10** 

**Nurul Azizi Hasibuan | 1203210090**

**Amanda Oktaviana Saragih | 1203210092**

```jsx
Stephanie dan Gede terpilih menjadi junior SysAdmin di salah satu BUMN pada program Magang Kampus Merdeka. Mereka ditugaskan untuk mendeploy website yang akan di launching 2 minggu lagi. Senior SysAdmin mereka yang bernama Sung Jin-Woo memberikan beberapa penjelasan dan masukan terkait persiapan yang dibutuhkan.
 
Terdapat 4 website yang akan di deploy
 
1. kelompokXX.local/ menggunakan framework laravel 8 dan php 7.4
2. news.kelompokXX.local menggunakan framework wordpress terbaru dan php 7.4
3. kelompokXX.local/product menggunakan framework yii 2.0 dan php 7.4
4. kelompokXX.local/app menggunakan framework codeigniter 3 dan php 5.6
 
silahkan replace XX dengan nomor kelompok.
 
## Teknologi yang digunakan
 
teknologi yang digunakan antara lain
 
1. VM Ubuntu 20.04 (menggunakan metode bridge network, dan static ip) atau menggunakan WSL
2. Nginx
3. 6 instance LXC ubuntu 20.04 PHP 7.4
4. 2 instance LXC debian 10 PHP 5.6
5. 1 instance LXC debian 10 mariadb server
6. Semua instalasi menggunakan Ansible ( kalau bisa )
7. kelompokXX.local/ menggunakan laravel 8
8. news.kelompokXX.local menggunakan wordpress terbaru
9. kelompokXX.local/product menggunakan YII 2.0
10. kelompokXX.local/app menggunakan Code Igniter 3 ( https://github.com/aldonesia/sas-ci )
 
## Arsitektur Jaringan
```

![Untitled](UAS%20SISTEM%20TERDISTRIBUSI%20e08286d2c95a4675a030fe661a2f50d4/Untitled.png)

**[ install lxc (ubuntu 20.04) ]**

```jsx
**sudo lxc-create -n lxc_php7_1 -t download -- --dist ubuntu --release focal --arch amd64 --force-cache --no-validate --server images.linuxcontainers.org

sudo lxc-create -n lxc_php7_2 -t download -- --dist ubuntu --release focal --arch amd64 --force-cache --no-validate --server images.linuxcontainers.org

sudo lxc-create -n lxc_php7_3 -t download -- --dist ubuntu --release focal --arch amd64 --force-cache --no-validate --server images.linuxcontainers.org

sudo lxc-create -n lxc_php7_4 -t download -- --dist ubuntu --release focal --arch amd64 --force-cache --no-validate --server images.linuxcontainers.org

sudo lxc-create -n lxc_php7_5 -t download -- --dist ubuntu --release focal --arch amd64 --force-cache --no-validate --server images.linuxcontainers.org

sudo lxc-create -n lxc_php7_6 -t download -- --dist ubuntu --release focal --arch amd64 --force-cache --no-validate --server images.linuxcontainers.org**
```

**[ install lxc (debian 10) ]**

```jsx
**sudo lxc-create -n lxc_php5_1 -t download -- --dist debian --release buster --arch amd64 --force-cache --no-validate --server images.linuxcontainers.org

sudo lxc-create -n lxc_php5_2 -t download -- --dist debian --release buster --arch amd64 --force-cache --no-validate --server images.linuxcontainers.org**
```

**[ install lxc untuk mariadb (debian 10) ]**

```jsx
**sudo lxc-create -n lxc_mariadb -t download -- --dist debian --release buster --arch amd64 --force-cache --no-validate --server images.linuxcontainers.org**
```

- selanjutnya, kita akan melakukan konfigurasi pada setting ip statis dan menginstall server ssh pada setiap lxc. lalu, untuk melakukan konfigurasi ip maka lakukanlah seperti pada gambar dibawah ini.

![Untitled](UAS%20SISTEM%20TERDISTRIBUSI%20e08286d2c95a4675a030fe661a2f50d4/e6f20758-96ac-4e83-9b7c-e508e18671e6.png)

- Registrasi domain masing-masing lxc seperti pada perintah dibawah ini.

```jsx
**sudo nano /etc/hosts**
```

- Hingga mengubah nano /etc/hosts seperti dibawah ini.
    
    ```jsx
    127.0.0.1 localhost
    127.0.1.1 sas02
    127.0.1.1 kelompok10.fpsas news.kelompok10.fpsas
    
    10.0.3.183      lxc_mariadb.dev
    10.0.3.184      lxc_laravel.dev
    10.0.3.163      lxc_wordpress.dev
    10.0.3.39        lxc_yii.dev
    10.0.3.177      lxc_codeigniter.dev
    
    #laravel
    10.0.3.12         lxc_php7_1L.dev
    10.0.3.243      lxc_php7_2L.dev
    10.0.3.51         lxc_php7_3L.dev
    10.0.3.54        lxc_php7_4L.dev
    10.0.3.81         lxc_php7_6L.dev
    
    #wordpress
    10.0.3.243      lxc_php7_2W.dev
    10.0.3.51         lxc_php7_3W.dev
    10.0.3.54        lxc_php7_4W.dev
    10.0.3.145      lxc_php7_5W.dev
    
    #yii
    10.0.3.12         lxc_php7_1Y.dev
    10.0.3.243      lxc_php7_2Y.dev
    10.0.3.51         lxc_php7_3Y.dev
    10.0.3.54        lxc_php7_4Y.dev
    10.0.3.81         lxc_php7_6Y.dev
    
    #codeigniter
    10.0.3.141       lxc_php5_1.dev
    10.0.3.105      lxc_php5_2.dev
    
    The following lines are desirable for IPv6 capable hosts
    
    ::1     ip6-localhost ip6-loopback
    ```
    
- sekarang, kita akan melakukan dengan menuliskan beberapa skrip yang mungkin untuk melakukan instalasi frame di setiap lxc seperti persyaratan dalam studi kasus.
    - membuat folder pada direktori ansibel menggunakan perintah `mkdir -p ~/ansible/tubes` untuk menampung semua script konfigurasi yang akan digunakan pada tugas kali ini.
    - pada direktori tubes, buatlah direktori role untuk menampung seluruh script pada setiap framework yang akan dipasang pada proyek ini.
    - membuat direktori laravel di direktori peran sudo mkdir laravel yang memungkinkan untuk menginstall framework.
- pada direktori laravel, kita memerlukan 3 folder lagi dengan nama tasks, handlers, dan templates yang akan digunakan pada instalasi laravel.

```jsx
**sudo mkdir -p laravel/tasks
sudo mkdir -p laravel/handlers
sudo mkdir -p laravel/templates**
```

- pada direktori tasks, tambahkan file dengan nama `main.yml` seperti pada gambar dibawah ini.
    
    ![Untitled](UAS%20SISTEM%20TERDISTRIBUSI%20e08286d2c95a4675a030fe661a2f50d4/bf1667ff-f16f-493e-b097-5e9f2357f77e.png)
    
    ![Untitled](UAS%20SISTEM%20TERDISTRIBUSI%20e08286d2c95a4675a030fe661a2f50d4/72a173f6-275c-47b4-bfb2-48e1d52c0343.png)
    
- pada direktori handlers, tambahkan file dengan nama `main.yml` seperti pada gambar dibawah ini.
    
    ![Untitled](UAS%20SISTEM%20TERDISTRIBUSI%20e08286d2c95a4675a030fe661a2f50d4/f45fde0c-2db3-49d0-b8bf-294f4d377ef0.png)
    
- jika pada direktori templates, kita akan menambahkan 3 skrip file seperti pada gambar dibawah ini.
    1. env.templates
    
    ![Untitled](UAS%20SISTEM%20TERDISTRIBUSI%20e08286d2c95a4675a030fe661a2f50d4/59cee882-9877-4e43-a173-65563e7186a0.png)
    
    ![Untitled](UAS%20SISTEM%20TERDISTRIBUSI%20e08286d2c95a4675a030fe661a2f50d4/28ad7aef-ce15-4ccf-9b93-3b45707d496e.png)
    
    1. lv.conf
    
    ![Untitled](UAS%20SISTEM%20TERDISTRIBUSI%20e08286d2c95a4675a030fe661a2f50d4/6ec6a147-cf7f-4852-b8d1-19dae60d1465.png)
    
    1. php7.conf
    
    ![Untitled](UAS%20SISTEM%20TERDISTRIBUSI%20e08286d2c95a4675a030fe661a2f50d4/6e774b9d-5be3-427b-adc6-e15b6d3a9780.png)
    
    ```jsx
    ; Start a new pool named 'www'.
    ; the variable $pool can we used in any directive and will be replaced by the
    ; pool name ('www' here)
    [www]
    
    ; Per pool prefix
    ; It only applies on the following directives:
    ; - 'slowlog'
    ; - 'listen' (unixsocket)
    ; - 'chroot'
    ; - 'chdir'
    ; - 'php_values'
    ; - 'php_admin_values'
    ; When not set, the global prefix (or /usr) applies instead.
    ; Note: This directive can also be relative to the global prefix.
    ; Default Value: none
    ;prefix = /path/to/pools/$pool
    
    ; Unix user/group of processes
    ; Note: The user is mandatory. If the group is not set, the default user's group
    ;       will be used.
    user = www-data
    group = www-data
    
    ; The address on which to accept FastCGI requests.
    ; Valid syntaxes are:
    ;   'ip.add.re.ss:port'    - to listen on a TCP socket to a specific address on
    ;                            a specific port;
    ;   'port'                 - to listen on a TCP socket to all addresses on a
    ;                            specific port;
    ;   '/path/to/unix/socket' - to listen on a unix socket.
    ; Note: This value is mandatory.
    listen = 127.0.0.1:9001
    
    ; Set listen(2) backlog. A value of '-1' means unlimited.
    ; Default Value: 128 (-1 on FreeBSD and OpenBSD)
    ;listen.backlog = -1
    
    ; Set permissions for unix socket, if one is used. In Linux, read/write
    ; permissions must be set in order to allow connections from a web server. Many
    ; BSD-derived systems allow connections regardless of permissions.
    ; Default Values: user and group are set as the running user
    ;                 mode is set to 0666
    listen.owner = www-data
    listen.group = www-data
    listen.mode = 0666
    
    ; List of ipv4 addresses of FastCGI clients which are allowed to connect.
    ; Equivalent to the FCGI_WEB_SERVER_ADDRS environment variable in the original
    ; PHP FCGI (5.2.2+). Makes sense only with a tcp listening socket. Each address
    ; must be separated by a comma. If this value is left blank, connections will be
    ; accepted from any ip address.
    ; Default Value: any
    ;listen.allowed_clients = 127.0.0.1
    
    ; Choose how the process manager will control the number of child processes.
    ; Possible Values:
    ;   static  - a fixed number (pm.max_children) of child processes;
    ;   dynamic - the number of child processes are set dynamically based on the
    ;             following directives. With this process management, there will be
    ;             always at least 1 children.
    ;             pm.max_children      - the maximum number of children that can
    ;                                    be alive at the same time.
    ;             pm.start_servers     - the number of children created on startup.
    ;             pm.min_spare_servers - the minimum number of children in 'idle'
    ;                                    state (waiting to process). If the number
    ;                                    of 'idle' processes is less than this
    ;                                    number then some children will be created.
    ;             pm.max_spare_servers - the maximum number of children in 'idle'
    ;                                    state (waiting to process). If the number
    ;                                    of 'idle' processes is greater than this
    ;                                    number then some children will be killed.
    ;  ondemand - no children are created at startup. Children will be forked when
    ;             new requests will connect. The following parameter are used:
    ;             pm.max_children           - the maximum number of children that
    ;                                         can be alive at the same time.
    ;             pm.process_idle_timeout   - The number of seconds after which
    ;                                         an idle process will be killed.
    ; Note: This value is mandatory.
    pm = dynamic
    
    ; The number of child processes to be created when pm is set to 'static' and the
    ; maximum number of child processes when pm is set to 'dynamic' or 'ondemand'.
    ; This value sets the limit on the number of simultaneous requests that will be
    ; served. Equivalent to the ApacheMaxClients directive with mpm_prefork.
    ; Equivalent to the PHP_FCGI_CHILDREN environment variable in the original PHP
    ; CGI. The below defaults are based on a server without much resources. Don't
    ; forget to tweak pm.* to fit your needs.
    ; Note: Used when pm is set to 'static', 'dynamic' or 'ondemand'
    ; Note: This value is mandatory.
    pm.max_children = 5
    
    ; The number of child processes created on startup.
    ; Note: Used only when pm is set to 'dynamic'
    ; Default Value: min_spare_servers + (max_spare_servers - min_spare_servers) / 2
    pm.start_servers = 2
    
    ; The desired minimum number of idle server processes.
    ; Note: Used only when pm is set to 'dynamic'
    ; Note: Mandatory when pm is set to 'dynamic'
    pm.min_spare_servers = 1
    
    ; The desired maximum number of idle server processes.
    ; Note: Used only when pm is set to 'dynamic'
    ; Note: Mandatory when pm is set to 'dynamic'
    pm.max_spare_servers = 3
    
    ; The number of seconds after which an idle process will be killed.
    ; Note: Used only when pm is set to 'ondemand'
    ; Default Value: 10s
    ;pm.process_idle_timeout = 10s;
    
    ; The number of requests each child process should execute before respawning.
    ; This can be useful to work around memory leaks in 3rd party libraries. For
    ; endless request processing specify '0'. Equivalent to PHP_FCGI_MAX_REQUESTS.
    ; Default Value: 0
    pm.max_requests = 100
    
    ; The URI to view the FPM status page. If this value is not set, no URI will be
    ; recognized as a status page. It shows the following informations:
    ;   pool                 - the name of the pool;
    ;   process manager      - static, dynamic or ondemand;
    ;   start time           - the date and time FPM has started;
    ;   start since          - number of seconds since FPM has started;
    ;   accepted conn        - the number of request accepted by the pool;
    ;   listen queue         - the number of request in the queue of pending
    ;                          connections (see backlog in listen(2));
    ;   max listen queue     - the maximum number of requests in the queue
    ;                          of pending connections since FPM has started;
    ;   listen queue len     - the size of the socket queue of pending connections;
    ;   idle processes       - the number of idle processes;
    ;   active processes     - the number of active processes;
    ;   total processes      - the number of idle + active processes;
    ;   max active processes - the maximum number of active processes since FPM
    ;                          has started;
    ;   max children reached - number of times, the process limit has been reached,
    ;                          when pm tries to start more children (works only for
    ;                          pm 'dynamic' and 'ondemand');
    ; Value are updated in real time.
    ; Example output:
    ;   pool:                 www
    ;   process manager:      static
    ;   start time:           01/Jul/2011:17:53:49 +0200
    ;   start since:          62636
    ;   accepted conn:        190460
    ;   listen queue:         0
    ;   max listen queue:     1
    ;   listen queue len:     42
    ;   idle processes:       4
    ;   active processes:     11
    ;   total processes:      15
    ;   max active processes: 12
    ;   max children reached: 0
    ;
    ; By default the status page output is formatted as text/plain. Passing either
    ; 'html', 'xml' or 'json' in the query string will return the corresponding
    ; output syntax. Example:
    ;   http://www.foo.bar/status
    ;   http://www.foo.bar/status?json
    ;   http://www.foo.bar/status?html
    ;   http://www.foo.bar/status?xml
    ;
    ; By default the status page only outputs short status. Passing 'full' in the
    ; query string will also return status for each pool process.
    ; Example:
    ;   http://www.foo.bar/status?full
    ;   http://www.foo.bar/status?json&full
    ;   http://www.foo.bar/status?html&full
    ;   http://www.foo.bar/status?xml&full
    ; The Full status returns for each process:
    ;   pid                  - the PID of the process;
    ;   state                - the state of the process (Idle, Running, ...);
    ;   start time           - the date and time the process has started;
    ;   start since          - the number of seconds since the process has started;
    ;   requests             - the number of requests the process has served;
    ;   request duration     - the duration in µs of the requests;
    ;   request method       - the request method (GET, POST, ...);
    ;   request URI          - the request URI with the query string;
    ;   content length       - the content length of the request (only with POST);
    ;   user                 - the user (PHP_AUTH_USER) (or '-' if not set);
    ;   script               - the main script called (or '-' if not set);
    ;   last request cpu     - the %cpu the last request consumed
    ;                          it's always 0 if the process is not in Idle state
    ;                          because CPU calculation is done when the request
    ;                          processing has terminated;
    ;   last request memory  - the max amount of memory the last request consumed
    ;                          it's always 0 if the process is not in Idle state
    ;                          because memory calculation is done when the request
    ;                          processing has terminated;
    ; If the process is in Idle state, then informations are related to the
    ; last request the process has served. Otherwise informations are related to
    ; the current request being served.
    ; Example output:
    ;   ************************
    ;   pid:                  31330
    ;   state:                Running
    ;   start time:           01/Jul/2011:17:53:49 +0200
    ;   start since:          63087
    ;   requests:             12808
    ;   request duration:     1250261
    ;   request method:       GET
    ;   request URI:          /test_mem.php?N=10000
    ;   content length:       0
    ;   user:                 -
    ;   script:               /home/fat/web/docs/php/test_mem.php
    ;   last request cpu:     0.00
    ;   last request memory:  0
    ;
    ; Note: There is a real-time FPM status monitoring sample web page available
    ;       It's available in: ${prefix}/share/fpm/status.html
    ;
    ; Note: The value must start with a leading slash (/). The value can be
    ;       anything, but it may not be a good idea to use the .php extension or it
    ;       may conflict with a real PHP file.
    ; Default Value: not set
    pm.status_path = /php-status
    
    ; The ping URI to call the monitoring page of FPM. If this value is not set, no
    ; URI will be recognized as a ping page. This could be used to test from outside
    ; that FPM is alive and responding, or to
    ; - create a graph of FPM availability (rrd or such);
    ; - remove a server from a group if it is not responding (load balancing);
    ; - trigger alerts for the operating team (24/7).
    ; Note: The value must start with a leading slash (/). The value can be
    ;       anything, but it may not be a good idea to use the .php extension or it
    ;       may conflict with a real PHP file.
    ; Default Value: not set
    ;ping.path = /ping
    
    ; This directive may be used to customize the response of a ping request. The
    ; response is formatted as text/plain with a 200 response code.
    ; Default Value: pong
    ;ping.response = pong
    
    ; The access log file
    ; Default: not set
    ;access.log = log/$pool.access.log
    
    ; The access log format.
    ; The following syntax is allowed
    ;  %%: the '%' character
    ;  %C: %CPU used by the request
    ;      it can accept the following format:
    ;      - %{user}C for user CPU only
    ;      - %{system}C for system CPU only
    ;      - %{total}C  for user + system CPU (default)
    ;  %d: time taken to serve the request
    ;      it can accept the following format:
    ;      - %{seconds}d (default)
    ;      - %{miliseconds}d
    ;      - %{mili}d
    ;      - %{microseconds}d
    ;      - %{micro}d
    ;  %e: an environment variable (same as $_ENV or $_SERVER)
    ;      it must be associated with embraces to specify the name of the env
    ;      variable. Some exemples:
    ;      - server specifics like: %{REQUEST_METHOD}e or %{SERVER_PROTOCOL}e
    ;      - HTTP headers like: %{HTTP_HOST}e or %{HTTP_USER_AGENT}e
    ;  %f: script filename
    ;  %l: content-length of the request (for POST request only)
    ;  %m: request method
    ;  %M: peak of memory allocated by PHP
    ;      it can accept the following format:
    ;      - %{bytes}M (default)
    ;      - %{kilobytes}M
    ;      - %{kilo}M
    ;      - %{megabytes}M
    ;      - %{mega}M
    ;  %n: pool name
    ;  %o: ouput header
    ;      it must be associated with embraces to specify the name of the header:
    ;      - %{Content-Type}o
    ;      - %{X-Powered-By}o
    ;      - %{Transfert-Encoding}o
    ;      - ....
    ;  %p: PID of the child that serviced the request
    ;  %P: PID of the parent of the child that serviced the request
    ;  %q: the query string
    ;  %Q: the '?' character if query string exists
    ;  %r: the request URI (without the query string, see %q and %Q)
    ;  %R: remote IP address
    ;  %s: status (response code)
    ;  %t: server time the request was received
    ;      it can accept a strftime(3) format:
    ;      %d/%b/%Y:%H:%M:%S %z (default)
    ;  %T: time the log has been written (the request has finished)
    ;      it can accept a strftime(3) format:
    ;      %d/%b/%Y:%H:%M:%S %z (default)
    ;  %u: remote user
    ;
    ; Default: "%R - %u %t \"%m %r\" %s"
    ;access.format = %R - %u %t "%m %r%Q%q" %s %f %{mili}d %{kilo}M %C%%
    
    ; The log file for slow requests
    ; Default Value: not set
    ; Note: slowlog is mandatory if request_slowlog_timeout is set
    ;slowlog = log/$pool.log.slow
    
    ; The timeout for serving a single request after which a PHP backtrace will be
    ; dumped to the 'slowlog' file. A value of '0s' means 'off'.
    ; Available units: s(econds)(default), m(inutes), h(ours), or d(ays)
    ; Default Value: 0
    ;request_slowlog_timeout = 0
    
    ; The timeout for serving a single request after which the worker process will
    ; be killed. This option should be used when the 'max_execution_time' ini option
    ; does not stop script execution for some reason. A value of '0' means 'off'.
    ; Available units: s(econds)(default), m(inutes), h(ours), or d(ays)
    ; Default Value: 0
    ;request_terminate_timeout = 0
    
    ; Set open file descriptor rlimit.
    ; Default Value: system defined value
    ;rlimit_files = 1024
    
    ; Set max core size rlimit.
    ; Possible Values: 'unlimited' or an integer greater or equal to 0
    ; Default Value: system defined value
    ;rlimit_core = 0
    
    ; Chroot to this directory at the start. This value must be defined as an
    ; absolute path. When this value is not set, chroot is not used.
    ; Note: you can prefix with '$prefix' to chroot to the pool prefix or one
    ; of its subdirectories. If the pool prefix is not set, the global prefix
    ; will be used instead.
    ; Note: chrooting is a great security feature and should be used whenever
    ;       possible. However, all PHP paths will be relative to the chroot
    ;       (error_log, sessions.save_path, ...).
    ; Default Value: not set
    ;chroot =
    
    ; Chdir to this directory at the start.
    ; Note: relative path can be used.
    ; Default Value: current directory or / when chroot
    chdir = /
    
    ; Redirect worker stdout and stderr into main error log. If not set, stdout and
    ; stderr will be redirected to /dev/null according to FastCGI specs.
    ; Note: on highloaded environement, this can cause some delay in the page
    ; process time (several ms).
    ; Default Value: no
    catch_workers_output = yes
    
    ; Limits the extensions of the main script FPM will allow to parse. This can
    ; prevent configuration mistakes on the web server side. You should only limit
    ; FPM to .php extensions to prevent malicious users to use other extensions to
    ; exectute php code.
    ; Note: set an empty value to allow all extensions.
    ; Default Value: .php
    ;security.limit_extensions = .php .php3 .php4 .php5 .php7
    
    ; Pass environment variables like LD_LIBRARY_PATH. All $VARIABLEs are taken from
    ; the current environment.
    ; Default Value: clean env
    ;env[HOSTNAME] = $HOSTNAME
    env[PATH] = /srv/www/phpcs/scripts/:/usr/local/bin:/usr/bin:/bin
    ;env[TMP] = /tmp
    ;env[TMPDIR] = /tmp
    ;env[TEMP] = /tmp
    
    ; Additional php.ini defines, specific to this pool of workers. These settings
    ; overwrite the values previously defined in the php.ini. The directives are the
    ; same as the PHP SAPI:
    ;   php_value/php_flag             - you can set classic ini defines which can
    ;                                    be overwritten from PHP call 'ini_set'.
    ;   php_admin_value/php_admin_flag - these directives won't be overwritten by
    ;                                     PHP call 'ini_set'
    ; For php_*flag, valid values are on, off, 1, 0, true, false, yes or no.
    
    ; Defining 'extension' will load the corresponding shared extension from
    ; extension_dir. Defining 'disable_functions' or 'disable_classes' will not
    ; overwrite previously defined php.ini values, but will append the new value
    ; instead.
    
    ; Note: path INI options can be relative and will be expanded with the prefix
    ; (pool, global or /usr)
    
    ; Default Value: nothing is defined by default except the values in php.ini and
    ;                specified at startup with the -d argument
    ;php_admin_value[sendmail_path] = /usr/sbin/sendmail -t -i -f [www@my.domain.com](mailto:www@my.domain.com)
    ;php_flag[display_errors] = off
    ;php_admin_value[error_log] = /var/log/fpm-php.www.log
    ;php_admin_flag[log_errors] = on
    ;php_admin_value[memory_limit] = 32M
    ```
    
    ![Untitled](UAS%20SISTEM%20TERDISTRIBUSI%20e08286d2c95a4675a030fe661a2f50d4/60aff377-4071-4fd3-9387-d5488edc7f64.png)
    
- selanjutnya, kita akan menambahkan folder dengan nama php untuk akomodasi php installasi di laravel roles. dan didalam roles, kita dapat menambhakan 2 direktori dengan nama handlers dan tasks.
    1. pada direktori tasks, kita dapat menambahkan file dengan nama `main.yml` seperti pada gambar dibawah ini.
    
    ![Untitled](UAS%20SISTEM%20TERDISTRIBUSI%20e08286d2c95a4675a030fe661a2f50d4/859dd213-b9e2-4fb7-8b43-74fc3dcf521c.png)
    
    1. pada direktori handlers, kita dapat menambahkan file dengan nama `main.yml` seperti pada gambar dibawah ini.
    
    ![Untitled](UAS%20SISTEM%20TERDISTRIBUSI%20e08286d2c95a4675a030fe661a2f50d4/3134452d-0962-4908-9dd5-7e3fb4e46e8c.png)
    
- tambahkan direktori codeigniter di dalam direktori roles menggunakan perintah sudo mkdir codeigniter untuk mengakomidasi ansibel dan templates skrip seperti dibawah ini.

```jsx
**sudo mkdir -p codeigniter/tasks
sudo mkdir -p codeigniter/handlers
sudo mkdir -p codeigniter/templates**
```

- pada direktori tasks, kita dapat menambahkan file dengan nama `main.yml` seperti pada gambar dibawah ini.
    
    ![Untitled](UAS%20SISTEM%20TERDISTRIBUSI%20e08286d2c95a4675a030fe661a2f50d4/77d3fbde-d4b3-4753-a61d-db641f7e30f0.png)
    
    ![Untitled](UAS%20SISTEM%20TERDISTRIBUSI%20e08286d2c95a4675a030fe661a2f50d4/92363546-4598-4558-af0e-93478e085ce2.png)
    
- pada direktori handlers, kita dapat menambahkan file dengan nama `main.yml` seperti pada gambar dibawah ini.
    
    ![Untitled](UAS%20SISTEM%20TERDISTRIBUSI%20e08286d2c95a4675a030fe661a2f50d4/4e042b0d-2007-4bd3-b30d-f44f151e8e42.png)
    
- pada direktori templates, kita dapat menambahkan file dengan nama file `app.conf` seperti pada gambar dibawah ini.
    
    ![Untitled](UAS%20SISTEM%20TERDISTRIBUSI%20e08286d2c95a4675a030fe661a2f50d4/129832e5-5e48-46c3-9e8c-0a68cdd39778.png)
    
- selanjutnya menambahkan direktori di dalam direktori roles menggunakan perintah sudo mkdir db untuk mengakomodasi skrip ansibel untuk menginstall db framework.
    - pada direktori db, kita dapat menambahkan 3 folder lagi intuk mengakomodasi tasks, handlers dan templates skript dengan menjalankan perintah seperti dibawah ini.
        
        ![Untitled](UAS%20SISTEM%20TERDISTRIBUSI%20e08286d2c95a4675a030fe661a2f50d4/85c60c0c-b8d9-4d58-8f20-0d8eac3ba1de.png)
        
        ![Untitled](UAS%20SISTEM%20TERDISTRIBUSI%20e08286d2c95a4675a030fe661a2f50d4/1bb9ff6d-1b14-43ee-966b-59184e1ffd9a.png)
        
    - pada direktori handlers dapat menambhakan file dengan nama file `main.yml` seperti pada gambar dibawah ini.
        
        ![Untitled](UAS%20SISTEM%20TERDISTRIBUSI%20e08286d2c95a4675a030fe661a2f50d4/bb62667c-1a37-4228-a746-668698f42fd3.png)
        
    - pada direktori template tambahkan file dengan nama `my.cnf` seperti pada gambar dibawah ini.
        
        ![Untitled](UAS%20SISTEM%20TERDISTRIBUSI%20e08286d2c95a4675a030fe661a2f50d4/4ceee96c-86f9-4fa7-b5e6-3b2670c6c9a1.png)
        
        ![Untitled](UAS%20SISTEM%20TERDISTRIBUSI%20e08286d2c95a4675a030fe661a2f50d4/a7ac74d0-a44e-4e9f-8030-838249db0efa.png)
        
        ![Untitled](UAS%20SISTEM%20TERDISTRIBUSI%20e08286d2c95a4675a030fe661a2f50d4/86e37037-d534-4a10-8869-90cfdb80cccd.png)
        
        ![Untitled](UAS%20SISTEM%20TERDISTRIBUSI%20e08286d2c95a4675a030fe661a2f50d4/fd1b159e-3e09-4975-ba18-4622e9e5b421.png)
        
        ![Untitled](UAS%20SISTEM%20TERDISTRIBUSI%20e08286d2c95a4675a030fe661a2f50d4/c6ade75e-ff99-43bb-9b97-64b3825e26ee.png)
        
    - selanjutnya kita kana menambahkan sebuah folder dengan nama pma untuk mengakomodasi pma instalasi di dalam pma roles. di dalam roles kita bida menambahkan 3 direktori dengan nama tasks, handlers dan templat.
    - pada direktori tasks tambahkan file dengan nama main.yml seperti pada gambar dibawah ini.
        
        ![Untitled](UAS%20SISTEM%20TERDISTRIBUSI%20e08286d2c95a4675a030fe661a2f50d4/12e1ded4-0e58-4a33-9394-af31156e5f42.png)
        
        ![Untitled](UAS%20SISTEM%20TERDISTRIBUSI%20e08286d2c95a4675a030fe661a2f50d4/91a16029-14dd-480b-b3f5-5deb7167fea3.png)
        
    - pada direktori handlers, kita dapat menambahkan file dengan nama `main.yml` seperti pada gambar dibawah ini.
        
        ![Untitled](UAS%20SISTEM%20TERDISTRIBUSI%20e08286d2c95a4675a030fe661a2f50d4/f0ece92e-0557-4de1-9e29-7cc8570b15dd.png)
        
    - pada direktori templates kita dapat menambahkan file dengan nama `pma.local` seperti pada gambar dibawah ini
        
        ![Untitled](UAS%20SISTEM%20TERDISTRIBUSI%20e08286d2c95a4675a030fe661a2f50d4/bfc1206b-f3fd-4d59-91f9-c13d32dc64c9.png)
        
    - tambahkan sebuah direktori wordpress didalam direktori roles dengan menjalankan perintah sudo mkdir wordpress untuk mengakomodasi skrip ansibel untuk menginstall wordpress framework.
    - pada direktori wordpress, kita dapat menambahkan 3 folder lagi untuk mengakomodasi tasks, handlers dan skrip templates didalam wordpress yang telah terinstall.
    
    ```jsx
    **sudo mkdir -p wordpress /tasks
    sudo mkdir -p wordpress /handlers
    sudo mkdir -p wordpress /templates**
    ```
    
    - pada direktori tasks, kita dapat menambahkan file dengan nama `main.yml` seperti pada gambar dibawah ini.
        
        ![Untitled](UAS%20SISTEM%20TERDISTRIBUSI%20e08286d2c95a4675a030fe661a2f50d4/4b386eb3-da72-499c-bee5-7edc22aec6b6.png)
        
        ![Untitled](UAS%20SISTEM%20TERDISTRIBUSI%20e08286d2c95a4675a030fe661a2f50d4/d21a3a89-6fe4-412b-9cf2-f1e54a5d9587.png)
        
        ![Untitled](UAS%20SISTEM%20TERDISTRIBUSI%20e08286d2c95a4675a030fe661a2f50d4/Untitled%201.png)
        
    - pada direktori handlers kita dapat menambahkan file dengan nama `main.yml` seperti pada gambar dibawah ini.
        
        ![Untitled](UAS%20SISTEM%20TERDISTRIBUSI%20e08286d2c95a4675a030fe661a2f50d4/215397b1-c119-4780-bd8c-7899a6161769.png)
        
- pada direktori templates kita dapat menambahkan 3 scrip file seperti pada skrip dibawah ini. dengan nama php7.conf
    
    ```jsx
    ; Start a new pool named 'www'.
    ; the variable $pool can we used in any directive and will be replaced by the
    ; pool name ('www' here)
    [www]
    
    ; Per pool prefix
    ; It only applies on the following directives:
    ; - 'slowlog'
    ; - 'listen' (unixsocket)
    ; - 'chroot'
    ; - 'chdir'
    ; - 'php_values'
    ; - 'php_admin_values'
    ; When not set, the global prefix (or /usr) applies instead.
    ; Note: This directive can also be relative to the global prefix.
    ; Default Value: none
    ;prefix = /path/to/pools/$pool
    
    ; Unix user/group of processes
    ; Note: The user is mandatory. If the group is not set, the default user's group
    ;       will be used.
    user = www-data
    group = www-data
    
    ; The address on which to accept FastCGI requests.
    ; Valid syntaxes are:
    ;   'ip.add.re.ss:port'    - to listen on a TCP socket to a specific address on
    ;                            a specific port;
    ;   'port'                 - to listen on a TCP socket to all addresses on a
    ;                            specific port;
    ;   '/path/to/unix/socket' - to listen on a unix socket.
    ; Note: This value is mandatory.
    listen = 127.0.0.1:9001
    
    ; Set listen(2) backlog. A value of '-1' means unlimited.
    ; Default Value: 128 (-1 on FreeBSD and OpenBSD)
    ;listen.backlog = -1
    
    ; Set permissions for unix socket, if one is used. In Linux, read/write
    ; permissions must be set in order to allow connections from a web server. Many
    ; BSD-derived systems allow connections regardless of permissions.
    ; Default Values: user and group are set as the running user
    ;                 mode is set to 0666
    listen.owner = www-data
    listen.group = www-data
    listen.mode = 0666
    
    ; List of ipv4 addresses of FastCGI clients which are allowed to connect.
    ; Equivalent to the FCGI_WEB_SERVER_ADDRS environment variable in the original
    ; PHP FCGI (5.2.2+). Makes sense only with a tcp listening socket. Each address
    ; must be separated by a comma. If this value is left blank, connections will be
    ; accepted from any ip address.
    ; Default Value: any
    ;listen.allowed_clients = 127.0.0.1
    
    ; Choose how the process manager will control the number of child processes.
    ; Possible Values:
    ;   static  - a fixed number (pm.max_children) of child processes;
    ;   dynamic - the number of child processes are set dynamically based on the
    ;             following directives. With this process management, there will be
    ;             always at least 1 children.
    ;             pm.max_children      - the maximum number of children that can
    ;                                    be alive at the same time.
    ;             pm.start_servers     - the number of children created on startup.
    ;             pm.min_spare_servers - the minimum number of children in 'idle'
    ;                                    state (waiting to process). If the number
    ;                                    of 'idle' processes is less than this
    ;                                    number then some children will be created.
    ;             pm.max_spare_servers - the maximum number of children in 'idle'
    ;                                    state (waiting to process). If the number
    ;                                    of 'idle' processes is greater than this
    ;                                    number then some children will be killed.
    ;  ondemand - no children are created at startup. Children will be forked when
    ;             new requests will connect. The following parameter are used:
    ;             pm.max_children           - the maximum number of children that
    ;                                         can be alive at the same time.
    ;             pm.process_idle_timeout   - The number of seconds after which
    ;                                         an idle process will be killed.
    ; Note: This value is mandatory.
    pm = dynamic
    
    ; The number of child processes to be created when pm is set to 'static' and the
    ; maximum number of child processes when pm is set to 'dynamic' or 'ondemand'.
    ; This value sets the limit on the number of simultaneous requests that will be
    ; served. Equivalent to the ApacheMaxClients directive with mpm_prefork.
    ; Equivalent to the PHP_FCGI_CHILDREN environment variable in the original PHP
    ; CGI. The below defaults are based on a server without much resources. Don't
    ; forget to tweak pm.* to fit your needs.
    ; Note: Used when pm is set to 'static', 'dynamic' or 'ondemand'
    ; Note: This value is mandatory.
    pm.max_children = 5
    
    ; The number of child processes created on startup.
    ; Note: Used only when pm is set to 'dynamic'
    ; Default Value: min_spare_servers + (max_spare_servers - min_spare_servers) / 2
    pm.start_servers = 2
    
    ; The desired minimum number of idle server processes.
    ; Note: Used only when pm is set to 'dynamic'
    ; Note: Mandatory when pm is set to 'dynamic'
    pm.min_spare_servers = 1
    
    ; The desired maximum number of idle server processes.
    ; Note: Used only when pm is set to 'dynamic'
    ; Note: Mandatory when pm is set to 'dynamic'
    pm.max_spare_servers = 3
    
    ; The number of seconds after which an idle process will be killed.
    ; Note: Used only when pm is set to 'ondemand'
    ; Default Value: 10s
    ;pm.process_idle_timeout = 10s;
    
    ; The number of requests each child process should execute before respawning.
    ; This can be useful to work around memory leaks in 3rd party libraries. For
    ; endless request processing specify '0'. Equivalent to PHP_FCGI_MAX_REQUESTS.
    ; Default Value: 0
    pm.max_requests = 100
    
    ; The URI to view the FPM status page. If this value is not set, no URI will be
    ; recognized as a status page. It shows the following informations:
    ;   pool                 - the name of the pool;
    ;   process manager      - static, dynamic or ondemand;
    ;   start time           - the date and time FPM has started;
    ;   start since          - number of seconds since FPM has started;
    ;   accepted conn        - the number of request accepted by the pool;
    ;   listen queue         - the number of request in the queue of pending
    ;                          connections (see backlog in listen(2));
    ;   max listen queue     - the maximum number of requests in the queue
    ;                          of pending connections since FPM has started;
    ;   listen queue len     - the size of the socket queue of pending connections;
    ;   idle processes       - the number of idle processes;
    ;   active processes     - the number of active processes;
    ;   total processes      - the number of idle + active processes;
    ;   max active processes - the maximum number of active processes since FPM
    ;                          has started;
    ;   max children reached - number of times, the process limit has been reached,
    ;                          when pm tries to start more children (works only for
    ;                          pm 'dynamic' and 'ondemand');
    ; Value are updated in real time.
    ; Example output:
    ;   pool:                 www
    ;   process manager:      static
    ;   start time:           01/Jul/2011:17:53:49 +0200
    ;   start since:          62636
    ;   accepted conn:        190460
    ;   listen queue:         0
    ;   max listen queue:     1
    ;   listen queue len:     42
    ;   idle processes:       4
    ;   active processes:     11
    ;   total processes:      15
    ;   max active processes: 12
    ;   max children reached: 0
    ;
    ; By default the status page output is formatted as text/plain. Passing either
    ; 'html', 'xml' or 'json' in the query string will return the corresponding
    ; output syntax. Example:
    ;   http://www.foo.bar/status
    ;   http://www.foo.bar/status?json
    ;   http://www.foo.bar/status?html
    ;   http://www.foo.bar/status?xml
    ;
    ; By default the status page only outputs short status. Passing 'full' in the
    ; query string will also return status for each pool process.
    ; Example:
    ;   http://www.foo.bar/status?full
    ;   http://www.foo.bar/status?json&full
    ;   http://www.foo.bar/status?html&full
    ;   http://www.foo.bar/status?xml&full
    ; The Full status returns for each process:
    ;   pid                  - the PID of the process;
    ;   state                - the state of the process (Idle, Running, ...);
    ;   start time           - the date and time the process has started;
    ;   start since          - the number of seconds since the process has started;
    ;   requests             - the number of requests the process has served;
    ;   request duration     - the duration in µs of the requests;
    ;   request method       - the request method (GET, POST, ...);
    ;   request URI          - the request URI with the query string;
    ;   content length       - the content length of the request (only with POST);
    ;   user                 - the user (PHP_AUTH_USER) (or '-' if not set);
    ;   script               - the main script called (or '-' if not set);
    ;   last request cpu     - the %cpu the last request consumed
    ;                          it's always 0 if the process is not in Idle state
    ;                          because CPU calculation is done when the request
    ;                          processing has terminated;
    ;   last request memory  - the max amount of memory the last request consumed
    ;                          it's always 0 if the process is not in Idle state
    ;                          because memory calculation is done when the request
    ;                          processing has terminated;
    ; If the process is in Idle state, then informations are related to the
    ; last request the process has served. Otherwise informations are related to
    ; the current request being served.
    ; Example output:
    ;   ************************
    ;   pid:                  31330
    ;   state:                Running
    ;   start time:           01/Jul/2011:17:53:49 +0200
    ;   start since:          63087
    ;   requests:             12808
    ;   request duration:     1250261
    ;   request method:       GET
    ;   request URI:          /test_mem.php?N=10000
    ;   content length:       0
    ;   user:                 -
    ;   script:               /home/fat/web/docs/php/test_mem.php
    ;   last request cpu:     0.00
    ;   last request memory:  0
    ;
    ; Note: There is a real-time FPM status monitoring sample web page available
    ;       It's available in: ${prefix}/share/fpm/status.html
    ;
    ; Note: The value must start with a leading slash (/). The value can be
    ;       anything, but it may not be a good idea to use the .php extension or it
    ;       may conflict with a real PHP file.
    ; Default Value: not set
    pm.status_path = /php-status
    
    ; The ping URI to call the monitoring page of FPM. If this value is not set, no
    ; URI will be recognized as a ping page. This could be used to test from outside
    ; that FPM is alive and responding, or to
    ; - create a graph of FPM availability (rrd or such);
    ; - remove a server from a group if it is not responding (load balancing);
    ; - trigger alerts for the operating team (24/7).
    ; Note: The value must start with a leading slash (/). The value can be
    ;       anything, but it may not be a good idea to use the .php extension or it
    ;       may conflict with a real PHP file.
    ; Default Value: not set
    ;ping.path = /ping
    
    ; This directive may be used to customize the response of a ping request. The
    ; response is formatted as text/plain with a 200 response code.
    ; Default Value: pong
    ;ping.response = pong
    
    ; The access log file
    ; Default: not set
    ;access.log = log/$pool.access.log
    
    ; The access log format.
    ; The following syntax is allowed
    ;  %%: the '%' character
    ;  %C: %CPU used by the request
    ;      it can accept the following format:
    ;      - %{user}C for user CPU only
    ;      - %{system}C for system CPU only
    ;      - %{total}C  for user + system CPU (default)
    ;  %d: time taken to serve the request
    ;      it can accept the following format:
    ;      - %{seconds}d (default)
    ;      - %{miliseconds}d
    ;      - %{mili}d
    ;      - %{microseconds}d
    ;      - %{micro}d
    ;  %e: an environment variable (same as $_ENV or $_SERVER)
    ;      it must be associated with embraces to specify the name of the env
    ;      variable. Some exemples:
    ;      - server specifics like: %{REQUEST_METHOD}e or %{SERVER_PROTOCOL}e
    ;      - HTTP headers like: %{HTTP_HOST}e or %{HTTP_USER_AGENT}e
    ;  %f: script filename
    ;  %l: content-length of the request (for POST request only)
    ;  %m: request method
    ;  %M: peak of memory allocated by PHP
    ;      it can accept the following format:
    ;      - %{bytes}M (default)
    ;      - %{kilobytes}M
    ;      - %{kilo}M
    ;      - %{megabytes}M
    ;      - %{mega}M
    ;  %n: pool name
    ;  %o: ouput header
    ;      it must be associated with embraces to specify the name of the header:
    ;      - %{Content-Type}o
    ;      - %{X-Powered-By}o
    ;      - %{Transfert-Encoding}o
    ;      - ....
    ;  %p: PID of the child that serviced the request
    ;  %P: PID of the parent of the child that serviced the request
    ;  %q: the query string
    ;  %Q: the '?' character if query string exists
    ;  %r: the request URI (without the query string, see %q and %Q)
    ;  %R: remote IP address
    ;  %s: status (response code)
    ;  %t: server time the request was received
    ;      it can accept a strftime(3) format:
    ;      %d/%b/%Y:%H:%M:%S %z (default)
    ;  %T: time the log has been written (the request has finished)
    ;      it can accept a strftime(3) format:
    ;      %d/%b/%Y:%H:%M:%S %z (default)
    ;  %u: remote user
    ;
    ; Default: "%R - %u %t \"%m %r\" %s"
    ;access.format = %R - %u %t "%m %r%Q%q" %s %f %{mili}d %{kilo}M %C%%
    
    ; The log file for slow requests
    ; Default Value: not set
    ; Note: slowlog is mandatory if request_slowlog_timeout is set
    ;slowlog = log/$pool.log.slow
    
    ; The timeout for serving a single request after which a PHP backtrace will be
    ; dumped to the 'slowlog' file. A value of '0s' means 'off'.
    ; Available units: s(econds)(default), m(inutes), h(ours), or d(ays)
    ; Default Value: 0
    ;request_slowlog_timeout = 0
    
    ; The timeout for serving a single request after which the worker process will
    ; be killed. This option should be used when the 'max_execution_time' ini option
    ; does not stop script execution for some reason. A value of '0' means 'off'.
    ; Available units: s(econds)(default), m(inutes), h(ours), or d(ays)
    ; Default Value: 0
    ;request_terminate_timeout = 0
    
    ; Set open file descriptor rlimit.
    ; Default Value: system defined value
    ;rlimit_files = 1024
    
    ; Set max core size rlimit.
    ; Possible Values: 'unlimited' or an integer greater or equal to 0
    ; Default Value: system defined value
    ;rlimit_core = 0
    
    ; Chroot to this directory at the start. This value must be defined as an
    ; absolute path. When this value is not set, chroot is not used.
    ; Note: you can prefix with '$prefix' to chroot to the pool prefix or one
    ; of its subdirectories. If the pool prefix is not set, the global prefix
    ; will be used instead.
    ; Note: chrooting is a great security feature and should be used whenever
    ;       possible. However, all PHP paths will be relative to the chroot
    ;       (error_log, sessions.save_path, ...).
    ; Default Value: not set
    ;chroot =
    
    ; Chdir to this directory at the start.
    ; Note: relative path can be used.
    ; Default Value: current directory or / when chroot
    chdir = /
    
    ; Redirect worker stdout and stderr into main error log. If not set, stdout and
    ; stderr will be redirected to /dev/null according to FastCGI specs.
    ; Note: on highloaded environement, this can cause some delay in the page
    ; process time (several ms).
    ; Default Value: no
    catch_workers_output = yes
    
    ; Limits the extensions of the main script FPM will allow to parse. This can
    ; prevent configuration mistakes on the web server side. You should only limit
    ; FPM to .php extensions to prevent malicious users to use other extensions to
    ; exectute php code.
    ; Note: set an empty value to allow all extensions.
    ; Default Value: .php
    ;security.limit_extensions = .php .php3 .php4 .php5 .php7
    
    ; Pass environment variables like LD_LIBRARY_PATH. All $VARIABLEs are taken from
    ; the current environment.
    ; Default Value: clean env
    ;env[HOSTNAME] = $HOSTNAME
    env[PATH] = /srv/www/phpcs/scripts/:/usr/local/bin:/usr/bin:/bin
    ;env[TMP] = /tmp
    ;env[TMPDIR] = /tmp
    ;env[TEMP] = /tmp
    
    ; Additional php.ini defines, specific to this pool of workers. These settings
    ; overwrite the values previously defined in the php.ini. The directives are the
    ; same as the PHP SAPI:
    ;   php_value/php_flag             - you can set classic ini defines which can
    ;                                    be overwritten from PHP call 'ini_set'.
    ;   php_admin_value/php_admin_flag - these directives won't be overwritten by
    ;                                     PHP call 'ini_set'
    ; For php_*flag, valid values are on, off, 1, 0, true, false, yes or no.
    
    ; Defining 'extension' will load the corresponding shared extension from
    ; extension_dir. Defining 'disable_functions' or 'disable_classes' will not
    ; overwrite previously defined php.ini values, but will append the new value
    ; instead.
    
    ; Note: path INI options can be relative and will be expanded with the prefix
    ; (pool, global or /usr)
    
    ; Default Value: nothing is defined by default except the values in php.ini and
    ;                specified at startup with the -d argument
    ;php_admin_value[sendmail_path] = /usr/sbin/sendmail -t -i -f [www@my.domain.com](mailto:www@my.domain.com)
    ;php_flag[display_errors] = off
    ;php_admin_value[error_log] = /var/log/fpm-php.www.log
    ;php_admin_flag[log_errors] = on
    ;php_admin_value[memory_limit] = 32M
    ```
    
1. **wordpress.conf**
    
    ![Untitled](UAS%20SISTEM%20TERDISTRIBUSI%20e08286d2c95a4675a030fe661a2f50d4/584413f8-2b3d-46a2-af3c-bcc1c8e7b1f4.png)
    
2. **wp.conf**
    
    ![Untitled](UAS%20SISTEM%20TERDISTRIBUSI%20e08286d2c95a4675a030fe661a2f50d4/0b0878f6-5e19-4424-8aae-41b84945e350.png)
    
    ![Untitled](UAS%20SISTEM%20TERDISTRIBUSI%20e08286d2c95a4675a030fe661a2f50d4/4928420c-04fd-4168-94b8-ac8352dd2df7.png)
    
    ![Untitled](UAS%20SISTEM%20TERDISTRIBUSI%20e08286d2c95a4675a030fe661a2f50d4/a183a16c-7f78-40bc-af93-5a58154975ab.png)
    
    - menambahkan direktori yii didalam direktori roles dengan menjalankan perintah `sudo mkdir yii` untuk mengakomodasi skrip ansibel untuk install framework yii.
        - pada direktori yii, kita dapat menambahkan 3 folder lagi untuk mengakomodasi tasks, handlers dan skrip templates pada instalasi yii.
    
    ```jsx
    **sudo mkdir -p yii /tasks
    sudo mkdir -p yii /handlers
    sudo mkdir -p yii /templates**
    ```
    
    - pada direktori tasks kita dapat menambahakn file dengan nama `main.yml` seperti pada gambar dibawah ini.
        
        ![Untitled](UAS%20SISTEM%20TERDISTRIBUSI%20e08286d2c95a4675a030fe661a2f50d4/05c96fd7-88f7-4dc6-a226-c3b7ff593d7f.png)
        
        ![Untitled](UAS%20SISTEM%20TERDISTRIBUSI%20e08286d2c95a4675a030fe661a2f50d4/efa72238-b972-48a2-9b3c-3c7a359a0972.png)
        
    - pada direktori handlers kita dapat menambahkan file dengan nama `main.yml` deperti pada gambar dibawah ini.
        
        ![Untitled](UAS%20SISTEM%20TERDISTRIBUSI%20e08286d2c95a4675a030fe661a2f50d4/ef83da83-f08a-4a08-997c-45aa5a5e7521.png)
        
    - pada direktori templates kita dapat menambahkan file dengan nama `yii.conf` seperti pada gambar dibawah ini.
        
        ![Untitled](UAS%20SISTEM%20TERDISTRIBUSI%20e08286d2c95a4675a030fe661a2f50d4/8aedb829-c098-4f4c-ae67-544ad097cadf.png)
        
        ![Untitled](UAS%20SISTEM%20TERDISTRIBUSI%20e08286d2c95a4675a030fe661a2f50d4/c0f16244-242f-4fbb-a2e6-bc0052be2a73.png)
        
        ![Untitled](UAS%20SISTEM%20TERDISTRIBUSI%20e08286d2c95a4675a030fe661a2f50d4/5225f110-16d0-473b-ba36-35d83b755473.png)
        
    - selanjutnya, kita dapat menambahkan file untuk menjalankan konfigurasi dan membuat file direktori
    - membuat file dengan nama hosts dengan menjalankan perintah `sudo nano hosts` seperti pada gambar dibawah ini.
        
        ![Untitled](UAS%20SISTEM%20TERDISTRIBUSI%20e08286d2c95a4675a030fe661a2f50d4/736d17ea-3e54-4dbb-9385-529bbfa3fd04.png)
        
    - membuat file dengan nama install-codeigniter.yml dengan menjalankan perintah `sudo nano install-codeigniter.yml` seperti pada gambar dibawah ini.
        
        ![Untitled](UAS%20SISTEM%20TERDISTRIBUSI%20e08286d2c95a4675a030fe661a2f50d4/74e542fe-8251-457f-aef7-4c65dcc1d5c6.png)
        
    - membuat file dengan nama install-laravel.yml dengan menjalankan perintah `sudo nano install-laravel.yml` seperti pada gambar dibawah ini.
        
        ![Untitled](UAS%20SISTEM%20TERDISTRIBUSI%20e08286d2c95a4675a030fe661a2f50d4/3347115a-7f7c-485e-9508-eb4d3eab4836.png)
        
    - membuat file dengan nama install mariadb.yml dengan menjalankan perintah `sudo nano install-wordpress.yml` seperti pada gambar dibawah ini.
        
        ![Untitled](UAS%20SISTEM%20TERDISTRIBUSI%20e08286d2c95a4675a030fe661a2f50d4/eb17ce26-1153-4a1a-a1e2-7ca223ef4d99.png)
        
    - membuat file dengan nama install-wordpress.yml dengan menjalankan perintah `sudo nano install-wordpress.yml` seperti pada gambar dibawah ini.
        
        ![Untitled](UAS%20SISTEM%20TERDISTRIBUSI%20e08286d2c95a4675a030fe661a2f50d4/9a58badd-02f5-4c1d-be5b-72de7dbd983b.png)
        
        ![Untitled](UAS%20SISTEM%20TERDISTRIBUSI%20e08286d2c95a4675a030fe661a2f50d4/59180059-5c5c-4f48-88af-f9393fe12581.png)
        
    - membuat file dengan nama install-yii.yml dengan menjalankan perintah `sudo nano install-yii.yml` seperti pada gambar dibawah ini.
        
        ![Untitled](UAS%20SISTEM%20TERDISTRIBUSI%20e08286d2c95a4675a030fe661a2f50d4/6d5236f1-d74b-4b40-9681-cd98206763b8.png)
        
    - Kemudian yang terakhir, kita perlu mengkonfigurasi pengaturan nginx. Buka cd situs yang tersedia `/etc/nginx/sites-available` dan buat file bernama kelompok12.fpsas. Masukkan file menggunakan `sudo nano kelompok10.fpsas` dan ketik skrip seperti di bawah ini untuk mengatur nginx termasuk konfigurasi load balancer.
        
        ![Untitled](UAS%20SISTEM%20TERDISTRIBUSI%20e08286d2c95a4675a030fe661a2f50d4/8e8c7300-fef8-4e83-850c-f6e09d7555af.png)
        
        ![Untitled](UAS%20SISTEM%20TERDISTRIBUSI%20e08286d2c95a4675a030fe661a2f50d4/4832af5a-3232-4b96-b50a-471cf916c773.png)
        
        ![Untitled](UAS%20SISTEM%20TERDISTRIBUSI%20e08286d2c95a4675a030fe661a2f50d4/90f87d6e-d4dc-4be2-9c4d-20874a2e5e44.png)
        
    - **selanjutnya cek sckrip dan restart nginx**

```jsx
**sudo nginx -t
sudo nginx -s reload
sudo service nginx restart**
```