# Examples of yaml-patch usage patterns for pcf-pipelines

### Index

- [Add a task to a job](#add-task-to-job)
- [Add a task to a job after a specific task](#add-task-to-job-after-another-task)
- [Replace a resource definition in the pipeline YML](#replace-resource-definition)
- [Change the trigger flag of a job resource](#change-trigger-flag)
- [Additional operations samples](#additional-operations-samples)


---

### <a name="add-task-to-job"></a> Add a task to a job

1. Source YAML: job.yml  
```  
---  
jobs:  
- name: my-job  
  plan:  
  - get: my-resource  
  - task: first-task  
    file: task1.yml  
  - task: second-task  
    file: task2.yml    
```  

1. Operations file: add-task.yml  
```  
- op: add  
  path: /jobs/name=my-job/plan/-  
  value:  
    task: third-task  
    file: task3.yml  
```  

1. Execute yaml-patch command  
   `cat job.yml | yaml-patch -o add-task.yml > result.yml`    

1. Resulting patched file: result.yml  
```  
---  
jobs:  
- name: my-job  
  plan:  
  - get: my-resource  
  - task: first-task  
    file: task1.yml  
  - task: second-task  
    file: task2.yml  
  - task: third-task  
    file: task3.yml      
```  


### <a name="add-task-to-job-after-another-task"></a> Add a task to a job after a specific task

The combined use of the `replace` operation from `yaml-patch` with the `do` tasks/resources grouping in Concourse pipelines allows you to "insert" tasks into the middle of the tasks of a job.

1. Source YAML: job-insert-task.yml  
```  
---  
jobs:  
- name: my-job  
  plan:  
  - get: my-resource  
  - task: first-task  
    file: task1.yml  
  - task: last-task  
    file: taskn.yml    
```  

1. Operations file: insert-task.yml  
```  
- op: replace  
  path: /jobs/name=my-job/plan/task=first-task   
  value:  
    do:  
    - task: first-task  
      file: task1.yml  
    - task: second-task   
      file: task2.yml  
```  

1. Execute yaml-patch command  
   `cat job-insert-task.yml | yaml-patch -o insert-task.yml  > result.yml`    

1. Resulting patched file: result.yml  
```  
---  
jobs:  
- name: my-job  
  plan:  
  - get: my-resource  
  do:  
    - task: first-task  
      file: task1.yml  
    - task: second-task   
      file: task2.yml   
  - task: last-task   
    file: taskn.yml        
```  


### <a name="replace-resource-definition"> Replace a resource definition in the pipeline YML

The example below is similar to the resource replacement action done by pcf-pipelines in operations file [use-pivnet-release.yml](operations/use-pivnet-release.yml) to use the pipelines release from PivNet instead of GitHub. When the resource name is updated, all the tasks that reference that name also need to be updated accordingly.    

1. Source YAML: resource-entry.yml  
```  
---  
resources:  
- name: pcf-pipelines  
  type: git  
  source:  
    uri: git@github.com:pivotal-cf/pcf-pipelines.git  
    branch: master  
    private_key: {{git_private_key}}  
```  

1. Operations file: replace-resource.yml  
```  
---  
- op: replace  
  path: /resources/name=pcf-pipelines  
  value:  
    name: pcf-pipelines
    type: pivnet  
    source:  
      api_token: "{{pivnet_token}}"  
      product_slug: pcf-automation  
      product_version: ~  
```  

1. Execute yaml-patch command  
   `cat resource-entry.yml | yaml-patch -o replace-resource.yml > result.yml`    

1. Resulting patched file: result.yml  
```  
---   
resources:  
- name: pcf-pipelines
  type: pivnet  
  source:  
    api_token: "{{pivnet_token}}"  
    product_slug: pcf-automation  
    product_version: ~    
```  


### <a name="change-trigger-flag"> Change the Trigger flag of a job resource

For a sample on how to update the *Trigger* parameter for the `apply-changes` job of the [upgrade-tile.yml](https://github.com/pivotal-cf/pcf-pipelines/blob/master/upgrade-tile/pipeline.yml#L103) pipeline, see sample [gated-apply-changes-job.yml](operations/gated-apply-changes-job.yml).

1. Source YAML: job-trigger.yml  
```  
---  
jobs:  
- name: my-job  
  plan:  
  - get: my-resource  
    trigger: true
  - get: her-resource  
    trigger: true
  - task: first-task  
    file: task1.yml  
```  

1. Operations file: replace-trigger.yml  
```  
- op: replace
  path: /jobs/name=my-job/plan/get=my-resource/trigger
  value: false
```  

1. Execute yaml-patch command  
   `cat job-trigger.yml | yaml-patch -o replace-trigger.yml > result.yml`    

1. Resulting patched file: result.yml  
```  
---  
jobs:  
- name: my-job  
  plan:  
  - get: my-resource  
    trigger: false
  - get: her-resource  
    trigger: true
  - task: first-task  
    file: task1.yml  
```  

### <a name="additional-operations-samples"> Additional operations samples

See [operations](operations) for more examples of yaml-patch operations.

- [Use Artifactory as the source for the Upgrade ERT pipeline resource](operations/upgrade-ert-use-artifactory.yml)
- [Use Artifactory as the source for the Upgrade Tile pipeline resource](operations/upgrade-tile-use-artifactory.yml)