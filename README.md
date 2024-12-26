<h1>SOC Automation (Home Lab) - Part 3</h1>

<p align="center">
<img src="https://snipboard.io/col95p.jpg" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<h2>Description</h2>
<br />
<p align="center">
Welcome to part three of five of the series on the SOC automation project. If you haven't seen the previous parts where we go over how to build a diagram for this lab and install what is required I highly encourage you to go and read that. First, today's objective is to configure our Hive and Wazuh servers so we can get them up and running properly. By the end of this part, you will have both the Hive and Wazuh servers up, along with a Windows 10 client reporting into Wazuh. Let's get started.
  
To begin, let's configure our Hive instance first, to get this up and running successfully. In part two, we installed multiple components for the Hive, and beginning with Cassandra, which is used for the Hive's database. We want to edit Cassandra's configuration files, and to do that we can type "nano /etc/cassandra/cassandra.yaml". 

So this is where we can customize our listen address or ports, along with the cluster name. For example, right here it says "cluster_name as "Test Cluster". You can leave it as is, but I'm going to delete this and change it to "mydfir". 


Next, we want to find the listen address and RPC address. Now we can do this a couple ways. We can scroll down until we find it or we can hold ctrl + w. This will open up a search bar for you and then we can type in "listen" and hit "Enter".

Let's scroll down just a little bit and we should see "listen_address" as "Localhost". We want to remove this and enter in our public IP of the hive. So this one is 143.110.217.149. Now your public IP will likely be different than mine, but just make sure it's reflecting your public IP for the Hive. 

Next, we want to find RPC address. So again, hold ctrl + w and then we'll search "rpc_address". Then scroll down just a little bit until we find "rpc_address". So let's remove the local host and do the same that we did earlier. Type in your public IP of the Hive.


Now that we changed our listen address and RPC address, we need to change one more configuration and that is the seed address. So I'll type in cntrl + w and type in "seed". Eventually, we'll see what is called "seed_provider". Now you can always go ahead and search for "seed_provider" and it will bring you straight to here. We want to change the local host address of 127.0.0.1 to our public IP of the hive and then we'll save this out by hitting cntrl + x + Y. 

The next step is to stop our Cassandra service and to do that we can type in, "Systemctl stop cassandra.service". I'll hit "Enter" and because we installed the Hive using their package, we must remove old files. To do that, we can type in "rm -rf /var/lib/cassandra/*". Hit enter.

Now that the older files have been deleted, we can now start up the service by typing "systemctl start cassandra.service and hit "Enter". It is always a good habit to just double-check your services, just to make sure it is running. So again, "systemctl status cassandra.service". Under the "Active", you can see that it's active in brackets "(running)".

We know that Cassandra is running now. We can exit by hitting Q on our keyboard and then I'll just clear out the screen for now. The next thing we want to set up is Elastic Search, which is used to manage data indices AKA querying data. The configuration files for Elastic Search is under "nano /etc/elasticsearch/elasticsearch.yml". 

In this configuration, I'll scroll down until I see "cluster.name" so I can remove the comment. I'll remove "my-application" and change this to "thehive". Scrolling down, we want to remove the comment for "node.name" and then I'll leave it as "node-1". Scroll down just a bit until we see "network.host". I will uncomment this and then put it in our public IP of the Hive.


Now by default, the HTTP Port is "9200". Now you can go ahead and uncomment it or comment it, because by default, it's 9200. I'm just going to uncomment it here. In order to start Elastic Service, it requires either a discovery seed or a cluster initial Master node either or. In my case, I will just uncomment cluster and I'll remove the node two because I do not have a second node but essentially the discovery seed is what you would configure to scale out elastic search now because we are only using this as a demo environment and there's no need to scale I'll just leave it as is let's go ahead and save that out now we can start the service for elastic search we'll type in system CTL start elastic search and hit enter once that is done we'll enable the service by typing system CTL enable elastic search great now that elastic search has been started and en enabled we can see the status of elastic search so we see that is active and running I do also want to double check Cassandra's service because sometimes it stops so let's type in system CTL status Cassandra and it's still running awesome the next section is to start configuring The Hive and before we actually configured the hive's configuration file we want to make sure that the hive user and group has access to a certain file path so let's double check that we'll type in ls-opt thp this is the file path that the hive requires access to as we can see right now root has access to the hive directory we need to change that to change that we'll type in ch wn- capital r The Hive colon The Hive SL opt SL thp and hit enter it's essentially saying change owner to The Hive user and the hive group over to the destination directories so if we were to type in ls- La again pointing to the directory we can now see it is the hive user and the hive group so now we are good to go and configuring the hive's configuration file the configuration file is located under SLC slash thehive and then it is called application. NF or com and hit enter so if we scroll down just a little bit here we notice that the database and index configuration this is what we need to configure I change the host name and remove the 127 address to our public IP of the hive next if you recall the cluster name for Cassandra I changed it to my dfir so I will remove thp and type in my dfir scrolling down a bit again the host name is 127 I'm going to remove that and type in my hive IP now let's scroll down we see that the storage is pointing to this directory now if you read the comments above right here it says the path can be updated and should belong to the usergroup running the hive that is why we have to change the ownership of those directories to The Hive otherwise it wouldn't have access to write in that directory if we scroll down we can see application based URL it is currently pointing to Local Host I will remove this and then type in my public IP of the hive so far so good it's looking pretty nice I'll scroll down and that's the end of the configuration by default The Hivehas both cortex and M enabled cortex is their data enrichment and response capability whereas misp is used as their CTI platform their cyber thread intelligence platform I'll leave this as is for now and save out the configuration file now let's go and start out the hive system CTL start the hive and now system CTL enable The Hive now that it's enabled let's go and check the services for the hive and it is active and running perfect one thing to keep note of is that if you cannot access the hive make sure to check all three services Cassandra elastic search and the hive all of them should be running otherwise The Hive won't start so for example let's take a look at Cassandra again just to make sure so Cassandra is running and then we'll take a look at elastic search elastic search is running and lastly we'll take a look at the hive and the hive is running perfect so in theory we should be able to access the hive by navigating to the public IP of the hive with Port 9000 and we are presented with the hive dashboard we can log in with their default credentials which is admin atthehive dolo and the password is secret if you have been following along you might experience an error on trying to log into the hive this could be trying to use default credentials and then it will tell you that your authentication failed if that's the case take a look at the status of elastic search likely your elastic search is down if that's the case if you are facing that error you want to create a custom jvm option file and we can do that by typing in Nano Etsy elastic search jvm. options. d/ jvm. options once you open up this file you want to paste in the following content essentially this is telling Java to limit its memory usage to 4 gigs our virtual machine has a total of 8 gigs however for the sake of making sure nothing crashes I'll change it to 2 gigs now this is telling elastic search to allocate 2 gigs of memory for Java you can go ahead and save this out and then you want to restart your elas IC search now again this is only if you are experiencing any errors when you're trying to log in using default credentials now that we have the hive configured we'll move on to Wasa and configure that to begin with configuring Configure Wazuh Wasa you want to log into the dashboard using the administrative credentials that we obtained in part two and immediately we see no agents were added to this manager and also I just wanted to very quickly show you if you you do not have the credentials you can head over to wasa's manager and then type in LS just to see if your was- install Das file exists it should technically if you've downloaded Wasa via the curl option now what you want to do is run the command t- xvf and then this will extract all the files from your tar file once those files have been extracted change into the was- install files directory from here we can type in LS and the file that we're looking for is called was- password.txt so we can cat that out and here will be all of our passwords the main one that we want of course is the admin password this is what we're going to be using to log into the dashboard with and secondly we want the Wasa API user we'll be using this later on in the lab to perform responsive capab cap abilities heading back to our wasal dashboard as we can see again we have no agents installed in order to install one we can simply click on ADD agent because we are using a Windows machine select Windows scroll down for the server address we'll put in wasa's public IP which in my case was 15923 1739 and your public IP is going to be different than mine on underneath we see assign an agent name this is optional but I'll type in my dfir and then it asks you if you want to select one or more existing groups I'll leave that as default now we can copy the following command so I'll go ahead and copy that and I am on my Windows machine I'll open up a Powershell window with administrative privileges once we open that up go ahead and paste in the command and then hit enter after the command is done installing we can then start the service by typing in net start Wasa SVC that is one way to start the service or you can go ahead and type in services and then from here you want to look for Wasa Service as we can see it's running so that means in our dashboard we should be able to see our agent I'll go ahead and close this one out and currently I do see the agent but it is disconnected so let's just wait a little bit and see what happens so after a couple of seconds we you can see that there is a total agent of one and an active agent as well that means our Windows machine is checking into Wasa successfully now we can click on security events and start querying four events you just configured both the hive and Wasa and now they are working as expected our end goal is to detect mimik cats usage on our Windows 10 client machine and in order to do that we must first generate Telemetry and create an alert related to mimik cats which is what we will do in the next episode that is it for the video and I hope this has been helpful for you so far if you stumbled across any kind of Errors along the way do let me know and I'll be happy to help you out.

