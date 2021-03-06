---
layout: docs
title: Notes for Distributors
category: sdk_distributors
sub_category: distributors
weight: 5
---

# Notes for Distributors

## Importing Images

Images of CoreOS are hosted at `http://storage.core-os.net/coreos/amd64-usr/`. At this URL there are directories for each individual version of CoreOS but also images that have promoted to a channel like master, alpha, beta, etc.

If you are importing images for use inside of your environment it is recommended that you import from a URL in the following format `http://storage.core-os.net/coreos/amd64-usr/${CHANNEL}/`. For example to grab the alpha OpenStack version of CoreOS you can import `http://storage.core-os.net/coreos/amd64-usr/alpha/coreos_production_openstack_image.img.bz2`. There is a `version.txt` file in this directory which you can use to label the image with a version number as well.

It is recommended that you also verify files using the [CoreOS Image Signing Key][signing-key]. The digests are simply the image URL, replacing `_image.img.bz2` with `.DIGESTS.asc`. You can verify the digest with `gpg --verify` after importing the signing key. Then the image itself can be verified based on one of the hashes in `.DIGESTS.asc`. For example:

    wget http://storage.core-os.net/coreos/amd64-usr/alpha/coreos_production_openstack_image.img.bz2
    wget http://storage.core-os.net/coreos/amd64-usr/alpha/coreos_production_openstack.DIGESTS.asc
    gpg --verify coreos_production_openstack.DIGESTS.asc
    sha512sum -c coreos_production_openstack.DIGESTS.asc

[signing-key]: {{site.url}}/security/image-signing-key

## Image Customization

Customizing a CoreOS image for a specific operating environment is easily done through [cloud-config]({{site.url}}/docs/cluster-management/setup/cloudinit-cloud-config/), a YAML-based configuration standard that is widely supported. As a provider, you must ensure that your platform makes this data available to CoreOS, where it will be parsed during the boot process.

Use cloud-config to handle platform specific configuration such as custom networking, running an agent on the machine or injecting files onto disk. CoreOS will automatically parse and execute `/usr/share/oem/cloud-config.yml` if it exists. Your cloud-config should create additional units that process user-provided metadata, as described below.

## Handling End-User Cloud-Config Files

End-users should be able to provide a cloud-config file to your platform while specifying their VM's parameters. This file should be made available to CoreOS at a known network address, injected directly onto disk or contained within a [config-drive][config-drive-docs]. Below are a few examples of how this process works on a few different providers.

[config-drive-docs]: http://docs.openstack.org/grizzly/openstack-compute/admin/content/config-drive.html

### Amazon EC2 Example

CoreOS machines running on Amazon EC2 utilize a two-step cloud-config process. First, a cloud-config file baked into the image runs systemd units that execute scripts to fetch the user-provided SSH key and fetch the [user-provided cloud-config][amazon-cloud-config] from the instance [user-data service][amazon-user-data-doc] on Amazon's internal network. Afterwards, the user-provided cloud-config, specified from either the web console or API, is parsed.

You can find the [code for this process on Github][amazon-github]. End-user instructions for this process can be found on our [Amazon EC2 docs][amazon-cloud-config].

[amazon-github]: https://github.com/coreos/coreos-overlay/tree/master/coreos-base/oem-ami
[amazon-user-data-doc]: http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AESDG-chapter-instancedata.html#instancedata-user-data-retrieval
[amazon-cloud-config]: {{site.url}}/docs/running-coreos/cloud-providers/ec2#cloud-config

### Rackspace Cloud Example

Rackspace passes configuration data to a VM by mounting [config-drive][config-drive-docs], a special configuration drive containing machine-specific data, to the machine. Like, Amazon EC2, CoreOS images for Rackspace contain a cloud-config file baked into the image that runs units to read from the config-drive. If a user-provided cloud-config file is found, it is parsed.

You can find the [code for this process on Github][rackspace-github]. End-user instructions for this process can be found on our [Rackspace docs][rackspace-cloud-config].

[rackspace-github]: https://github.com/coreos/coreos-overlay/tree/master/coreos-base/oem-rackspace
[rackspace-cloud-config]: {{site.url}}/docs/running-coreos/cloud-providers/rackspace#cloud-config
