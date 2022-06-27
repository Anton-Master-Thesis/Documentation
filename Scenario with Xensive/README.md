# Scenario
This scenario will describe the setup and running of the XENSIVE predictive maintenance kit with Arrowhead and the Dataprocessor found in the organization [github page](https://github.com/Anton-Master-Thesis).

## Setup

### Xensive
You will need to follow the guide in the [repository](https://github.com/Anton-Master-Thesis/pred-main-xmc4700-kit) in order to setup the IDE so that you can compile and run the code.
After that the network need to be configured, this is done in the file ''demos/include/aws_credentials.h".
This is simply the wifi name and password.

After this the Arrowhead module need to be configured.
This is done in "application_code/Arrowhead/ah_config.h".
Here is the configuration of the ip address and port of the ServiceRegistry as well as the ip address and port of the Orchestrator.
The serviceUri is also configured here but they seldom change and therefore can be left as is.

There is also configuration of the service which is to be published to the SR in the "AH_SERVICEREGISTRY_REQUEST_BODY".
It is important that the systemName in the providerSystem field is the same as the certificate name if running in secure mode (not available currently).
The full providerSystem field in "AH_SERVICEREGISTRY_REQUEST_BODY" also need to match the requesterSystem field in "AH_ORCHESTRATE_REQUEST_BODY" and also the JSON object in "AH_EVENTHANDLER_EVENT_SOURCE".

The "AH_EVENTHANDLER_EVENT_TIMESTAMP" need to be configured since the XENSIVE kit does not have a Real Time Clock.

The "AH_EVENTHANDLER_EVENT_METADATA" is what allows filtering of the different "AH_EVENTHANDLER_EVENT_TYPE" and is recommended to explain the physical location of the sensor system.

The different AH_HTTPS_* is used when connection with secure mode, however the TCP library used currently does not support this connection.

### Arrowhead Local Cloud
The Arrowhead local cloud configurations are simple.
There is a need to run the Service Registry, the Orchestrator, the Authorization, and the EventHandler.
All of these system need to run in insecure mode until the TCP issue in the Xensive system is fixed.
This configuration is done in the file "\<system>/target/application.properties" after the compiling of the systems when cloning the core-java-spring [repository](https://github.com/eclipse-arrowhead/core-java-spring).
The value which need to be changed for secure mode is "server.ssl.enable".
The Arrowhead local cloud can not run on the Infineon Laptops since they have extra firewall security and thus are not allowed to receive requests.

The EventHandler need to be configured in the same file as the secure mode changes where made.
The changes to be made are the "domain.name" field which is the IP address that is published in the SR, this need to be set to the network IP the device running the EventHandler have.
The other field to configure is the "time_stamp_tolerance_seconds" which changes the allowed timestamp when a system publishes an event.

### DataProcessor
The data processor also need to receive requests and as such can not be run on an Infineon laptop.

The configuration for the dataprocessor are done in the Arrowhead module and in the "run.py" file.
In the file "Arrowhead/SystemConfig.json" the address field need to be configured to be the network ip address of the device running the dataprocessor.
The port field need to match the port argument of "app.run" function in the run.py file.
The systemName field need to match the certificate common name (only needed if the cloud is running in secure mode).
And the cert field need to be configured to have a working certificate (only needed if the cloud is running in secure mode).

The orchestrator configuration is located in "Arrowhead/Orchestration/OrchestrationConfig.json".
Here the ip and port fields need to coincide with the ip address and port of the orchestrator.
The secure field tells the application if the local cloud runs in secure mode or insecure mode.

The EventHandler does not have any configuration files on it's own however there are configuration files for the different subscriptions.
Subscription files can be added and removed as needed but they need to be located in the "Arrowhead/EventHandler/Subscriptions" folder.
They need to have a "eventType" field which expresses which event type to subscribe to.
Optionally there is a "filterMetaData" field which configures which matching metadata are required in order to be notified of an event.
And it also requires a "notifyUri" field which need to coincide with a route of the flaskapp located in "flaskapp/data/routes.py".

## Cloud management
After going through the process of configuring the systems there is a need to allow them to connect with the other systems.
If the local cloud is running in secure mode there is a need to add the "sysop.p12" certificate for the cloud to the browser.

The data processor need to be added manually to the Service Registry, This is done by using the swagger UI via a browser for the SR (\<SRIP>:\<SRPort>) open the "ALL" dropdown and scroll down to the "POST /serviceregistry/mgmt/systems" function, press "try it out" on the right, and add the different fields which corresponds to the fields in the data processor system config ("/Arrowhead/SystemConfig.json").

The Xensive system needs to be run once when it is configured in order for it to publish its system and service to the SR.
After this is done there is a need to go to the authorization swagger UI (\<AuthIP>:\<AuthPort>), drop down "ALL" and scroll until "POST /authorization/mgmt/intracloud", "try it out", and input the different fields for the Xensive system.
* The consumerId is the Xensive system id.
* The providerId is the EventHandler id.
* The serviceDefinitionId is the "event-publish" id.
* The interfaceId is the interface used.
All of these ids can be found in the swagger UI for the SR and the "GET /serviceregistry/mgmt" function.

When this is done there is a need to connect the data processor to the Eventhandler.
This is done in the same way as for with the Xensive system but with different values of the fields.
* The consumerId is the Dataprocessor system id gotten when adding the dataprocessor to the SR.
* The providerId is the EventHandler id.
* The serviceDefinitionId is the "event-subscribe" and "event-unsubscribe" ids.
* The interfaceId is the interface used.

There is also a need to authorize the Dataprocessor to connect with the Xensive system.
This is done in the same way but with different values.
* The consumerId is the Dataprocessor system id.
* The providerId is the Xensive system id.
* The serviceDefinitionId is the service configured in the ah_config.h files id.
* The interfaceId is the interface used.

After this is done everything should be able to connect and run.

## Future work (Copied from thesis)
There are quite a lot of improvements which can be done to the artifact presented in this thesis.
The most important parts to fix are two bugs which have been encountered.
The first bug is that the status code is not updated from the HTTP library used when when sending a HTTP request and then reading the response.
This bug has been brought up to the developers of the library but there was not enough time to follow through with a more rigorous search for a cause and solution.
The second bug is that sometimes the library for creating the TCP connection returns a false positive that a TCP connection has been established.
Because of this the functions try to send the HTTP request to an unconnected socket which results in an error.
Both of these bugs have been built around in a less than ideal way and therefore needs to be addressed if there is a continuation of the development of the artifact.

Another thing to add to the artifact is the ability to establish a TCP connection using TLS.
This could not be done to the artifact due to a need to change the library which establishes the TCP connection.
Some alternatives have been found possible to use in this regard, mainly a freeRTOS implementation of either ""mbed_tls" or "wolfssl".
Both of these have been looked at on a surface level and seem as good canditates but where unable to be implemented for this thesis because of the complexity to change to either of these libraries.

There are also performance improving changes which could be made.
Currently there is a TCP connection being established from the sensor system to the EventHandler each time a publish request is made.
This might be changed to persistent TCP connection with which to send the HTTP requests and thus reducing the overhead of the publishing functionality of the sensor system.