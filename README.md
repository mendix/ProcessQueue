# ProcessQueue
This module enables you to control the amount of these microflows that are executed at once by assigning them to queues.  Each of these queues can be configured to handle a subset of these microflows and you can also set a limit to the number of microflows each queue can execute at once. This allows you to control the maximum load put on your application during peak usage by these microflows while still ensuring all microflows will be executed eventually.  The queues use a FIFO approach (first-in, first-out) and will automatically restart themselves (and any microflows still left to execute) after a server restart.

NOTE: This module is deprecated in favor of the [native Task Queue in Mendix 9](https://docs.mendix.com/refguide/task-queue/) and is no longer maintained in this repository.

## Description
This module enables you to control the load on your application by configuring different Queues. The amount of parallel processes and the number of queues can be controlled from the runtime and you’ll be able to see the progress real-time in your application.
Typical usage scenario
- You want to schedule a job but don’t want to execute the job at a specific time.
- To control a process which crashes a server if to many users execute the action at the same time.
- Create unlimited different queues.
- Configure multiple different microflows to run in a single queue.
- Configure multiple different microflows to run each in a different queue.
- Control the nr of parallel threads per queue.

## Dependencies
- Mx Reflection Model module


## Configuration
After importing the module, you should connect the “QueueOverview” form to your application. This is the starting place for defining the different queues and processes. Add the microflow "ASu_InitialiseQueue" as a startup event to instantiate the queue. Before configuring the queue you need to synchronize the Mx Model Reflection module, make sure you sync the "ProcessQueue" as well. 

Each microflow you’ll configure here should have one input parameter of the type: “ProcessQueue.QueuedAction”.

Off course you’ll want to retrieve the object for which you’ve created an queuedAction, the best way to do this is create an association from your object to the QueuedAction entity. This can either be a reference(set) with the default owner or the owner both, but try to prevent to add an association from QueuedAction to your entity. You should minimize the changes to this module.

In the microflow you’ve configured in the queue there should be an QueuedAction input parameter, using this parameter you can retrieve your object.
In the folder: “Example / Test” you will found an example how to queue an action.


## Other

The constant: "FinishedQueuedActionsCleanupInDays" can be used to automatically clean up finished queued actions (through the scheduled event SE_CleanupFinishedQueuedActions):
Negative value = disabled.0 = clear all finished actions1 or more = clear all finished actions that completed [1 or more] days ago.

Please note that (when dealing a large amount of actions in a short period of time):
-> Create QueuedAction (no commit) -> add to list -> append to queue -> commit list of queued actions 
is inferior to:
-> Create QueuedAction (commit) -> append to queue.

However both constructions should work now (tested batch sizes up to 10000 objects in size).

### Automatic retry behavior:

The module will keep retrying exponentially for up to 11 retries. Initial retry will have a delay of 1 second (2^0), the second retry will take (2^1) seconds and the nth retry (2^n-1) seconds for a maximum of 2^10 = 1024 seconds for a combined total of 2047 seconds (=34 minutes) which is excessive but finite on purpose. In tests even adding 10000 actions at once (batch commit) will only take 5 retries (and only the first action is affected). This is basically the time it takes the microflow doing the batch commit (of queued actions) to complete.
