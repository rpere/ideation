# How to introduce a concept of config management for halyard

See [DEV-1733]

## What's halyard ?

Halyard is essentially the spine of a spinnaker instance - a CLI to configure how spinnaker will be deployed and to also add updates/features to an existing cluster.

Right now it is a statefulset kubernetes kind (backed by a PVC) that we exec in and run some `hal foo` [commands].

Once a change is introduced, it is verified by `hal config` and if it sounds valid deployed by `hal deploy apply`

Noting that one good thing about it : halconfig is just a file `./hal/config` so (good and bad) can be easily manipulated.

## What's the problem ?

This approach is very manual and depends on human running commands on a production instance.

We all know what it means in terms of a human having access to the spine of a system to make a production change - a potential disaster.

The other problem, is that it is difficult to know what command was ran against the current config state.

## What we would like to have ?

Something, a very well detailed process (at the very least), but ideally something more robust, more automated and leveraging some sort of source control (SC) changes/reviews/applies.

## Solution(s)?

Incredibly, there is nothing out there that is solving this big problem.

We're just left with brainstorming it, and be creative about it, and by no means this is aiming to be THE solution(s).

- **Terraform** : Right now a few of Spinnaker's bits are deployed using terraform, but they're limited to (ingresses, svc accounts, keys..) we could extend it to manage halyard config this way :
  - CRB's as a data resource using `template_file` something like : 
```
data "template_file" "my_grant_to_spin" {
  template = <<EOF
kubectl create clusterrolebinding spin-admin \
  --clusterrole=cluster-admin \
  --serviceaccount=<ns>:<account>
EOF
}
```
  - Then grant the CRB to spin svc account
  - Create a dedicated SA account/GCS bucket (used to store halconfig)
  - (...probably a few more little things in between ...)
  - And finally create another data resource using for halconfig - something like :
```
data "template_file" "halconfig_spinnaker" {
  template = <<EOF
HAL_CMD=`hal -q config`
$HAL_CMD provider kubernetes enable \
$HAL_CMD storage gcs edit --project <project_name> --bucket-location <bucket_location> --json-path .... \
$HAL_CMD features edit --artifacts true \
...
...
$HAL_CMD deploy apply
EOF
}
```

- **Jenkinsfile** : The concept is described [here], we could just use GCS instead of S3 and target the current halyard pod for it.

# Conclusion :

TBD :-)


[here]: https://medium.com/@jwenz723/using-jenkins-to-install-spinnaker-1af7d87281fe
[DEV-1733]: https://liveramp.atlassian.net/browse/DEV-1733
[commands]: https://spinnaker.io/reference/halyard/commands/
