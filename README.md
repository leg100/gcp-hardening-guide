# gcp-hardening-guide
Comprehensive guidelines for hardening the security of your GCP organization.

This guide is targeted at those running a large complex infrastructure. You'll need a team with necessary expertise and resources to apply these guidelines.

## APIs and Services

### Enablement

Restrict the enablement of APIs and services using the [Organization Policy](https://cloud.google.com/resource-manager/docs/organization-policy/org-policy-constraints) "Define allowed APIs and services". Unfortunately, only several APIs can be restricted. [DiD]

For each project, enable only those APIs and services that are needed [PoLP].

### Authorization

Bind to users and service accounts only IAM roles and permissions they need to perform their job [PoLP].

Apply VPC Service Controls. [TODO]

## Compute Engine

### SSH

Use [OS Login](https://cloud.google.com/compute/docs/oslogin). OS Login uses IAM to manage SSH access rather than SSH keys. The distribution of SSH keys cannot be controlled (whereas of course IAM permissions can).
Set the [Organization Policy](https://cloud.google.com/resource-manager/docs/organization-policy/org-policy-constraints) "Require OS Login" to `true`. [DiD]

Note: this may cause GKE instances to malfunction, so set this policy to `false` on projects containing GKE clusters.

`sudo`...

## Princples

* [PoLP]: Principle of Least Privilege
* [DiD]: Defence in depth
