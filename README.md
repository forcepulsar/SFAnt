# SFAntUtility
Set of Ant commands to leverage SF Metadata api
Salesforce ANT scripts: metadata retrieval, deployment, post UAT disablement v48

#Pre requisites and Set Up

Using ANT, Metadata API and Sublime we can create a script to disable metadata in SF relatively easy.

To get familiar with ANT and Metadata API follow this guide:

https://developer.salesforce.com/page/Force.com_Migration_Tool

Prerequisites:

https://developer.salesforce.com/docs/atlas.en-us.daas.meta/daas/forcemigrationtool_prereq.htm

- Java
- Ant

Download zip file: AntScripts v48.zip from this repo.  



Folder structure:

AntScripts
    ant
    build.xml
    build.properties
newSandbox
    retrieveSource
      package.xml
      package-basic-metadata.xml
      package-code-only.xml
      package-full-copy.xml
      package-deactivate.xml
    lib
      ant-salesforce.jar
      ant-contrib-1.0b3.jar
    Readme.txt

Set your credentials and org url in the **build.properties file**.  You can set up the credentials for PROD, UAT and any sandbox.  For dev sandbox ensure the sandbox name is written at the end of the credentials:

eg: sf.username.<sandbox name>


# New Sandbox Metadata Deployment

When a new sandbox is created, this script is run to set certain metadata ready for sandbox usage (ie: removing prod details from fields, disabling certain wf, etc)

In terminal

cd ../AntScripts/ant
ant deployNewSandboxMetadata -Dorg=uat

*Replace the -Dorg=<ORGNAME> with the sandbox name

This will deploy all the items in the folder newSandbox to the target org


# Data masking metadata process (Used in UAT refresh and data migrations)

Generally used when doing data masking after a full sandbox refresh. The steps would be as follow:
1. Set Sandbox Ready for Masking
   Get metadata
   Disable Metadata locally
   Deploy metadata to be disabled
2. Restore original metadata

## Set Sandbox Ready for Masking

This command combines the Get Metadata, Disable Metadata Locally and Deploy Metadata to be disabled steps into one command. If using this command, no need to perform the other three commands

In terminal

cd ../AntScripts/ant
ant setSandboxReadyForMasking -Dorg=uat
- Replace the -Dorg=<ORGNAME> with the sandbox name

## Get metadata

The build.xml has a retrieve function that uses the package-deactivate.xml; this file contains the metadata to be disabled

In terminal

cd ../AntScripts/ant
ant getMetadataForDeactivation -Dorg=uat

*Replace the -Dorg=<ORGNAME> with the sandbox name

This commands does the following:

- Retrieves metadata from SF
- Puts metadata in folder originalMetadata-metadata-deactivate
- Creates a copy of metadata in folder backupMetadata-metadata-deactivate
- Creates a zip of metadata with name backupMetadata-metadata-deactivate-<TIMESTAMP>.zip

## Disable Metadata locally

In terminal
cd ../AntScripts/ant
ant deActivateLocal

This commands does the following to the metadata stored in originalMetadata-metadata-deactivate:

- Marks all Validation Rules as Inactive
- Marks all Workflows as Inactive
- Set all the flows version to 0. (making them deactive) 
- Sets the duplication rules to inactive

## Deploy metadata to be disabled

In terminal
cd ../AntScripts/ant
ant deActivateSandboxMetadata  -Dorg=uat

After this command finishes, all the WF,VR,Flows and duplication rules are disabled in the target org


## Restore original metadata (done after Data Cleansing tasks are finished on UAT refresh)

In terminal
cd ../AntScripts/ant
ant restoreActivatedMetadata -Dorg=uat

After this command finishes, all original WF rules are deployed back in the target org from the backupMetadata-metadata-deactivate folder


# Get Metadata From Orgs


## Retrieve full copy of metadata (excluding: emails, dashboards, reports)

The package-full-copy.xml is used. This contains all the metadata types that will be extracted.

In terminal:

cd ../AntScripts/ant
ant retrieveFullCopy -Dorg=prod

*Retrieves metadata from SF, puts it in folder originalMetadata-prod-<Timestamp>

For sandboxes replace the -Dorg=<ORGNAME> with the right sandbox


## Retrieve only code (classes,pages, triggers,components, workflows)

The package-code-only.xml is used.

In terminal:

cd ../AntScripts/ant
ant retrieveCode -Dorg=prod

* For sandboxes replace the -Dorg=<ORGNAME> with the right sandbox


## Retrieve emails

In terminal:
ant bulkRetrieveEmails -Dorg=prod

This retrieves all the emails and adds them to originalmetadata-<ORGNAME>-emails

This function first gets a list of all the folders that contain emails in the SF org. Then dynamically makes a bulk retrieve of all the emails in each folder.


# Projects/BAU deployments

The following commands have been created to facilitate deployment for Projects/BAU

## Retrieve

ant retrieve -Dorg=<sandbox name>

retrieve will:

- Get metadata data from retrieve/package.xml 
- Put metadata in folder original-metadata-<sandbox name>
- Put metadata in folder original-metadata-deploy

## Validate

ant validate -Dorg=<sandbox name>

Validate will be done for all the metadata specified in folder original-metadata-deploy


## validate running specific tests

ant validateBAU -Dorg=<sandbox name>

Validate will be done for all the metadata specified in folder original-metadata-deploy.  To set which tests to run, edit the build.xml file and add the test classes in the validateBAU command:

<target name="validateBAU" depends="init"><property name="org" value="DEV" /><setCredentials org="${org}" /><echo message="Target Org: ${org}"/><echo message="User name: ${orguname}"/>
<sf:deploy username="${orguname}" checkOnly="true" password="${orgpwd}" sessionId="" serverurl="${orgserver}" maxPoll="${sf.maxPoll}" deployRoot="${originalMetadata}-deploy" rollbackOnError="true" testLevel="RunSpecifiedTests">
<runTest>TESTTORUN1</runTest>
<runTest>TESTTORUN2</runTest>
</sf:deploy></target>

## Quick Deploy

ant quickDeploy -Dorg=<sandbox name> -DvalidationId=<VALIDATION_ID>

Deploy a previously validated deployment in target org


## Deploy

ant deploy -Dorg=<sandbox name>

Deploy will be done for all the metadata specified in folder original-metadata-deploy


# To Do

There are plenty of things that can be done to expand the functionality, here are some ideas:
- Encrypt the password and security tokens
- Make a script that runs all the data masking process in one command.
- deploy the backup in an easier way without having to rename the created folder
- Create a script that gets dynamically the latest changes for new sandbox metadata, rather than hardcoding them.

