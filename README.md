# Order Service

The Order Service accepts order requests and publishes them to the durable RabbitMQ `order_queue`. Lab 2 used a dedicated Azure VM. Lab 3 runs this Node.js service on Azure App Service and keeps RabbitMQ on its dedicated Azure VM.

## Configuration

Create an untracked .env file in this repository root:

RABBITMQ_CONNECTION_STRING=amqp://orderapp:URL_ENCODED_PASSWORD@RABBITMQ_VM_PUBLIC_IP:5672/
PORT=3000

Use the orderapp account created on the RabbitMQ VM, percent-encode special characters in its password, and replace RABBITMQ_VM_PUBLIC_IP with the RabbitMQ VM's public IP. Existing environment variables take precedence. Keep real credentials in .env only; .env.example contains placeholders. Restart the process after configuration changes.

## Install and run

Use Node.js 22 LTS for the Lab 3 App Service runtime. From the repository root, run `npm ci`, `node --test`, then `node index.js`. Locally the service listens on port 3000. On App Service it reads the platform-provided `PORT`.

The `package.json` test script matches the instructor's Lab 3 manifest. The behavioral tests remain in `test/` and run separately with `node --test` (17 tests).

For App Service, configure `RABBITMQ_CONNECTION_STRING` in the application's environment settings and set the startup command to `node index.js`. Keep the connection credential out of GitHub. The deployed endpoint is `https://chuk8915order.azurewebsites.net/orders`.

For Lab 2, the order-service VM NSG allowed TCP 3000 from the laptop and RabbitMQ allowed TCP 5672 from the order VM. For Lab 3, RabbitMQ permits TCP 5672 from the order App Service's documented outbound addresses. Its management interface is accessed through a laptop SSH tunnel for the demonstration.

## Verify

Run `node --test` for the broker-independent suite. For a live check, POST this order to `http://localhost:3000/orders` or the deployed endpoint: `{"product":{"id":1,"name":"Dog Food","price":19.99},"quantity":2,"totalPrice":39.98}`. Expect HTTP 200 with `Order received` only after RabbitMQ confirms the persistent message.

On the RabbitMQ VM, run sudo rabbitmqctl list_queues name durable messages and verify order_queue is durable and its message count increased. This lab has no order-processing consumer, so accepted orders remain queued.
