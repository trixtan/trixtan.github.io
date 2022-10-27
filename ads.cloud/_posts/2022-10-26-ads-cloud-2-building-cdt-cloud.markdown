---
layout: post
title:  "Building containerized CDT.cloud"
date:   2022-10-26 09:58:54 +0100
---
The first step I decided to do is to install the already existing [CDT.cloud][cdt-cloud] in my remote virtual server (a VPS I have by the provider Hetzner).
I want to manage everything with Docker.

## Building CDT.cloud docker image
The CDT.cloud documentation suggests the command to create the docker image:

```sh
docker build -t cdt-cloud-blueprint:latest -f dockerfile/Dockerfile .
```

### webpack and SIGKILL
I faced the following issue building the Docker image:

```sh
cdt-cloud-blueprint-browser: Error: webpack exited with an unexpected signal: SIGKILL.
cdt-cloud-blueprint-browser:     at ChildProcess.<anonymous> (/home/theia/cdt-cloud-blueprint/node_modules/@theia/application-manager/lib/application-process.js:59:28)
cdt-cloud-blueprint-browser:     at ChildProcess.emit (events.js:400:28)
cdt-cloud-blueprint-browser:     at maybeClose (internal/child_process.js:1088:16)
cdt-cloud-blueprint-browser:     at Socket.<anonymous> (internal/child_process.js:446:11)
cdt-cloud-blueprint-browser:     at Socket.emit (events.js:400:28)
cdt-cloud-blueprint-browser:     at Pipe.<anonymous> (net.js:686:12)
cdt-cloud-blueprint-browser: error Command failed with exit code 1.
cdt-cloud-blueprint-browser: info Visit https://yarnpkg.com/en/docs/cli/run for documentation about this command.
cdt-cloud-blueprint-browser: error Command failed with exit code 1.
cdt-cloud-blueprint-browser: info Visit https://yarnpkg.com/en/docs/cli/run for documentation about this command.
lerna ERR! yarn run prepare exited 1 in 'cdt-cloud-blueprint-browser'
```
Linux has a mechanism that can kill processes when they drain too much resources. Since my server is quite limited in resources (only 2GB of RAM and 1 core), I thought the problem might be that webpack is too heavy for this configuration.

In fact, a call to `dmesg` shows:
```sh
[ 1077.609009] Out of memory: Kill process 9229 (webpack) score 303 or sacrifice child
[ 1077.610169] Killed process 9229 (webpack) total-vm:33465112kB, anon-rss:1166192kB, file-rss:0kB, shmem-rss:0kB
```

#### Solution
1. Build the Docker image on a stronger machine (I used my laptop)
2. Export the image:
	```sh
	docker save cdt-cloud-blueprint | gzip > cdt-cloud-blueprint.tgz
	```
3. Upload the tgzipped image to the remote server
4. Load and retag the image on the server
	```sh
	docker load -i cdt-cloud-blueprint.tgz
	```

[cdt-cloud]: https://github.com/eclipse-cdt-cloud/cdt-cloud
