---
layout: docs
title: Build Worker API
---

# Build Worker API

AppVeyor Build Worker is the service running on build VM. It provides REST API for sending real-time messages and test results to the build console, and pushing artifacts.

To access Build Worker API from your scripts, AppVeyor provides the command-line utility `appveyor.exe` and PowerShell module.

API URL is stored in `APPVEYOR_API_URL` environment variable and it is `localhost` with a random port, e.g. `http://localhost:9023/`

* [Add message](#add-message)
* [Set environment variable](#set-environment-variable)
* [Add compilation message](#add-compilation-message)
* [Add tests](#add-tests)
* [Update tests](#update-tests)
* [Push artifact](#push-artifact)
* [Update build details](#update-build-details)
* [Start new build](#start-new-build)


## Add message

![build messages](/assets/images/docs/build-messages.png)

### PowerShell

    Add-AppveyorMessage -Message <string>
           [-Category <category> {Information | Warning | Error}]
           [-Details <string>]

Example:

```powershell
Add-AppveyorMessage "This is a test message"
```

### Command line

    appveyor AddMessage <message> [options]

Options are similar to PowerShell cmdlet parameters.

### REST

    POST api/build/messages

Request body (JSON):

```json
{
    "message": "This is a test message",
    "category": "warning",
    "details": "Additional information for the message"
}
```




## Add compilation message

### PowerShell

    Add-AppveyorCompilationMessage -Message <string>
           [-Category <category> {Information | Warning | Error}] [-Details <string>]
           [-FileName <string>] [-Line <int>] [-Column <int>] [-ProjectName <string>]
           [-ProjectFileName <string>]

Example:

```powershell
Add-AppveyorCompilationMessage "Unreachable code detected" -Category Warning -FileName "Program.cs" -Line 1 -Column 3
```

### Command line

    appveyor AddCompilationMessage <message> [options]

Options are similar to PowerShell cmdlet parameters.

### REST

    POST api/build/compilationmessages

Request body (JSON):

```json
{
    "message": "This is a test message",
    "category": "warning",
    "details": "Additional information for the message",
    "fileName": "program.cs",
    "line": "1",
    "column": "10",
    "projectName": "MyProject",
    "projectFileName": "MyProject.csproj"
}
```



## Set environment variable

### PowerShell

    Set-AppveyorBuildVariable -Name <string> -Value <string>

Example:

```yaml
Set-AppveyorBuildVariable 'MyVar1' 'This is a test message'
```

### Command line

    appveyor SetVariable -Name <string> -Value <string>

### REST

    POST api/build/variables

Request body (JSON):

```json
{
    "name": "variable_name",
    "value": "hello, world!"
}
```



## Add tests

![build messages](/assets/images/docs/build-tests.png)

### PowerShell

    Add-AppveyorTest -Name <string> -Framework <string> -FileName <string>
           [-Outcome <outcome> { None | Running | Passed | Failed | Ignored | Skipped
           | Inconclusive | NotFound |  Cancelled | NotRunnable}] [-Duration <long>]
           [-ErrorMessage <string>] [-ErrorStackTrace <string>]
           [-StdOut <string>] [-StdErr <string>]

AddTest options:

      -Name             - Required. The name of test.
      -Framework        - Required. The name of testing framework, e.g. NUnit, xUnit, MSTest.
      -FileName         - Required. File name containg test.
      -Outcome          - Test outcome: None, Running, Passed, Failed, Ignored, Skipped, Inconclusive, NotFound, Cancelled, NotRunnable
      -Duration         - Test duration in milliseconds.
      -ErrorMessage     - Error message of failed test.
      -ErrorStackTrace  - Error stack trace of failed test.
      -StdOut           - Standard console output from the test.
      -StdErr           - Error output from the test.

Example:

```powershell
Add-AppveyorTest -Name "Test A" -Framework NUnit -Filename a.exe -Outcome Passed -Duration 1000 # in milliseconds
```

### Command line

    appveyor AddTest <name> [options]

Options are similar to PowerShell cmdlet parameters.

### REST

To add a single test:

    POST api/tests

Request body (JSON):

```json
{
    "testName": "Test A",
    "testFramework": "NUnit",
    "fileName": "tests.dll",
    "outcome": "Passed",
    "durationMilliseconds": "1200",
    "ErrorMessage": "",
    "ErrorStackTrace": "",
    "StdOut": "",
    "StdErr": ""
}
```

To add tests in a batch:

    POST api/tests/batch

Request body (JSON):

```json
[
    {
        "testName": "Test A",
        "testFramework": "NUnit",
        "fileName": "tests.dll",
        "outcome": "Passed",
        "durationMilliseconds": "1200",
        "ErrorMessage": "",
        "ErrorStackTrace": "",
        "StdOut": "",
        "StdErr": ""
    },
    {
        "testName": "Test B",
        "testFramework": "xUnit",
        "fileName": "tests.dll",
        "outcome": "Passed",
        "durationMilliseconds": "10"
    },
    ...
]
```


## Update tests

### PowerShell

    Update-AppveyorTest -Name <string> -Framework <string> -FileName <string>
           -Outcome <outcome> { None | Running | Passed | Failed | Ignored | Skipped
           | Inconclusive | NotFound |  Cancelled | NotRunnable}] [-Duration <long>
           [-ErrorMessage <string>] [-ErrorStackTrace <string>]
           [-StdOut <string>] [-StdErr <string>]

Example:

```powershell
Add-AppveyorTest -Name "Test A" -Framework NUnit -FileName a.exe -Outcome Running
Start-sleep -s 10
Update-AppveyorTest -Name "Test A" -Framework NUnit -FileName a.exe -Outcome Passed -Duration 10000
```

### Command line

    appveyor UpdateTest -Name <name> [options]

Options are similar to PowerShell cmdlet parameters.

### REST

To update a single test (test is matched by name):

    PUT api/tests

Request body (JSON):

```json
{
    "testName": "Test A",
    "testFramework": "NUnit",
    "fileName": "tests.dll",
    "outcome": "Passed",
    "durationMilliseconds": "1200",
    "ErrorMessage": "",
    "ErrorStackTrace": "",
    "StdOut": "",
    "StdErr": ""
}
```

To update tests in a batch (tests are matched by name):

    PUT api/tests/batch

Request body (JSON):

```json
[
    {
        "testName": "Test A",
        "outcome": "Passed",
        "durationMilliseconds": "1200"
    },
    {
        "testName": "Test B",
        "outcome": "Passed",
        "durationMilliseconds": "10"
    },
    ...
]
```



## Push artifact

![build messages](/assets/images/docs/build-artifacts.png)

### PowerShell

    Push-AppveyorArtifact <path> [-FileName <string>] [-DeploymentName <string>]
           [-Type <type> {Auto, WebDeployPackage}]
           [-Verbosity <string> {Normal, Minimal, Quiet}]

Example:

```powershell
Push-AppveyorArtifact mypackage.nupkg
```

### Command line

    appveyor PushArtifact <path> [options]

Options are similar to PowerShell cmdlet parameters.

### REST

    POST api/artifacts

Request body (JSON):

```json
{
    "path": "c:\projects\myproject\mypackage.nupkg",
    "fileName": "mypackage.nupkg",
    "name": null,
    "type": "NuGetPackage"
}
```

Response body (JSON) contains temporary URL for uploading artifact. If this is an appveyor.com address, like this:

    "https://ci.appveyor.com/api/artifacts/abc123/mypackage.nupkg"

Then the file can be uploaded using WebClient, e.g. in PowerShell:

```powershell
(New-Object System.Net.WebClient).UploadFile( `
    "https://ci.appveyor.com/api/artifacts/abc123/mypackage.nupkg", `
    "c:\projects\myproject\mypackage.nupkg")
```

However, if the URL contains `storage.googleapis.com`, things are a little different. First, send a PUT to the returned URL containing the contents of the file in the body. Then, the artifact must be finalised with another call to the API endpoint:

    PUT api/artifacts

Request body (JSON):

```json
{
    "fileName": "mypackage.nupkg",
    "size": <file_size_in_bytes>
}
```

## Update build details

![build details](/assets/images/docs/build-details.png)

### PowerShell

    Update-AppveyorBuild [-Version <string>] [-Message <string>]
           [-CommitId <string>] [-Committed <DateTime>]
           [-AuthorName <string>] [-AuthorEmail <string>]
           [-CommitterName <string>] [-CommitterEmail <string>]

Be careful - if you set `Version` make sure it is unique for the current project.

Example:

```powershell
$version = Get-Date -Format "mmddyyyy-HHmm"
Update-AppveyorBuild -Version "1.0-$version"
```

### Command line

    appveyor UpdateBuild [options]

Options are similar to PowerShell cmdlet parameters. Example:

    appveyor UpdateBuild -Version "1.0-$version"

### REST

    PUT api/build

Request body (JSON):

```json
{
    "version": "1.0.2-rc1",
    "message": "This is my custom build commit message",
    "commitId": "12345abc",
    "committed": "Sat, 22 Feb 2014 00:39:25",
    "authorName": "John",
    "authorEmail": "john@smith.com",
    "committerName": "Jack",
    "committerEmail": "jack@brown.com"
}
```

## Start new build

Call AppVeyor API to start a new build of specified project.

### PowerShell

    Start-AppveyorBuild -ApiKey <string>
           [-AccountName <string>]
           -ProjectSlug <string>
           [-Branch <string>]
           [-EnvironmentVariables <Hashtable>]

Example:

```powershell
Start-AppveyorBuild -ApiKey $env:api_key -ProjectSlug 'deploy-web' -EnvironmentVariables @{
   var1 = 'value1'
   var2 = 'value2'
}
```

Parameters:

    -ApiKey                - Required. AppVeyor account API key.
    -AccountName           - Optional. AppVeyor account name.
    -ProjectSlug           - Required. Project slug (from URL).
    -Branch                - Optional. Default project branch is used if not specified.
    -EnvironmentVariables  - Optional. Environment variables in the form var1=value1,var2=value2, ...

We recommend placing `ApiKey` value to environment variable (either General tab of project settings or as [secure variable](/docs/build-configuration#secure-variables) in `appveyor.yml`).

### Command line

    appveyor Start-AppveyorBuild [options]

Options are similar to PowerShell cmdlet parameters. The only exception is `-EnvironmentVariables` that should have the following format:

    var1=value1,var2=value2, ...
