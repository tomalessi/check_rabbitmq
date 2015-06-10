# RabbitMQ Nagios Plugin (check_rabbitmq)

The purpose of this Nagios plugin is to check the health status of one or more (single or clustered/mirrored) Rabbitmq servers.

## Functionality

The check_rabbitmq plugin has the following functionality (these all relate to a specific vhost):

- Check that all queues are synchronized across a cluster
- Check the number of queues that reside on a RabbitMQ server
- Check the number of consumers attached to each queue on a RabbitMQ server
- Check the number of messages in all queues on a RabbitMQ server


## Examples
The examples below have the following assumptions:

- There are two RabbitMQ servers: queue1, queue2
- The queue servers are clustered and all queues are configured to be mirrored
- Username and password for RabbitMQ access are stored in Nagios's resource.cfg as $USER2$ and $USER3$
- All queues are contained in the vhost /vhost1


### Nagios Command Configuration Examples

```text
# 'check_rabbitmq_queue_synchronisation' command definition
define command{
        command_name    check_rabbitmq_queue_synchronisation
        command_line    $USER1$/check_rabbitmq -H $HOSTADDRESS$ -u $USER2$ -p $USER3$ -c queue_synchronisation -v $ARG1$ -e $ARG2$
        }

# 'check_rabbitmq_queue_count' command definition
define command{
        command_name    check_rabbitmq_queue_count
        command_line    $USER1$/check_rabbitmq -H $HOSTADDRESS$ -u $USER2$ -p $USER3$ -c queue_count -v $ARG1$ -e $ARG2$
        }

# 'check_rabbitmq_queue_consumers' command definition
define command{
        command_name    check_rabbitmq_queue_consumers
        command_line    $USER1$/check_rabbitmq -H $HOSTADDRESS$ -u $USER2$ -p $USER3$ -c queue_consumers -v $ARG1$ -q $ARG2$
        }

# 'check_rabbitmq_queue_messages' command definition
define command{
        command_name    check_rabbitmq_queue_messages
        command_line    $USER1$/check_rabbitmq -H $HOSTADDRESS$ -u $USER2$ -p $USER3$ -c queue_messages -v $ARG1$ -e $ARG2$
        }
```


### Nagios Service Configuration Examples

```text
# Queue Synchronization Status - Check that queues are synchronised to at least one other node
define service{
        use                             generic-service
        name                            rabbitmq_queues_synchronised
        service_description             RabbitMQ Queue Synchronisation [/vhost1]
        check_command                   check_rabbitmq_queue_synchronisation!%2fvhost1!1
        host_name                       queue1, queue2
        }

# Ensure that the correct number of queues resides on each Rabbitmq server
define service{
        use                             generic-service
        name                            rabbitmq_queue_count
        service_description             RabbitMQ Queue Number [/vhost1]
        check_command                   check_rabbitmq_queue_count!%2fvhost1!3
        host_name                       queue1, queue2
        }

# Ensure that the correct number of consumers are attached to each queue
define service{
        use                             generic-service
        name                            rabbitmq_queue_consumers
        service_description             RabbitMQ Queue Consumers [/vhost1]
        check_command                   check_rabbitmq_queue_consumers!%2fvhost1!queue1:2,queue2:2,queue3:2
        host_name                       queue1, queue2
        }

# Ensure that the queues don't have messages (they should mostly be empty if we are being efficient)
define service{
        use                             generic-service
        name                            rabbitmq_queue_messages
        service_description             RabbitMQ Queue Messages [/vhost1]
        check_command                   check_rabbitmq_queue_messages!%2fvhost1!0
        host_name                       queue1, queue2
        }
```