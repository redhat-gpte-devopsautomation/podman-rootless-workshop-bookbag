
# Running a Rootless Container With Podman, Buildah, and Skopeo on RHEL

## 1. Download podman, buildah, skopeo and check versions:
```bash
sudo yum install -y buildah podman skopeo
buildah --version
podman --version
skopeo --version
```
## 2. Create an image from a container file (simple httpd web service):

Create a new container from httpd
```bash
container=$(buildah from docker.io/library/httpd:latest)
```

Create html directory

```bash
buildah run $container mkdir -p /usr/local/apache2/htdocs/
```

Create html file

```bash
echo "<html><body><h1>Welcome to my web service</h1></body></html>" | buildah run $container tee /usr/local/apache2/htdocs/index.html
```

Commit the changes to an image named mywebapp

```bash
buildah commit $container mywebapp
```
## 3. Push to registry:
```bash
buildah login quay.io
buildah push mywebapp:latest docker://quay.io/<Quay Username>/mywebapp:latest
```

## 4. Create the container by pulling the image from registry:

Inspect the image in the repository before pulling
```bash
skopeo inspect docker://quay.io/<Quay Username>/mywebapp:latest
```

Pull the image from the repository and run it

```bash
podman pull quay.io/<Quay Username>/mywebapp:latest
podman run -d --name mywebapp-demo -p 8080:80 quay.io/<Quay Username>/mywebapp:latest
```
## 5. Check the status:
```bash
podman ps
```
## 6. Check if the web service is accessible using curl:
```bash
curl -N localhost:8080
```
## 7. Create a systemd service to get it started after system reboot automatically:

Create a systemd service file to manage the `mywebapp-demo` container

```bash
mkdir -p ~/.config/systemd/user/
cd ~/.config/systemd/user
```

Create the unit file for the `mywebapp-demo` container

```bash
podman generate systemd --name mywebapp-demo --files --new
```

Stop and then delete the `mywebapp-demo` container.

```bash
podman stop mywebapp-demo
podman rm mywebapp-demo
```

Reload the systemd daemon configuration, and then enable and start your new `container-mywebapp-demo` user service

```bash
systemctl --user daemon-reload
systemctl --user enable --now container-mywebapp-demo
```

Verify that the web server responds to requests.

```bash
curl http://localhost:8080
```

Verify that the container is running.

```bash
podman ps
```

Use the container ID information to confirm that the systemd daemon creates a container when you restart the service.

```bash
systemctl --user stop container-mywebapp-demo
podman ps --all
systemctl --user start container-mywebapp-demo
podman ps
```

Ensure that the services for your user start at system boot, then restart your machine.

```bash
loginctl enable-linger
loginctl show-user adamz
sudo systemctl reboot
```


Log back in and verify that the systemd daemon started the `mywebapp-demo` container, and that the web content is available.

```bash
ssh node1
podman ps
curl http://localhost:8080

```
##  Done!

To clear containers and user:

```bash
podman rm -af
podman rmi -af
exit
pkill -9 -u student
```
