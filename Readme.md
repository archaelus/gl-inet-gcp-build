# gl-inet mifi firmware build (in the cloud!)
By `Geoff Cant <geoff+github@archant.us>`.

This collection of cloud builder and docker files will let you build [gl-inet firmware](https://github.com/gl-inet/imagebuilder-cc-ar71xx) using [Google Cloud Container Builder](https://cloud.google.com/container-builder/docs/how-to).

## Pre-Requisites

1. Install the google-cloud-sdk - you'll need the `gcloud` and `gsutil` utilities.
2. Check out this repository locally.
3. Create a google storage bucket to store the build results
4. Edit the `id: publish` step in [cloudbuild.yaml](cloudbuild.yaml) and replace the "gl-mifi-firmware" in the google storage url `gs://gl-mifi-firmware/$BUILD_ID` with the name of the bucket you created. You should keep the `$BUILD_ID` if you want to keep all build results (otherwise it'll overwrite each time).
5. Edit the firmware [Dockerfile](Dockerile.gl-mifi-firmware-build]. Change the `RUN make image` file to the GL-inet device you have and the options you want. This repo builds for the [gl-mifi](https://www.gl-inet.com/mifi/).

# Build and publish firmware

1. Run `gcloud container builds submit --config cloudbuild.yaml .` from the directory containing this repository.

That should be it! You should see the artifacts if you run `gsutil gs://YOUR_STORAGE_BUCKET/$BUILD_ID`. For example:

    $ gsutil ls gs://gl-mifi-firmware/7ddc967a-eb7d-4aaf-8452-a473df011601/
    gs://gl-mifi-firmware/7ddc967a-eb7d-4aaf-8452-a473df011601/md5sums
    gs://gl-mifi-firmware/7ddc967a-eb7d-4aaf-8452-a473df011601/openwrt-ar71xx-generic-gl-mifi-squashfs-sysupgrade.bin
    gs://gl-mifi-firmware/7ddc967a-eb7d-4aaf-8452-a473df011601/openwrt-ar71xx-generic-root.squashfs
    gs://gl-mifi-firmware/7ddc967a-eb7d-4aaf-8452-a473df011601/openwrt-ar71xx-generic-root.squashfs-64k
    gs://gl-mifi-firmware/7ddc967a-eb7d-4aaf-8452-a473df011601/openwrt-ar71xx-generic-uImage-gzip.bin
    gs://gl-mifi-firmware/7ddc967a-eb7d-4aaf-8452-a473df011601/openwrt-ar71xx-generic-uImage-lzma.bin
    gs://gl-mifi-firmware/7ddc967a-eb7d-4aaf-8452-a473df011601/openwrt-ar71xx-generic-vmlinux-lzma.elf
    gs://gl-mifi-firmware/7ddc967a-eb7d-4aaf-8452-a473df011601/openwrt-ar71xx-generic-vmlinux.bin
    gs://gl-mifi-firmware/7ddc967a-eb7d-4aaf-8452-a473df011601/openwrt-ar71xx-generic-vmlinux.elf
    gs://gl-mifi-firmware/7ddc967a-eb7d-4aaf-8452-a473df011601/openwrt-ar71xx-generic-vmlinux.gz
    gs://gl-mifi-firmware/7ddc967a-eb7d-4aaf-8452-a473df011601/openwrt-ar71xx-generic-vmlinux.lzma
    gs://gl-mifi-firmware/7ddc967a-eb7d-4aaf-8452-a473df011601/sha256sums

# Things that are complicated and weird.

Using docker containers for build steps was good... right up until I needed a non-docker build step to use the results of a docker build step. At that point you need an insane work around of a three step: 1) create container from previous image, 2) docker cp files out of the container, 3) remove the container. That's the trick to getting the firmware images out of the docker build container and uploaded to google cloud storage.
