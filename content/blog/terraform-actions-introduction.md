+++
title = 'Plan. Apply Act. Bringing Day 2 Work into Terraform'
date = 2025-01-05T21:56:26-05:00
draft = false
tags = ["terraform", "iac"]
+++


### Introduction to Terraform Actions

For what seems like forever now, every Terraform repo has carried a stray shell step: a cache purge after deploy, a webhook ping, an Ansible playbook to finish configuration and while useful, ultimately grafted on with provisioners or CI glue that Terraform can’t see. Actions give those steps a much needed home: provider defined operations you can declare in HCL and invoke on demand or at specific lifecycle moments. They’re purpose-built for “make something happen” without pretending to be resources.

Think of your terraform operations in three layers:
+ Resources – durable, CRUD-managed objects (what Terraform has always done).
+ Ephemeral – temporary values you use during a run but never persist (e.g., a short-lived token or password).
+ Actions – imperative side effects, invoked either on lifecycle hooks or ad-hoc, with zero impact on state.

### Model & Approach
+ action blocks are defined by providers; they expose a small schema and an invoke entrypoint. They do not produce values, so you can’t reference them like data sources or resources. That design keeps them predictable.
+ Use lifecycle { action_trigger { … } } on a resource to run actions before/after create or update. Or call an action directly with terraform apply -invoke action.type.name to retry a flaky step without replaying the whole plan
+ Scope and boundaries: triggers can only reference actions declared in the same module however the CLI can still invoke across modules.
+ No resource state changes: actions don’t modify resource state; if a side effect changed a computed attribute, a refresh syncs it later. 


### Using Actions

```
action "webhook_post" "deploy_notice" {
  config {
    url     = var.slack_webhook_url
    message = "New app version deployed to ${var.env}"
  }
}

action "ansible_playbook" "configure_vm" {
  config {
    playbook_path   = "${path.module}/playbooks/app.yml"
    host            = azurerm_linux_virtual_machine.app.public_ip_address
    ssh_public_key  = azurerm_ssh_public_key.app.public_key
  }
}

resource "azurerm_linux_virtual_machine" "app" {
  # ...vm config...

  lifecycle {
    action_trigger {
      events  = [after_create, after_update]
      actions = [
        action.ansible_playbook.configure_vm,
        action.webhook_post.deploy_notice
      ]
      condition = var.enable_post_deploy_actions
    }
  }
}

```

In certain situations, if you  would like to just run an action you can use the new -invoke flag to just invoke one action: __terraform apply -invoke action.ansible_playbook.configure_vm__ just invokes the action that runs the Ansible Playbook

