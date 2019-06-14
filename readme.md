## This reposistory is created with learning purposes for Terraform, focusing on how to move an already created resource into a separate module.

## Purpose :

- It provides a simple example of how to use `terraform state mv` command to move existing resource to a separate module.

## How to install terraform : 

- The information about installing terraform can be found on the HashiCorp website 
[here](https://learn.hashicorp.com/terraform/getting-started/install.html)

## How to use it :

- In a directory of your choice, clone the github repository :
    ```
    git clone https://github.com/martinhristov90/terraformMoveState.git
    ```
- Change into the directory :
    ```
    cd terraformMoveState/dir1
    ```
- Run `terraform init` to download the needed providers and `terraform apply` to create the resources described in the `dir1/main.tf`. Your output should look like:
    ```
    An execution plan has been generated and is shown below.
    Resource actions are indicated with the following symbols:
      + create

    Terraform will perform the following actions:

      + null_resource.hello
          id:        <computed>

      + random_pet.name
          id:        <computed>
          length:    "4"
          separator: "-"


    Plan: 2 to add, 0 to change, 0 to destroy.

    Do you want to perform these actions?
      Terraform will perform the actions described above.
      Only 'yes' will be accepted to approve.

      Enter a value: yes

    random_pet.name: Creating...
      length:    "" => "4"
      separator: "" => "-"
    random_pet.name: Creation complete after 0s (ID: supposedly-miserably-optimal-raptor)
    null_resource.hello: Creating...
    null_resource.hello: Provisioning with 'local-exec'...
    null_resource.hello (local-exec): Executing: ["/bin/sh" "-c" "echo Hello supposedly-miserably-optimal-raptor"]
    null_resource.hello (local-exec): Hello supposedly-miserably-optimal-raptor
    null_resource.hello: Creation complete after 0s (ID: 6725223686421428936)
    ```
- Now, the resource `random_pet.name` is going to be moved to a separate module. For this purpose,file `dir1/main.tf` is needs to be changed to :
    ```
    module "example" {
      source = "../dir2"
    }

    resource "null_resource" "hello" {
      provisioner "local-exec" {
        command = "echo Hello ${module.example.display}"
      }
    }
    ```
- Now, you need to run `terraform init` to get the content of the `example` module. If `terraform plan` is run, terraform is going to suppose that we want to remove resource `random_pet.name` that was originally crated and we want to create new one `module.example.random_pet.name`. For Terraform those are two completely different resources, but to us they are the same. So, to move the state of `random_pet.name` resource, we need to use `terraform state mv random_pet.name module.example`. After executing this command Terraform knows that resource is moved to `module.example`. Output should look like this :
    ```
    Moved random_pet.name to module.example
    ```
- After moving the state of the `random_pet.name` resource, you can run `terraform plan` and see that no changes are needed. Output should look like this :
    ```
    random_pet.name: Refreshing state... (ID: supposedly-miserably-optimal-raptor)
    null_resource.hello: Refreshing state... (ID: 6725223686421428936)

    ------------------------------------------------------------------------

    No changes. Infrastructure is up-to-date.

    This means that Terraform did not detect any differences between your
    configuration and real physical resources that exist. As a result, no
    actions need to be performed.
    ```
- To destroy the created resources, you should execute :
    ```
    terraform destroy
    ```
