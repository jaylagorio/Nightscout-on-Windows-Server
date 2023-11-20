# Table of Contents

- [Introduction](#introduction)
   - [Goals and Thanks](#goals-and-thanks)
- [Tools and Services](#tools-and-services)
- [Installation](#installation)
   - [Installing Windows Server](#installing-windows-server)
   - [The MongoDB Database Server](#the-mongodb-database-server)
      - [Database Configuration](#database-configuration)
      - [The Connection String](#the-connection-string)
   - [Install other 3rd Party software](#install-other-third-party-software)
   - [Windows Updates](#windows-updates)
   - [Windows Firewall Rules](#windows-firewall-rules)
- [Download and Run Nightscout](#download-and-run-nightscout)
   - [Download Nightscout](#download-nightscout)
   - [Start Nightscout on Boot](#start-nightscout-on-boot)
- [Reverse Proxy with IIS](#reverse-proxy-with-iis)
   - [Create the Proxy Site](#create-the-proxy-site)
   - [Set up the URL Rewrite Reverse Proxy](#set-up-the-url-rewrite-reverse-proxy)
   - [URL Rewrite and Compression](#url-rewrite-and-compression)
- [Conclusion](#conclusion)

<a name="introduction"></a>

# Introduction

[Nightscout](http://www.nightscout.info/) is an effort by the open source community to make it easier to visualize diabetes treatment data in a way that lets any device or system contribute data while letting any other device or system retrieve data. Each instance of Nightscout is customized to the patient and serves as a central point of data aggregation in the cloud. Many people who work on Nightscout are part of the [#WeAreNotWaiting](https://openaps.org/) community. Allowing users to control their data gives them more flexibility to treat themselves the way they see fit and gives parents of children a better window into their child's health. To date, insulin pumps, continuous glucose monitors, and other medical devices lock data into proprietary formats, protocols, and web sites, limiting the user's ability to view and manipulate their data in more meaningful ways. Nightscout aims to combat these limitations.

<a name="goals-and-thanks"></a>

## Goals and Thanks

Most users don't have the infrastructure to run and maintain Nightscout at home in a way that's safe and accessible from the Internet. For those users [Azure](http://www.azure.com/) and [Hiroku](https://www.heroku.com/) are excellent resources. The goal of this walk through is to install a local instance of Nightscout on [Windows Server 2016, 2019, or 2202](https://www.microsoft.com/en-us/cloud-platform/windows-server) and to run it when the server boots, even if a user doesn't log on. Further, because I already run a front-facing web server, I'll also detail how to setup a reverse proxy using [IIS](http://www.iis.net/) using my primary web server. The reverse proxy isn't required but documentation may help someone through a snag or two that I ran into.

Nightscout is still in development so there are some tricks needed because of dependencies on different open source libraries. I'd like to thank [@bewest](https://github.com/bewest), [@MilosKozak](https://github.com/MilosKozak), [@PieterGit](https://github.com/PieterGit), and the other users at [https://gitter.im/nightscout/public](https://gitter.im/nightscout/public) for the pointers they provided.

<a name="tools-and-services"></a>

# Tools and Services
The following software was used to set up the local Nightscout server. All of the tools are free except for the Windows Server operating system.

1. [Windows Server 2016, 2019, or 2022](https://www.microsoft.com/en-us/cloud-platform/windows-server)
1. [Visual Studio Express 2015 for Windows Desktop](https://my.visualstudio.com/Downloads?q=Visual%20Studio%20Express%202015%20for%20Windows%20Desktop)
1. [MongoDB Community Server 4.4.25](https://fastdl.mongodb.org/windows/mongodb-windows-x86_64-4.4.25-signed.msi)
1. [Python 2.7.11](https://www.python.org/downloads/release/python-2711/)
1. [Git for Windows 2.42.0](https://github.com/git-for-windows/git/releases/download/v2.42.0.windows.2/Git-2.42.0.2-64-bit.exe)
1. [Node.js 16.20.2 x64](https://nodejs.org/download/release/v16.20.2/node-v16.20.2-x64.msi)

<a name="installation"></a>

# Installation
To get started with the project you first must install Windows Server, then the database server. Other third party applications will be installed later.

<a name="installing-windows-server"></a>

## Installing Windows Server

1. Install Windows Server 2016:

   ![](screenshots/WindowsInstallScreen.png)

   ![](screenshots/ServerWithGui.png)

   ![](screenshots/WindowsInstalling.png)

1. Change the workstation name to Nightscout and, optionally, join it to a domain if you have one:

   ![](screenshots/DomainChange.png)

1. Open Windows Update, click Advanced Options and check the "Give me updates for other Microsoft products when I update Windows" checkbox. Click the back button and then click Check for Updates. While you do other tasks Windows Updates will download in the background and ultimately take less time to download and install later.

   ![](screenshots/WindowsUpdateInitialSettings1.png)

   ![](screenshots/WindowsUpdateInitialSettings2.png)

1. Reboot the server

1. Install Visual Studio Express 2015 for Windows Desktop using the default options.

<a name="the-mongodb-database-server"></a>

## The MongoDB Database Server
Locally hosting the MongoDB database is much more automated in the latest version of Mongo than in previous versions. Start the installation process and leaving all default options checked except for the following modifications.

1. Choose Complete installation as the installation type.

   ![](screenshots/MongoDBCompleteInstall.png)

1. When the Service Configuration screen appears be sure that Install MongoD as a Service is checked and that Run service as Network Service user is selected.

   ![](screenshots/MongoDBServiceConfiguration.png)

1. When prompted to install Mongo Compass, the database GUI tool, uncheck the box.

   ![](screenshots/MongoDBCompass.png)

Once Mongo is installed the service is started automatically.

<a name="database-configuration"></a>

### Database Configuration
Configuration involves creating the Nightscout database and assigning it credentials. You'll need this information and some information above to put together the connection string so Nightscout knows where to find the database.

1. Open a new Administrator command prompt
1. Open the Mongo client by running `C:\Program Files\MongoDB\Server\4.4\bin\mongo.exe`
1. At the resulting `>` prompt create the `Nightscout` database:

      ```
      use Nightscout
      ```

1. Create the username and password to access the database and allow the account read/write access to the database (for simplicity, use `username` and `password` as the account credentials):

      ```
      db.createUser({user: "username", pwd: "password", roles:["readWrite"]})
      ```

When you see the message about the user being successfully added you will have completed installation and configuration of the MongoDB service.

<a name="the-connection-string"></a>

### The Conection String
Now that the service is installed and configured you will need to note the connection string. This string will be used to configure Nightscout so it knows where the database is and how to connect to it. The connection string is formatted with the username, password, host name or address, the MongoDB port, and the database name:

```
mongodb://<username>:<password>@<hostname>:<port>/<database>
```

The connection string for Nightscout running on the same host with all of the settings above would be: 

```
mongodb://username:password@localhost:27017/Nightscout
```

<a name="install-other-third-party-software"></a>

## Install other 3rd Party software
1. Install Git:

   ![](screenshots/GitInstallOptions.png)

   ![](screenshots/GitEditorOptions.png)

   ![](screenshots/GitBranchNames.png)

   ![](screenshots/GitSshOptions.png)

   ![](screenshots/GitPullOptions.png)

   ![](screenshots/GitUnixOptions.png)

   ![](screenshots/GitCheckoutOptions.png)

   ![](screenshots/GitTerminalEmulator.png)

   ![](screenshots/GitCredentialHelper.png)

   ![](screenshots/GitCacheOptions.png)

   ![](screenshots/GitExperimentalOptions.png)

1. Install Python, making sure to add Python to the `PATH` environment variable:

   ![](screenshots/PythonInstallOptions.png)

1. Install Node.js

   ![](screenshots/NodeInstallOptions.png)

1. Reboot the server

<a name="windows-updates"></a>

## Windows Updates
Windows Update has improved substantially in this release of Windows Server. Expect to download a couple of cumulative updates and reboot once to complete installation. Just in case, make sure you always check for more update after each reboot.

   ![](screenshots/WindowsUpdateRestartPC.png)

When you're done installing You'll see this screen when all Windows Updates are installed:

   ![](screenshots/WindowsUpdateFinished.png)

Windows updates are now delivered and installed automatically. This lets you continue to be a good steward of Internet-accessible systems (read: probably not contributing to botnets) while keeping maintenance tasks low. When Windows Updates install the system will reboot and the Nightscout process will start automatically once you configure that later.

<a name="windows-firewall-rules"></a>

## Windows Firewall Rules
By default Windows Firewall will allow you to connect to the Nightscout server locally but will deny connections to external hosts. To allow standard HTTP traffic to the system you need to create a Windows Firewall rule.

1. Open Windows Firewall with Advanced Security and click Inbound Rules, then New Rule

   ![](screenshots/WindowsFirewall.png)

1. Select "Port" and click Next

   ![](screenshots/WindowsFirewallRuleType.png)

1. Select "Specific local ports" and enter 80, the standard port for HTTP traffic, and click Next

   ![](screenshots/WindowsFirewallProtocolAndPorts.png)

1. Leave the "Allow the connection" option selected and click Next

   ![](screenshots/WindowsFirewallAction.png)

1. Leave all three Domain checkboxes checked and click Next

   ![](screenshots/WindowsFirewallProfile.png)

1. Name the rule `Allow HTTP Inbound` and click Finish

   ![](screenshots/WindowsFirewallName.png)

HTTP traffic will now be allowed through the firewall and passed to Nightscout. This process doesn't need to be done for MongoDB because Nightscout is going to connect to Mongo on a local port from the same server. If you need to access Mongo from an external system you can repeat the above process for port 27017.

<a name="download-and-run-nightscout"></a>

# Download and Run Nightscout
Now that the operating system and database are installed you're ready to clone and install Nightscout.

<a name="download-nightscout"></a>

## Download Nightscout
Run the NodeJS command prompt as Administrator. Create a folder on your server for Nightscout to download into. I chose `C:\Nightscout\`. Clone the Nightscout master repository into this directory, making sure to observe the trailing period on the `git` command:

```
cd \
mkdir Nightscout
cd Nightscout
git clone https://github.com/nightscout/cgm-remote-monitor.git .
```

Git will have downloaded Nightscout files into the Nightscout directory.

<a name="installation-and-configuration"></a>

## Installation and Configuration
The next step is to install your environment variables at the system level, not the user level. This is necessary because you want any user, especially the `SYSTEM` user, to access these variables when the system boots but a user isn't logged in.

1. Right-click "This PC" on the Desktop and select Properties
1. Click "Advanced system settings" on the left panel of the System Properties window
1. Click the Environment Variables button

   ![](screenshots/EnvironmentVariables.png)

1. Add the following system variables by entering the variable names and values in the marked text box by clicking the New button for each variable:

   1. `PUMP_FIELDS` controls which fields you want in the Pump pillbox and includes things like the reservoir and battery levels
   1. `ENABLE` explicitly calls out which features to enable for the user
   1. `API_SECRET` is required to be set to allow an uploader device use the REST API to add data to Nightscout (directly writing to MongoDB isn't supported)
   1. `MONGO` is the database connection string and is required for Nightscout to retrieve your data
   1. `PORT` is the port the system will listen on, commonly 80 for HTTP if you're going to put a reverse proxy in front of Nightscout, or 443 if not
   1. `INSECURE_USE_HTTP` allows Node to use HTTP and not require HTTPS, which is only needed if you plan to put a reverse proxy in front of Nightscout (see the sections on IIS for details)

```
PUMP_FIELDS = reservoir battery status
ENABLE = careportal iob cob openaps pump bwg rawbg basal
API_SECRET = YOUR_API_SECRET_HERE
MONGO = mongodb://username:password@localhost:27017/Nightscout
PORT = 443
INSECURE_USE_HTTP = true
```

Open the NodeJS command line prompt as an Administrator and navigate to Nightscout folder, then run the Node Package Manager (NPM) installation process:

```
npm install --msvs_version=2015
```

Once the installation process is complete start the server:

```
node server.js
```

The first time you start Nightscout it will take a little longer to be ready because the database needs to be initialized. When you see the `emitted clear_alarm to all clients` line try to connect to it using Internet Explorer by going to [http://localhost/](http://localhost/). If you see the Nightscout site and you're redirected to the Profile Editor your setup is complete. Terminate the Nightscout server using Ctrl+C to begin setting it up to run automatically on system boot.

<a name="start-nightscout-on-boot"></a>

## Start Nightscout on Boot
The Task Scheduler must be used to start Nightscout when the system boots.

1. Open the Task Scheduler and click Create Basic Task:

   ![](screenshots/TaskScheduler.png)

1. Name the new task `Nightscout Server` and click Next

   ![](screenshots/TaskSchedulerNightscoutCreateTask.png)

1. Select "When the computer starts" as the trigger and click Next

   ![](screenshots/TaskSchedulerTrigger.png)

1. Choose "Start a program" as the action to perform and click Next

   ![](screenshots/TaskSchedulerAction.png)

1. Configure the task to start Node and to start in the Nightscout folder, then click Next:
   * Program/script: `C:\Program Files\nodejs\node.exe`
   * Arguments: `server.js`
   * Start in: `C:\Nightscout\`

   ![](screenshots/TaskSchedulerNightscoutStartProgram.png)

1. Check the "Open the Properties dialog for this task when I click Finish" checkbox and click Finish

   ![](screenshots/TaskSchedulerNightscoutFinish.png)

   If you get a warning about the program name and arguments click the No button

   ![](screenshots/TaskSchedulerNightscoutWarning.png)

1. On the General tab select "Run whether user is logged on or not"

   ![](screenshots/TaskSchedulerNightscoutGeneral.png)

1. On the Trigger tab double-click the trigger in the list and add a delay by checking the "Delay task for" checkbox and setting it to 2 minutes, then click OK

   ![](screenshots/TaskSchedulerNightscoutTriggerEdit.png)

1. On the Conditions tab under Power, uncheck "Start the task only if the computer is on AC power"

   ![](screenshots/TaskSchedulerNightscoutConditions.png)

1. On the Settings tab, check all of the following:

   - Run task as soon as possible after a scheduled start is missed
   - If the task fails, restart every (default is 1 minute)
   
   *Important*: Uncheck the "Stop the task if it runs longer than" or your Nightscout service will stop three days after every reboot.

   ![](screenshots/TaskSchedulerMongoSettings.png)

If prompted to enter the Administrator password do so and then click OK. Close the Task Scheduler and reboot the server. Wait at least 2 minutes from the time you can press Ctrl+Alt+Del to log in and then attempt to connect to the service.

<a name="reverse-proxy-with-iis"></a>

# Reverse Proxy with IIS
Ordinarily forwarding port 80 from your home router to the Nightscout server is enough to enable traffic from the Internet to reach the service. In my case I already run an IIS web server accessible via the Internet so I needed to set up a reverse proxy. By creating a new Web Site on the existing IIS server and setting up URL Rewrite to proxy traffic to the Nightscout server for a certain domain name you can run more than one web server accessible via the same external port. All of these steps are performed on the existing, primary web server and are not done on the Nightscout system. You will need to download and install [Microsoft Application Request Routing](https://www.iis.net/downloads/microsoft/application-request-routing) (ARR).

<a name="create-the-proxy-site"></a>

## Create the Proxy Site
Setting up the site that will proxy traffic is quick:

1. Open IIS, right click the Sites item, and click "Add Website..."

   ![](screenshots/IISProxyAddWebsite.png)

1. Name the site `NightscoutProxy` and choose a physical path for the site. This path will remain empty. Though not shown below, the Host Name must be the fully qualified domain name you expect to reach via the Internet (such as `nightscout.domainname.com`):

   ![](screenshots/IISProxyAddWebsiteConfig.png)

Click OK and observe that the Web Site starts successfully.

<a name="set-up-the-url-rewrite-reverse-proxy"></a>

## Set up the URL Rewrite Reverse Proxy
The URL Rewrite module will act as a reverse proxy by taking incoming traffic with the outside domain name (e.g., nightscout.domainname.com), rewriting it with the internal name for the destination web server (e.g., nightscout.internal.local), and then forwarding it on to the internal Nightscout web server. When the Nightscout server returns data URL Rewrite will process the traffic in reverse by receiving the response from Nightscout, rewriting it to match the external domain name, and sending it to the Internet system that made the original request.

Because Application Request Routing doesn't support compression in the destination host, in this case Nightscout, we will need to impose an additional rule that tells Nightscout not to compress its output. Traffic between the web browser and IIS, however, will be compressed.

1. In the "NightscoutProxy" site click the URL Rewrite icon and then click View Server Variables. Click Add, and then enter `HTTP_ACCEPT_ENCODING`. This will allow us to disable compression between Nightscout and IIS later. Click OK and then click Back to Rules.

1. Click Add Rule

   ![](screenshots/IISProxyURLRewriteIcon.png)

1. Select Reverse Proxy and then click OK

   ![](screenshots/IISProxyURLRewriteAddRule.png)

1. For Inbound Rules, enter the name or IP address of the internal Nightscout server and optionally check the "SSL Offloading" checkbox. Check the Outbound Rules checkbox, enter the name of the internal Nightscout server in the "From" text box (`nightscout`), then enter the name of the external domain name in the "To" box (`nightscout.domainname.com`).

   ![](screenshots/IISProxyURLRewriteRule.png)

At this point for most web servers you would attempt to access the Nightscout server from a smartphone over the cellular data connection and it would work. Doing that now will result in an error because URL Rewrite hates compression and will need that to be disabled on the external facing IIS server and in Nightscout.

<a name="url-rewrite-and-compression"></a>

## URL Rewrite and Compression
Data compression between IIS and Nightscout will need to be disabled because URL Rewrite doesn't support it. We will do that by rewriting the `HTTP_ACCEPT_ENCODING` header between IIS and Nightscout. The `Accept-Encoding:` request header delivered to Nightscout will be set to an unsupported compression name (`x`) which will cause Node to deliver its data without any compression at all.

1. In the "NightscoutProxy" site and double-click the URL Rewrite icon

1. Double-click the `ReverseProxyInboundRule1` in the Inbound Rules

1. In the Server Variables section of the screen, click the Add button. Enter `HTTP_ACCEPT_ENCODING` in the Server variable name field and `x` in the Value field. Click OK to add the rule and then click Apply.

   ![](screenshots/IISProxyURLAcceptEncoding.png)

   ![](screenshots/IISProxyURLEditInboundRule.png)

Once this modification is made you should be able to view your Nightscout instance through the IIS reverse proxy.

<a name="conclusion"></a>

# Conclusion
Nightscout is billed as [CGM in the Cloud](http://nightscout.github.io/) and it's really easy to set that scenario up on Azure or Hiroku - a great attribute if you only have access to cloud computing resources. For people who have access to local servers and prefer to assert more control over their data they can use Windows Server to keep their information in-house.