# IMS Devops Zowe Jenkins Pipeline
​
In this tutorial, we will create a Jenkins pipeline that will automate the process of compiling and deploying COBOL code stored in a Github repository and refresh the IMS application resources for the updates.
​
This tutorial uses Zowe CLI to submit JCL for compiling the sample application.  It uses the Zowe CLI IMS Plugin and IMS Operations API to submit IMS Type 2 commands to start and stop IMS MPP region to refresh the application resources for the updates. 
​
A sample jenkins file, COBOL application source and the compile JCL is provided in the repo.
​
## Requirements 
Before you run the sample pipeline, you need to do the following:
1. Zowe server and CLI components (you need the following Zowe components):  
    a. Zowe server on the host that supports submitting JCL  
    b. Zowe CLI   
    c. Zowe CLI IMS Plugin  
​
2. Jenkins   
    a. You need a Jenkins server to run the sample pipeline.  For Jenkins installation information, see here.
​
## Set up Jenkins Pipeline
We will start navigating to the home screen of Jenkins and selecting **New Item**. From there, enter an item name and select **Pipeline** as the project type as shown in the screenshot below. Click **OK**.
![](images/create-pipeline.png)  
​
Now you will be taken to the configuration page for your newly created pipeline. In the General tab, check the box next to **Github project** and paste the URL of your project's github repository in the text field provided.
![](images/configure-general.png)  
​
Scroll down to the Pipeline tab, and next to **Definition**, click on the dropdown menu and select **Pipeline script from SCM**. Selecting this indicates to your Jenkins pipeline that there will be a Jenkinsfile in your repository that will define how the pipeline will run and how it will be separated into separate stages.
![](images/configure-pipeline.png)  
​
You will now see an expanded view of the Pipeline tab. Next to **SCM** click on the dropdown menu and select **Git**. You will now need to provide the Repository URL (shown in the screenshot below) and any credentials if applicable.
![](images/repo-url.png)  
​
Specify the branch in which you want your Jenkins build to run on, and in the text field next to **Script Path**, make sure to provide the correct path to your Jenkinsfile in the Github repository.
![](images/modify-pipeline.png)
​
Select **Save** at the bottom of the page. Your Jenkins pipeline is ready to build and deploy your COBOL code!
​
## Jenkinsfile Description
The Jenkinsfile provided in the `ims-bank-app` is divided into six separate stages. Notice that there piece of code in each of the stages along the lines of:
```
script {
    withEnv(["PATH+ZOWE=/usr/local/bin"]) {
        ...
    }
}
```
This block of code relies on the fact that there is an instance of Zowe installed on the host running this Jenkins build.
### Clone Repository 
This is the stage that checks out the source code from the source code manager, in this case Github, in order to use and manipulate the files within the repository.
### Setup Environment  
Here, the ims plugin will be installed for Zowe by calling making a shell command call to download the extension from the remote repository.
### Create Zowe Profiles  
This stage is split into three portions. The first portion is checking whether or not the intended profile has already been created or not. If the profile already exists, it will be deleted to maintain consistency with any changes that have been made to the Jenkinsfile that pertain to the actual profile.  
The next portion is the actual creation of the two Zowe profiles (ims-profile and zosmf-profile). A Zowe profile allows you to store configuration details such as username, password, host, and port for a mainframe system. 
And the final section of the stage is setting the profiles that were created to default. That way any future shell commands made with Zowe will use those configuration details specified.
### Submit JCL Job  
This stage calls the Zowe command to submit a local JCL file pertaining to this sample COBOL application.
### Verify Output of Job  
After the JCL job has been submitted, the Job ID will be identified and will continuously use another Zowe command to check the output status of that Job until completion.
### Refresh IMS Region  
Now that the Job is complete, we will refresh the IMS Region by using the zowe ims plugin to stop and start the specified region. 