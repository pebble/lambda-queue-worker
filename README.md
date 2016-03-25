# lambda-queue-worker

This is a template that can be used to create an infinitely scalable SQS queue
worker! It is expecting you to use NodeJS. The index.js currently only logs
outputs.

The dockers created with the files found in this directory allow you to easily
create lambda queue workers. This gives you infinite scalability and lower
costs than having to constantly be running machines that may or may not be
handling your queue at times.

## Usage

Please note that you will have to launch both of the Dockers on one instance,
and then keep that instance running to make sure the queue items reach the 
lambda functions.

Make sure to set and pass all the environment variables found in `.env`.

Create the `Dockerfile.lambda` first, so you have a function deployed
already. Please note that initially on cloning this repo you will only
have an `index.js` which simply logs out what it receives, you will have to
add your business logic from here.

Next up, the `Dockerfile`, this facilitates the notifications coming in on the
SQS queue so it makes sure they land at the right lambda function. The instance
that runs this continuous docker service can be very low, as it is just a pass-
through until AWS officially supports SQS as an event source.

You now have a service that can instantly handle the queue and infinitely scale.

### fleet Service files

Since we are already building the main Dockerfile, you only have to build the 
`Dockerfile.lambda`.

`lambda-queue-worker@.service`:

```
[Unit]
Description=lambda-queue-worker

[Service]
User=core
TimeoutStartSec=0
Restart=always
ExecStartPre=-/usr/bin/docker pull your-registry/your-lambda-function
ExecStartPre=-/usr/bin/docker pull pebbletech/lambda-queue-worker
ExecStartPre=-/usr/bin/docker kill sqs-to-lambda
ExecStartPre=-/usr/bin/docker rm sqs-to-lambda
ExecStartPre=-/bin/sh -c "/usr/bin/docker run --rm --name your-lambda-function --env-file=.env your-registry/your-lambda-function"
EnvironmentFile=/etc/environment
ExecStart=/bin/sh -c "/usr/bin/docker run --rm --name sqs-to-lambda --env-file=.env pebbletech/lambda-queue-worker:latest"
ExecStop=/usr/bin/docker stop sqs-to-lambda
```

### Development testing

Install `npm install -g git+https://git@github.com/deviavir/node-lambda#vpc-config-support`
locally, you can then adjust the `event.json` with how you expect your event to
arrive at your Lambda function.

You can test your function by running `node-lambda run` in your project
directory.

## Cost

Depending on the amount of time your finished lambda function needs to process
your queue items, you can make the following assumptions:

```
20.000.000 SQS items
128 MB memory
3 seconds execution time

$3.80 / month
```

```
200.000.000 SQS items
128 MB memory
3 seconds execution time

$39.80 / month
```

```
250.000.000 SQS items
128 MB memory
3 seconds execution time

$49.80 / month
```

```
250.000.000 SQS items
512 MB memory
10 seconds execution time

$63.97 / month
```

Feel free to calculate for your own use case, Matthew D Fuller created a
cost calculcator: http://blog.matthewdfuller.com/p/aws-lambda-pricing-calculator.html