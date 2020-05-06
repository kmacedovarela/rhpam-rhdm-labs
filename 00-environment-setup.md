# Environment Setup



## Pre-reqs

In order to try out the labs and have an overview of [Red Hat Decision Manager](https://www.redhat.com/pt-br/technologies/jboss-middleware/decision-manager) (a.k.a. [Drools](drools.org) ), [Red Hat Process Automation Manager](https://www.redhat.com/pt-br/technologies/jboss-middleware/process-automation-manager) and its components and capabilities, there's a couple of prerequisites you should check and set up.

You can choose to run RHDM on your local environment or run it on OpenShift (_To run on OpenShip you need subscriptions to access Red Hat images registry_). To run RHDM or RHPAM on your local environment, make sure you have:

* An up-to-date browser, such as Google Chrome or Firefox
* JDK 8 or 11 (For example, [OpenJDK 8](https://openjdk.java.net/install/)  ) (_Make sure you set up JAVA_HOME environment variable_)

To follow up the labs, you'll need:

* Apache Maven ( You can find it at the official [Apache Maven Site](https://maven.apache.org/download.cgi)) 
*  Git

## Confirming you're ready to go

Run locally these commands to confirm your environment is properly set. Open a terminal or CLI, and 

1. Test your Java installation by running:

````shell
$ java -version	
````

You should see an output with your Java version.

2. Test your maven installation by running: 

````shell
$ mvn -version
````

You should see an output with your maven home and version.

3. Test your git installation by running:

````shell
$  git --version
````

You should see an output with your git version.

After confirming you have the three tools properly set, you can choose to configure RHDM or RHPAM on your environment. 

## Set Up Red Hat Decision Manager Environment

Follow the steps described on [this](https://github.com/jbossdemocentral/rhdm7-install-demo) repository to install Red Hat Decision Manager. 

In the guide, you'll get the link to download the most recent version of Red Hat Decision Manager (Decision Central and Kie Server deployables) and also to download Red Hat JBoss EAP 7.2. 

You can also find three installation options:

1. Option 1 - Install on your machine
2. Option 2 - Run on OpenShift
3. Option 3 - Run in Docker 

Now, access this non-official [RHDM installation guide](https://github.com/jbossdemocentral/rhdm7-install-demo/blob/master/README.md), choose the most suitable for you and install RHDM.

The credentials to your environment should be: 

| Decision Central URL                   | User      | Password     |
| -------------------------------------- | --------- | ------------ |
| http://localhost:8080/decision-central | `dmAdmin` | `redhatdm1!` |

## Set Up Red Hat Process Automation Manager Environment

Follow the steps described on [this](https://github.com/jbossdemocentral/rhpam7-install-demo) repository to install Red Hat Process Automation Manager. 

In the guide, you'll get the link to download the most recent version of Red Hat Process Automation Manager (Business Central, Kie Server, and Add-ons package ) and also to download Red Hat JBoss EAP 7.2. 

You can also find three installation options:

1. Option 1 - Install on your machine
2. Option 2 - Run on OpenShift
3. Option 3 - Run in Docker 

Now, access this non-official [RHPAM installation guide](https://github.com/jbossdemocentral/rhpam7-install-demo/blob/master/README.md), choose the most suitable for you and install RHPAM.

The credentials to your environment should be: 

| Business Central URL                   | User         | Password     |
| -------------------------------------- | ------------ | ------------ |
| http://localhost:8080/business-central | `rhpamAdmin` | `redhatdm1!` |