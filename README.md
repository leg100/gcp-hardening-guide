# gcp-hardening-guide
Comprehensive guidelines for hardening the security of your GCP organization.

This guide is targeted at those running a large complex infrastructure. You'll need a team with necessary expertise and resources to apply these guidelines.

## APIs and Services

### Enablement

Restrict the enablement of APIs and services using the [Organization Policy](https://cloud.google.com/resource-manager/docs/organization-policy/org-policy-constraints) "Define allowed APIs and services" [DiD]

Note: Unfortunately, only several APIs can be restricted.

For each project, enable only those APIs and services that are needed [PoLP].

### Authorization

Bind to users and service accounts only IAM roles and permissions they need to perform their job [PoLP].

Apply VPC Service Controls. [TODO]

In IAM policies, specify [Cloud Identity groups](https://cloud.google.com/identity/docs/concepts/groups) rather than individual users; groups indicate the role of members via the group name. [Auditing]

#### Custom Roles

Use [custom IAM roles](https://cloud.google.com/iam/docs/understanding-custom-roles) [PoLP]

Note: there are thousands of individual IAM permissions. It can be difficult to manage the complexity of assigning large numbers of permissions to a custom role (e.g. managing the release of new permissions for existing and new services as well as their deprecation). Custom roles may make more sense for small, focused roles, that include only a small number of permissions.

Note: custom roles [do not support all IAM permissions](https://cloud.google.com/iam/docs/custom-roles-permissions-support).

## Compute Engine

### SSH

Use [OS Login](https://cloud.google.com/compute/docs/oslogin). OS Login uses IAM to manage SSH access rather than SSH keys. The distribution of SSH keys cannot be controlled (whereas of course IAM permissions can).
Set the [Organization Policy](https://cloud.google.com/resource-manager/docs/organization-policy/org-policy-constraints) "Require OS Login" to `true`. [DiD]

Note: this may cause GKE instances to malfunction, so set this policy to `false` on projects containing GKE clusters.

#### Prevent users from working around OS Login

1. (Legitimate) User A connects to their VM: `gcloud compute ssh <vm>`. They have necessary IAM perms, so it succeeds.
2. User A adds User B's public key to `~/.ssh/authorized_keys`.
3. User B connects to the VM: `ssh -i <key> <vm_ip>`. It succeeds, because their key is authorized and they have direct connectivity to port 22, even though they don't have the necessary OS Login IAM permissions.

To prevent this, compel users to use IAP to SSH into a VM. Configure a GCP firewall rule to block access to port 22 on the VM from anything other than the IAP IP range.

Alternatively if a user does not need `sudo` privileges on the VM (assuming it's Linux), disabling the user of the authorized keys file is sufficient:

1. Set `AuthorizedKeysFile none` in `/etc/ssh/sshd_config`
2. Restart SSH daemon: `systemctl restart sshd`

## Princples

* [PoLP]: Principle of Least Privilege
* [DiD]: [Defence in depth](https://en.wikipedia.org/wiki/Defense_in_depth_(computing)). Every control adds depth, but for the purpose of this guide, this principle refers to **secondary** controls, which typically have a broader scope than **primary** control, which is closer to the entity being protected.
* [Auditing]: Assists with auditing
