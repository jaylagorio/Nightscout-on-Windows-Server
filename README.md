# Table of Contents

- [Introduction](#introduction)
   - [Goals and Thanks](#goals-and-thanks)
- [Tools and Services](#tools-and-services)
- [Installation](#installation)
   - [Installing Windows Server](#installing-windows-server)
   - [Visual Studio Express 2013 with Update 4](#visual-studio-express-2013-with-update-4)
   - [The MongoDB Database Server](#the-mongodb-database-server)
      - [Run Mongo on System Startup](#run-mongo-on-system-startup)
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

Most users don't have the infrastructure to run and maintain Nightscout at home in a way that's safe and accessible from the Internet. For those users [Azure](http://www.azure.com/) and [Hiroku](https://www.heroku.com/) are excellent resources. The goal of this walk through is to install a local instance of Nightscout on [Windows Server 2012 R2](https://www.microsoft.com/en-us/server-cloud/products/windows-server-2012-r2/) and to run it when the server boots, even if a user doesn't log on. Further, because I already run a front-facing web server, I'll also detail how to setup a reverse proxy using [IIS](http://www.iis.net/) using my primary web server. The reverse proxy isn't required but documentation may help someone through a snag or two that I ran into.

Nightscout is still in development so there are some tricks needed because of dependencies on different open source libraries. I'd like to thank [@MilosKozak](https://gitter.im/MilosKozak) and the other users at [https://gitter.im/nightscout/public](https://gitter.im/nightscout/public) for the pointers they provided.

<a name="tools-and-services"></a>

# Tools and Services
The following software was used to set up the local Nightscout server. All of the tools are free except for the Windows Server operating system.

1. [Windows Server 2012 R2](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2012-r2)
1. [Visual Studio Express 2013 with Update 4](https://www.microsoft.com/en-US/download/details.aspx?id=44914)
1. [MongoDB 3.2.6](https://www.mongodb.com/download-center?jmp=nav#community) for Windows Server 2008 R2 and later, with SSL support
1. [Python 2.7.11](https://www.python.org/downloads/release/python-2711/)
1. [Git for Windows 2.7.2](https://www.git-scm.com/downloads)
1. [Node.js 0.12.10 x64](https://nodejs.org/download/release/v0.12.10/x64/node-v0.12.10-x64.msi)

<a name="installation"></a>

# Installation
To get started with the project you first must install Windows Server, then Visual Studio and the database server. Other third party applications will be installed later.

<a name="installing-windows-server"></a>

## Installing Windows Server

1. Install Windows Server 2012 R2:

   ![](screenshots/WindowsInstallScreen.png)

   ![](screenshots/ServerWithGui.png)

   ![](screenshots/WindowsInstalling.png)

1. Install .NET Framework 3.5 using the Server Manager utility:

   ![](screenshots/DotNetFramework35Checkbox.png)

   ![](screenshots/SourcePathNotify.png)

   ![](screenshots/SourcePathEntered.png)

   ![](screenshots/DotNetFramework35InstallSucceed.png)

1. Change the workstation name to Nightscout and, optionally, join it to a domain if you have one:

   ![](screenshots/DomainChange.png)

1. Change the settings on Windows Update to "Download but let me choose when to install," and check both the "Give me recommended updates the same way I receive important updates" and "Give me updates for other Microsoft products when I update Windows" checkboxes. Don't start installing Windows Updates yet. While you do other tasks Windows Updates will download in the background and ultimately take less time to download and install later.

   ![](screenshots/WindowsUpdateInitialSettings.png)

1. Reboot the server

<a name="visual-studio-express-2013-with-update-4"></a>

## Visual Studio Express 2013 with Update 4
1. Install Visual Studio:

   ![](screenshots/VSInstallScreen.png)

1. Reboot the server

<a name="the-mongodb-database-server"></a>

## The MongoDB Database Server
Locally hosting the MongoDB database requires little effort after the initial setup, but the initial setup is very manual. Running the installation file is the only automated step. When prompted for the installation type choose the Complete installation.

   ![](screenshots/MongoDBCompleteInstall.png)

Now that the files have been installed the service must be activated.

1. Open an Administrator command prompt
1. Create the MongoDB database folder in the root of the `C:\` drive: 
      ```
      cd C:\
      mkdir data
      mkdir data\db
      ```
1. Change to the MongoDB `bin` directory and start the service:
      ```
      cd C:\Program Files\MongoDB\Server\3.2\bin\
      mongod
      ```

When you see the `[initandlisten] waiting for connections on port 27017` message the service has started sucessfully. You'll need this port number later to build the connection string. Terminate the service using Ctrl+C and close the command prompt.

<a name="run-mongo-on-system-startup"></a>

### Run Mongo on System Startup
To configure MongoDB to run when the system starts up you'll use the Windows Task Scheduler. Most servers and daemons will install a Windows Service and do this automatically but Mongo doesn't.

1. Open the Task Scheduler and click Create Basic Task

   ![](screenshots/TaskScheduler.png)

1. Name the new task `MongoDB` and click Next

   ![](screenshots/TaskSchedulerMongoCreateTask.png)

1. Select "When the computer starts" as the trigger and click Next

   ![](screenshots/TaskSchedulerTrigger.png)

1. Choose "Start a program" as the action to perform

   ![](screenshots/TaskSchedulerAction.png)

1. Configure the task to start the MongoDB daemon and to start in the folder where the executable is located:
   * Program/script: `C:\Program Files\MongoDB\Server\3.2\bin\mongod.exe`
   * Start in: `C:\Program Files\MongoDB\Server\3.2\bin\`

   ![](screenshots/TaskSchedulerMongoStartProgram.png)

1. Check the "Open the Properties dialog for this task when I click Finish" checkbox and click Finish

   ![](screenshots/TaskSchedulerMongoFinish.png)

   If you get a warning about the program name and arguments click the No button

   ![](screenshots/TaskSchedulerMongoWarning.png)

1. On the General tab select "Run whether user is logged on or not"

   ![](screenshots/TaskSchedulerMongoGeneral.png)

1. On the Conditions tab under Power, uncheck "Start the task only if the computer is on AC power"

   ![](screenshots/TaskSchedulerMongoConditions.png)

1. On the Settings tab, check all of the following:

   - Run task as soon as possible after a scheduled start is missed
   - If the task fails, restart every (default is 1 minute)
      
   *Important*: Uncheck the "Stop the task if it runs longer than" or your Nightscout server will fail three days after every reboot

   ![](screenshots/TaskSchedulerMongoSettings.png)

Close the Properties window, entering the Administrator password if prompted. Reboot the system and login. Open the Task Scheduler again and you should see that the MongoDB task is listed as "Running." MongoDB is now configured to start with the system.

<a name="database-configuration"></a>

### Database Configuration
Configuration involves creating the Nightscout database and assigning it credentials. You'll need this information and some information above to put together the connection string so Nightscout knows where to find the database.

1. Open a new Administrator command prompt
1. Go back to the Mongo `bin` directory as you did when you ensured the `mongod` service started successfully
1. Open the Mongo client with the `mongo` command
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

   ![](screenshots/GitUnixOptions.png)

   ![](screenshots/GitCheckoutOptions.png)

   ![](screenshots/GitTTYOptions.png)

   ![](screenshots/GitCacheOptions.png)

1. Install Python, making sure to add Python to the `PATH` environment variable:

   ![](screenshots/PythonInstallOptions.png)

1. Install Node.js

   ![](screenshots/NodeInstallOptions.png)

1. Reboot the server

<a name="windows-updates"></a>

## Windows Updates
Windows Updates take the longest amount of time in the installation process because as of this writing there are over 250 of them for the operating system and other Microsoft software packages. I tend to install them 50 at a time in case a problem is encountered so I don't end up back at square one after a long roll-back process. Because I'm working in a virtual machine I also tend to take a snapshot before starting each round of updates.

After each round a reboot is required so get ready to see this screen often:

![](screenshots/WindowsUpdateRestartPC.png)

When you're done installing You'll see this screen when all Windows Updates are installed:

![](screenshots/WindowsUpdateFinished.png)

Once Windows Updates are done you should change the installation settings to "Install updates automatically (recommended)" This lets you continue to be a good steward of Internet-accessible systems (read: probably not contributing to botnets) while keeping maintenance tasks low. When Windows Updates install the system will reboot and the Nightscout process will start automatically when you configure that later.

![](screenshots/WindowsUpdateFinalSettings.png)

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
Now that the operating system, database, and Visual Studio build system are installed you're ready to clone and install Nightscout.

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
   1. `DEVICESTATUS_ADVANCED` shows uploader device information
   1. `ENABLE` explicitly calls out which features to enable for the user
   1. `API_SECRET` is required to be set to allow an uploader device use the REST API to add data to Nightscout (directly writing to MongoDB isn't supported)
   1. `MONGO` is the database connection string and is required for Nightscout to retrieve your data
   1. `PORT` is the port the system will listen on, commonly 80 for HTTP

```
PUMP_FIELDS = reservoir battery status
DEVICESTATUS_ADVANCED = true
ENABLE = careportal iob cob openaps pump bwg rawbg basal
API_SECRET = YOUR_API_SECRET_HERE
MONGO = mongodb://username:password@yourmongodbaddress.mlab.com:1234/mongodbname
PORT = 80
```

Open the NodeJS command line prompt as an Administrator and navigate to Nightscout folder, then run the Node Package Manager (NPM) with the option to indicate that Visual Studio 2013 should be used as the build system:

```
npm install --msvs_version=2013
```

Once the installation process is complete start the server:

```
node server.js
```

The first time you start Nightscout it will take a little longer to be ready because the database needs to be initialized. When you see the `emitted clear_alarm to all clients` line try to connect to it using Internet Explorer by going to [http://localhost/](http://localhost/). If you see the Nightscout site and you're redirected to the Profile Editor your setup is complete. Terminate the Nightscout server to begin setting it up to run automatically on system boot.

<a name="start-nightscout-on-boot"></a>

## Start Nightscout on Boot
Like MongoDB, the Task Scheduler must be used to start Nightscout when the system boots. The process to do this is largely the same as with MongoDB but a delay must be added so that it starts shortly after MongoDB. If MongoDB isn't started first Nightscout won't be able to connect to the service and it will fail even if Mongo starts successfully afterwards.

1. Open the Task Scheduler and click Create Basic Task (the MongoDB task is not pictured below)

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
   
   *Important*: Uncheck the "Stop the task if it runs longer than" or your Nightscout server will fail three days after every reboot

   ![](screenshots/TaskSchedulerMongoSettings.png)

If prompted to enter the Administrator password do so and then click OK. Close the Task Scheduler and reboot the server. Wait at least 2 minutes from the time you can press Ctrl+Alt+Del to log in and then attempt to connect to the service.

<a name="reverse-proxy-with-iis"></a>

# Reverse Proxy with IIS
Ordinarily forwarding port 80 from your home router to the Nightscout server is enough to enable traffic from the Internet to reach the service. In my case I already run an IIS web server accessible via the Internet so I needed to set up a Reverse Proxy. By creating a new Web Site on the existing IIS server and setting up URL Rewrite to proxy traffic to the Nightscout server for a certain domain name you can run more than one web server accessible via the same external port. Except for the step at the very end all of these steps are performed on the existing, primary web server and are not done on the Nightscout system.

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

1. In the "NightscoutProxy" site click the URL Rewrite icon and then click Add Rule

   ![](screenshots/IISProxyURLRewriteIcon.png)

1. Select Reverse Proxy and then click OK

   ![](screenshots/IISProxyURLRewriteAddRule.png)

1. For Inbound Rules, enter the name or IP address of the internal Nightscout server and optionally check the "SSL Offloading" checkbox. Check the Outbound Rules checkbox, enter the name of the internal Nightscout server in the "From" text box (`nightscout`), then enter the name of the external domain name in the "To" box (`nightscout.domainname.com`).

   ![](screenshots/IISProxyURLRewriteRule.png)

At this point for most web servers you would attempt to access the Nightscout server from a smartphone over the cellular data connection and it would work. Doing that now will result in an error because URL Rewrite hates compression and will need that to be disabled on the external facing IIS server and in Nightscout.

<a name="url-rewrite-and-compression"></a>

## URL Rewrite and Compression
Compression will need to be disabled in IIS for static and dynamic content.

1. Click the "NightscoutProxy" site and double-click the Compression icon

   ![](screenshots/IISProxySiteCompressionIcon.png)

1. Clear both checkboxes to disable static and dynamic compression, then click the Apply button

   ![](screenshots/IISProxySiteCompressionSettings.png)

Finally, compression needs to be disabled in Nightscout. Nightscout doesn't have an environment variable that can be set to disable compression so a two line modification to the [`app.js`](https://github.com/nightscout/cgm-remote-monitor/blob/5dc49692432229892613fb022c9aa176eb62f5ff/app.js#L16) file must be made. At that point in the history of the file, the relevant lines start on line 16:

```javascript
   app.use(compression({filter: function shouldCompress(req, res) { 
       //TODO: return false here if we find a condition where we don't want to compress 
       // fallback to standard filter function 
       return compression.filter(req, res); 
   }})); 

```

This allows the `compression` object to decide whether compression is used for the HTTP response to the request, which is something we never want it to do in this case. To implement this modification, change the return statement to `return false;` as seen below:

```javascript
  app.use(compression({filter: function shouldCompress(req, res) {
    //TODO: return false here if we find a condition where we don't want to compress
    // fallback to standard filter function
    return false;
  }}));
```

Restart your Nightscout server to cause Node to read the code change. Keep in mind that when Nightscout is updated you may need to remake this modification.

<a name="conclusion"></a>

# Conclusion
Nightscout is billed as [CGM in the Cloud](http://nightscout.github.io/) and it's really easy to set that scenario up on Azure or Hiroku - a great attribute if you only have access to cloud computing resources. For people who have access to local servers and prefer to assert more control over their data they can use Windows Server to keep their information in-house.