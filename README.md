1. Signin to `azure.dev.com` (n.ishantsingh87@gmail.com)

2. Go to create free pipleine

3. Create an organization

   ![image-20191108214527100](images/create-organization.png)

   

4. Create a project
  ![image-20191108214637075](images/create-project.png)



### Create build pipeline

1. Go to pipeline, and click on **new pipeline**


6. Choose github as source code repository 

   ![image-20191108215015138](images/choose-source-repo.png)

 

7. After Oauth with GitHub, choose the repository **(saxo-university)**

8. Choose the starter pipeline for configuration options

   ![image-20191108215301965](images/starter-pipeline.png) 



9. Now we can edit teh starter pipeline boilerplate

   ![image-20191108215355000](images/stater-pipeline-boilerplate.png)



10. Edit the pipeline yam to create three task (as scripts)

   - yarn install

   - yarn lint

   - yarn build

     ```yaml
     # This sets trigger to any update in master  
     # trigger:
     # - master
     
     #  Defines the host for agent (MS hosted)
     pool:
       vmImage: 'ubuntu-latest'
     
     #  Build steps for the pipeline
     steps:
     - script: yarn 
       displayName: 'Installing node modules'
     
     - script: yarn lint
       displayName: 'Checking for lint errors'
     
     - script: |
         yarn build
       displayName: 'Running webpack build'
     
     # Save artifacts to refer them in release builds
     - task: PublishPipelineArtifact@1
       inputs:
         path: $(Agent.BuildDirectory)/s/build
         artifact: react-static-website-build-artifacts
     ```
     
     
   
11. Select the yaml path in repository, this will be saved in repo on commit 

    ![image-20191108220006270](images/set-build-yaml-path.png)



Define work directory for all steps : https://github.com/MicrosoftDocs/vsts-docs/issues/6315



12. Click on save and run, choose to commit changes in master branch



**Publishing artifacts**

- To be able to deploy the bundled resources (e.g. to Azure Storage), we will need to retain and share the results (files) of this build with release build 

- Lets instruct the azure to save the result of our build aka **Pipeline Artifacts**

- Add following task to the pipeline.yml

  ```yaml
      # Save artifacts to refer them in release builds
      - task: PublishPipelineArtifact@1
        inputs:
          path: $(Agent.BuildDirectory)/s/projects/react-static-website/build
          artifact: react-static-website-build-artifacts
  ```

- Now after the build is run, open the build result, we will see an option to view/download artifacts : 

  ![image-20191110115958567](images/download-artifacts.png)







### Create release pipeline

- Go to release tab and click on **New Pipeline**
- ![image-20191110154011530](images/create-release-pipeline.png)





- Click on **Artifacts | + Add** to select artifacts that will be deployed in this build

- **Choose the build from dropdown** to get the artifacts from

  ![image-20191110154310776](images/choose-artifacts-build-create-release-pipeline.png)

  


- Click on add and now we will see the build pipeline as : 

  ![image-20191110154547260](images/added-artificats-create-release-pipeline.png)





### Create release task to deploy to the dev environment

- Click on **Stages | +Add**

- Choose empty job and click on **Apply**

  ![image-20191110154818151](images/create-stage-empty.png)



- Choose name of the stage and click on close button of stage dialogue 

  ![image-20191110154937460](images/set-stage-name.png)





- Click on '1 job, 0 task' to add a task to the stage

  ![image-20191110155122013](images/add-task-to-release-job.png)



- Click the **+** icon on the agent job to create a task

  ![image-20191110155350673](images/add-release-job-task.png)



- Search for "Azure file copy" for available azure tasks:

  ![image-20191110155505798](images/stoage-task-for-release.png)



- Click on newly created task **File Copy** to start editing it

  ![image-20191110160640110](images/edit-file-copy-task.png)
  



- Edit the task as 

  - Display name as **Deploy to Dev Server**

- Click on **...** button next to Source input, this will show a dialog to select available artifacts

  - select the **react-static-website-build-artifacts** (we create this in our build )
- make sure you select the right dir here.
  
  ![image-20191110165638275](images/select-correct-artifacts-build-create-release-pipeline.png)



- Choose free trial from available **Azure subscription** dropdown

  ![image-20191110161403652](images/select-file-task-azure-subscription-dropdown.png)



- Click on authorize to allow release task to connect to the **Azure Storage** service, using our free trial subscription

  - this will open and OAuth window with microsoft live

  ![image-20191110161534413](/Users/dawn/Library/Application Support/typora-user-images/image-20191110161534413.png)



- Select **Destination Type** from dropdown as **staticwebsiteazure** (our dev server)
- In container name, enter **$web**
- ![image-20191110161844446](images/select-container-destination-name-copy-task.png)



- Click on save button to save the create build task

  ![image-20191110162039502](images/save-release-to-dev-file-copy-task.png)





### Configure Auto Deployment of artifacts from master branch

- Go back to pipelines tab to continue editing the release pipeline

- Click on **trigger icon** over the artifacts: 

  ![image-20191110162948752](images/continuous-deployment-trigger.png)



- Click on enabled in the dialogue 

  ![image-20191110163108484](images/enable-continuous-deployment-trigger.png)



- Add filter to make sure the continuous deploy only happens for master branch

- click on **+ Add ** button below the **Build branch filters** and enter **master** in branch name

- now click on close on top of dialog

  ![image-20191110163339448](images/select-branch-for-continuous-deployment-trigger.png)

  



- Click on the **Save** on top right to save the new trigger

![image-20191110163603892](images/save-continuous-deployment-trigger.png)



​ ![image-20191110163647355](images/save-comment-continuous-deployment-trigger.png)

