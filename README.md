# BOSH Release for VM fine tuning

The purpose of this release is provide a way to fine tune and customize a VM.

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
* `hosts` manages static dns entries defined in /etc/hosts

## Usage

Example manifest with job properties:
```
  jobs:
  - name: sysctl
    release: vm-tuning
    properties:
      sysctl:
        config:
        - name: enable-ipv6
          index: 70
          config: |
            # undo BOSH's stemcell's IPv6 lockdowns
            # 10-ipv6-privacy.conf
            net.ipv6.conf.all.use_tempaddr = 0
            net.ipv6.conf.default.use_tempaddr = 0
            # 60-bosh-sysctl.conf
            net.ipv6.conf.all.accept_ra=1
            net.ipv6.conf.default.accept_ra=1
            net.ipv6.conf.all.disable_ipv6=0
            net.ipv6.conf.default.disable_ipv6=0
            net.ipv6.conf.default.accept_redirects=1
            net.ipv6.conf.all.accept_redirects=1
            net.ipv6.route.flush=0
  - name: hosts
    release: vm-tuning
    properties: {}
      hosts:
        "192.168.1.10": ['fqdn.domain.com', 'alias', 'name'] 
        "fe00::0": []
  - name: resolvconf
    release: vm-tuning
    properties:
      resolvconf:
        search: local springer.com
        options:
        - "timeout:3"
        - "attempts:1"
  - name: scheduler
    release: vm-tuning
    properties:
      scheduler:
        default: noop
        devices:
        - name: sdb
          scheduler: cfq
  - name: users
    release: vm-tuning
    properties:
      users:
      - name: jriguera
        uid: 1010
        disable: false
        sudo: true
        ssh: true
        public_key:
         - ssh-rsa AAAA .... kna jriguera@thinkpad
  - name: login_banner
    release: vm-tuning
    properties:
      login_banner:
        text: |
          This server is managed by Bosh Director
           ___ ___
          | _ \ __|   It's PE,
          |  _/ _|      baby!
          |_| |___|             platform-engineering@example.com
          
          # $JOB_FULL on $DEPLOYMENT_NAME
```


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

Based on https://github.com/cloudfoundry/os-conf-release

Springer Nature Platform Engineering, Jose Riguera

Copyright 2017 Springer Nature


# License

MIT License

