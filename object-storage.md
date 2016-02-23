---
layout: post
title: Object Storage
permalink: /object-storage/
---

Bluemix Object Storage Service provides unstructured cloud storage to manage your files.

In this tutorial you will learn how to deploy the Object Storage Application using Devops Delivery Pipeline and to familiarize yourselves in using Object Storage service as well. 

####**Copy a Github Repository**

1. Open a web browser and login to [Github](https://github.com/). 

2. Go to the Github Repository [`https://github.com/danicacorpuz/Object-Storage`](https://github.com/danicacorpuz/Object-Storage)

3. Fork the repository by clicking the `Fork` button to get a copy to your account.

####**Create a Bluemix Devops Project based on the Github Repository**

1. Using the same browser, open a new tab and login to [Bluemix Devops](https://hub.jazz.net/). 

2. Click the `CREATE PROJECT` button.

3. Name your project `Object-Storage`.

4.  Click `Link to an existing Github repository`.

5. Select the repository `<username>/Object-Storage` from the dropdown list.

6. On the `Select project contents` section, make sure that the following options are selected:

||||
|---|---|---|
	|**Private Project**| checked |
	| **Add features for Scrum development** | checked |
	| **Make this a Bluemix Project** | checked |
	| **Region** | IBM Bluemix US South |
	| **Organization** | you may leave the default selection |		
	| **Space** | dev |

7. Click `CREATE`.

####**Create a Build Stage**

1. Click the `BUILD & DEPLOY` button.

2. Click the `ADD STAGE` button. Change the stage name `MyStage` to `Build Stage`.

3. On the `INPUT` tab, set the following values:

||||
|---|---|---|
	| **Input Type** | SCM Repository |
	| **Git URL** | https://github.com/danicacorpuz/Object-Storage.git |
	| **Branch** | master |
	| **Stage Trigger** | Run jobs whenever a change is pushed to Git |

4. On the `JOBS` tab, click the `ADD JOB` button and select `BUILD` as the Job Type.

5. Change the job name `Build` to `Gradle Assemble` and set the following values:

||||
|---|---|---|
	| **Builder Type** | Gradle |		
	| **Build Shell Command** | `#!/bin/bash`<br>`export PATH="$GRADLE2_HOME/bin:$PATH"`<br>`gradle assemble`  |	
	| **Stop running this stage if this job fails** | checked |

6. Click the `SAVE` button.

####**Create a Test Stage**

1. Create another stage by clicking the `+ ADD STAGE` button. Change the stage name `MyStage` to `Test Stage`.

2.  On the `INPUT` tab, set the following values:

||||
|---|---|---|
	| **Input Type** | Build Artifacts |
	| **Stage** | Build Stage |
	| **Job** | Gradle Assemble |
	| **Stage Trigger** | Run jobs when the previous stage is completed |

3. On the `JOBS` tab, click the `ADD JOB` button and select `TEST` as the Job Type.

4. Change the job name `Test` to `JUnit Test through Gradle` and set the following values:

||||
|---|---|---|
	| **Tester Type** | Simple |		
	| **Test Command** | `#!/bin/bash`<br>`gradle test`  |	
	| **Stop running this stage if this job fails** | checked |

5. Click the `SAVE` button.


####**Create a Deploy Stage**

1. Click the `+ ADD STAGE` button to create a deploy stage. Change the stage name `MyStage` to `Deploy Stage`.

2.  On the `Input` tab, set the following values:

||||
|---|---|---|
	| **Input Type** | Build Artifacts |
	| **Stage** | Build Stage |
	| **Job** | Gradle Assemble |
	| **Stage Trigger** | Run jobs when the previous stage is completed |

3. On the `JOBS` tab, click the `ADD JOB` button and select `DEPLOY` as the Job Type.

4. Change the job name `Deploy` to `CF Push to Dev Space` and set the following values:

||||
|---|---|---|
	| **Deployer Type** | Cloud Foundry |		
	| **Target** | IBM Bluemix US South - https://api.ng.bluemix.net |		
	| **Organization** | you may leave the default selection |		
	| **Space** | dev |	
	| **Application Name** | blank |		
	| **Deploy Script** | `#!/bin/bash`<br>`cf push os-<your_name> -m 512M -p build/libs/ObjectStorage.war`  |	
	| **Stop running this stage if this job fails** | checked |

5. Click `SAVE`.

####**Deploy the Application using Delivery Pipeline**

1. On the `BUILD & DEPLOY` tab, click the `Run Stage` icon of the `Build Stage`.

	>Make sure all the stages are completed and passed before you proceed to the next step.

2. Go to [IBM Bluemix](ibm.biz/bluemixph) Website and login.

3. Click the widget of your application `os-<your_name>`.

4. Click the `ADD A SERVICE OR API` to add the Object Storage service to your application.

5. In the `CATALOG` page, look for the `Object Storage` service and click it.

6. Type `ObjectStorage-<your name>` in the `service name` text box and select the `Free` plan.

7. Click `CREATE`. When asked to restage your application, click the `RESTAGE` button. Wait for your application to restage.
8. Open a new tab and go to `http://os-<your_name>.mybluemix.net/home.jsp` to test if the application can connect to the Object Storage service. 

9. Choose any file you want to upload by clicking the `Choose File` button.

10. Click the `Upload File` button.

11. After clicking the button, you will be redirected to `home.jsp` page and this time, the file you uploaded earlier is displayed on the page.

12. Try to click the `Download` button and check if the application can successfully download the file.

####**Analyze how the Object Storage Application works**

In this tutorial, you were able to communicate with the Object Storage service to upload and download files directly. 

The code shown below shows the connection to the service.  The credentials needed to successfully connect to the service are `auth_url`, `userId`, `password`, `domainName`, and `project`.  The domainName and project attributes are passed to Identifier class to create an Identifier which is a name based authentication.

```text
Identifier domainIdent = Identifier.byName(domainName);
Identifier projectIdent = Identifier.byName(project);

OSClient os = OSFactory.builderV3()
                .endpoint(auth_url)
                .credentials(userId, password)
                .scopeToProject(projectIdent, domainIdent)
                .authenticate();
```

The Object Storage service has different functions where you can `add container`, 	`delete container`, `upload file`, `download file`,  and `delete file`. However, only the upload file and download file were demonstrated in the tutorial.

The code below shows some of the functions you can do to the service.

```text
	public boolean createContainer(String cName) {
        return os.objectStorage().containers().create(cName).isSuccess();
    }
    
    public boolean deleteContainer(String cName) {
        return os.objectStorage().containers().delete(cName).isSuccess();
    }
    
    public String uploadFile(String cName, String fName, Payload payload) {
        return os.objectStorage().objects().put(cName, fName, payload);
    }
	
	public boolean deleteFile(String cName, String fName) {
        return os.objectStorage().objects().delete(cName, fName).isSuccess();
    }
    
	public SwiftObject getFile(String cName, String fName) {
		return os.objectStorage().objects().get(cName, fName);

	public SwiftAccount getAccount() {
        account = os.objectStorage().account().get();
        return account;
    }
	
	public List listAllObjects(String containerName) {
        return os.objectStorage().objects().list(containerName);
    }
```
Before you can upload and download a file, there should be an existing container where you can store and retrieve your files. All the files or objects are created within a container. This tutorial used one container only which is named as `sample`. You can create many containers as long as you do not go beyond your usage limit. In addition, containers work independently to one another. You can upload the same file to different containers since it represents two different objects.

####**Delete the Application and Object Storage Service**
1. Go to [IBM Bluemix](ibm.biz/bluemixph) Website and click the `Dashboard`.
2. From the `Applications` section, click the `gear` icon in the widget of the `os-<your_name>` application.
3. Select the `Delete App` from the list.
4. In the `Services` tab, select the `Object Storage` service.
5. Make sure that the route is selected as well.
6. Click the `Delete` button.

####End of Tutorial