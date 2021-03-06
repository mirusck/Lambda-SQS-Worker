# Lambda SQS Worker

This is a simple library for creating SQS Lambda workers that can be invoked by the SQS event mapping. By default, AWS invokes SQS worker Lambdas in batches (default of 10) and if one fails and returns an error, then all messages are marked as incomplete and will be retried. This library not only handles these messages concurrently, but removes the messages from the SQS queue when they are completed. This means that only failed messages will be retried.

## Usage

Install with `go get github.com/helpfulhuman/lambda-sqs-worker`

```go
package main

import (
  "github.com/aws/aws-lambda-go/events"
  "github.com/aws/aws-lambda-go/lambda"
  "github.com/aws/aws-sdk-go/aws/session"
  "github.com/aws/aws-sdk-go/service/sqs"
  sqsworker "github.com/helpfulhuman/lambda-sqs-worker"
)

func main() {
  // create a new AWS client session
  aws := session.New()

  // create an SQS client using our AWS client session
  sqsClient := sqs.New(aws)

  // pass any external services to use in your handler
  services := make(map[string]interface{})

  // for example, you can set initialize your database session
  // outside of message processor and pass it to the services map
  services["dbSess"] = *sess // mongodb session

  // create your Lambda handler
  worker := sqsworker.NewHandler(sqsClient, HandleMessage, services)

  // start Lambda using your new worker handler
  lambda.Start(worker.Handle)
}

func HandleMessage(ctx context.Context, message events.SQSMessage, h *sqsworker.Handler) error {
  // process the SQS message

  if dbSess, ok := h.Services["dbSess"].(mgo.Session); ok {
    // use your DB session here
  }

  return nil
}
```
