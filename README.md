# MsgBroker
Simple message publish/subscribe infrastructure for deeply embedded applications

MsgBroker library provides an extremely simple mechanism to add publish/subscription semantics to any C/C++ project.
Publish/subscribe mechanisms is formed by:
- Topics: messages exchanged between software agents: publishers and subscribers.
- Publishers: agents which update data topics and notify those topic updates.
- Subscribers: agents which attach to topic updates and act according their new values each time they are updated.

Example of use:

```

#include "MsgBroker.h"

/** GPS topic data structure */
struct gps_data_t{
	float lat;
	float lng;
};

/** CMSIS Threads */
Thread *publisher;
Thread *observer;

/** prototypes of publisher/observer tasks */
void publisher_task(void*);
void observer_task(void*);

/** main program*/
int main(void){
	/** Init Message Broker and install topics */
	MsgBroker::init();
	MsgBroker::installTopic("/gps", sizeof(gps_data_t));

	/** CMSIS-RTOS initialization */
	osKernelInitialize();
	
	/** CMSIS Tasks creations */
	observer = new Thread(&observer_task);
	publisher = new Thread(&publisher_task);
	
	/** CMSIS-RTOS startup */
	osKernelStart();
	
	/** loop forever */
	while(1){
	}
}

/** Publisher task */
int publisher_task(void *arg){
	// gps data
	gps_data_t gps_data;
	MsgBroker::Exception e;
	
	while(1){
		// waits 5 seconds (CMSIS Thread wait)
		wait(5000);
		// read gps data from receiver
		ReadGps(&gps_data);
		// publish gps update
		MsgBroker::publish("/gps", &gps_data, sizeof(gps_data_t), &e);
	}
}

/** Observer topic notification callback */
void gps_updated(void *arg, const char * topicname){
	//checks that topic update is relative to "/gps" topic
	if(strcmp(topicname, "/gps") == 0){
		Thread * th = (Thread*)arg;
		// sets signal flag 1.
		th->signal_set(1);
	}
}

/** Observer task */
int observer_task(void *arg){
	MsgBroker::Exception e;
	// attaches to "/gps" topic updates. On each update, function "gps_updated" will be invoked.
	MsgBroker::attach("/gps", observer, &gps_updated, &e);
	
	while(1){
		// Wait for signal flag 1 ... 
		osEvent oe = consumer->signal_wait(1, 0); 
		// if flag set
		if(oe.status == osEventSignal && (oe.value.signals & 1) != 0){	
			// clear flag 1
			observer->signal_clr(1);
			// get updated gps data
			gps_data_t *data = (gps_data_t *)MsgBroker::getTopicData("/gps", &e);
			// do something with gps data
			...
			// mark topic update as consumed
			MsgBroker::consumed("/gps", &e);			
		}
	
	}
}
```

## Content

- /src/ Developement source files
- /dist/ Distributions source files


## Changelog

> 05.05.2015-001
	- Added example code to README.
	
> 20.04.2015-001
	- First release on /dist folder. Version 1.0.20150420001
	
