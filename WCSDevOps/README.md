
# Before Deploy #

Even though, the WCSDevOps helm charts can help deploy Nexus/Vault_Consul/DeployController, there still have some dependence tools you need to prepare ahead

## Docker Image Repository ##
There have lots of option user can choose for the Docker Image Repository ( Docker Registry / Harbor / Nexus Repository ... ), so we not include it in Helm Charts.

When you have prepared the Docker Image Repository, please create "commerce" library and upload `all Commerce OOTB Docker image` under it. Meanwhile, please upload

DeploySlave / SupportContainer and all other containers in WC-DevOps-Utilities project under 'commerce' library with latest tag. Because DevOps Utilities will download them with latest tag as default.

Current Docker Image Repository  url will be the value for parameter 'DockerRepo' when you config Values.yml for DeployController.

Here are the image list you need to upload to Docker Repository for DevOps Utilities

deployslave:latest
supportcontainer:latest



## Helm Charts Repository ##

When you trigger job to deploy Commerce V9, DeployController will auto fetch Helm Charts you specified from Helm Charts Repository. Since the Helm Charts reposiroty can be any web server. so we not include in
in Helm Charts, we assume user already prepare it.

The Quick way to setup Helm Charts Repository is leverage the Helm Client ( You can get help by run `helm serve --help` )

1. Logon Server Witch has installed Helm Client
2. Create Folder  "HelmChartsRepo"
3. Upload Commerce V9 Helm Chart package "WCSV9" to ./HelmChartsRepo
4. Run Helm package command to package "WCSV9" folder
```
  helm pacakge <FULLPATH>/WCSV9
```
5. Run command to start Helm Serve service: helm serve --repo-path ./HelmChartsRepo --address ServerIPAddress:8879 ( replace the ServerIPAddress with real IP )

When you finish all steps you will see WCSV9-ChartVersion.tgz file and index.yaml file under ./HelmChartsRepo

you can check helm repo server by open index.yml.  There should have one record like below:

```
apiVersion: v1
entries:
  WCSV9:
  - apiVersion: v1
    created: 2018-08-02T17:06:31.80912053+08:00
    description: Commerce Helm chart
    digest: 8fac0a50d9c5831dc046b29645a98f3948aac97d8e5aecebe284830571fc4b38
    name: WCSV9
    urls:
    - http://x.x.x.x:8879/WCSV9-0.1.x.tgz
    version: 0.1.x
```

The Helm Server Address http://ServerIPAddress:8879 will be the value for parameter "HelmChartsRepo" when you config values.yml for DeployController

# Deploy With Helm Charts #

## Prepare Values.yml ##
Before you use this Helm Chart, please edit vaules.yml

1. Correct the Docker Image url
   Vault/Consul and Nexus will try to download docker image from internet as default. Since DeployController need to 
   be build by yourself, please provide a valid Docker Image name for it.
2. Set appropriate values for DeployController. ( please see below parameters table )
3. Vault/Consul and Nexus are option, if you want use existed Vault/Consul or Nexus, you can change the "Enable" value to "false"
4. IF you want to reuse exist Vault/Consul. Please make sure DeployController be deployed in same namespace with Vault/Consul and set "InCluster" to "true"


Deploycontroller Container support some environment parameters to enable auto-configuration, such as:

Parameter  |  Usage
------------- | -------------
VaultUrl |  Specify Vault URL ( e.g http://9.112.245.194:30552 ). If InCluster equal true, this value is not mandatory
VaultToken  | Specify Vault Root Token for rest access . If InCluster equal true, this value is not mandatory
BundleRepo | Specify customize package repository, Nexus is the default bundle repository ( e.g  http://nexus:8081/nexus/content/repositories/releases/commerce ),  Since service for Nexus will be create as NodePort model, you also can K8s Node IP in URL.
DockerRepo | Specify Docker Image repository (e.g DockerRepoHostname:RepoPort )
KubernetesUrl | Specify Kubernetes url for remote call from Jenkins. If InCluster equal true, this value is not mandatory
DockerRepoUser   | Specify User Name of Docker Image Repository for logon when download Docker Image
DockerRepoPwd  | Specify User Password of Docker Image Repository for logon when download Docker Image
HelmChartsRepo  | Specify Helm Charts Repository for update Helm Charts | handle helm pre and post install hook / as InitContainer to controller startup sequence (e.g http://9.112.245.194:8879/charts)
InCluster | true or false. If InCluster equal true, DeployController will auto defect Vault and Kubernetes ULR and config related parameters ( see parameters which marked not mandatory, when InCluster equal true). Please make sure Vault and DeployController in same namespace.

Note:

IF you don't know how to fill in the parameter for DeployController, you can set InCluster parameter to "false" and leave all values empty. The Helm Deploy also will get success.

Then you can logon DeployController and manually config those parameters in Manage Jenkins --> Configure System --> Environment variables view

 <img src="https://github.com/IBM/wc-devops-utilities/raw/master/doc/images/DeployController-GlobalConfig1.png" width = "700" height = "450" alt="Overview" align=center /><br>

 <img src="https://github.com/IBM/wc-devops-utilities/raw/master/doc/images/DeployController-GlobalConfig2.png" width = "700" height = "450" alt="Overview" align=center /><br>


## Deploy with Helm Charts ##
```
helm install --name=wcsdeploycontroller ./WCSDevOps
```

## Access DevOps Utilities ##

http://NodeIP:31895  ( DeployController will be start with NodePort service, so you can use any K8s Node IP to access it, IF you are using ICP, you can use Ingress IP address with 31895 to access it )

## Initial Predefined Job ##
Some of job work with dynamic parameter plugin. So before you launch them, please open and save it to make dynamic parameter plugin can be load.

# Upload Customized Assets (Option) #
As default, DeployController integrate with Nexus to support build customized Docker Image by run pre-defined Job "BuildDockerImage_Base".  This need user store and manage customized package
on Nexus. As default, all the custom package should be store in release repository under group commerce.Tenant ( commerce.demo )


<img src="https://github.com/IBM/wc-devops-utilities/raw/master/doc/images/NexusPackageStructure.png" width = "700" height = "450" alt="Overview" align=center /><br>

For upload custom pacakge to Nexus, please refer below gradle scritps to create "build.gradle" file.
```
apply plugin: "maven"
apply plugin: "maven-publish"

publishing{
        repositories {
            maven {
                url "http://NexusServerAddress:8081/nexus/content/repositories/releases"
                credentials {
                    username 'NexusUser'
                    password 'NexusUserPwd'
                }
            }
        }
        publications {
            zip(MavenPublication) {
                groupId 'commerce.TenantName'
                artifactId 'ComponentName'
                version 'PackageVersion'
                artifact 'PackageLocation'
            }
    }
}

```

Please correct the value for NexusServerAddress / NexusUser / NexusUserPwd / ComponentName / PackageVersion / PackageLocation

For example:

groupId:commerce.demo <br>
artifactId: crs-app <br>
version: 20170807 <br>
type: zip  <br>

When you prepared the build.gradle file, you can run belwo command at the same location with build.gradle
```
gradle publish
```


# Start DeployController on local Without Configurtion #
```
docker run -d -p 8080:8080 -p 50000:50000 deploycontroller:latest
```

by this way, deploycontroller will be startup without any configuration. When it startup, you can logon it and edit

Manage Jenkins --> Configure System --> Environment variables

to input those parameter by manually then save the change


# Start your deployment journey #

1. Create Tenant ( use CreateTenant_Base job )
2. Populate Environment Data to Vault ( use ManageVaultConfig_Base job )
3. Deploy (update) Commerce  ( use DeployWCSCloud_Base )

For details about the pre-defined job, please see [DeployControllerDesign](https://github.com/IBM/wc-devops-utilities/blob/master/doc/DeployControllerDesign.md)

