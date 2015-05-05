# MsgBroker
Simple message publish/subscribe infrastructure for deeply embedded applications

MsgBroker library provides an extremely simple mechanism to add publish/subscription semantics to any C/C++ project.
Publish/subscribe mechanisms is formed by:
- Topics: messages exchanged between software agents: publishers and subscribers.
- Publishers: agents which update data topics and notify those topic updates.
- Subscribers: agents which attach to topic updates and act according their new values each time the get updated.

Example of use:

```
/** GPS topic data structure */
struct gps_data_t{
	float lat;
	float lng;
};

/** CMSIS Threads */
Thread *producer;
Thread *observer;

/** prototypes of producer/consumer tasks */
void producer_task(void*);
void consumer_task(void*);

/** main program*/
int main(void){
	/** Init Message Broker and install topics */
	MsgBroker::init();
	MsgBroker::installTopic("/gps", sizeof(gps_data_t));
	
	/** CMSIS-RTOS initialization */
	osKernelInitialize();
	osKernelStart();
	
	/** CMSIS Tasks creations */
	observer = new Thread(&consumer_task);
	publisher = new Thread(&producer_task);
	
	/** loop forever */
	while(1){
	}
}

/** Producer task */
int producer_task(void *arg){
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

/** Consumer topic notification callback */
void gps_updated(void *arg, const char * topicname){
	// on topic update sets signal flag 1.
	consumer->signal_set(1);
}

/** Consumer task */
int consumer_task(void *arg){
	MsgBroker::Exception e;
	// attaches to "/gps" topic updates. On each update, function "gps_updated" will be invoked.
	MsgBroker::attach("/gps", 0, &gps_updated, &e);
	
	while(1){
		// Wait for signal flag 1 ... 
		osEvent oe = consumer->signal_wait(1, 0); 
		// if flag set
		if(oe.status == osEventSignal && (oe.value.signals & 1) != 0){	
			// clear flag 1
			consumer->signal_clr(1);
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
	
