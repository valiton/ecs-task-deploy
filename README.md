# ECS Task Deploy

A script to increment an active task definition on [ECS](https://aws.amazon.com/ecs) with an updated Docker image, followed by a service update to use it.

Originally forked from [`mikestead/ecs-task-deploy`](https://github.com/mikestead/ecs-task-deploy) and extended with
the following features:
- preserves `exetutionRoleArn`s in task definitions(was previously stripped out)
- allows updating stand-alone task-definitions without service

## What does it do?

Sequence of steps performed by the script:

1. Download the active task definition of an ECS service.
1. Clone it.
1. Given the Docker image name provided, find *any* containers in the task definition with references to it and replace them with the new one. Docker tags are ignored when searching for a match.
1. Register this new task definition with ECS.
1. Update the service to use this new task definition, triggering a blue/green deployment.

## How to use

```sh
npm i @valiton/ecs-task-deploy
```

### Environment Varaiables

For all the container definitions found when looking up your defined `image`, you have the option to add or update environment variables for these.

This can be done using the `--env` option.

    --env "SERVICE_URL=http://api.somedomain.com"

You can supply as many of these as required.

## Spare Capacity

In order to roll a blue/green deployment there must be spare capacity available to spin up a task based on your updated task definition.
If there's not capacity to do this your deployment will fail.

This can be done via ECS service `minimum healthy percent` but as a brute force option this tool supports `--kill-task`.
This will attempt to stop an existing task, making way for the blue/green rollout.

If you're only running a single task you'll experience some down time. **Use at your own risk.**

## Usage

    ecs-task-deploy [options]

    Options:

    -h, --help                output usage information
    -V, --version             output the version number
    -k, --aws-access-key <k>  aws access key, or via AWS_ACCESS_KEY_ID env variable
    -s, --aws-secret-key <k>  aws secret key, or via AWS_SECRET_ACCESS_KEY env variable
    -r, --region <r>          aws region, or via AWS_DEFAULT_REGION env variable.
    -c, --cluster <c>         ecs cluster, or via AWS_ECS_CLUSTER env variable
    -n, --service <n>         ecs service, or via AWS_ECS_SERVICE_NAME env variable
    -d, --task-def <d>        task definition name, only needed when service and cluster are not given. can be defined via AWS_ECS_TASK_DEF env variable
    -i, --image <i>           docker image for task definition, or via AWS_ECS_TASK_IMAGE env variable
    -t, --timeout <t>         max timeout (sec) for ECS service to launch new task, defaults to 90s
    -v, --verbose             enable verbose mode
    -e, --env <e>             environment variable in "<key>=<value>" format
    --kill-task               stop a running task to allow space for a rolling blue/green deployment

**Hint: You can pass a service OR a task definition name. If a service is passed
 that service will be updated, otherwise only the given task definition.**

### CLI

To run via cli.

    npm install -g ecs-task-deploy

```javascript
ecs-task-deploy \
    -k 'ABCD' \
    -s 'SECRET' \
    -r 'us-west-1' \
    -c 'qa' \
    -n 'website-service' \
    -i '44444444.ddd.ecr.us-east-1.amazonaws.com/website:1.0.2' \
    -e 'SERVICE_URL=http://api.somedomain.com' \
    -e 'API_KEY=clientapikey' \
    -v
```

To run in code.

```sh
npm install @valiton/ecs-task-deploy
```

### Node API

```javascript
const ecsTaskDeploy = require('@valiton/ecs-task-deploy');

ecsTaskDeploy({
  awsAccessKey: 'ABCD',
  awsSecretKey: 'SECRET',
  region: 'us-east-1',
  cluster: 'cache-cluster',
  service: 'cache-service',
  image: 'redis:2.8',
  env: {
    SERVICE_URL: 'http://api.somedomain.com',
    API_KEY: 'clientapikey'
  }
})
.then(
  newTask => console.info(`Task '${newTask.taskDefinitionArn}' created and deployed`),
  e => console.error(e)
)
```

