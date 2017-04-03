---
layout: post
title:  "Integrate SpecFlow Report using NUnit 3 in TeamCity"
date:   2017-03-11 18:45:00 +0100
categories: bdd tools continuous integration delivery 
tags: specflow nunit teamcity report test bdd
---
As we started using [SpecFlow][SpecFlow] for integration and system tests in one project, we wanted to see SpecFlow reports right inside our build pipeline. We are running our builds with [TeamCity 10][TeamCity] and testing our code with the unit test driver [NUnit in version 3][NUnit]. The SpecFlow Report currently has no direct support for the resulting test output of Nunit 3. Luckily I can show you how to get this running benefitting from NUnit's command line configuration options. 
<!--more-->

The full experience is covered by first running the unit tests within TeamCity with the NUnit3 Runner. After that, you should add an additional step to process the test output file to create the report and adding the report as artefact. One extra benefit can be reached by defining an additional build tab, that shows the report. I assume your build is running (e.g. you have defined a Build Step for NuGet and Build/Visual Studio).

## NUnit3 Build Step
Create a new build step inside your TeamCity build configuration. Choose `NUnit` as Runner type and `NUnit3` as NUnit runner. Now specify the Path to NUnit console tool. Since we are using NuGet to load this dependency, we can add the following line: `packages\NUnit.ConsoleRunner.3.4.1\tools\nunit3-console.exe`. You should transfer this line to your specific version of NUnit. In the Run tests from field, you can write down all dlls, that should be run, especially the dlls containing your SpecFlow tests. That is why we are here. This should get you running your tests. 

But the most important part is missing. Please insert this into Additional command line parameters: `--labels=On --result=%teamcity.build.checkoutDir%/SpecFlowTestResult.xml;format=nunit2 --out=%teamcity.build.checkoutDir%/SpecFlowTestResult.txt`
Here you tell NUnit to output the result in NUnit2 format, which is needed by SpecFlow.

## SpecFlow Report Build Step
Now create another build step. Choose `Command Line` as Runner type. Choose `Executable with parameters` for Run. Again, as we are using NuGet for our dependency management, we can run SpecFlow from within the NuGet `packages` folder. So, we type `packages\SpecFlow.2.1.0\tools\specflow.exe` into the field Command executable and the following into the Command parameters field: `nunitexecutionreport
your.project.specs\your.project.specs.csproj
/xmlTestResult:%teamcity.build.checkoutDir%/SpecFlowTestResult.xml
/testOutput:%teamcity.build.checkoutDir%/SpecFlowTestResult.txt
/out:%teamcity.build.checkoutDir%/SpecFlowTests.html` Refer to the files you produced in the NUnit3 Build Step accordingly and define the path to your SpecFlow project inside your solution folder hierarchy. This will produce the SpecFlow report into the `SpecFlowTests.html` file.

## Build Artifact
In order to save the produced output files, change to the General Settings tab in your build configuration and add the line  `SpecFlowTests.html` (and if you like the other two files from NUnit Build Step) into the field Artifact paths. After running the build configuration, you can open the `SpecFlowTests.html` from the artifacts output of the build run.

## SpecFlow Report tab
Navigate to your project settings and click on Report Tabs. Here click Create new build report tab. As Tab Title choose `SpecFlowReport` and for start page type `SpecFlowTests.html`. You could also add a Project Report tabs, where you additionaly have to choose the build configuration from which to take the specified file

## The result
Now, the work is done, so let's have a look at the result. Here is one screenshot from our pipeline, that demonstrates the added functionality.

![team city build pipeline with specflow report tab]({{ site.url }}/assets/teamcity-specflow-report.png)

*This post was also published on [Softwareentwicklung Köln Blog][se-koeln-blog] (german).*

[SpecFlow]: http://specflow.org/
[NUnit]: https://github.com/nunit/nunit
[TeamCity]: https://www.jetbrains.com/teamcity/
[se-koeln-blog]: http://www.softwareentwicklung-koeln.de/integration-von-specflow-reports-mit-nunit-3-in-teamcity/