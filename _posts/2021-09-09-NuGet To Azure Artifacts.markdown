# Introduction
In the project I am currently working in we have decided to move from our self-hosted (in Azure App Service) NuGet server to use Azure DevOps Artifacts, as we have been moving more and more towards utilizing its capabilities. It should reduce our costs and add some features we have been missing like a reasonable UI to manage the packages. In this short article I will share with you how we have managed to migrate all our old packages to Azure Artifacts to make the transition easier.

// Note: this article assumes you are using Windows environment.

# How?

## Download __nuget.exe__ tool

In order to migrate the packages you need a CLI tool named __nuget.exe__. You can download latest recommended version from [here](https://www.nuget.org/downloads).

## Directories structure

I have organized my directories like below:
[![](/_posts/2021-09-09-NuGetToAzureArtifacts/nuget-migration-directory.png)](/_posts/2021-09-09-NuGetToAzureArtifacts/nuget-migration-directory.png)

- `Packages` is a directory containing \*.nupkg files. I have downloaded them all from my server via KUDU (literally just grabbed it as a ZIP file with all files from the directory).
- `migrate.ps1` is powershell script with the content presented later in the article.
- `nuget.exe` is the CLI tool you probably have already downloaded by this point.

## Scripts

### NuGet source

Firstly, we need to specify our Azure Artifacts NuGet feed. To do that navigate to the root directory of your project (in my case it's: C:\Projects\NugetMigration) and run the following command:

```PowerShell
.\nuget.exe sources add -Name "MyFeedName" -Source "MyFeedUrl"
```

where:
- `MyFeedName` should be the name of the NuGet source to be used in the script. I have named used the same name as the name of my Azure Artifacts feed name (something like in example: AzureArtifactsNuGet) to be clear where do I push the package to (taken from __1__ in the picture below).
- `MyFeedUrl` should be the URL of the index.json from Azure Artifacts (taken from __2__ in the picture below).

[![](/_posts/2021-09-09-NuGetToAzureArtifacts/artifacts-feed-settings.png)](/_posts/2021-09-09-NuGetToAzureArtifacts/artifacts-feed-settings.png)

// Note: to get to the page displayed in the picture go to your Azure Artifacts and hit `Connect to feed` button.

### Packages migration script

The content of the script named `migrate.ps1` should be as follows:

```PowerShell
param ($SourceName)

$packages = Get-ChildItem -Path .\Packages -Recurse -ErrorAction SilentlyContinue -Force `
            | %{$_.FullName} `
            | where {$_ -like "*.nupkg" -and $_ -notlike "*symbols*"}

Foreach ($package in $packages) {
    .\nuget.exe push -Source $SourceName -SkipDuplicate -ApiKey az $package
}
```

After saving the content, run the script with the following parameter:

```PowerShell
.\migrate.ps1 "MyFeedName"
```

For the first push, you'll be promped to log in to your Azure Active Directory account associated with Azure DevOps. After that, you should then see the script iterating through the all packages and pushing them one by one to your newly created Azure Artifacts feed.

# Summary
Microsoft is providing us with its [own migration tool](https://docs.microsoft.com/en-us/azure/devops/artifacts/tutorials/migrate-packages?view=azure-devops), but it is supporting only `MyGet` hosting. In order to achieve the same with self-hosted NuGet server, the solution presented in this article should be suitable for you. Actually it should work for every NuGet migration, no matter where from, as long as you have your \*.nupkg files exported into Packages directory. I hope it will be useful! :)

# Author
Dawid Chróścielski

Node.js & .NET Programmer | Microsoft Azure Developer | Adaptivity Integration Engineer 

[www.chroscielski.pl](https://www.chroscielski.pl)
