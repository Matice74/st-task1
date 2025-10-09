### Task 6 

- gained Access to RAbbitMQ via
```bash
Docker docker run -d --hostname my-rabbit --name rabbitmq -p 5672:5672 -p 15672:15672 rabbitmq:3-management
```
- Tested localhost:15672 afterwards which worked fine
- Added RabbitMQ to dependencies

- Added Credentials to application.properties

- added class RabbitConfig that contains Beans declaring Exchange,app-queue and binding to the broker

- Edited PollManager:

- added amqpAdmin and pollsExchange
- edited the CreatePoll-Method to assign option ids and link to poll and) declare a per-poll durable queue and bind it to the topic exchange

- Added a PollEventListener-class which listens to polls.app.queue, parses JSON payload and calls manager.createVote(...).

- To ensure that external subscribers are informed immediately, I edited VoteController to send an event to polls.exchange after manager.createVote(...).

- Struggled with some needed changes in the existing classes

- Afterwards created a small StandAlone Gradle-Project to listen to Incoming votes --> worked
- There are no pending issues with this assignment
