# Google Play Music Manager for Docker

The [Google Play Music
Manager](https://support.google.com/googleplaymusic/answer/1075570?hl=en) is
becoming increasingly hard to run, depending on deprecated libraries with known
security vulnerabilities, like QtWebKit.  This Docker image is inteded to keep
Music Manager alive in a stable and semi-isolated environment long after
distros have removed its dependencies.

This image is loosely modeled after
[`ruippeixotog/google-musicmanager`](https://hub.docker.com/r/ruippeixotog/google-musicmanager/),
but modified to run as a client on your host's X server like any other X
application, rather than running in an embedded Xvfb server.

## Installation

```
docker create \
  --name=google-musicmanager \
  --ipc=host \
  --net=host \
  -e DISPLAY \
  -v $XAUTHORITY:/root/.Xauthority \
  -v /tmp/.X11-unix \
  -v $HOME/.config/google-musicmanager:/config \
  -v </path/to/music>:/music \
  iamjamestl/google-musicmanager
```

You can optionally replace `--ipc=host` with `-e QT_X11_NO_MITSHM=1` for
increased isolation but lower graphical performance.  You can also remove `-v
$XAUTHORITY:/root/.Xauthority` if you open your X server to local connections
from `root` with `xhost +local:root`.  If you choose to open your X server in
this way, you can also remove the `--net=host` option for increased network
isolation, though be aware that Google Play Music does some form of client
tracking by MAC address, so that could lead to unpredictable behavior.

### Volumes

* `/config`: This is where the Music Manager will store files for preserving
  its configuration between runs.  It also logs to this directory.  In the
  example shown, Docker will map the default configuration directory from your
  host, which is great if you want to migrate from running the app on the host
  into this container.  Otherwise, you can specify any directory you want.
* `/music`: This is the location Music Manager will scan for music to upload to
  Google Play Music.  Map your music library to this volume.  You can map
  multiple music volumes into the container, but you must manually configure
  Music Manager to read from them.

## Starting

Start the container with `docker start google-musicmanager`.  The first time
you start the container with an empty config directory, the application will
launch a Google login screen.  Every time thereafter, the application will
launch directly to a system tray icon.  Click it to restore the main window.
If you do not have a system tray, you can get the main window to show by
running:

```
docker exec google-musicmanager google-musicmanager
```

## Stopping

When you are done uploading your music, you can keep the container running to
monitor your music library for changes, or stop the container with `docker stop
google-musicmanager`.

## Removing

Run `docker rm google-musicmanager`.  You can always restore the container by
specifying the same `/config` and `/music` volumes that you used before.
