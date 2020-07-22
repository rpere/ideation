# Berglas VS Vault

## What's Berglas ? 

It's a tool that interfaces with GCP services by abstracting away all the complexity.
Uses KMS to encrypt secrets and buckets to store them.
Compatible with the new Google Secret Manager

It can be used as a CLI or as a library and has kubernetes integration ; for more details look at the [project]

## What's vault ? 

Vault is a tool for securely accessing secrets.
We've been using vault at LiveRamp for many years and have a fairly robust set up in GCP - check our [runbook] for more details

The kubernetes integration relies on an in house tool that involves some steps that needs to be done by both devs and ops - see this [doc] for more details.

## What are we trying to solve ? 

Vault has been set up and used in a time where LiveRamp was a very much different company : an on-premise company.

Since then, we have improved considerably the vault architecture to be more cloud centric and therefore reliable and redundant.

Vault is still causing times to times issues to people around the company (ranging from permissions to just plain usage of the binary)

Berglas appears to resolve most of these issues on the surface and was built and designed for GCP.

## What are the pros ? 

- Binary is dead simple to get and to interact with : `brew install berglas`
- CLI usage for random password retrieval is also dead simple : `berglas access cryptic/infra-root-password`
- Sharing access to a password is again dead simple : `berglas grant cryptic/infra-root-password --member user:aditya@liveramp.com`
- It's secure without _obscurity_ (uses KMS)
- Can also be used with Secret Manager Storage (there is a small cost per password associated with google secret manager storage)
- There is no servers / pods / services / dns to maintain for ops.
- Could be used anywhere (in code) by wraping any process with a simple `berglas exec --` a good example [here]
- GCP projects owners can essentially become owners of their own secrets (via buckets in their respective projects)
- (specific to kubernetes integrations), yaml files will essentially be a bunch of env variables that uses "pointers" beginning with "berglas://"  

## What are the cons ? 

- Kubernetes integration (see POC) is _not_ dead simple : 
  - Relies on creating a [MutatingWebhook] on each clusters - could be done with a fairly simple cloud function.
- Has a limitation where there is a requirement for containers to specify a `command` in their manifest.
- No UI (unless it is coupled with Google Secret Manager)

## Conclusion

Beyond the kubernetes integration, I think this tool beats vault in all the use cases it offers, granted that we're now in GCP and that vault while robust is more complex to use and maintain.







[MutatingWebhook]: https://kubernetes.io/blog/2019/03/21/a-guide-to-kubernetes-admission-controllers/#mutating-webhook-configuration]
[here]: https://medium.com/weareservian/berglas-with-node-js-on-cloud-run-d7cecfa5aa49
[doc]: https://github.com/LiveRamp/team_devops/blob/master/articles/K8S_SECRETS.md
[project]: https://github.com/GoogleCloudPlatform/berglas
[runbook]: https://github.com/LiveRamp/team_devops/blob/master/articles/VAULT_RUNBOOK.md
