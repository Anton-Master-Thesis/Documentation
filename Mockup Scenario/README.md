# Scenario
This scenario is for the use of the python implementation of the Dataprocessor and Dataproducer found on the [github page](https://github.com/Anton-Master-Thesis).

# Setup
(Note, all links assumes the AH-cloud is hosted locally)

First you need to setup the Arrowhead local cloud found [here](https://github.com/eclipse-arrowhead/core-java-spring).

When starting the local cloud you only need to start the core systems (Authorization, Orchestrator, Serviceregistry) and the EventHandler.

Then install both the [dataproducer](https://github.com/Anton-Master-Thesis/Dataproducer) and [dataprocessor](https://github.com/Anton-Master-Thesis/Dataprocessor), in their respective repository is their individual setup guide.

Both the dataproducer and dataprocessor need to be authorized to access the eventhandler. 
This can be done via the Authorization's [swagger-ui](https://127.0.0.1:8445/).

Use the "intracloud" endpoint inserting the dataproducer's/dataprovider's id in the consumerId value and the eventhandler's id in the providerId value.
The interfaceId is either 1 for "HTTP-SECURE-JSON" or 2 for "HTTP-INSECURE-JSON".
For the dataprocessor the serviceDefinitionId values need to be the EventHandler's "event-subscribe" and "event-unsubscribe" services id.
For the dataproducer the serviceDefinitionId value need to be the EventHandler's "event-publish" service id.

All of the different Id's can be found in the serviceregistry's ["mgmt"](https://127.0.0.1:8443/swagger-ui.html#/All/getServiceRegistryEntriesUsingGET) endpoint.
I recommend copying the response and pasting it into an IDE where you can format json and search the response.

Then the dataprocessor need to be registered in the serviceregistry. 
This can be done via the serviceregistry's [swagger-ui](https://127.0.0.1:8443/). 
The dataproducer will register itself when started.

The dataprocessor then has to get the authorization to access the dataproducer, this is done via the Authorization's [swagger-ui](https://127.0.0.1:8445/).
This is done the same way as for the authorization towards the Eventhandler previously but with the providerId being the dataproducer and the serviceDefinitionId being the service which the dataproducer registers.