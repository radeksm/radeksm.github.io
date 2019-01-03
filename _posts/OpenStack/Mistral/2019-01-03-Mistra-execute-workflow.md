---
layout: post
title: OpenStack Mistral demystification (part 1) - execute workflow
tags:
    - OpenStack
    - Mistral
author: Radosław Śmigielski
---

Simple way how to execute OpenStack Mistral workflow.

### Steps ###
1. Check input parameters of your workflow.

   All of them in below example have default values, so all of them are
   optional. That means, that workflow can be executed without any params.
   ```
   $ openstack workflow show tripleo.deployment.v1.config_download_deploy -c Input
   +-------+----------------------------------------------------------------------------------------------+
   | Field | Value                                                                                        |
   +-------+----------------------------------------------------------------------------------------------+
   | Input | timeout=240, queue_name=tripleo, plan_name=overcloud, work_dir=/var/lib/mistral, verbosity=1 |
   +-------+----------------------------------------------------------------------------------------------+
   ```
2. Create ```~/input.json``` with workflow input parameters.
   ```
   echo '{"verbosity": "2", "work_dir": "/var/tmp/config-download"}' > ~/input.json
   ```
3. Create ```~/input.json``` with workflow input parameters.
   ```
   $ openstack workflow  execution create tripleo.deployment.v1.config_download_deploy ~/input.json 
   +--------------------+----------------------------------------------+
   | Field              | Value                                        |
   +--------------------+----------------------------------------------+
   | ID                 | 24aca1f6-8bef-402b-9daa-7c7328c2dc10         |
   | Workflow ID        | 32991067-7a10-40e3-ad59-42ae4a0a140b         |
   | Workflow name      | tripleo.deployment.v1.config_download_deploy |
   | Workflow namespace |                                              |
   | Description        |                                              |
   | Task Execution ID  | <none>                                       |
   | State              | RUNNING                                      |
   | State info         | None                                         |
   | Created at         | 2019-01-03 07:09:16                          |
   | Updated at         | 2019-01-03 07:09:16                          |
   +--------------------+----------------------------------------------+
   ```
4. Check status of workflow execution
   ```
   openstack workflow execution show <ID value from previus point>
   ```
