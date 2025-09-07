- Loom Video
  - https://www.loom.com/share/52be1eb096cc44abbaeac0e53670c1c2?sid=e4bbe42e-71b3-4f81-b35d-340e3b907e60
- APIs
  - api/v1/sendNotification
  - api/notificaton/status?id={transaction_id}s
  - api/user/preferences?user_id={user_1}

- Architecture diagrams
  - Foboh notification system architecture diagram
  - Excalidraw.svg
  - Database.png

- Improvements:
  - support multiple user_ids in API
  - Set max limits for each type of notification per day. email = 3, sms = 2, push = 5.
  - Templates can be created and stored in our system at which point the APIs will support the template ids instead of the template texts.
  - Have custom limit per client.

  - Alternative approach for queuing will be to use a pub-sub kafka model where we publish to a single topic and all subscribers (email, sms, push) will 
 consume this message and do the processing.

- Rate limiting options:
  - Build our own
  - Apigee, AWS API Gateway
  - Process from EC2 request per user/ according to the timing like donot disturb.


- Monitoring and alerting system
  - Message in slack channel or team chat
  - New relic/ data dog alerts, AWS sentry alerts, cloudwatch


- Scaling
  - Autoscaling ec2 instances, increasing worker pool.
  - Containerise kubernitise 
  - ECS fargate is one more option for workers
  - Pub sub model - kafka


- Coding:
  - Unit tests
  - Independently testable components
  - Independent Deployments
  - Versioning, backward compatible with other microservies

- Examples:
api/v1/sendNotification
{
  "userId": "user123", //required

  //required
  "channels" : [
    
    //optional
    "email": {
      "subject": "Test subject",
      "body": "Hello {{first.name}} {{last.name}}, This is a test email body"
    },

    //optional
    "sms": {
      "body": "Hi {{first.name}}, This is a test sms"
    },

    //optional
    "push": {
      "Test push notification"
    }
  ],
  "type": "marketing" //optional
}


response:
//on success
{
  "status": "received",
  "transaction_id": "d3e2ec54-dc12-41f8-aaa5-bf4500a711f9" // string uuid
}

//on failure
{
  "error": true,
  "status": "rate_limit" //invalid_user, opt_out, 
}



//API to get the status of the notificaion
//GET
api/v1/notificaton/status?id={transaction_id}
response
{
  "email": "done",
  "sms": "failed",
  "push": null
}
//other status: null, opt_out, ready, queued, retry, failed, done


//response on failure
invalid header respoinse
{
    "error": "invalid or expired transaction_id"
}




//API to get user preferences:
//GET
header token
api/user/preferences?user_id=user_1

// success response:
{
  "email": true,
  "sms": false,
  "push": true,
  "marketing": false,
  "reminders": true,
  "notifications": true
}


//failre response
{
  "error": "invalid user",
}

