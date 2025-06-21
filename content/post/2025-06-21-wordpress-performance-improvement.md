---
date: "2025-06-21T00:00:00Z"
title: How to improve Wordpress performance
---

In this one I will show you how I improved the performance of a WordPress site using CloudFlare Caching and PHP FPM pool settings.

But a bit of context. I have 2 WP websites running on the same VPS behind nginx, with 2 different conf file. Pretty standard deployment so far. On this VPS I have root access, so I'm able to play a bit more with other stuff, not just standard cPanel settings.

Now, because I have 2 websites running on same server, I have one that I want to prioritize and not have the other one consume all the resources. So for this I've created 2 separate fpm pools with different settings:

```
[first]
user = www-data
group = www-data
listen = /run/php/php8.3-fpm-first.sock
pm = dynamic
pm.max_children = 15
pm.start_servers = 3
pm.min_spare_servers = 2
pm.max_spare_servers = 6
pm.max_requests = 500
```
```
[second]
user = www-data
group = www-data
listen = /run/php/php8.3-fpm-second.sock
listen.owner = www-data
listen.group = www-data
pm = dynamic
pm.max_children = 5
pm.start_servers = 1
pm.min_spare_servers = 1
pm.max_spare_servers = 2
pm.max_requests = 200
```
now, on top of this, in nginx I've added:
* to second one:
```
limit_req_zone $binary_remote_addr zone=throttled:10m rate=3r/s;
```
plus some other rate limits to php processing. 

* and this to first one:
```
limit_req_zone $binary_remote_addr zone=priority:10m rate=10r/s;
```

beside this, I've enabled Caching in CloudFlare, and here are the results for the prioritized website:
* before:
```
 ~/ ab -k -c 10 -n 100 https://redacted.website/                                                                                                                                                                                                     [85%] ...
This is ApacheBench, Version 2.3 <$Revision: 1913912 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/

Benchmarking redacted.website (be patient)...
..done


Server Software:        cloudflare
Server Hostname:        redacted.website
Server Port:            443
SSL/TLS Protocol:       TLSv1.2,ECDHE-ECDSA-CHACHA20-POLY1305,256,256
Server Temp Key:        ECDH X25519 253 bits
TLS Server Name:        redacted.website

Document Path:          /
Document Length:        173176 bytes

Concurrency Level:      10
Time taken for tests:   79.794 seconds
Complete requests:      100
Failed requests:        0
Keep-Alive requests:    0
Total transferred:      17455892 bytes
HTML transferred:       17317600 bytes
Requests per second:    1.25 [#/sec] (mean)
Time per request:       7979.403 [ms] (mean)
Time per request:       797.940 [ms] (mean, across all concurrent requests)
Transfer rate:          213.63 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:       21   36  42.9     27     409
Processing:  1445 7573 1557.1   7466   13679
Waiting:     1433 7473 1571.7   7397   13582
Total:       1499 7609 1559.1   7512   13731

Percentage of the requests served within a certain time (ms)
  50%   7512
  66%   7943
  75%   8347
  80%   8515
  90%   9427
  95%  10245
  98%  11821
  99%  13731
 100%  13731 (longest request)
```

* after:
```
 ~/ ab -k -c 10 -n 100 https://redacted.website/                                                       
This is ApacheBench, Version 2.3 <$Revision: 1913912 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/

Benchmarking redacted.website (be patient).....done


Server Software:        cloudflare
Server Hostname:        redacted.website
Server Port:            443
SSL/TLS Protocol:       TLSv1.2,ECDHE-ECDSA-CHACHA20-POLY1305,256,256
Server Temp Key:        ECDH X25519 253 bits
TLS Server Name:        redacted.website

Document Path:          /
Document Length:        171448 bytes

Concurrency Level:      10
Time taken for tests:   32.678 seconds
Complete requests:      100
Failed requests:        0
Keep-Alive requests:    0
Total transferred:      17297624 bytes
HTML transferred:       17144800 bytes
Requests per second:    3.06 [#/sec] (mean)
Time per request:       3267.803 [ms] (mean)
Time per request:       326.780 [ms] (mean, across all concurrent requests)
Transfer rate:          516.93 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:       22   36  16.8     31     139
Processing:   746 3101 635.8   3127    4682
Waiting:      733 2957 638.8   2987    4609
Total:        805 3137 632.5   3160    4741

Percentage of the requests served within a certain time (ms)
  50%   3160
  66%   3323
  75%   3458
  80%   3616
  90%   3782
  95%   4073
  98%   4573
  99%   4741
 100%   4741 (longest request)
```

As you can see the performance gains are:
* Requests per second: 1.25 → 3.06 (**+145% improvement**)
* Total test time: 79.8s → 32.7s (**59% faster**)
* Mean response time: 7979ms → 3268ms (**59% faster**)
* Transfer rate: 213 KB/s → 517 KB/s (**+142% improvement**)


If you're struggling with your wordpress website performance, send me a message through contact form.
