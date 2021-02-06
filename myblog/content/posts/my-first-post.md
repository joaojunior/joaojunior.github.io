---
title: "Chrome with selenium, python and docker"
date: 2019-09-16T09:15:55-04:00
draft: false
tags: ["Python", "Selenium", "Chrome", "Docker", "Crashed", "WebDriverException"]
---

Sometimes, when I execute the Chrome inside a docker container I receive the error below:

``selenium.common.exceptions.WebDriverException: Message: unknown error: Chrome failed to start: crashed``

It is caused because Chrome uses the `/dev/shm` to share memory and the docker, by default, set 64MB for this partition.

Here, I will describe how to reproduce this error and how to resolve it.

How to reproduce
================

To reproduce this error, we need to create a docker image with chrome and python. Then, I created this [Dockerfile](https://github.com/joaojunior/python_selenium_chrome_docker/blob/master/Dockerfile) which uses a [multi-stage build](https://docs.docker.com/develop/develop-images/multistage-build/) to install the chrome and the python.

After clone this [repository](https://github.com/joaojunior/python_selenium_chrome_docker
), we need to create the image with the command:

``docker build -t python_selenium_chrome .``

And then, we can run one container with the command:

``docker run --shm-size=1b -it python_selenium_chrome bash``

Finally, we call the chrome from python with the command:
``python run_chrome.py``

After you run it, you will receive one message error similar as we have shown before(in the first session).

How to resolve
==============
To resolve this problem we only need to configure the correct size of the `/dev/shm`. We can run the container with the command:

``docker run --shm-size=1g -it python_selenium_chrome bash``

Now, we are running the container with 1 gigabyte of memory to `/dev/shm`. After this is possible to run the chrome normally inside the container.

References
==========
 - https://developers.google.com/web/tools/puppeteer/troubleshooting
 - https://docs.docker.com/engine/reference/run/
