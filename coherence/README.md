# Deploying a Java Application with Coherence on Azure Using a WebLogic Virtual Machine Cluster
This demo shows how you can deploy a Java application with Coherence to Azure into a WebLogic cluster.

## Setup

* Install the latest version of [Oracle JDK 8](https://www.oracle.com/java/technologies/javase/javase-jdk8-downloads.html) (we used 8u301).
* Install [the 2020-03 release of Eclipse for Enterprise Java Developers](https://www.eclipse.org/downloads/packages/release/2020-03/r/eclipse-ide-enterprise-java-developers-includes-incubating-components) (this is the latest Eclipse IDE version that supports Java SE 8 and works reliably with WebLogic 12.2.1.3).
* Install WebLogic 12.2.1.3 (note - not the latest version) using the Quick Installer by downloading it from [here](https://www.oracle.com/middleware/technologies/weblogic-server-downloads.html). You need this locally even if you are not running WebLogic locally because the Eclipse WebLogic deployment support needs it.
* Download this repository somewhere in your file system (easiest way might be to download as a zip and extract).
* You will need an Azure subscription. If you don't have one, you can get one for free for one year [here](https://azure.microsoft.com/en-us/free).

## Start Managed PostgreSQL on Azure
We will be using the fully managed PostgreSQL offering in Azure for this demo. Please set it up now unless you have done so already. Below is how we set it up. 

* Go to the [Azure portal](http://portal.azure.com).
* Select 'Create a resource'. In the search box, enter and select 'Azure Database for PostgreSQL'. Hit create. Select a single server.
* The steps in this section use `<your suffix>`. The suffix could be your first name such as "reza".  It should be short and reasonably unique, and less than 10 charracters in length.
* Create and specify a new resource group named weblogic-cafe-db-group-`<your suffix>` . 
* Specify the Server name to be weblogic-cafe-db-`<your suffix>`.
* Specify the region to be a location close to you.
* Leave the Version at its default.
* In Compute + Storage click "Configure Server" then choose Basic.
   * Set vCore to the minimum.
   * Set Storage to the minimum.
   * Click 'OK'
* Specify the Admin username to be postgres. 
* Specify the Password to be Secret123!. 
* Hit 'Review+create' then 'Create'. It will take a moment for the database to deploy and be ready for use.
* In the portal, go to 'All resources'. Enter `<your suffix>` into the filter box and press enter.
* Find and click on weblogic-cafe-db-`<your suffix>`. 
* Under Settings, open the connection security panel.
   * Toggle "Allow access to Azure services" to "Yes"
   * Toggle "Enforce SSL connection" to "DISABLED". 
   * Hit Save.

## Create User-Assigned Managed Identity

* Go to the [Azure portal](http://portal.azure.com).
* Select 'Create a resource'. In the search box, enter and select 'User Assigned Managed Identity'. Click create.
* The steps in this section use `<your suffix>`. The suffix could be your first name such as "reza".  It should be short and reasonably unique, and less than 10 charracters in length.
* Create and specify a new resource group named weblogic-cafe-managed-identity-group-`<your suffix>`.
* Specify the region to be a location close to you.
* Specify the Instance name to be weblogic-cafe-managed-identity-`<your suffix>`. 
* Click Review + create -> Create.

## Create the WebLogic Cluster on Azure
The next step is to get a WebLogic cluster up and running. Follow the steps below to do so.

* Go to the [Azure portal](https://ms.portal.azure.com/).
* Use the search bar on the top to navigate to the Marketplace.
* In the Marketplace, type in 'Oracle WebLogic Server Cluster' in the search bar and click Enter.
* Locate the offer named 'Oracle WebLogic Server Cluster' and click 'Create'.
* Create and specify a new resource group named weblogic-cafe-group-`<your suffix>`.
* Select a region closest to you.
* For the WebLogic image, select a 12.2.1.3 version.
* For "Credentails for Virtual Machines and WebLogic"
   * For the "Password for admin account of VMs", enter 'Secret123456'. 
   * For the "Password for WebLogic Administrator", enter 'Secret123456'.
* For Number of VMs, change the value to 3.
* For "Optional Basic Configuration", ensure  `Yes` is selected to accept default for optional configuration.
* Click Next.
* In "TLS/SSL Configuration", leave "Configure WebLogic Administration Console on HTTPS (Secure) port, with your own TLS/SSL Certificate?" with `No` selected.
* In the "Azure Application Gateway" use these values
   * Toggle "Connect to Azure Application Gateway" to `Yes`.
   * Choose "Generate a self-signed certificate" for the "Select desired TLS/SSL certificate option".
   * To auto-generate a self-signed certificate, you will need to add user-assigned managed identities(at least one), click "Add".
   * In the window pops from right, choose weblogic-cafe-managed-identity-`<your suffix>`, then click "Add"
   * Ensure the managed identity is selected.
* Click Next.
* In "DNS Configuration", leave "Configure Custom DNS Alias?" with `No` selected.
* Click Next.
* In "Database" use these values
   * Toggle "Connect to DataBase" to `Yes`. 
   * For "Choose database type", from the dropdown menu, select the option for "Azure Database for PostgreSQL".
   * Specify JNDI Name to be 'jdbc/WebLogicCafeDB'. 
   * Specify DataSource Connection String to be 'jdbc:postgresql://weblogic-cafe-db-`<your suffix>`.postgres.database.azure.com:5432/postgres'
   * Specify the Database Username to be 'postgres@weblogic-cafe-db-`<your suffix>`'
   * Enter the Database Password as 'Secret123!' 
* Click Next.
* In "Azure Active Directory", leave "Connect to Active Directory" with `No` selected.
* Click Next.
* In "Elasticsearch and Kibana", leave "Export logs to Elasticsearch?" with `No` selected.
* Click Next.
* In "Coherence", toggle "Use Coherence cache?" to `Yes`. Leave other values to default.
* Click 'Next: Review + create'
   * On the Summary blade you must see "Validation passed".  If you don't see this, you must troubleshoot and resolve the reason.  After you have done so, you can continue.
   * On the final screen, click Create.
* It will take some time for the WebLogic cluster to properly deploy (could be up to an hour). Once the deployment completes, in the portal go to 'All resources'.
* Find and click on adminVM. Copy the DNS name for the admin server. You should be able to log onto http://`<admin server DNS name>`:7001/console successfully using the credentials above.  If you are not able to log in, you must troubleshoot and resolve the reason why before continuing.
* In the portal go to 'All resources'. Find and click on wls-nsg. In the Overview page, find `WebLogicManagedChannelPortsDenied` under the Inbound Security Rules, click on it. In the pop window, select `Allow` for Action and click `Save`, wait for the update to be completed. This will open portal 8501 which we will use later.

## Setting Up WebLogic in Eclipse
The next step is to get the application up and running.

* Start Eclipse.
* If you have not yet installed the Oracle WebLogic Server Tools, do so now.
   * Go to the 'Servers' panel, secondary click. Select New -> Server
   * Select Oracle -> Oracle WebLogic Server Tools. Click next. Accept the license agreement, click 'Finish'.  Eclipse may ask to be restarted. If so, comply with the request.
* Go to the 'Servers' panel, secondary click. 
* Select New -> Server -> Oracle -> Oracle WebLogic Server. Click Next.
* Choose remote, for the remote host enter `<admin server DNS name>`.
* Enter the WebLogic admin username/password from above.
* Click "Test connection".  If "Test connection succeeded!" appears, click OK and you may continue.  Otherwise, troubleshoot and resolve the reason for the connection failure.
* Click 'Finish'.

## Open weblogic-cafe in the IDE
* Get the stateful Coherence version of the weblogic-cafe application into the IDE. In order to do that, go to File -> Import -> Maven -> Existing Maven Projects. Click Next.
* Then browse to where you have this repository code in your file system and select coherence/weblogic-cafe and click "Open".
* Accept the rest of the defaults and click "finish".
* Once the application loads, you should do a full Maven build by going to the application and secondary clicking -> Run As -> Maven install.
   * You must see `BUILD SUCCESS` in the Eclipse console in order to proceed.  If you do not, troubleshoot the build problem and resolve it.  Once the application has successfully built, you may continue.

## Deploying the Application
Ensure that the deployment action from Eclipse will target the WebLogic Cluster running on Azure.
* Secondary click on the weblogic-cafe in the Project Explorer and choose Properties.
* Click on "Server" in the left navigation pane.
* You should see your local and remote WebLogic servers.  Select the cluster one and choose Apply and Close.
* Go to the 'Servers' panel, find the WebLogic cluster instance, secondary click -> Properties -> WebLogic -> Publishing -> Advanced. 
* Remove 'admin' as target by clicking on the red X. 
* Add a new target by clicking green plus.
   * Click on the little menu icon that appears near the red X and select 'cluster1' in the dialog that appears. Click OK.
   * Click Apply and close. 
* Secondary click on weblogic-cafe in the Project Explorer and choose Run As -> Run on Server.  
   * If a dialog appears saying "Select which server to use", select the cluster one, check the "Always use this server when running this project, and click Finish.
* Once the application runs, Eclise will try to open it up in a browser. The browser will fail with a 404. This is normal. We delibarately did not deploy the appllication to the admin server.
* In the azure portal go to 'Resource groups'. Find and click weblogic-cafe-group-`<your suffix>`.
* Find and click on myAppGateway. Find and copy the DNS name for the Application Gateway.
* The application will be available at `<App Gateway DNS>`/weblogic-cafe.

## Connect to the Coherence JMX Server using JConsole
The Coherence Mbean server will be started on the oldest machine of the cluster, so we need to find it and connect to it. We can connect using JConsole. [JConsole](https://en.wikipedia.org/wiki/JConsole) is a GUI tool that can monitor remote JVMs, and it comes for free with the JDK.
* Go to `<admin server DNS name>:7001/console`.
* Sign in with username `weblogic` and password that was specified in section [Create the WebLogic Cluster on Azure](#create-the-weblogic-cluster-on-azure).
* Click `Diagonostics` in "Domain Structure" window, and go to `Log Files`.
* Look for `ServerLog` of server `msp1`, select and click the `View` button.
* Click `Customize this table` and change `Time Interval` to `All` and `Number of rows displayed per page` to `5000`, click `Apply`.
* Use the browser's search function to look for `OldestMember`, note the machine name for `OldestMember` (it will be something like `mspVM1`).
* In the azure portal go to 'Resource groups'. Find and click weblogic-cafe-group-`<your suffix>`.
* Search for the oldest virtual machine using it's name, click on it to enter the Overview page.
* Note the Public IP address for the machine.
* Open JConsole using the following command for the Windows command prompt:
```
[<Path to JDK>\bin\]jconsole -J-Djava.class.path="<Path to JDK>\lib\jconsole.jar;<Path to JDK>\lib\tools.jar;<Path to Oracle Home>\wlserver\server\lib\weblogic.jar" -J-Djmx.remote.protocol.provider.pkgs=weblogic.management.remote
```
* For PowerShell, please use the following:
```
[<Path to JDK>\bin\]jconsole '-J-Djava.class.path="<Path to JDK>\lib\jconsole.jar;<Path to JDK>\lib\tools.jar;<Path to Oracle Home>\wlserver\server\lib\weblogic.jar"' '-J-Djmx.remote.protocol.provider.pkgs=weblogic.management.remote'
```
* For Mac and Linux, please use the following:
```
[<Path to JDK>/bin/]jconsole -J-Djava.class.path=<Path to JDK>/lib/jconsole.jar:<Path to JDK>/lib/tools.jar:<Path to Oracle Home>/wlserver/server/lib/weblogic.jar -J-Djmx.remote.protocol.provider.pkgs=weblogic.management.remote
```
* In the 'JConsole: New Connection' window, select `Remote Process` and fill the address as following:
```
service:jmx:t3://<Oldest Machine IP>:8501/jndi/weblogic.management.mbeanservers.runtime
```
* Fill in username as `weblogic`, password as what was specified in section [Create the WebLogic Cluster on Azure](#create-the-weblogic-cluster-on-azure), click Connect. If a window pops up and says 'Secure connection failed. Retry insecurely?', click `Insecure connection`.
* After the connection is established, verify it by checking there is a green connection icon on the top right.

## Test the cache
* In JConsole switch to the `Mbeans` tab.
* Step into the Coherence cache attribute window by clicking `Coherence->Cache->"oracle.coherence.web:DistributedSessions"->session-storage->myCoherence->mspStorage1->3->back->Attributes`.
* You can see the size now is `0`. That is because you haven't accessed the webLogic-cafe application yet.
* Now go to `<app gateway DNS>`/weblogic-cafe. The application just created a session scoped bean. Click the 'Refresh' button in JConsole and you will see the cache size increment by 1.  Accessing the application in another tab won't affect this number because you will hit the session cache.
* Open a new incognito browser window and access the webLogic-cafe application again. The cache size will increase because the new window does not share sessions with previous window.

## Cleaning Up
Once you are done exploring all aspects of the demo, you should delete the weblogic-cafe-group-`<your suffix>`, weblogic-cafe-managed-identity-group-`<your suffix>` and weblogic-cafe-db-group-`<your suffix>` resource groups. You can do this by going to the portal, going to resource groups, finding and clicking the groups and clicking delete. This is especially important if you are not using a free subscription! If you do keep these resources around (for example to begin your own prototype), you should in the least use your own passwords and make the corresponding changes in the demo code.
