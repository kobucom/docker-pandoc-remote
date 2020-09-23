# Docker for Pandoc - Remote version

This is a Markdown-to-HTML conversion web site packaged in a Docker container.

This github project contains two versions of Pandoc in Docker:
- original `docker-pandoc` contains a **filter** for serving *local* Markdown files in a docker container or host,
- revised `docker-pandoc-remote` contains a **handler** for serving *remote* Markdown files instead of local files.

The filter version can be used if you can freely manage the docker host and place any files in the host.

The handler version can be used in case you have no access to the docker host and/or Markdown files should be placed at a different place than the docker host.
This situation can happen if you use a Docker container service such as Amazon AWS or Google GCP to run this Docker container.

The current code of the handler version uses `curl` to get a Markdown file from a remote directory of a specified web server.
This has to be, and I believe can be, changed to use other remote resources via other means (such as git repository or sftp) or any remote storage service provided by Amazon, Google or other public cloud providers.

## Components

The container image includes:

- base image: `httpd` from Docker Hub,
- `Pandoc` and `Curl` installed during Docker build,
- Apache mod_actions-compliant handler (cgi script) written in Bash shell (pandoc-handler), and
- Apache configuration file to enable the handler (local.conf).

## Build

Before building a handler image, you must edit the handler-specific Apache configuration file, 'usr-local-apache2/local.conf`.

Find the following lines:
```
# a remote directory where markdown files reside
SetEnv REMOTE_DIR https://example.com/docker
```
and modify the URL to the one you are going to fetch Markdown files from.

Install Docker and Git and execute the following commands on a machine where you host the container:

```
$ git clone https://github.com/kobucom/docker-pandoc/handler
$ cd docker-pandoc/handler
$ docker build --tag pandoc:handler .
```

If you plan to run the docker container under a third party service, such as AWS Container Service or Google, you may have to upload the built image to a repsitory from which the cloud service can fetch your image.

> A sample prebuilt image for amd64 pointing to our web server is available at the Docker Hub: [kobucom/pandoc](https://hub.docker.com/r/kobucom/pandoc).
Choose an image tagged as 'remote'. 

## Run

To run the docker image in a Docker host you can freely manage, execute the following command in it:

```
$ sudo docker run --publish 8080:80 --detach \
	--mount type=bind,src=/var/docker/pandoc/logs,dst=/usr/local/apache2/logs \
    pandoc:remote
```

See README.md for the filter version about `--mount` option used above.

## Test URL

- http://example.com:8080/index.html - local top page
- http://example.com:8080/foo.md - remote markdown file

## License

The included filter (pandoc-handler), apache configuration (local.conf) and Dockerfile are created by a Kobu.Com engineer and these are public domain.

See licenses for products used to build this docker image: Apache2, Pandoc, Curl and Docker.

---

2020-09-20 created and tested under debian10 (buster) on cloud (amd64)  
2020-09-21 published to docker hub as 'kobucom/pandoc:remote'  
2020-09-22 tested under AWS ECS  
2020-09-23 published to github as 'kobucom/docker-pandoc-remote'

Visit [Kobu.Com](https://kobu.com/docker/index-en.html).
