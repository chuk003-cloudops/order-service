# Order Service

The Order Service accepts order requests and publishes them to the durable RabbitMQ order_queue. For Lab 2 it runs on a dedicated Azure VM and connects to RabbitMQ on a separate VM.

## Configuration

Create an untracked .env file in this repository root:

RABBITMQ_CONNECTION_STRING=amqp://orderapp:URL_ENCODED_PASSWORD@<RABBITMQ-VM-PUBLIC-IP>:5672/
PORT=3000

Use the orderapp account created on the RabbitMQ VM, percent-encode special characters in its password, and replace the broker placeholder with the RabbitMQ VM public IP. Existing environment variables take precedence. Keep real credentials in .env only; .env.example contains placeholders. Restart the process after configuration changes.

## Install and run

On the order-service VM, install Node.js 24 LTS and npm. From the repository root, run npm ci, npm test, then node index.js. The service listens on all IPv4 interfaces at port 3000.

The order-service NSG should allow TCP 3000 only from the laptop public IP. The RabbitMQ NSG should allow TCP 5672 only from this VM public IP. Port 15672 is optional and should remain closed for this lab.

## Verify

Run npm test for the broker-independent suite. For a live check, POST this order to http://localhost:3000/orders from another terminal on the VM: {"product":{"id":1,"name":"Dog Food","price":19.99},"quantity":2,"totalPrice":39.98}. Expect HTTP 200 with Order received only after RabbitMQ confirms the persistent message.

On the RabbitMQ VM, run sudo rabbitmqctl list_queues name durable messages and verify order_queue is durable and its message count increased. This lab has no order-processing consumer, so accepted orders remain queued.
