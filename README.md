# owncloud-letsencrypt-docker-compose
A docker-compose.yml and supporting files making using of letsencrypt and enabling Big File Uploads support
---
## USE CASE
The purpose of the respository is to offer an example of:
a) You already have a docker environment installed.
b) how to get owncloud running using Docker via 'docker-compose'. The running and management of docker is left up to you. E.g. do not attempt to a shell connection the file time you bring this up, as getting the images to create the container requires yo to obe on the local machine. After the images are obtained, you can ssh in.
b) Using letsencypt to maintain your SSL certificate. n.b. Management of your firewall and routing is left up to you. It is assumed that port 80 and 443 are accessible.
c) You need to manage files that are <=16G.
d) The longest upload is going to tale less than 10 minuntes.

## QUICK SETUP
1. Place these files in the same directory you expect to store the owncloud files
2. Edit the `.env` file with your specific infromation
3. `docker-compose up`
4. `docker-compose down`
5. Edit the owncloud `./data/config/config.php`
6. Insert the following, just before the closing of the file where you find );

```
  'filelocking.enabled' => true,
  'memcache.locking' => '\OC\Memcache\Redis',
  'redis' =>
  array (
    'host' => 'redis',
    'port' => '6379',
    'timeout' => 0,
  ),
```
## OVERVIEW:

### `.env`

This file is used to contain your custom information, e.g. your password for owncloud

### `docker-compose.yml`

This is the file `docker-compose` requires to start up your environment

### `owncloud.apache.php.ini` to override `/etc/php/7.4/apache2/php.ini`

This file overrides what the container produces and changes the timeouts to 10 minutes (600s), so that big file can upload.

Change these values as you need. The unit is seconds:
- `max_execution_time = 600` 
- `max_input_time = 600`

### `owncloud.nginx.conf` to override `/etc/nginx/nginx.conf`

This file makes a few changes to the proxy to allow big file uploads, e.g. file size limit, and time outs to align with owncloud timeout values
```
    #keepalive_timeout  65;
    client_max_body_size 16G;
    keepalive_timeout 10m;
    proxy_connect_timeout  600s;
    proxy_send_timeout  600s;
    proxy_read_timeout  600s;
```
as well as changing the logging level to warn:

```
# error_log  /var/log/nginx/error.log notice;
error_log  /var/log/nginx/error.log warn;
```

### Connect redis - `./data/config/config.php`

Lastly to help with some peformance, redis is conntected to owncloud per above.

Once you have owncloud up and running, shut it down and edit the `./data/config/config.php`. `./data` is a subdirectory you will find in the same directory as the docker-compose.yml file.

```
 'filelocking.enabled' => true,
  'memcache.locking' => '\OC\Memcache\Redis',
  'redis' =>
  array (
    'host' => 'redis',
    'port' => '6379',
    'timeout' => 0,
  ),
```

### CAVEATS

- The location of the `./data`, `./nginx`, and `./override` folders can be moved to where you need. You must update the `docker-compose.yml` if you do change their locations.
