---
name: tf-destroy
description: Tear down all AWS infrastructure managed by Terraform to stop incurring charges. Empties the S3 bucket first, then runs terraform destroy with interactive confirmation.
allowed-tools: Bash, Read
disable-model-invocation: true
---

Destroy all AWS resources provisioned by Terraform in `terraform/`.

**Always prompt the user for explicit confirmation before any destructive step.** Do not proceed without a clear yes.

Steps:
- [ ] Show what will be destroyed (informational only): `cd terraform && terraform plan -destroy -no-color`
- [ ] Ask the user to confirm before proceeding with the destroy.
- [ ] Read the bucket name and distribution id from outputs: `cd terraform && terraform output -json`
- [ ] Empty the S3 bucket (the bucket has `force_destroy = false` and versioning enabled, so plain destroy will fail if objects exist):
  - Delete all current objects: `aws s3 rm s3://<bucket> --recursive`
  - Delete all object versions and delete markers (versioning is on):
    ```
    aws s3api list-object-versions --bucket <bucket> \
      --query '{Objects: Versions[].{Key:Key,VersionId:VersionId}}' --output json \
      | aws s3api delete-objects --bucket <bucket> --delete file:///dev/stdin || true
    aws s3api list-object-versions --bucket <bucket> \
      --query '{Objects: DeleteMarkers[].{Key:Key,VersionId:VersionId}}' --output json \
      | aws s3api delete-objects --bucket <bucket> --delete file:///dev/stdin || true
    ```
- [ ] Run destroy: `cd terraform && terraform destroy -no-color` (CloudFront teardown takes 5–15 min).
- [ ] Verify cleanup:
  - `aws s3 ls | grep <bucket>` should return nothing
  - `aws cloudfront get-distribution --id <dist-id>` should return `NoSuchDistribution`
- [ ] Report which resources were destroyed and confirm no further charges will accrue.

If any step fails, STOP and report the error — do not retry, do not continue. Common failures:
- **`BucketNotEmpty`**: rerun the object-version cleanup above (a new write may have landed).
- **CloudFront `DistributionNotDisabled`**: wait — Terraform disables it first; full deletion takes 5–15 min.
- **State lock**: another apply/destroy is running, or a previous run died holding the lock. Investigate before running `force-unlock`.


Notes:
- The S3 backend (if migrated to remote state) and its DynamoDB lock table are NOT managed by this Terraform config — they were created out of band per `backend.tf`. Destroying them is a separate manual step the user must perform if they want a fully clean account.
