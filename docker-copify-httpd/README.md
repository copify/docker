## Docker image: Apache/2.4.10 (Debian)

View full details at [copify/httpd](https://hub.docker.com/r/copify/httpd/) on Docker hub.

## Requirements

#### Linux

Docker requires a 64-bit installation regardless of your Linux flavour. Additionally, your kernel must be 3.10 at minimum. The latest 3.10 minor version or a newer maintained version are also acceptable.

#### Docker

Version >= 1.9
https://docs.docker.com/engine/installation/

## Installation

Once Docker is installed, pull the image.

`docker pull copify/httpd`

Run the image, exposing port 80 to the host machine, and mount the directory you want as your DocumentRoot.

`docker run -d -p 80:80 -v /local/path:/var/www/html copify/httpd`

Apache will now be running at http://127.0.0.1 and serving whatever files are mounted from  `/local/path`. You can edit these files in real-time without restarting the Docker container.

If you need to attach to the running container;

`docker exec -it {container id} bash`

To detach, use `CTRL + P` and then `CTRL + Q`.
