### Adding Function subscribers

Add subscribers after the topic has been created.

```js {5}
const topic = new Topic(this, "Topic", {
  subscribers: ["src/subscriber1.main", "src/subscriber2.main"],
});

topic.addSubscribers(this, ["src/subscriber3.main"]);
```

### Lazily adding Function subscribers

Create an _empty_ topic and lazily add the subscribers.

```js {3}
const topic = new Topic(this, "Topic");

topic.addSubscribers(this, ["src/subscriber1.main", "src/subscriber2.main"]);
```

### Configuring Function subscribers

#### Specifying function props for all the subscribers

You can extend the minimal config, to set some function props and have them apply to all the subscribers.

```js {2-6}
new Topic(this, "Topic", {
  defaults: {
    function: {
      timeout: 20,
      environment: { tableName: table.tableName },
      permissions: [table],
    },
  },
  subscribers: ["src/subscriber1.main", "src/subscriber2.main"],
});
```

#### Using the full config

Configure each Lambda function separately.

```js
new Topic(this, "Topic", {
  subscribers: [{
    function: {
      srcPath: "src/",
      handler: "subscriber1.main",
      environment: { tableName: table.tableName },
      permissions: [table],
    },
  }],
});
```

Note that, you can set the `defaultFunctionProps` while using the `function` per subscriber. The `function` will just override the `defaultFunctionProps`. Except for the `environment`, the `layers`, and the `permissions` properties, that will be merged.

```js
new Topic(this, "Topic", {
  defaults: {
    function: {
      timeout: 20,
      environment: { tableName: table.tableName },
      permissions: [table],
    },
  },
  subscribers: [
    {
      function: {
        handler: "subscriber1.main",
        timeout: 10,
        environment: { bucketName: bucket.bucketName },
        permissions: [bucket],
      },
    },
    "subscriber2.main",
  ],
});
```

So in the above example, the `subscriber1` function doesn't use the `timeout` that is set in the `defaultFunctionProps`. It'll instead use the one that is defined in the function definition (`10 seconds`). And the function will have both the `tableName` and the `bucketName` environment variables set; as well as permissions to both the `table` and the `bucket`.

#### Giving the subscribers some permissions

Allow the subscriber functions to access S3.

```js {5}
const topic = new Topic(this, "Topic", {
  subscribers: ["src/subscriber1.main", "src/subscriber2.main"],
});

topic.attachPermissions(["s3"]);
```

#### Giving a specific subscriber some permissions

Allow the first subscriber function to access S3.

```js {5}
const topic = new Topic(this, "Topic", {
  subscribers: ["src/subscriber1.main", "src/subscriber2.main"],
});

topic.attachPermissionsToSubscriber(0, ["s3"]);
```

#### Configuring the subscription

Configure the internally created CDK `Subscription`.

```js {8-14}
import { SubscriptionFilter } from "aws-cdk-lib/aws-sns";

new Topic(this, "Topic", {
  subscribers: [
    {
      function: "src/subscriber1.main",
      cdk: {
        subscription: {
          filterPolicy: {
            color: SubscriptionFilter.stringFilter({
              allowlist: ["red"],
            }),
          },
        },
      },
    },
  ],
});
```

### Configuring Queue subscribers

#### Specifying the Queue directly

You can directly pass in an instance of the Queue construct.

```js {4}
const myQueue = new Queue(this, "MyQueue");

new Topic(this, "Topic", {
  subscribers: [myQueue],
});
```

#### Configuring the subscription

Configure the internally created CDK `Subscription`.

```js {10-16}
import { SubscriptionFilter } from "aws-cdk-lib/aws-sns";

const myQueue = new Queue(this, "MyQueue");

new Topic(this, "Topic", {
  subscribers: [
    {
      queue: myQueue,
      cdk: {
        subscription: {
          filterPolicy: {
            color: SubscriptionFilter.stringFilter({
              allowlist: ["red"],
            }),
          },
        },
      },
    },
  ],
});
```

### Creating a FIFO topic

```js {4-6}
new Topic(this, "Topic", {
  subscribers: ["src/subscriber1.main", "src/subscriber2.main"],
  cdk: {
    topic: {
      fifo: true,
    },
  },
});
```

### Configuring the SNS topic

Configure the internally created CDK `Topic` instance.

```js {4-6}
new Topic(this, "Topic", {
  subscribers: ["src/subscriber1.main", "src/subscriber2.main"],
  cdk: {
    topic: {
      topicName: "my-topic",
    },
  },
});
```

### Importing an existing topic

Override the internally created CDK `Topic` instance.

```js {6}
import * as sns from "aws-cdk-lib/aws-sns";

new Topic(this, "Topic", {
  subscribers: ["src/subscriber1.main", "src/subscriber2.main"],
  cdk: {
    topic: sns.Topic.fromTopicArn(this, "MySnsTopic", topicArn),
  },
});
```