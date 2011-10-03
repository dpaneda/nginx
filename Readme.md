# Nginx with syslog rate-limited logging

## Description

Nginx with [1] syslog logging patch and rate-limited log

[1] - https://github.com/yaoweibin/nginx_syslog_patch

## Compilation

./configure --add-module=syslog_patch

## Configuration

### Syslog

Sytax: syslog auth|authpriv|cron|daemon|ftp|kern|local0-7|lpr|mail|news|syslog|user|uucp [program_name]
Default: none
Context: main

Enable the syslog and set its facility. The default program name is nginx.

### Error log

Syntax: error_log [syslog[:emerg|alert|crit|error|warn|notice|info|debug]]['|'file] [ debug | info | notice | warn | error | crit ] [rate_limit=limit]
Default: ${prefix}/logs/error.log
Context: main, http, server, location

Enable the error_log with the syslog and set its priority. The default priority is error. The error log can be sent to syslog, file or both. 
If the rate-limit option is not used, it defaults to 1 (log everything).

Note: if you set the error_log directive in the main block, the syslog is switched on by default.

### Access log

Syntax: access_log off|[syslog[:emerg|alert|crit|error|warn|notice|info|debug]]['|'path] [format [buffer=size] [rate_limit=limit]]
Default: access_log logs/access.log combined
Context: http, server, location

Enable the access_log with the syslog and set its priority. The default priority is notice. The access log can be sent to syslog, file or both.
If the rate-limit option is not used, it defaults to 1 (log everything).


## Example

Example configuration

```
worker_processes  1;  

#Activate syslog logging and define facility and program name
syslog local6 nginx;

events {
        worker_connections  1024;
}

http {
    include       mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] $request '
        '"$status" $body_bytes_sent "$http_referer" '
        '"$http_user_agent" "$http_x_forwarded_for"';

    server {
        listen       80; 
        server_name  localhost;

        #send the log to syslog and file.
        # Log all error logs and a 1 % of access logs
        access_log  syslog:notice|logs/host1.access.log main rate_limited=100;
        error_log syslog:notice|logs/host1.error.log;

        location / { 
            root   html;
            index  index.html index.htm;
        }   
    }   

    server {
        listen       80; 
        server_name  www.example.com;

        access_log  syslog:warn|logs/host2.access.log main;
        error_log syslog:warn|logs/host2.error.log;

        location / { 
            root   html;
            index  index.html index.htm;
        }   
    }   

    server {
        listen       80; 
        server_name  www.test.com;

        #send the log just to syslog.
        access_log  syslog:error main;
        error_log syslog:error;

        location / { 
            root   html;
            index  index.html index.htm;
        }   
    }   
}
```
