# BOSH Release for tuning up a VM

The purpose of this release is provide a way to tune up and customize a VM.

Capabilities of the jobs:

* `users` job manages the users on the server. It allows defining several public keys,
  the password, uid, ssh and sudo access, shell, the profile and a way to disable a user.
* `sysctl` manages the sys settings on the system, starting the job will apply the 
  new settings defined on the properties and stopping will restore the orginal settings
  on the system.
* `scheduler` defines udev rules to manage the IO scheduler, global default or per device
* `resolvconf` job manages domain, search and options for the dns resolver. See 
  resolv.conf manual to define the options.
* `login_banner` manages the content of the file /etc/issue.net. There are
  some variables predefined you can use in the text.


## Development

Steps to build a new release:

```
git clone https://github.com/SpringerPE/vm-tuning-boshrelease.git
cd vm-tuning-boshrelease
bosh target BOSH_HOST
bosh create release --force && bosh upload release
```

### Final releases

In order to publish a final release, you can run `./bosh_final_release` for:

* Upload all blobs to the public S3 bucket (and set them to public)
* Create a final Bosh release (increasing the version number)
* Upload and publish the new boshrelease tgz file in GitHub releases section, so
  you can point it directly in the manifest file (sha1 checksum is also calculated)
* Pushes and commits the changes to git

After that, you can also, upload the new release to bosh director `bosh upload release`


# Author

Springer Nature Platform Engineering, Jose Riguera

Copyright 2017 Springer Nature


# License

MIT License

