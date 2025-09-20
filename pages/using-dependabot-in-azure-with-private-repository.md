# Using Dependabot in Azure with a private repository

Regular dependency updates are important for security, quality and compatibility of applications over time. Using Dependabot is one way to keep the packages you use updated to the latest versions.
You can either use the dependabot-script or dependabot-azure-devops in Azure to configure it for your repository.

Let us look at an example using the dependabot extension for Azure DevOps. You enable version updates by checking a dependabot.yml configuration file in to your repository's .github directory. Dependabot then raises pull requests to keep the dependencies you configure up-to-date. You can go through the [official spec](https://docs.github.com/en/code-security/dependabot/dependabot-version-updates/configuration-options-for-the-dependabot.yml-file) for all the configuration options.

A simple configuration in /.github/dependabot.yml for a maven project would be for example:
~~~
version: 2
updates:
- package-ecosystem: "maven"
  directory: "/"
  schedule:
  interval: "monthly"
  target-branch: "development"
  open-pull-requests-limit: 10
~~~

You need a way to trigger dependabot. You can do that in an Azure pipeline using the [dependabot task](https://marketplace.visualstudio.com/items?itemName=tingle-software.dependabot). In our simple example, that would be:
~~~
trigger: none

stages:
- stage: CheckDependencies
  displayName: 'Check Dependencies'
  jobs:
    - job: Dependabot
      displayName: 'Run Dependabot'
      pool:
      vmImage: 'ubuntu-latest'
      steps:
        - task: dependabot@1
          displayName: 'Run Dependabot'
~~~

If you trigger this pipeline manually, it would work if all the libraries in your project's pom.xml are fetched from public maven repository.
If you are using a private repository which requires authentication in your pom.xml. then this pipeline will fail and no PRs will be initiated as it cannot access the private repository.
You would see HTTP 401 or 404 in the pipeline logs with an error message that may look like this:
> The following source could not be reached as it requires authentication (and any provided details were invalid or lacked the required permissions): <<URL of the repository>> (Dependabot::PrivateSourceAuthenticationFailure)

To solve that, you need to pass the access credentials to dependabot. In our example, the private repository is an incoming maven feed in Azure which requires a Personal Access Token (PAT) for authentication.
Typically, you can use the system access token as PAT. System.AccessToken is a predefined variable that carries the security token used by the running build. This variable is agent-scoped, and can be used as an environment variable in a script and as a parameter in a build task. You can also create a PAT just for this purpose in Azure as well and use it if you really need it in your setup. Just remember to create it with Full Code scope.

In our example with Azure incoming maven feed, we first map the System.AccessToken into the pipeline using a variable. Like so:
~~~
steps:
- task: dependabot@1
  displayName: 'Run Dependabot'
  env:
  SYSTEM_ACCESSTOKEN: $(System.AccessToken)
~~~

Then we can use the SYSTEM_ACCESSTOKEN variable in the dependabot configuration as password to authenticate to the incoming maven feed URL, the user name being your azure organization name.
For example:
~~~
version: 2
registries:
maven-github:
type: maven-repository
url:  https://pkgs.dev.azure.com/AZURE_DEVOPS_ORGANIZATION/XxxXxx/_packaging/XXXXX-incoming-maven-feed-for-java/maven/v1
username: AZURE_DEVOPS_ORGANIZATION
password: ${{SYSTEM_ACCESSTOKEN}}
updates:
- package-ecosystem: "maven"
  directory: "/"
  schedule:
  interval: "monthly"
  target-branch: "development"
  open-pull-requests-limit: 10
~~~

With this, we are all set for the authentication to succeed. Dependabot shall fetch all the updates for libraries using your private repository and trigger pull requests if there are updates till it reaches the configured pull request limit.

![Visitor Count](https://visitor-badge.laobi.icu/badge?page_id=kumaresh.github.io.using-dependabot-in-azure-with-private-repository)