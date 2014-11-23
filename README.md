# docker-localtunnel

[localtunnel.me](http://localtunnel.me/) is a service that allows you to share a
web server running on your local machine (or local network) with a publicly
accessible URL.

This Docker image packages the `lt` tool so you don't have to install node, npm,
or localtunnel on your development machine.

## Diagram

When packaged in a Docker image, the `lt` tool will run inside of a Docker
container, possibly inside a [boot2docker](http://boot2docker.io) VM. Here's the
flow of an HTTP request using this image.

    user's browser -> yourendpoint.localtunnel.me -> lt in docker container
                                                              |
                                                             \|/
                                                     <boot2docker VM host>:8000
                                                             OR
                                                     <dev machine>:8000

The key point is that development webserver needs to be accessible from the
Docker container running `msparks/localtunnel`. That means you won't be able to
listen on localhost (`127.0.0.1` or `[::1]`). Instead, listen on all interfaces
(or at least one accessible from your Docker containers). Start your development
webserver listening on `0.0.0.0` or `[::]` and things should work.

If you're using boot2docker, the IP address defaults are:

* `192.168.59.3` for the VM host side (i.e., the physical machine)
* `192.168.59.103` for the VM guest side (i.e., the boot2docker VM)

These IP addresses are used below in the Usage section.

## Usage

Set up a tunnel to a webserver running on port 8000 on the boot2docker host
machine.

    docker run --rm msparks/localtunnel --local-host 192.168.59.3 --port 8000

Set up a tunnel to a webserver running on port 8000 in another Docker container
running in the boot2docker VM.

    docker run --rm msparks/localtunnel --local-host 192.168.59.103 --port 8000

Set up a tunnel to a webserver running on port 8000 somewhere else accessible
from the Docker container.

    docker run --rm msparks/localtunnel \
      --local-host some.other.example.com --port 8000

## Tips

Here's a quick way to serve static files on the public web from the current
shell directory (`$PWD`). Note that this is the second case above, with two
Docker containers in a boot2docker VM.

First terminal (webserver serving from the current directory):

    docker run --rm -p 8000:80 -v $PWD:/usr/share/nginx/html:ro nginx

Second terminal (localtunnel server):

    docker run --rm msparks/localtunnel --local-host 192.168.59.103 --port 8000
