Nomad podman Driver (Unoffical)
==================

This is a modified version of the Hashicorp Nomad podman driver with some extra features. It was forked from version 0.6.1 but I try to merge all new releases into this branch whilst keeping the extra features (that I personally use) working.

## Extra features

* Support for multiple podman `socket` blocks. This allows you to run each task under a different podman socket, segregating workloads from running (by accident?) under the same host user.
* Fixed journald logging empty lines
* Allow the use of `container_user` in the task to overwrite the user that executes the entrypoint in the container.

## Thanks

Many thanks to [@towe75](https://github.com/towe75) and [Pascom](https://www.pascom.net/) for contributing this plugin to Nomad!

## Example of using multiple sockets

In your Nomad client config, update the plugin driver with all the sockets:

```hcl
plugin "nomad-driver-podman" {
  config {
    socket {
      default = true
      name = "giteaApplication"
      host_user = "gitea"
      socket_path = "unix://run/user/1000/podman/podman.sock"
    }
    socket {
      name = "navidromeApplication"
      host_user = "navidrome"
      socket_path = "unix://run/user/1001/podman/podman.sock"
    }
  }
}
```

You can then reference the socket name in the tasks:

```hcl
job "deploy_applications" {
  datacenters = ["GRA"]

  group "reggie" {
    network {
      port "gitea_port" {
        to = 3000
      }
      port "navidrome_port" {
        to = 4533
      }
    }   

    task "deploy_gitea_container" {
      driver = "podman"
      config {
        image = "localhost/mygitea:latest"
        ports = ["gitea_port"]
        privileged = false
        socket = "giteaApplication"
        logging = {
          driver = "journald"
          options = [
            {
              "tag" = "gitea"
            }
          ]
        }
      }
    }   

    task "deploy_navidrome_container" {
      driver = "podman"
      config {
        image = "localhost/mynavidrome:latest"
        ports = ["navidrome_port"]
        privileged = false
        socket = "navidromeApplication"
        logging = {
          driver = "journald"
          options = [
            {
              "tag" = "navidrome"
            }
          ]
        }
      }
    }   
  }
}
```

## Building The Driver from source

This project has a `go.mod` definition. So you can clone it to whatever directory you want.
It is not necessary to setup a go path at all.
Ensure that you use go 1.21 or newer.

```shell-session
git clone git@github.com:hashicorp/nomad-driver-podman
cd nomad-driver-podman
make dev
```

The compiled binary will be located at `./build/nomad-driver-podman`.

If you want to test the driver, install podman then run

```sh
make test
```

