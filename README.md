# Concourse resource for sonar scanning a dotnet core application

Provides support for [SonarQube](https://www.sonarqube.org/) scanning a C# .NET core 2 application.

Includes support for:
* Core SonarQube scans
* Test coverage using [MiniCover](https://github.com/lucaslorentz/minicover)

## Usage

### Scan application and upload to SonarQube

Source parameters:
* host_url - the URL to the SonarQube server (e.g. http://localhost:9000)
* login - the SonarQube login token

Parameters:
* project_name - the name of the project to show in SonarQube
* project_key - the configured project key in SonarQube
* source - the directory containing the source code to analyze
* version - reference to the project version file - as generated by the semver resource (optional)
* coverage_threshold - the desired minimum coverage percentage (defaults to 70)

Expects source directory to contain one or more directories containing ```.csproj``` files.  Test projects are identified by ```.csproj``` files including ```Microsoft.NET.Test.Sdk```.  Any other ```.csproj``` is considered to be source.  All ```.csproj``` files must include a ```ProjectGuid``` field.

```
resource_types:
- name: sonarqube-scanner
  type: docker-image
  source:
    repository: subnova/concourse-dotnet-sonarscanner-resource
    tag: "latest"

resources:
- name: sonarqube-scan
  type: sonarqube-scanner
  source:
    host_url: ((sonarqube_host_url))
    login: ((sonarqube_login_token))

jobs:
- name: sonarqube
  plan:
  - get: version
  - get: my-source
  - put: sonarqube-scan
    params:
      project_name: MyProjectName
      project_key: myproject
      version: version/version
      source: my-source
      coverage_threshold: 60
```

### Retrieve quality gates status

Source parameters:
* host_url - the URL to the SonarQube server (e.g. http://localhost:9000)
* login - the SonarQube login token

Parameters:
* project_key - the configured project key in SonarQube

```
resource_types:
- name: sonarqube-scanner
  type: docker-image
  source:
    repository: subnova/concourse-dotnet-sonarscanner-resource
    tag: "latest"

resources:
- name: sonarqube-scan
  type: sonarqube-scanner
  source:
    host_url: ((sonarqube_host_url))
    login: ((sonarqube_login_token))

jobs:
- name: sonarqube-qualtity-status
  plan:
  - get: sonarqube-scan
    params:
      project_key: myproject
```
