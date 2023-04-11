---
title: Speed up docker pull in China
slug: speed-up-docker-pull-in-china
date: 2022-03-06 00:00:00+0000
categories:
    - dev
tags:
    - docker
---

If you are in China and your docker pull kept failing/retrying, try editing your docker `daemon.json`.

The default location for linux is at `/etc/docker/daemon.json`.

```json
{
  "registry-mirrors": [
    "https://dockerproxy.com"
  ]
}
```

Then restart docker service with:

```bash
sudo systemctl daemon-reload
sudo systemctl restart docker
```

To check if it is correct, run `docker info | grep -A1 'Registry Mirrors:'`

If you can understand Chinese it is better to just go to the website directly and learn, [dockerproxy.com](tab:https://dockerproxy.com/docs)

Another registry mirror that you can use (at this time of writing):

- https://docker.mirrors.ustc.edu.cn/
- https://hub-mirror.c.163.com/
- https://reg-mirror.qiniu.com
