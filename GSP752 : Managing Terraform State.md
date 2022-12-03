# GSP752 - [Managing Terraform State](https://www.cloudskillsboost.google/focuses/15845?parent=catalog)

## Task 1. Working with backends
&nbsp;

:point_right:  Add a local backend
```
gcloud config list --format 'value(core.project)'

cat > main.tf <<EOF
provider "google" {
  project     = "$GOOGLE_CLOUD_PROJECT"
  region      = "us-central-1"
}
resource "google_storage_bucket" "test-bucket-for-state" {
  name        = "$GOOGLE_CLOUD_PROJECT"
  location    = "US"
  uniform_bucket_level_access = true
}
terraform {
  backend "local" {
    path = "terraform/state/terraform.tfstate"
  }
}
EOF

terraform init

terraform apply -auto-approve

terraform show
```
&nbsp;

:point_right:  Add a Cloud Storage backend
```
cat > main.tf <<EOF
provider "google" {
  project     = "$GOOGLE_CLOUD_PROJECT"
  region      = "us-central-1"
}
resource "google_storage_bucket" "test-bucket-for-state" {
  name        = "$GOOGLE_CLOUD_PROJECT"
  location    = "US"
  uniform_bucket_level_access = true
}
terraform {
  backend "gcs" {
    bucket  = "$GOOGLE_CLOUD_PROJECT"
    prefix  = "terraform/state"
  }
}
EOF

yes | terraform init -migrate-state

gsutil label ch -l key:value gs://$GOOGLE_CLOUD_PROJECT

terraform refresh

terraform show
```
&nbsp;

:point_right: Clean up your workspace
```
cat > main.tf <<EOF
provider "google" {
  project     = "$GOOGLE_CLOUD_PROJECT"
  region      = "us-central-1"
}
resource "google_storage_bucket" "test-bucket-for-state" {
  name        = "$GOOGLE_CLOUD_PROJECT"
  location    = "US"
  uniform_bucket_level_access = true
}
terraform {
  backend "local" {
    path = "terraform/state/terraform.tfstate"
  }
}
EOF

yes | terraform init -migrate-state

cat > main.tf <<EOF
provider "google" {
  project     = "$GOOGLE_CLOUD_PROJECT"
  region      = "us-central-1"
}
resource "google_storage_bucket" "test-bucket-for-state" {
  name        = "$GOOGLE_CLOUD_PROJECT"
  location    = "US"
  uniform_bucket_level_access = true
  force_destroy = true
}
terraform {
  backend "local" {
    path = "terraform/state/terraform.tfstate"
  }
}
EOF

terraform apply -auto-approve

yes | terraform destroy
```
&nbsp;
## Task 2. Import Terraform configuration
&nbsp;

:point_right: Create a Docker container

```
docker run --name hashicorp-learn --detach --publish 8080:80 nginx:latest

docker ps
```
&nbsp;

:point_right: Import the container into Terraform
```
git clone https://github.com/hashicorp/learn-terraform-import.git

cd learn-terraform-import

terraform init

sed -i 's/host/# host/' main.tf

echo 'resource "docker_container" "web" {}' > docker.tf

terraform import docker_container.web $(docker inspect -f {{.ID}} hashicorp-learn)

terraform show
```
&nbsp;

:point_right: Create configuration
```
terraform plan

terraform show -no-color > docker.tf
```
