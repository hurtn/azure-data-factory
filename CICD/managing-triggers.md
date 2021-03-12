You may chose to have different trigger states in different environment for example running pipelines frequently in production but not in dev or test. If the factory is under control, there are two options:
1. Do not implement triggers in Dev, use the debug option only. Or test triggers in dev but delete the trigger from dev just prior to merging to the master branch or Publishing. 
2. Publish the trigger in a disabled state:
Below is a powershell script which has been from the [documentation](https://docs.microsoft.com/en-us/azure/data-factory/continuous-integration-deployment#script) which will perform various tasks. Please read through the information [here](https://docs.microsoft.com/en-us/azure/data-factory/continuous-integration-deployment#updating-active-triggers) to understand how this script is necessary to update triggers which may be currently running. This script is well maintained by product group but cleans up resources before the ARM template redeploys them so please use it cautiously if you are making updates/[hotfixes](https://docs.microsoft.com/en-us/azure/data-factory/continuous-integration-deployment#hotfix-production-environment) in production which are not yet deployed to dev. 
The only change made to the script was to remove a filter condition which only updates active triggers, so with the change in mind the assumption is that you will test your trigger in development ADF on your feature branch, then set the Activated state to No

![image](https://user-images.githubusercontent.com/5063077/110925119-b8ef2200-831a-11eb-8b76-14ab526e56c6.png)


Perform the pull request as normal and then Publish to the master branch once approved. The script is run in a pre and post task as follows

![image](https://user-images.githubusercontent.com/5063077/110925390-08355280-831b-11eb-8b7f-5036a934a539.png)



An example of the script arguments for Pre Task are 
-armTemplate "$(System.DefaultWorkingDirectory)/_adf/adf-ccmdev01/ARMTemplateForFactory.json" -ResourceGroupName ccm-prod -DataFactoryName adf-ccmprod -predeployment $true -deleteDeployment $false

An example of the script arguments for Post Task are
-armTemplate "$(System.DefaultWorkingDirectory)/_adf/adf-ccmdev01/ARMTemplateForFactory.json" -ResourceGroupName ccm-test -DataFactoryName adf-ccmtest -predeployment $false -deleteDeployment $true

The post task will ensure that all triggers are enabled/set to Activated Yes in that environment for which this post task runs. It is recommended you only apply this version of the script to production 

Script location:

Pre script:

https://github.com/hurtn/azure-data-factory/blob/master/CICD/adf-pre-post-custom-script.ps1

Post script:

https://github.com/hurtn/azure-data-factory/blob/master/CICD/adf-pre-post-script.ps1
