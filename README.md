# RabbitMQ
### What is RabbitMQ ?
  * The most widely used message broker
  * Open source
  * Lightweight and easy to deploy
  * Supports different messaging protocols
  * Can be deployed to clusters to provide high availability and scalability , neccessary in enterprise solutions
  * Used by many companies on a large scale including Google , VMWare and Mozilla
### Main Features
  * Variable level of reliability , generally configuring for increased reliability will reduce th performance so this can be managed as required.
  * Complex routing capabilities
  * Different configurations to group together brokers for different purposes eg. Clusters , Federation , Shovel models 
  * Highly available message queues
  * Support for multiple protocols eg amqp , mqtt 
  * Client available in a large number of languages including C# , Java , Go , Erlang etc
#### Installation on a Mac
 Click to [install on MAC](https://www.rabbitmq.com/install-homebrew.html) then to enable the RabbitMQ management plugin run this command
 `rabbitmq-plugins enable rabbitmq_management`
 If RabbitMQ is running and the management plugin is enabled , accessing http://localhost:15672 should load the management UI.
 The default credentials for the management console are : 
 Username : guest
 Password : guest
 *Troubleshooting*
  * Make sure that no other service was already running on port 15672 of your local machine
  * Check that RabbitMQ service is started
  * Installation may require a restart
### RabbitMQ UI 
* Connections : is a TCP connection between an application and the rabbitMQ broker.
* Channels : is a virtual connection inside the TCP connection , publishing/consuming is done over a connection rather than a connection and each TCP connection can hold multiple channels.
* Nodes let us see the health of our cluster
* Admin TAB 
     * We can create user then also virtual host. Virtual host is simply a way to segregate applications within the same rabbitmq instance so different users can have different right virtual host and queues and exchanges can be configured when created so that they exist only in one particular virtual host
### Introduction to Messages , Queues and Exchanges
#### Messages
  * A message is a binary blob of data that is handled by RabbitMQ and can be distributed to queues and exchanges
Throughout this material
  * we will use the term __Producer__ to refer to a generic program producing messages and sending them to a RabbbitMQ queue/exchange
  * and the term __Consumer__ to refer to any generic program recieving messages from RabbitMQ
#### Queues
  * A queue is where messsages flowing though RabbitMQ can be stored functioning similar to a post box 
  * Can also be seen as a large message buffer
  * A queue's message storing limit is only bound by the host's memory and disk limits
#### Exchanges
  * An exchange receives messages from producers and pushes them to queues
  * An exchange can be set to forward messages to zero or more queues depending on the routing parameters and exchange configurations
  * If we relate this to the post office analogy , where the queues are the post boxes then exchange are the post men thar deliver messages to the post boxes
___
### Messages
###### Message Acknowledgements
which is a key mechanism in guaranting reliable message transfer in RabbitMQ
   * If a connection fails between a RabbitMQ server and a client (producer or consumer), messages in transit may not have all been processed correctly and need to be re-sent. 
   * To detect such instances, message acknowledgements can be used. If the sender does not recieve a positive acknoledgement (ack) before the connection fails it will re-queue the message
   * It is therefore good practice to acknowledge a message __after__ any required operations on the message are performed.
   * There are different configurations of message acknowledgements , by enabling __automatic__ acknowledgement mode the message will be considered acknowledged as soon as it is sent - acting in a `fire and forget` mode.
   * This will reduce the safety check that a message has been recieved succesfully but allows for increased throughput.
   * Consumers can also send a negative acknowledgement for a message and instruct the message broker to re-queue them.
   * Both positive and negative message acknowledgements can be sent in bulk by setting the *multiple* flag of the acknowledgement command to true.
   * Protocol methods for acknowledgements: 
         * basic.ack is used for positive acknowledgements
         * basic.nack is used for negative acknowledgements
         * basic.reject is also used for negative acknowledgements but is only capable rejecting one message at a time 
__NOTE__ : the exact command name may vary slightly between client libraries of different programming languages
###### Message Ordering
 * By default , message ordering in queues in First In First Out (FIFO)
 * However, queues can be configured to act as priority queues, in which case messages will be ordered depending on their priority which is set by the sender
###### Durability 
 * Durable queues are persisted to disk and thus survive broker restarts. Queues that are not durable are called transient.
 * Setting a queue to durable does not make *messages* that are routed to that queue durable. If the message broker is restarted, a durable queue will be re-declared during broker startup, however, only *persistent* messages will be recovered.
 * The message sender can set the delivery_mode property for a message to set it to *persistent* , in which case it will be persisted to disk as soon it is received by a durable queue.
 * In some cases non-persistent messages are also written to disk when there is shortage of memory. However , this will not make them durable
___
### Queues
###### Temporary Queues
  * Queues can be configured to be deleted automatically in 3 ways : 
      * __Exclusive queues__ can only be used by their declaring connection, and will be automatically deleted once this connection is closed.
      * An __expiry time__ (also known as time to live) can be set for the queue. If the queue is left unused for a duration exceeding this period , the brokoer will automatically delete the queue.
      * __Auto-delete queues__ will be automatically deleted once their last consumer has cancelled through the *basic.cancel* protocol or gone (e.g closed connection)
___
### Exchange
  * An exchange receives messages from producer (sender) and pushes them to queues or rather exchanges.
  * An exchange can bbe set to forward messages to zero or more queues depending on the routing parameters and exchange configurations 
  * We will now go through each of the following exchanges types and learn about each of their routing capabilites 
     * Types of exchanges
             * Fanout
             * Direct
             * Topic
             * Headers
  ###### Fanout Exhange 
     * This is the most simple kind of exchange.
     * A Fanout Exchange routes a message to all queues bound to it
     * Ignores any routing key provided with the message 

![Screenshot-2019-04-05-at-6.44.22-PM.png](https://www.tryimg.com/u/2019/04/05/Screenshot-2019-04-05-at-6.44.22-PM.png)

  ###### Direct Exchange
    * A direct exchange routes a message with a particular routing key to queues bound to that exchange with that exact routing key.
  
[![Screenshot-2019-04-05-at-7.00.13-PM.png](https://www.tryimg.com/u/2019/04/05/Screenshot-2019-04-05-at-7.00.13-PM.png)](https://www.tryimg.com/image/Lpw7R)

 ###### Topic Exchange 
  * A Topic exchange routes a message with a particular routing key to queues whose routing key matches all or a portion of a routing key
  * Messages are published with routing keys containing one or more words seperated by a dot eg multi.word.test
  * Queues that bind to a topic exchange supply a matching routing key pattern for the server to use when routing the message. These patterns may contain an asterisk (`*`) to match any word in a specific position of the routing key , or a hash (`#`) to match zero or more words.
  *  Examples : a message published with routing key `multi.word.test` 
        * will match queues with routing key `multi.#`, `*.word*`, `multi.word.test`, `#`
        * but will not match queues with routing key `multi.*` , `single.#` or `multi.foo.test`
        * The `#` can replace a single or more word while the `*` will replace a single word which matches the pattern 

![Screenshot-2019-04-05-at-7.10.12-PM.png](https://www.tryimg.com/u/2019/04/05/Screenshot-2019-04-05-at-7.10.12-PM.png)
  
  ###### Headers Exchange 
 *  A Headers exchange routes messages based upon a matching of the message's headers to the expected headers specified by the binding queue.
 * It is important to note the difference between the headers exchages and the topic exchange type :  
   
            * The topic exchange type matches on the routing key
            * The header exchange matches on the message header
  * More than one header criteria can be specified as a filter by the binding queue, in which case the binding queue can specify if the message headers need to contain 'any' or 'all' of the header criteria
  * Message header can be matched in any order  

![Screenshot-2019-04-05-at-7.10.12-PM.png](https://www.tryimg.com/u/2019/04/05/Screenshot-2019-04-05-at-7.10.12-PM.png)

### Consistent-Hash Exchange Plugin
It allows the system to scale better without introducing possible race condition. 
  ###### What does this plugin do
* adds a new exchange type : the consistent-hash exchange
* the consistenct-hash exchange 
