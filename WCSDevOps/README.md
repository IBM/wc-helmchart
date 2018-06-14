WCSDevOps is used for doing preparation before deploying and running CI/CD pipeline. WCSDevOps contains three deployments: vault & consul, jenkins and nexus. 

Vault is used to provide PKI service, and encrypt data of Consul. Consul is used to keep the information of Commerce environment, or you could keep environment parameters with env key-values in value.yaml.

Jenkins is a deploy controller, with all the CI/CD pipeline in it, including:
- AddCerts
- BuildDockerImage
- BundleCert
- CreateDockerfile
- CreateGroup
- DeployWCSCloud
- PopCustomTemp
- PrepareEnv
- RollBackGroup
- TriggerBuildIndex
- TriggerIndexReplica

Jenkins Container support some environment parameters to enable auto-configuration, such as:

- Incluster ( true & false ) : Jenkins will get the vault_token from current environment. This feature only support the Vault_token created by this helm package.
- vault_url: define the vault_url of your cluster. default value: "http://vault-consul.default.svc:8200/v1"
- jenkins_server: this jenkins container's url in your cluster. In general, we will use Kubernetes service for connection. default_value: "http://jenkins.default.svc:8080"
- cloud_url: If this jenkins is out of your WCS runtime based Kubernetes, you will configure this so that the jenkins cloud plugin could connect to the Kubernetes cluster.
- bundleRepo: 
- dockerRepoHost: The docker repository you just use now.
- dockerRepoPwd: The docker repository password to login
- dockerRepoUser: The docker repository username to login
- helmChartsRepo: The chart repository of your environment.

All the environment parameters have default value, it is just for feature testing. In most case, you only have to change one or two of these parameters based your cluster information.

nexus server is used to:.....