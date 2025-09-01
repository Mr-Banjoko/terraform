# simple definition 
* Think of a null_resource in Terraform as an empty box you can fill with “do‑this-now” instructions — it doesn’t represent an AWS resource like an EC2 instance or S3 bucket.
* Normal resources → Create “real” things in AWS, Azure, GCP, etc.
* null_resource → Doesn’t create anything in the cloud; it just lets you run actions on your local machine (or remote) as part of terraform apply.

## good usecases 
* Run AWS CLI commands
* Execute scripts
* Copy files or set up config
* Perform steps Terraform doesn’t natively support

## running commands locally example
```

# task: copying the contents from an s3 bucket and then deleting that bucket.

# Step 1 & 2: Copy contents from an s3 bucket and emptying the bucket
resource "null_resource" "s3_backup_and_cleanup" {
  provisioner "local-exec" {
    command = <<EOT
      mkdir -p /opt/s3-backup
      aws s3 cp s3://devops-bck-2490 /opt/s3-backup/ --recursive
      aws s3 rm s3://devops-bck-2490 --recursive
    EOT
  }
}

# Step 3: Delete the bucket itself
resource "null_resource" "delete_s3_bucket" {
  depends_on = [null_resource.s3_backup_and_cleanup]

  provisioner "local-exec" {
    command = "aws s3api delete-bucket --bucket devops-bck-2490 --region us-east-1"
  }
}
```

## example 2
```
#task: copying the contents from one s3 bcuket to another

resource "null_resource" "s3_backup_and_cleanup" {
  provisioner "local-exec" {
    command = "aws s3 sync s3://${aws_s3_bucket.wordpress_bucket.bucket} s3://${aws_s3_bucket.nautilus_sync_bucket.bucket}"
  }
  depends_on = [
    aws_s3_bucket.wordpress_bucket,
    aws_s3_bucket.nautilus_sync_bucket
  ]
}

```
