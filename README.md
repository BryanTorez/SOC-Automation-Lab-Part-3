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


Now by default, the HTTP Port is "9200". Now you can go ahead and uncomment it or comment it, because by default, it's 9200. I'm just going to uncomment it here. In order to start Elastic Service, it requires either a discovery seed or a cluster initial master node, either or. In my case, I will just uncomment cluster and I'll remove the node too, because I do not have a second node, but essentially the discovery seed is what you would configure to scale out elastic search.

Now because we are only using this as a demo environment and there's no need to scale, I'll just leave it as it is. Let's go ahead and save that out. Now we can start the service for elastic search. We'll type in "systemctl start elasticsearch" and hit "Enter". Once that is done, we'll enable the service by typing "systemctl enable elasticsearch".

Now that Elastic Search has been started and enabled, we can see the status of Elastic Search. So we see that it is active and running. I do also want to double check Cassandra's service because sometimes it stops. So let's type in "systemctl status cassandra.service". It's still running. 

Awesome, the next section is to start configuring The Hive and before we configure the Hive's configuration file, we want to make sure that the Hive user and group has access to a certain file path. So let's double-check that. We'll type in "ls -la /opt/thp". This is the file path that the Hive requires access to, as we can see right now, "root" has access to the Hive directory. 

We need to change that. We'll type in "chown -R thehive:thehive /opt/thp" and hit "Enter". It's essentially saying, "Change owner to the Hive user and the Hive group over to the destination directories!" So if we were to type in "ls- la /opt/thp" again. Pointing to the directory, we can now see it is the Hive user and the Hive group. So now we are good to go to configure the Hive's configuration file. The configuration file is located under "nano /etc/thehive/application.conf" and hit "Enter". 

So if we scroll down just a little bit here, we notice that the database and index configuration, this is what we need to configure. I'll change the hostname and remove the "127" address to our public IP of the Hive. Next, if you recall the cluster name for Cassandra I changed it to "mydfir". So I will remove "thp" and type in "mydfir". 

Scrolling down a bit again, the hostname is "127". I'm going to remove that and type in my Hive IP address. Now let's scroll down, we see that the storage is pointing to this directory. Now if you read the comments above right here, it says, "The path can be updated and should belong to the user/group running the Hive". That is why we have to change the ownership of those directories to the Hive, otherwise it wouldn't have access to write in that directory.

If we scroll down we can see "application.basedUrl". It is currently pointing to "localhost". I will remove this and then type in my public IP address of the Hive. So far so good. I'll scroll down and that's the end of the configuration. By default, the Hive has both "Cortex" and "Misp" enabled. Cortex is their data enrichment and response capability, whereas Misp is used as their CTI platform, their, Cyber Thread Intelligence Platform.


I'll leave this as is for now and save out the configuration file. Now let's go and start the Hive, "systemctl start thehive" and now "systemctl enable thehive". Now that it's enabled, let's go and check the services for the Hive. And it is active and running. Perfect. One thing to keep note of is that, if you cannot access the Hive make sure to check all three services, Cassandra, Elastic Search, and the Hive. All of them should be running, otherwise The Hive won't start. In theory, we should be able to access the Hive with Port 9000.


We are presented with the hive dashboard. We can log in with their default credentials which are "admin@thehive.local and the password is "secret". If you have been following along, you might experience an error on trying to log into the Hive. This could be trying to use default credentials and then it will tell you that your authentication failed. If that's the case take a look at the status of Elastic Search likely your Elastic Search is down.



If that's the case, you want to create a custom JVM option file and we can do that by typing in "nano /etc/elasticsearch/jvm.options.d/jvm.options". Once you open up this file, you want to paste in the following content. Essentially this is telling Java to limit its memory usage to 4 gigs. Our virtual machine has a total of 8 gigs, however for the sake of making sure nothing crashes, I'll change it to 2 gigs.


Now this is telling Elastic Search to allocate 2 gigs of memory for Java. You can go ahead and save this out. Then, you want to restart your Elastic Search. Now again, this is only if you are experiencing any errors when you're trying to log in using default credentials. Now that we have the Hive configured, we'll move on to Wazuh and configure that.

To begin with configuring Wazuh, you want to log into the dashboard using the administrative credentials that we obtained in part two. Immediately, we see no agents were added to this manager. Incase you do not have the credentials, you can head over to Wazuh's manager and then type in "ls", just to see if your "wazuh-install-file" exist. It should, technically. If you've downloaded Wazuh via the curl option. 


Now what you want to do is run the command "tar -xvf wazuh-install-files.tar" and then this will extract all the files from your tar file. Once those files have been extracted, change into the "wazuh-install-files" directory. From here, we can type in "ls" and the file that we're looking for is called wazuh-password.txt. So we can cat that out. 

Here will be all of our passwords. The main one that we want of course, is the admin password. This is what we're going to be using to log into the dashboard. Secondly, we want the Wazuh API user. We'll be using this later on in the lab to perform responsive capabilities.

Heading back to our wazuh dashboard. As we can see again, we have no agents installed. In order to install one, we can simply click on "Add Agent" because we are using a Windows machine. Select "Windows". Scroll down for the "Server address". We'll put in Wazuh's public IP. Which in my case was "159.203.17.39". Your public IP is going to be different than mine.

Underneath, we see "Assign an agent name". This is optional, but I'll type in "mydfir". It'll then ask you if you want to select one or more existing groups. I'll leave that as default. Now we can copy the following command. So I'll go ahead and copy that. I am on my Windows machine so I'll open up a Powershell window with administrative privileges.

Once we open that up, go ahead and paste in the command and then hit "Enter". After the command is done installing, we can then start the service by typing in "Net Start wazuhsvc". That is one way to start the service, or you can go ahead and type in "Services" in the windows searchbar. Then from here you want to look for Wazuh Service. As we can see, it's running. 

So that means in our dashboard, we should be able to see our agent. I'll go ahead and close this one out. Currently, I do see the agent but it is disconnected, so let's just wait a little bit and see what happens. So after a couple of seconds, we can see that there is a total agent of one and an active agent as well

That means our Windows machine is checking into Wasa successfully now we can click on security events and start querying four events you just configured both the hive and Wasa and now they are working as expected our end goal is to detect mimik cats usage on our Windows 10 client machine and in order to do that we must first generate Telemetry and create an alert related to mimik cats which is what we will do in the next episode that is it for the video and I hope this has been helpful for you so far if you stumbled across any kind of Errors along the way do let me know and I'll be happy to help you out.

