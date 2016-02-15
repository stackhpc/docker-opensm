OpenSM Infiniband Subnet Manager
================================


[![Image Size](https://img.shields.io/imagelayers/image-size/jumanjiman/opensm/latest.svg)](https://imagelayers.io/?images=jumanjiman/opensm:latest 'View image size and layers')&nbsp;
[![Image Layers](https://img.shields.io/imagelayers/layers/jumanjiman/opensm/latest.svg)](https://imagelayers.io/?images=jumanjiman/opensm:latest 'View image size and layers')&nbsp;
[![Docker Registry](https://img.shields.io/docker/pulls/jumanjiman/opensm.svg)](https://registry.hub.docker.com/u/jumanjiman/opensm)&nbsp;
[![Circle CI](https://circleci.com/gh/jumanjihouse/docker-opensm.png?circle-token=49cfeda576ec04d63924da128e314e8796b173fa)](https://circleci.com/gh/jumanjihouse/docker-opensm/tree/master 'View CI builds')

Project URL: [https://github.com/jumanjihouse/docker-opensm](https://github.com/jumanjihouse/docker-opensm)
<br />
Docker hub: [https://registry.hub.docker.com/u/jumanjiman/opensm/](https://registry.hub.docker.com/u/jumanjiman/opensm/)


Overview
--------

OpenSM is an infiniband subnet manager from the
OpenFabrics Enterprise Distribution (OFED).
This repo provides a way to build OpenSM
into a docker image and run it as a container.


### Build integrity

The repo is set up to build the image on CircleCI.

![workflow](assets/docker_hub_workflow.png)

An unattended test harness runs simple test scripts.
If all tests pass on master branch in the unattended test harness,
it pushes the built images to the Docker hub.


How-to
------


### Build locally

On a host with Docker:

    docker build --rm -t opensm src/


### Pull an already-built image

These images are built as part of the test harness on CircleCI.
If all tests pass on master branch, then the image is pushed into the docker hub.

    docker pull jumanjiman/opensm:latest

The `latest` tag always points to the latest version.
Additional tags include `${opensm_rpm_version}-${build_datetime}-${git_hash}`
to correlate any image to both the opensm version and a git commit from this repo.

We push the tags automatically from the test harness, and
we occasionally delete old tags from the Docker hub by hand.


### Run OpenSM

On a host with Docker:

    docker run -d \
      --name opensm.service
      --read-only \
      --memory=10G \
      --privileged \
      --net=host \
      -v /lib/modules:/lib/modules:ro \
      jumanjiman/opensm


### Run diagnostics

The image contains the infiniband-diags package.
You can run the tools in two ways:

* Enter a running opensm container via `docker exec -it <cid> bash`
  and call the utils directly.

* Start a fresh container with something like:

  ```
  docker run --rm -it --net=host --privileged --entrypoint bash jumanjiman/opensm
  ```

Sample output from `ibnetdiscover` (one of the utils) is shown at
[`test/ibnetdiscover.out`](test/ibnetdiscover.out).


### Test

Run the test harness:

    bats test/*.bats

The test harness uses [BATS](https://github.com/sstephenson/bats).
Output resembles:

    1..8
    ok 1 LICENSE file exists
    ok 2 image exists
    ok 3 image contains opensm package
    ok 4 RDMA config allows IPoIB
    ok 5 RDMA config does not load iSCSI over RDMA
    ok 6 image contains infiniband-diags package
    ok 7 image does not contain doc files
    ok 8 sminfo poll interval is 5 seconds

:warning: I need to figure out how to run acceptance tests
on a host without infiniband devices (such as CircleCI).
On a host with IB, `docker logs <cid>` could show output
similar to [`test/opensm.log`](test/opensm.log), which was
obtained on an unconfigured, tiny test fabric.


License
-------

See [LICENSE](LICENSE).