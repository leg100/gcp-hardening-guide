# GCP Hardening Guide

Comprehensive guidelines for hardening the security of your Google Cloud organization.

This guide is targeted at those running a large complex infrastructure. You'll need a team with necessary expertise and resources to apply these guidelines.

## 1. IAM

### 1.1. Disable service account key creation [DiD]

#### Description

Disable the ability to create service account keys.

#### Control

Enforce the Organization Policy "Disable service account key creation" at the organization level.

#### Reasoning

* Keys are not non-repudiable: any actions carried out using the key cannot be associated with a given user.
* Distribution of keys cannot be controlled: they can be leaked to outside parties.

### 1.2. Specify groups in IAM policies, not users [Auditing].

#### Description

In IAM policies, bind permissions to [Cloud Identity groups](https://cloud.google.com/identity/docs/concepts/groups) rather than individual users

#### Control

N/A

#### Reasoning

* Directly assigning IAM permissions to users increases the overhead of managing IAM policies; when adding or removing permissions from a certain job function (e.g. web developers), changes are necessary to each and every username with that job function.
* The name of a group can (and should) meaningfully indicate the job function of its members, e.g. `web-developers@example.com`, whereas usernames do not.

### 1.3. Enforce 2-step verification [DiD].

#### Description

Require all users to use 2-step verification when logging into their Google accounts. They'll need to provide a piece of information in addition to their password (such as a security code from an app on their phone).

#### Control

https://support.google.com/a/answer/9176657?hl=en

#### Reasoning

* Passwords can be leaked and should not be relied upon alone.


## 2. Compute Engine

### 2.1. Require OS Login [DiD]

#### Description

[OS Login](https://cloud.google.com/compute/docs/oslogin) uses IAM to manage SSH access to VMs, allowing access to be granted and revoked centrally. Once enabled, the default mechanism - which stores users' SSH keys in project and instance metadata - is disabled.

#### Control

Set the [Organization Policy](https://cloud.google.com/resource-manager/docs/organization-policy/org-policy-constraints) "Require OS Login" to `true`.

Note: this may cause GKE instances to malfunction, so set this policy to `false` on projects containing GKE clusters.

OS Login can otherwise be enabled by setting [`enable-oslogin=TRUE` in project or instance metadata](https://cloud.google.com/compute/docs/instances/managing-instance-access#enable_oslogin). However, a user with the necessary IAM permissions can disable OS Login by setting `enable-oslogin=FALSE` (these permissions are included in IAM roles typically assigned to users working with compute engine resources).

#### Reasoning

* The distribution of SSH keys cannot be controlled; they can be leaked to outside parties.

### 2.2. Prevent OS Login loophole (Linux)

#### Description

Prevent the following scenario:

1. (Legitimate) User A connects to their VM: `gcloud compute ssh <vm>`. They have necessary IAM permissions, so it succeeds.
2. User A adds User B's public key to `~/.ssh/authorized_keys`.
3. User B connects to the VM: `ssh -i <key> <vm_ip>`. It succeeds, because their key is authorized and they have direct connectivity to port 22, even though they don't have the necessary OS Login IAM permissions.

#### Control

Compel users to [create an IAP tunnel](https://cloud.google.com/iap/docs/using-tcp-forwarding) in order to SSH into a VM. Configure a GCP firewall rule to block access to port 22 on the VM from anything other than the IAP IP range (`35.235.240.0/20`).

Alternatively, if a user does not need `sudo` privileges on the VM, disabling the user of the authorized keys file is sufficient:

1. Set `AuthorizedKeysFile none` in `/etc/ssh/sshd_config`
2. Restart SSH daemon: `systemctl restart sshd`

### 2.3. Remove default network

#### Description

Remove the network named `default` that is automatically supplied with a newly created project.

#### Control

For each given project:

1. Create a new network
2. Remove the network named `default`.

#### Reasoning

* The default network is configured with permissive firewall rules.

### 2.4. Disable Serial Ports [DiD]

#### Description

Disable serial ports on VMs.

#### Control

Enforce the organization policy "Disable VM serial port access" at the organization level.

#### Reasoning

* The serial port exposes sensitive information
* The serial port can accept commands that could permit changes to made
* GCP firewall rules do not apply to serial ports. Any client with an appropriate SSH key and trivial details of the instance (its name, project, etc) can connect to the ports.

## 3. APIs and Services

### 3.1. Restrict API enablement [DiD]

#### Description

Restrict which APIs users are permitted to enable.

#### Control

Enforce the [Organization Policy](https://cloud.google.com/resource-manager/docs/organization-policy/org-policy-constraints) "Define allowed APIs and services".

Note: Only several APIs can be specified.

#### Reasoning

* A user with appropriate IAM permissions can enable APIs on a given project. Setting the organisation policy above restricts what they are allowed to enable.

### 1.3. Use IAM custom roles instead of predefined roles

Rather than use predefined IAM roles, use [custom IAM roles](https://cloud.google.com/iam/docs/understanding-custom-roles). [PoLP].

Note: there are thousands of individual IAM permissions. It can be difficult to manage the complexity of assigning large numbers of permissions to a custom role (e.g. managing the release of new permissions for existing and new services as well as their deprecation). Custom roles may make more sense for small, focused roles, that include only a small number of permissions.

Note: custom roles [do not support all IAM permissions](https://cloud.google.com/iam/docs/custom-roles-permissions-support).

## Principles

* [PoLP]: Principle of Least Privilege
* [DiD]: [Defence in depth](https://en.wikipedia.org/wiki/Defense_in_depth_(computing)). Every control adds depth, but for the purpose of this guide, this principle refers to **secondary** controls, which typically have a broader scope than **primary** control, which is closer to the entity being protected.
* [Auditing]: Assists with auditing
