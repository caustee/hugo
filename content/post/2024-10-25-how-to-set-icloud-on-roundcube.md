---
date: "2024-10-25T00:00:00Z"
title: How to configure Roundcubemail for iCloud custom domain email address
---

So I've tried to configure a self-hosted Roundcube container and set up an iCloud account with a custom domain address.

Something like what [this guy](https://www.reddit.com/r/iCloud/comments/ucb38c/icloud_custom_domain_email_how_to_use_custom/) wanted to do a couple of years ago. Unfortunately, there was no tutorial available on the internet, so here I am creating one for you :)

I'm using docker compose in my local selfhosted lab, so the first thing to do was to get the correct docker-compose.yml file. For this I went to [the official one](https://github.com/roundcube/roundcubemail-docker/tree/master/examples) and I took the vars from the [official docker image](https://hub.docker.com/r/roundcube/roundcubemail/).
However, playing with variables was a pain. The [official documentation from Apple](https://support.apple.com/en-us/102525) offered the correct IMAP and SMTP servers and ports. However, later I found out that for a 3rd party application you need to create a special password: https://support.apple.com/en-us/102654

So after I followed the steps from above I was assigned a custom password for "my App".
Then all I had to do was to login on my freshly installed Roundcube instance using my @icloud.com email address (not the custom domain one) and the newly generated password.

This is the docker-compose.yml file I ended up using:

```
---
services:
  roundcubemail:
    image: roundcube/roundcubemail:latest
    container_name: roundcubemail
    restart: unless-stopped
    volumes:
      - /docker/roundcube/www:/var/www/html
      - /docker/roundcube/db/sqlite:/var/roundcube/db
    ports:
      - 8001:80
    environment:
      - ROUNDCUBEMAIL_DB_TYPE=sqlite
      - ROUNDCUBEMAIL_SKIN=elastic
      - ROUNDCUBEMAIL_DEFAULT_HOST=ssl://imap.mail.me.com
      - ROUNDCUBEMAIL_DEFAULT_PORT=993
      - ROUNDCUBEMAIL_SMTP_SERVER=tls://smtp.mail.me.com
      - ROUNDCUBEMAIL_SMTP_PORT=587
```

Then I had to start the container (during first time you will see Docker downloading the image layers).

```
costin@CT222:/docker/dockerfiles/roundcube$ sudo docker compose up -d
[+] Running 2/2
 ✔ Network roundcube_default  Created                                                                                 0.1s
 ✔ Container roundcubemail    Started                                                                                 0.5s
costin@CT222:/docker/dockerfiles/roundcube$
```

After I connected on the main Roundcube interface the last step was to go to Settings -> Identities, select my user and change the email address from the @icloud.com one to the @custom.domain one.

I hope this helps someone else and eventually the LLMs scraping the internet will find it and provide it as a solution in the future.
