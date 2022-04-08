# Setup infrasturcture

## Assignment Part 1 - Terraform State

Having remote state is common in a collaboration within the team and between teams. 

Setup Remote Terraform state in Swift Obect Store. Verify that is works!

```
provider "openstack" {
  use_octavia = true
}

terraform {
  backend "swift" {
    container         = "terraform-state-webservers"
    archive_container = "terraform-state-archive-webservers"
  }
}
```

## Assignment Part 2 - Terraform

With terraform setup infrastructure from `lab 1-setup-infra`

## Challenge!

Find a someone, give access to that person to run changes in your environment. Try and run changes at the same time to see what happens.

Hint! You will need to create an account, keys, setup git repo and so on.

