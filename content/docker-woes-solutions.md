Title: Docker inheirtance of configuration: a source of devops headaches
Date: 2022-01-17
Category: Writeups
Tags: docker
Status: draft

Problem
-------

mysql stops reading unicode. the docker library/mysql image uses debian:ver-slim which has reduced its disk size by not including locales. mysql freaks out without locale information and starts running latin1! The docs say it should be "utf8" a 3 byte limited encoding that becomes outdated with 8 or whatever and we use the m4 one in the future. pulls being unsafe.

Mitigation
----------

OS-Query scanning -> diffs

