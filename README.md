# delivery-phase2
The evolution of a process: phase 1: the operating system

Phase 2 is about making a safe home for containers in the form of a Amazon AMI. 

## Some Setup
Before starting, use homebrew to install packer: `brew install packer packer-completion`

Next, the `AWS_REGION` must be assigned a value and exported to the environment. The value is whatever region AWS assigned to _you_. Example: `export AWS_REGION='us-west-2'`.

Also, the work that comes later cannot be validated until there is [access to the VPC]:

* FROM: the point of origin: your current gateway
* TO: the destination: _your_ AWS account

We may as well get it out of the way now. _NOTE: This is only necessary if:_ 

* The gateway address is DHCP (keeps changing) or 
* You are on the road, staying in hotels for example. 
* The script only needs to be run once per IP address change. 
* If the gateway address hasn't changed the script will simply report that the correct settings are already in place. 

Access can be [gained easily] by running this script:

`scripts/aws-tools/access-aws-securitygroup.sh`

Now that the requisite access is in place the AMI can be built.

Finally, the Ansible requirements file must be satisfied else the playbooks will fail:

`ansible-galaxy install -r scripts/requirements.yml`

***
## Build the AMI
Building an AMI means describing it in a JSON file that will be consumed by Packer. In this case that file is `debian-jessie.json`. It needs to be built from an existing AMI so, we'll use the one [Debian] offers in the AWS Marketplace. Since finding the latest AMI `ImageId` requires the manual steps of selecting the **Manual Launch** (tab) and indexing the  `ImageId` in your region the exorcise becomes tedious. We'll get the automation to do it for us.

### Process Description
The `build-ami.sh` script takes 2 arguments

1. the `amiName`: base-debian
2. the packer file: `debian-jessie.json`

Next, `build-ami.sh` calls `scripts/aws-tools/find-latest-amis.sh` to [find the latest] Debian Jessie AMI `ImageId` which is then sent to Packer as a variable; for example: `latestImageId=ami-12345678`. _NOTE: this script does factor for `AWS_REGION`_.

**WARNING: this step can take 8-12 minutes.**

After the script has been started it's time to go make a sandwich, do some laundry, whatever. It takes some time to build, which varies based on available Internet connection bandwidth and workstation horsepower. 

To build an AMI simply run this script:

`./build-ami.sh base-debian debian-jessie.json 2>&1 | tee /tmp/packer-debian-build.out`


### Post-Build
When you're ready, review the output in `/tmp/packer-debian-build.out`. These are the build details.

Now that you have a tailored AMI. You are ready to [start Terraforming].

[access to the VPC]:http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-network-security.html#adding-security-group-rule
[gained easily]:https://github.com/todd-dsm/aws-tools/wiki/access-aws-securitygroup
[Debian]:https://aws.amazon.com/marketplace/pp/B00WUNJIEE?qid=1487285341617
[find the latest]:https://github.com/todd-dsm/aws-tools/wiki/find-latest-ami
[start Terraforming]:https://github.com/todd-dsm/process-ph3

