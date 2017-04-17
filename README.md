# pcf

PCF Notes
---------
IAAS - Hardware + OS + Network Ex:AWS, Rackspace,Microsoft Azure,Vmware vCloud Air
PAAS - IAAS + platform for running apps. Ex: Cloud foundry, Heroku, Google App Engine, AWS Elastic Beanstalk
SAAS - Complete software application. Ex: salesforce, google apps, gotomeeting etc.

Cloud foundry is open source. It removes vendor lock. It can be used as public or private. It is language independent using buildpacks.

Cloud foundry components
------------------------
Router
Cloud controller
Service Brokers
EA(execution agent has one or more containers)
BOSH

Note: Execution agents are VM's where applications run. Until CF 1.0 EA type is DEA(Droplet Execution Agent), since CF 1.5 EA type is Diego(A VM running one or more Droplets, docker images)

IAAS is VM centric. For scaling more VM's are created from templates.
PAAS is app centric. For scaling creating containers in a VM pool.

PCF Ops Manager: One PCF Ops manager instance is hosted across a cluster of VMs.
Typical enterprise deployment contains 2-3 PCF ops manager instances(dev/test + prod x2 for HA)

PCF charges for each app instance, each ops manager and each managed service.

PCF concepts
------------
Applications -- It is the unit of deployement. Developer focus on apps, not runtimes or services.

Build packs -- PCF supports multiple languages and deployment environments. For ex, Buildpacks for Java, Ruby, Rails, Java script, Node js, Tomcat etc.

Manifests -- A deployment template for an application. redeploy using same manifest.

Organizations -- Organizations are top most administrative unit. It contains spaces and users.

PCF is designed for use by Organizations. Typically a company, department etc.

An organization will have Users, Spaces, Domains and Quotas.

Spaces
------
An organization contains multiple spaces like Dev, Prod etc. Default pws space is development.
Users can create additional spaces.

Spaces provide users access to shared location for application development, testing, deployment to prod, maintenance.

Applications and services scoped to a space. It means Many applications and services are scaled with in a space.

Users and Roles
---------------
Users are members of an organization. you can invite users to share your cloud. 
User have specific roles. Roles control access to domains and spaces.
Roles control access to manage routes, deploy applications and add/bind/remove services.

Organization Roles
------------------
Organization Manager -- invite/manage users, select/change plan, establish spending limits.
Organization Auditor -- read only access to all org and space info, settings, reports etc.

Space Roles
-----------
Space Manager -- invite/manage users, enable features for a given space.
Space Developer -- create , manage and delete applications and services, access to reports and logs.
Space Auditor -- Read only acess to all space info, settings, reports and logs.

Administrator User/Roles
------------------------
These are defined for cf installation. These are different from users defined at org/space.

There are serveral cf commands restricted to administrators only. For ex, Setting organization space/quota, Defining security groups, administering services and add,modify and remove users.

These commands throw 403, if used without access.

Quotas
------
Restrictions on available resources. They are 
Total memory available to all applications
Total no of routes
Max app instance size
Total no of services

These can be viewed using CLI or app manager(on org homepage)

Domains
-------
Each cf instance has a default app domain cfapps.io. you can register custom domains and give it cf for use and manage.
Each app will have a subdomain, so that URL of the app will be subdomain.domain
For ex, if subdomain is myapp, then URL of app is http://myapp.cfapps.io

Routes
------
Route define how to get to an application. A unique route exists to each application in every space.

Domains are mapped to multiple spaces. Route can be mapped to only one space. Same app can be deployed to multiple spaces. Then each will have different URL. For ex, different spaces for dev, test and prod.

Services
--------
services are usually bound to 1 or more applications.
Connection info and credentials are put in an environment variable : VCAP_SERVICES

Note: All configuration data to applications should be passed via environment variables, can't use configuration files. CF uses no file system.

Three interface options exists for cf. CLI, web based app manager console and Eclipse/STS plugin.

CLI
---
To get any help w.r.to any <command>, use 
cf help <command>
 
To login,
cf login -a api.run.pivotal.io -u <username>

cf utility makes restful requests to to the api endpoint(api.run.pivotal.io). This is actually Cloud Controller.

CF URLS
-------
System domain - run.pivotal.io
API End point - api.run.pivotal.io
Apps Manager - console.run.pivotal.io
Apps Domain - cfapps.io
API End points of CC - apidocs.cloudfoundry.org
cf plugins -- https://plugins.cloudfoundry.org/

CF creates a .cf directory in your home directory. It stores context,logs, crash reports etc.
It remembers CF api endpoint, so no need to provide this for next time login.

.cf contains config.json file, with all configuration details.

Note: If you want to change .cf location you need to set using CF_HOME.

If you want to see the details about User, org and space from CLI, use "cf target".

Organization details
--------------------
cf orgs
cf org <org_name>

Managing spaces
---------------
cf spaces
cf create-space <space-name> (creates space in current org by default)
cf create-space <space-name> -o <org-name>

use target command to change space/org point. For ex,
cf target -s <space-name>
cf target -o <org-name>

Deploy to CF
------------
cf push <name_of_the_app>
There are many options
-i -- no of instances
-m -- memory limit
-n -- hostname (by default hostname is app name)
-p -- local path to app directory
-f -- manifest file path
--docker-image -- docker image name

Note: make sure hostname is unique, otherwise CF throws http status code 400 

"cf push myapp" results in myapp.cfapps.io if myapp is already not deployed to CF.
  
cf push myapp -i 1 -m 512M -n myapp-test123 -p build/libs/myapp.war

The above command creates an URL myapp-test123.cfapps.io

Note: The above cf push command stages your application. i.e. It recognizes java war file, prepares a droplet with a JRE and Tomcat server. The droplet is deployed to a container and starts running.

Here are the steps what happen inside
-------------------------------------
1)Updates CF meta data (app name, instances, memory)
2)Establish route(myapp-test123.cfapps.io)
3)uploads app bits
4)Buildpack selected and executed
5)Buildpack configures java, spring for cloud environment, obtains tomcat and creates droplet.
6)Runs droplet on container
7)Health check
8)Shows status of the application

cf apps --> displays apps running with in a space
cf apps <app_name> --> List all instances of the application with in the space.
cf logs myapp --> shows logs of myapp 

Configuring a Deployed application
----------------------------------
cf scale <app> -i <newvalue> --> New instances are added or some existing instances stopped
cf scale <app> -m <newvalue> --> Requires a restart to take effect.

cf stop --> Sends SIGTERM message to app. Sends SIGKILL 10 seconds later if app is still running.
cf start --> starts existing application.
cf restart --> cf stop followed by cf start.
cf restage --> Repeats the staging process and starts the app. Used when env variables, bound services changed.

Add/Remove Routes
-----------------
cf map-route <app> <domain> -n <hostname>
Ex: cf map-route myapp cfapps.io -n myappdev -->myappdev.cfapps.io is also mapped to myapp.

cf unmap-route <app> <domain> -n <hostname>
Ex: cf unmap-route myapp cfapps.io -n myapp-test123 --> myapp-test123.cfapps.io no longer mapped.

cf routes --> list all routes in a space.
cf delete-route <route_name>--> remove route
cf delete-orphaned-routes --> removes un used routes

Logging Terminology
-------------------
Logging server process is called Doppler.

Loggregator: Entire Log Aggregation architecture is called Loggregator. It has Metron Agents + logging servers + traffic controllers.

Components
----------
Metron Agents : Receive both logs and metrics.
Doppler servers: Accumulates data from agents.
Traffic Controllers: Allow access to metrics and logs.
 
Logging sources
---------------
There are two logging sources.
1)Application instances --> sysout and syserr
2)CF components --> Router, Cloud Controller , Execution Agents etc.

There are two ways to get logs from Loggregator.
1)CLI --> cf logs <appname> (Note: provides tail of logs. Use CTRL+C to exit)
2)app manager UI

Note: cf logs <appname> --recent (display recent logs and exit)

Note: cf logs receive data on incoming port 4443. This must be open in your firewall. Otherwise use app manager.

Third party log managers can be configured to get logs from logging servers. There are quite a few third party managers like splunk, Logentries, logstash, Papertrail etc.

Log types
---------
API - cloud controller (change in application state)
APP/# - application/instance number
RTR - Router
CELL - Execution agent
STG - logging from staging
SSH - ssh logging

To login to the container running your application
cf ssh <appname> 

Note: The above command defaults to instance 0. If you want to login to instance 2,
cf ssh <appname> -i 2.

Access to Files/logs without logging in
---------------------------------------
cf ssh <appname> -C <command>

For ex,
cf ssh myapp -c ls
cf ssh myapp -c "cat /logs/stdout.log" -i 1

cf events <app_name> --> display event log related to an application.  

CF_TRACE
--------
set CF_TRACE environment variable on your local machine like this.
set CF_TRACE = true --> For unix based systems use 'export' in place of set.
set CF_TRACE = /path/to/local/file

This environment varaible is detected by cf utility and generates extra debugging output. Also,
displays rest messages b/w cf and cloud foundry.

Common CF deployment issues
---------------------------
Memory set too low --> warden container fails due to memory, discarded, new container created, fails again, restarted, fails ...

Note: The above one is also called flapping.

JAR/WAR too large , upload fails --> Resolve by push JAR/WAR only (or by decomposing application)

Slow start, health check fails --> Specify Tomcat BindOnInit = false (or alter timeout settings)

Best practices - Ruby
---------------------
1)precompile assets --> It reduces staging time
2)Don't push unncessary files , use .cfignore.

Best Practices - Node js
------------------------
1)package.json mandatory for configuration
2)use .cfignore
3)specify application's web start command. i.e. cf push -command "node myapp.js"

Timeouts
--------
when a new application is pushed , timeouts may occur. For ex, cf push timeout is 60 seconds. Application staging timeout is 15 mins. Application start-up timeout is 5 mins. 

we can change setting using the following command.
cf push -t 120. (Max timeout is 180).

For staging and startup use environment variables.
CF_STAGING_TIMEOUT and CF_STARTUP_TIMEOUT

Deployment process
------------------
1. Establish route
2. Stage Application
3. Start Application
4. EA's send heartbeats

Diego uses Auctioneer component to pick best EA for better load balancing.

Manifest file
-------------
Manifest file describes the application deployment options. i.e. same options used along with cf push plus other options only available in the manifest.

cf create-app-manifest <app_name> creates a manifest file which reflects how app was pushed.
Default name is manifest.yml.

cf push automatically detects manifest in the current/parent directories with name manifest.yml
If you want to override, cf push -f /path/to/myapp-manifest.yml

Note: If no manifest file is found, then cf will default all deployement options.

To ignore an existing manifest file, use cf push <appname> --no-manifest 

Sample yaml format
------------------
---
applications:
- name: myapp
  memory: 512M
  instances: 2
  host: myapp #comment
  domain: cfapps.io
  path: . #path to executable
  timeout: 120
  env:
    CF_STAGING_TIMEOUT: 20
	CF_STARTUP_TIMEOUT: 10
  #comment
  
- name: myapp-test
  memory: 256M
  ...
  
3 dahses indicate start of document.
- defines group.
Each indent should be 2 spaces.

Manifest Inheritance
--------------------
you may have multiple manifests for an app. For ex, a base manifest file and environment specific manifest file. To inherit a base-manifest.yml file in app-test-manifest.yml
---
inherit: base-manifest.yml

Note: STS does not use manifest when pushing.
*****
Note: space cannot be set in manifest file. you need to set this using "cf target -s dev" and then push.

Note: Options specified via CLI override options specified in manifest.

Environment varaibles
---------------------
To set environment variables using CLI,
cf set-env <appname> <env-var-name> <value>

Note: This needs cf restage or cf push to take effect.

*****
Note: Environment variables set via manifest file takes precedence over CLI. This is opposite of other push options.

Note: Environment variables retain their values whether app is running or not. so, to remove them

cf unset-env <appname> <var-name> #restage again.

*****
Difference between restage and restart
--------------------------------------
`restage` will stop your application, run the application bits through the staging process to create a new droplet, and then start the new droplet. It's a lot like `push` but without actually pushing new application bits.

`restart` will simply stop your application and start it with the existing droplet.The staging process has access to env variables, etc. so the env can affect the contents of the droplet.

Once environment variables are set, If the buildpack needs to read that variable and take some action like for your agent activation or change stuff like memory or jvm arguments, then
restage, otherwise just restart is fine.

To access environment variables inside an app,
Java: System.getenv("var-name");
Ruby: ENV['var-name']
Node js: process.env.var-name

Environment variables - VCAP_APPLICATION
----------------------------------------
The environment varaible VCAP_APPLICATION gives information on instances, memory etc.

Environment variables - VCAP_SERVICES
----------------------------------------
The environment varaible VCAP_SERVICES gives information on all bound services 

Integration with spring
-----------------------
Activates a profile in java spring application. we need to set in manifest like this.
---
env:
  spring_profiles_active: dev
applications:
  ...

Note: Environment variables set by cf runtime cannot be modified. To view all environment variables of an app,
cf env <app-name>

Scaling applications
---------------------
Update instances is horizontal scaling. Updating memory is vertical scaling.

CPU allocation
--------------
Each EA has 256 shares irrespective of CPU. Allocation is based on the formula
Allocation = 256*container-memory-limit/Total-EA-memory.

For ex, if app size is set to 1G and EA size is set to 32G(generally set by Ops), then CPU shares 
is equal to 256*1/32. i.e. 8 cpu shares for the application.

Scaling Application
-------------------
1)Cloud Controller indicates instances should be started/stopped via CCDB. 
2)Converger reads from CCDB and add tasks to start/stop to central bulletin board.
3)Auctioneer determines least busy cells to use.

Repush
------
Cf push command override the no of instances based on CLI/manifest.
If CLI/manifest do not specify then cf push will stop the application and re push the application.

Note: Use manifest for default settings (or omit settings from manifest)

File system
-----------
Application instances (droplets) run in an isolated warden containers. scaling up creates new isolated file system. scaling down destroys local file storage.So, Do not rely on files because they are transient and not shareable.

Cloud Foundry Services
----------------------
Service is an external application component/dependency such as Db, Msg Queue, Monitoring app, Security etc. Because application has to be self contained anything else they need provided by paas as a service. These are one of the chargeable units in PAAS.

Services provide functionality to your application. 
A service can be shared among multiple applications.
Services are bound to an application using service broker.
Provide connection information to application via env variables called VCAP_SERVICES.

Types of services
-----------------
There are two types of services
1)Managed services --> can be selected from marketplace. These are provisioned by paas.
2)User Defined services --> These are external to CF. PAAS only provides connection information.

User Defined Services
---------------------
These exists already. For ex, Mainframe systems, applications developed in house etc.
There are two ways to access them.
1)User Defined Services --> your ops people continue to manage and provision.
2)Custom Service --> CF uses service broker to provision and bind. These can be fully integrated with CF. They appear in CF marketplace once added. These are again come under Managed services because CF will manage them.

Requirements of Custom Service
------------------------------
Custom development of service broker by developer. Operator install service broker into CF. Optionally create an Ops Manager Tile.

Question: In a CF app How to connect to db, messaging system, email system, nosql db, read/write to files, save/retrieve sessions, anything not coded in app?
Ans: via services.

Managed Services
----------------
These services are preconfigured and made available to CF by operations personnel. 

If it is private/on-premise then your company controls what is available and these services run in company data center.

If it is public cloud, cloud provider controlled and these services may run anywhere.

In private cloud, services are added after PCF installation by ops team. List of available services can be downloaded from https://network.pivotal.io. These services are available with .pivotal extension. These are installed by ops team and are also called "Tiles".

Note: After installation they can be seen in marketplace like marketplace in app manager UI of pws.

Developers can create instances of services once they are available in market place.

PWS and APP Direct
------------------
pws is public cloud foundry instance. It is hosted on AWS. provides extensive marketplace of services via app-direct. Some of them are free and some of them are payable.

Ex: Postgres, MySQL, MongoDB etc.

App-Direct
----------
App-Direct is commerical provider of services. It provides third party services like Redis, Mongo etc to marketplace. These will have various plans and fees. These are provided by pws.
These are run by providers. For ex, Redis instance would run at Redis Labs.

private/on-premise CF can also use App-Direct, but they are run out of your data center. Also, there may be connectivity,security and legal issues.

Cloud Foundry Services
----------------------
All Services provision service instances. For ex, ClearDB service provisions MySQL instances.
You may get a dedicated server or share a multi-tenant server.

Available services depend on CF setup.i.e. They must be installed and configured by CF Ops either
Ops Manager, CLI or Bosh provisioning tool. For this service broker has to be created and enabled using cf create-service-broker and cf enable-service-access commands.

Once Ops have deployed a service it appears in market place. To made available a service to an application is called provisioning.

A Developer can provision a service to the app using the following 2 steps.
1)create a service (also called provisioning a service) using cf create-service <service-name>
2)Bind it to the application (cf bind-service <app-name> <service-name> )

Note: cf create-service <service-name> is available to current space. If you want for multiple spaces run in each space.

CLI commands
------------
cf marketplace --> list all services available in marketplace
cf services --> list all existing service instances in current space.

To provision a new service instance,
cf create-service <service-name> <plan-name> <instance-name>
Ex:
cf create-service elephantsql turtle mypg

Accessing Service instance from App
-----------------------------------
In traditional way say for ex Jdbc, we put all connection properties in a properties file. In cloud foundry, we bind the configuration using the following.
	cf bind-service <app-name> <service-name>

Ex: cf bind-service myapp mypg
Note: use cf restage to take effect of env changes.

A service section can be added in manifest file like this.
---
applications:
- name: myapp
  memory: 256M
  services:
  - mypg
  - mydb

Service details are injected to app by VCAP_SERVICES env variable and any changes are managed external to the application. These details are in json format. They can be accessed inside the app in 3 ways.
Manual --> parsing the VCAP_SERVICES json data manually.
Custom Library --> CF aware helper code can be written like this.
	for(ServiceInfo info: cloud.getServiceInfos()){
		if(info instanceof MySqlServiceInfo){
			connectionURI = ((MySqlServiceInfo)info).getJdbcURL();
		}
	}
Note: The above code is dependent on Language/Framework.
Auto Configuration --> This is dependent on buildpack/framework and uniqueness of service type.

User Provided Services
----------------------
These are provisioned outside of CF. Typically some legacy services. When bound to app, they provide service instance configuration including credentials to the app. Even if you have multiple app instances you pass same credentials into each app. Service can be defined using "cf create-user-provided-service". For ex,

cf cups -p "hostname, port, username, password, instancename" 

Note: The above command prompts to enter values for all the fields mentioned. You can define them in manifest file.

you can access user provided services from VCAP_SERVICES env variable section named "user-provided".

Service Brokers
---------------
Custom Managed Services can be integrated into CF by implementing service broker API for which Cloud Controller is the client.

Service broker API typically exposes 5 rest end points.
catalog, create,delete,bind,unbind.

CF commands are converted into rest calls by Cloud Controller and invoke the Service Broker.
To deploy a service broker use the command "cf create-service-broker".

cf services --> fetch catalog(plans) from service broker.
cf create-service --> creates(or provision) a service instance.(PUT /v2/service_instance/:id)
cf bind-service --> create binding of a service instance(PUT /v2/service_bindings/:service_instance_id/bind/:binding_id)
cf unbind-service --> remove binding of a service instance(DELETE /v2/service_instance/:service_instance_id/bind/:binding_id)
cf delete-service --> remove a service instance (DELETE /v2/service_instance/:id)
cf update-service --> updates binding of a service instance(PATCH /v2/service_bindings/:id)

Note:The marketplace must authenticate with the service broker using HTTP basic authentication (the Authorization: header) on every request. The broker is responsible for validating the username and password and returning a 401 Unauthorized message if credentials are invalid. It is recommended that brokers support secure communication from platform marketplaces over TLS.

Note:Broker can be implemented as a seperate application or along with the service.

Deployment Models
-----------------
Many deployment models are possible.
1)Entire service and broker are deployed by BOSH alongside CF.
2)Broker deployed alongside CF but service may be running outside.
3)Broker pushed as an application to CF.
4)Entire service and broker is deployed and maintained outside of CF.

To register a service broker with CF,
cf create-service-broker <service-broker-name> username password URL

Buildpacks
----------
Applications typically contain source code. Source code may use frameworks to create an application. There are different types of applications like Java/Spring, Ruby/Rails, Node js etc.

Configuration of server to run an application needs OS, Runtimes(Java/Ruby/Node etc), Containers(Tomcat,Apache httpd etc), app binaries. It's a good idea to write scripts to execute the above steps. Buildpack does the same thing.

So, Buildpack is nothing but combination of scripts that assembles OS, runtimes, containers, frameworks and app into a droplet.

Droplets run inside a warden container. Warden containers run inside EA's.

Note:Buildpacks build the droplet. This processing of building is also called as staging.
Buildpacks do not run locally. They run on CF during staging process.

Buildpacks often written as Ruby scripts which contains three parts.
1)Detect --> finds which buildpack should be applied.(If package.json is present then node)
2)Compile --> Assemble app code with runtimes, frameworks,OS (Assemble/pack is a better name than compile)
3)Release --> Release the app to be deployed to EA.

Note: Buildpacks are responsible for preparing the machine image for an application.

Available Buildpacks
--------------------
Buildpacks are installed into CF. They can also be downloaded from external location at push time.

CF community provides buildpacks for other languages or you can write your own by extending an existing buildpack.

Note:Buildpacks can be compatible with multiple PAAS offerings. For ex, CF buildpacks follow HEROKU buildpack design.

To list available buildpacks "cf buildpacks".
To create a buildpack use,
	cf create-buildpack BUILDPACK-NAME PATH POSITION [--enable|--disable]

PATH --> local directory to zip file or URL.
POSITION --> is a positive integer which sets priority and is sorted from lowest to highest.
disable/enable --> To disable/enable the buildpack for staging.

Note: Admin permission are required for the above command. Also, update,delete and rename commands are available.

To push an application with buildpack,
	cf push -b <buildpack-name>/URL

You can specify in manifest file also.
	buildpack: https://github.com/cloudfoundry/java-buildpack
	
Note:Options set in push command override manifest.

Customizing Buildpacks
----------------------
Buildpack API
-------------
/bin/detect app_directory --> Inspect application to determine buildpack applicability.
-->checks for file Gemfire for ruby, package.json for node, setup.py for python.

/bin/compile app_directory cache_directory --> Download and install runtime,container,libraries etc.
-->builds the droplet. Download and install necessary runtime like jre, webserver, jars etc.
These are downloaded from sources external to CF. CF uses a location(cache) for storing downloaded artifacts to speed up subsequent staging operations.

/bin/release app_directory --> build application start command.
-->Builds a yaml formatted hash with three possible keys like below.
	addons: []
	config_vars: []
	default_process_types: 
		web: <start_command>

Note:CF uses only the web command to start the application.

Java Buildpack
--------------
Java Buildpack supports a variety of JVM languages, frameworks, containers with a modular, configurable and extensible design. For ex, Groovy, Scala, Tomcat, Spring etc.

Java Buildpack concepts
-----------------------
Containers --> Executable jars, Groovy, Spring Boot CLI, Play etc.
Frameworks --> App Dynamics, Spring auto configuration, New Relic etc.
JRE's --> OpenJdk, Oracle JDK etc.

Container Detection criteria
----------------------------
Java main() --> META-INF/manifest.mf exists with Main-class attribute set.
Tomcat --> WEB-INF directory exists.
Groovy --> .groovy file with main() method or with no classes or with shebang(#!)
Spring Boot CLI --> .groovy files with no main() method and no WEB-INF directory.
Play --> start and lib/play.play_*.jar exist.

Framework detection criteria
----------------------------
App Dynamics --> service bound to app.
New Relic --> service bound to app.
Spring Auto configuration --> spring-core*.jar in the app directory.

Note: To see Buildpack files during staging process, "cf files <app-name>"

Customizing Buildpack
---------------------
Java buildpack may be altered by configuring standard JRE's, Containers and frameworks or extend the buildpack with your own JRE's , Container and frameworks.

Customization is done either by forking or downloading/modifying.

Most configuration options found in /config. This determines behaviour of JRE, container and framework. Config directory contains yaml files like this.
app_dynamics_agent.yml
cache.yml
components.yml
groovy.yml
open_jdk_jre.yml
oracle_jre.yml

Here is sample openjdk file.

---
repository_root: "{default.repository.root}/openjdk/{platform}/{architecture}"
version: 1.8.0_+
memory_sizes:
  metaspace: 64m..
memory_heuristics:
  heap: 75
  metaspace: 10
  stack: 5

Note: default.repository.root contains download location. For ex, download.pivotal.io.s3.amazonws.com.

So, repository_root URL will be like  download.pivotal.io.s3.amazonws.com/openjdk/lucid/x86_64/index.yml.

Note:index.yml holds location of each version.

Similary, using tomcat.yml file provides customization of tomcat. You can do resource configuration like context.xml/server.xml also. 

To extend the Java buildpack i.e. to add different jre, container and framework implement support class(Ruby) in the appropriate directory with additional support classes as necessary. For ex,
	lib/java_buildpack/jre
	lib/java_buildpack/container
	lib/java_buildpack/framework

Add these new support classes to config/components.yml file.

Customization without modifying buildpack
-----------------------------------------
Simple customization of properties can be done without modifying the buildpack. Using cf set-env or manifest file we can set options like JAVA_OPTS/JBP_CONFIG.

Ex: cf set-env myapp JAVA_OPTS -showversion
Note:Most JVM optons possible except the one that govern memory sizing like -Xms, -Xmx etc.

JBP_CONFIG
----------
This can be used to set java buildpack configuration. For ex,
cf set-env myapp JBP_CONFIG_MY_FILE (where my_file.yml is your config file)
To set default version of java to 7, 
cf set-env myapp JBP_CONFIG_OPEN_JDK_JRE '[version: 1.7.0_+,memory_heuristics: {heap:85}]'

Note: The above one will override open_jdk_jre.yml file configuration.

Offline Buildpacks
------------------
To avoid download buildpacks from internet. Pivotal CF ships with offline buildpacks. Java buildpacks can be packages as offline buildpacks, which is used to build droplets without internet.
 
It is a one-time build process. Generally comes with latest dependencies. It disables remote downloads. It is generally about 180M in size. Install using "cf create-buildpack/update-buildpack".
 
Note: To add a java-buildpack to CF use the following command.
"cf create-buildpack java-buildpack java-buildpack-offline-v<VERSION>.zip"

Cloud Foundry Architecture
--------------------------
CF components
-------------
1)Load Balancer (HA proxy)
This is a seperate component from router. It sits in front of the router. It is a single instance and single point of failure.CF can have only one IP address available to outside world i.e. Load balancer.

Note:Load balancer balances across multiple routers and also supports SSL termination.
Typically you need to configure your own HA proxy or other load balancer like F5,NSX etc for better performance, SSL, handling and high availability.

2)Router
Routes all incoming HTTP traffic(both cf commands and also application traffic). It maintains a dynamic routing table for each load balanced app. Multiple routers are configured by Ops manager.
From CF 1.4, this is rewritten in GO(Go router).

Note:Router has access to logs. It supports web sockets and SSL termination.

3)Cloud controller
Cloud controller responds to all clients.i.e. CLI, web UI, sts/eclipse etc. It does account and provisioning control. It provides restful endpoints for domain objects like apps,services, organizations,spaces,user roles etc. Multple cloud controllers can be configured by Admin.

Note: CC is responsible for application state transitions and desired convergence. It is responsible for permissions/authorization , app placement, auditing and billing events and Blob storage.

4)Blob store
It stores binary large objects. It eliminates the need for re-staging when scaling apps. Currently it is NFS mounted storage(Amazon S3 store).

Note:It stores app packages and droplets.

5)Cloud controller Database
It stores app metadata. It is used by Cloud Controller.

Note:It stores information on App name, no of instances requested,memory limit,routes, bound services etc. It is a postgres DB instance.

6)Availability Zones
Seperate points of failure like power,network,cooling,roof,floor etc. It allows CC to locate instances on seperate zones to boost redundancy.

Note:It provides enhancing app redundancy and high availability.

7)Service broker
Provides an interface for native or 3rd party services like mail server,db,messaging etc.

Note:It is responsible for advertising service catalog(pricing info), exposes api end points like create/delete/bind/unbind. It also requests information from CC of existing instances and bindings for caching, orphan management.

8)Loggregator
It is master logging subsystem. It accepts logs from app instances and other CF components.
This makes log data available to external sinks.

9)UAA and Login services
UAA (user authentication and authorization) provides identity(user management),security and authorization services. It can manage 3rd party Oauth 2.0 access credentials. It provides application access and identity as a service for CF apps. It consists of UAA server,CLI and library. Multiple UAA/Login servers are possible.

Note:It is responsible for token management(token server),user management(ID server),OAuth scopes(Groups) and SCIM. It works as Login server which has UAA database, it supports SAML.

10)CF Bosh
Control primitives(CPI) written for each underlying Infrastructure(IAAS) provider. It is the glue between CF and underlying IAAS. This is the tool for managing large scale distributed systems. It does deployment and lifecycle management. It does continous/predictive updates with minimum downtime.

Note:Iaas installer is currently vSphere(comes default with PCF). It is responsible for VM creation and management. It restarts failed CF internal processes for HA. It can also restart execution agent VMs.

DIEGO
-----
It is an internal project name for elastic runtime enhancements. 
1)It is written to support multiple Operating Systems on the EA VM's(particulary Windows). 
2)It is more scable to avoid bottleneck for CC and Health Manager.
3)It supports containers other than Warden(Particularly Docker).

DIEGO Vs DEA
------------
DEA is original CF internal architecture before CF 1.5. Apps live in warden container in DEA(droplet execution agent). Health manager handled restart of failed app instances. It was written in Ruby, running on Linux. 

DIEGO - Rewrite of DEA and other components in Google GO. Major refactoring of CC and other components since CF1.5.

DIEGO was optional in CF 1.5, but default since CF 1.6. It refactors CC and delegates to new components Auctioneer and Converger. CELLs replace DEAs.

Warden containers support multple implmentations not just running droplets,not just linux etc.
Lets see the components and their functionality.

1)The Cloud Controller passes requests to stage and run applications to the Cloud Controller Bridge (CC-Bridge).
2)The CC-Bridge translates staging and running requests into Tasks and Long Running Processes (LRPs), then submits these to the Bulletin Board System (BBS) through an API over HTTP.
3)The BBS submits the Tasks and LRPs to the Auctioneer, part of the Diego Brain.
4)The Auctioneer distributes these Tasks and LRPs to Cells through an Auction. The Diego Brain communicates with Diego Cells using SSL/TLS protocol.
5)Once the Auctioneer assigns a Task or LRP to a Cell, an in-process Executor creates a Garden container in the Cell. The Task or LRP runs in the container.
6)The BBS tracks desired LRPs, running LRP instances, and in-flight Tasks. It also periodically analyzes this information and corrects discrepancies to ensure consistency between ActualLRP and DesiredLRP counts.

CC-Bridge
---------
Ensures that requests placed onto BBS take place eventually. Provides info about running tasks and LRPs to Cloud Controller.
Note:It allows existing components to interact with DIEGO.

Bulletin Board System
---------------------
Holds CF actions like start/stop app instances,one-off task processes. It is an action queue currently backed by etcd(key/value store). Other implementaions are possible. It is currently written in GO and uses raft protocol.

Note:BBS decouples control components like CC from DIEGO.
Note:etcd does cluster coordination and state management. It is the consistent store at the heart of Diego.

Converger
---------
Converger makes sure that the system is eventually consistent. Converger component is a refactored one which was present in Old DEA Health Manager and cloud controller for instance recover functionality.

Note:It compares desired system state with actual state and takes remedial action if they are different. It places start/stop,restart requests on the BBS. If it spots requests on BBS too long, force BBS to try again.

Auctioneer
----------
Auctioneer determines which cell to run an app instance. It is based on auctioning placement algorithm. Also determines which instance to stop when scaling down. one off tasks are not managed by auction yet.

Note:Auctioneer tells the cell that win the auction to either start/stop the app instance by placing the action on BBS.

CELLs
-----
CELL is a secure and isolated container typically a linux vm. It can also be a windows VM. It is a replacement of DEA. CELL periodically broadcast messages about their state. CELL runs an executor internally to manage containers.

CELLs participate in auctions. CELLs manage app lifecycle(Executor) i.e. start/stop/build etc. 
CELLs manage app containers typically pivotal's secure containers or anyother container with same interface, for ex Docker.
CELLs monitor resource pools like process, file system, network and memory.

DEA architecture
----------------
DEA is previous internal architecture prior to DIEGO and almost does same as that of a CELL. It has Health manager inplace of Converger. DEA broadcast messages(heartbeats) about their state to CC via NATS message bus.

Warden Container
----------------
It is an isolated lightweight process running inside a VM(DEA). It runs a droplet. It is a pivotal's secure implementation of LXC(Secure Linux Container). It isolates apps from each other.
It allows multiple apps running in each VM(DEA).

It uses kernal namespaces to isolate network, cpu, disk and memory etc. It uses Linux cgroups to do resource management. It secures apps from environments.

Health manager
--------------
It is same as converger. But, unlike converger it can't take action when actual state is different from desired state. It listens to NATS message bus for mismatched app states and reports to CC. CC can only make state changes(or actions).

Messaging(NATS--> Not another transport system)
-----------------------------------------------
NATS is an internal messaging bus which maintains system wide communication. It uses pub-sub model. Single NATS per CF installation.

NATS is a non peristent messaging system. Queues are nothing but app events.

Note:Diego uses direct communcation over http i.e. REST.

Log Managers
------------
Third party Log managers are recommonded because they can save far more logging info than CF. Also, they allow persitence,searching,analyzing and metrics. There are variety of them in the market like papertrail,sumologic,logstash,loginsight,splunk etc.

Connecting Third party log managers
------------------------------------
Bind the log manger with a user provided service command like,
cf cups <service> -l syslog://<host>:<port>

Bind the above service and restage the app to take effect. CF sinks loggregator output to this drain for the application.

Note: We know all output is collected from Doppler to Loggregator. Loggregator opens socket to host:port and sends all log info for the app in syslog format. 

Papertrail setup
----------------
a)create an account in papertrailapp.com and use add system button.
b)papertrail will provide you the url to use for syslog. For ex,logs2.papertrailapp.com:41846
c)choose cf in alternatives link, name your system
d)cf cups mydrain -l syslog://logs2.papertrailapp.com:41846
e)cf bind-service myapp mydrain
c)cf restage app

Syslog
------
This is a de facto standard for unix/linux machines. It can log to a file or to a server syslogd(via protocol). All major third party log managers provide syslog servers.

Note:CF generates a TCP or UDP message in the right format and open a socket to your syslog server and send.

Application Performance Monitoring
----------------------------------
APM tools are used to monitor application while running. There are several choices like PWS - New Relic and AppDynamics, Pivotal Spring insight etc.

PWS offers simple interface for new relic at market place. To do this,
create new relic service using "cf create-service newrelic standard apm"
Bind the service using "cf bind-service myapp apm" and restage the application.

Scaling options
---------------
CF allows horizontal scaling, controlling the no of instances of an app running behind a common router(load balancer). This can be controlled via CLI,manifest and through app manager console.

Note: All the above require manual intervention.

Autoscaling
-----------
CF can allow apps to be automatically scaled. This is a service named "App Autoscaler" must be installed by admin. It is not available in marketplace(pws).

Autoscaling plan has to be selected(provisioned), then bind the service and configure.

There are multple options available to configure. Instances can be added when high threshold is reached and remove instances when low threshold is reached.

Autoscaling events can be scheduled. i.e. you can change autoscaling behaviour for a given date and time.

Note:Manual scaling disables autoscaling.

Zero Downtime Deployments
-------------------------
CF supports Zero downtime deployments or release techniques. They are
1)Blue/Green Deployments
2)Canary Deployments

Blue/Green Deployments
----------------------
CF push stops old instances and then start new. Blue/Green deployement eliminates user downtime. This is also called "Zero-downtime", A/B deployment. Steps to do this.

1)cf push blueapp -n myapp --> router (myapp.cfapps.io)
2)cf push greenapp -n myapplatest --> router(myapplatest.cfapps.io)
3)cf map-route greenapp cfapps.io -n myapp --> router(myapp.cfapps.io URL is loadbalanced by router and sends to both blue and green, whereas any request come to myapplatest.cfapps.io only comes to green)
4)cf unmap-route blueapp cfapps.io -n myapp --> router(All request to myapp.cfapps.io comes to green)
5)cf unmap-route greenapp cfapps.io -n myapplatest --> router(removes the route myapplatest.cfapps.io)
6)cf delete-route greenapp

Canary Deployments
------------------
1)Starts with desired no of blue instances.(cf push blueapp -n myapp -i 4)
2)Start with a single green instance and route traffic to both. Green instance is the Canary.
cf push greenapp -n myapplatest -i 1
3)watch the canary, if it behaves scale green up and blue down until blue reaches 0.
cf scale greenapp -i 2
cf scale blueapp -i 3
4)delete blue app.
cf delete blueapp

Continous Delivery
------------------
SCM - source control management can be done using Git,SVN,CVS,Clear Case etc.
CI - Continuous integration tools like bamboo/hudson/jenkins etc run automated compilation,checks,tests on frequent basis.
CD - continuous delivery is CI + automated deployment.

High Availability
-----------------
There are many ways HA possible.
1)BOSH And Health Monitoring,
2)Availability Zones, 
3)Handling App instance failure(converger/or Old Health Manager).

BOSH
----
It is an opensource software, which deploys and maintains the lifecycle of any software. It is designed for distributed systems like CF. BOSH can run on any infrastructure like vSphere, AWS, vagrant VM etc.

BOSH is middle layer between cluster and infrastructure. It creates VMs, start/stop processes, enables continous delivery.

Note:Installting CF actually requires installing BOSH first. BOSH then deploys everything else.
BOSH deploys releases like template VM's(stem cells), softwares to run on VM's and a manifest 
to drive the whole process.

Note:stemcell is like a droplet for VM creation.

Note:The elastic runtime and services are BOSH releases. CF operators typically use BOSH.

OPS Manager
-----------
OPS manager is an UI on top of BOSH. When you install or make changes in Ops manager, BOSH level changes are being made(you can see it in logs).

BOSH components
---------------
Director --> Coordinates BOSH(like CC). It maintains own dbs.
Health Monitor --> Receives heartbeats from VM's.
Worker --> Used to run or modify VM's.
CPI --> cloud provider interface which is an interface to underlying Iaas.

Note:Health monitor does the work of converger i.e.compares the desired state of the system with actual state and if they differ it arranges for failed VM's or processes to be restarted.
typically it sends NATS message, which will be seen by Director and asks Worker to restart via CPI.

Availability Zones
------------------
CF scales app instances across Availability zones. Zones correspond to independent infrastructure segments like different racks, or even data centers.

Converger
---------
Converger compares actual with desired state. State may changes due to new app is deployed or may be app is scaled up/down or app instance failed. Any descrepency causes a request to be placed on BBS. CC is not involved in restart. 

12 Factor applications
----------------------
It outlines architecture principles for modern apps.
1)Codebase
2)Dependencies
3)Configuration
4)Backing services
5)Build,Release and Run
6)Processes
7)port binding
8)Concurrency
9)Disposability
10)Dev/Prod parity
11)Logs
12)Admin processes

Questions
---------
CF_TRACE

when app is crashed data stored in local file system can be accessed by other instances or not
