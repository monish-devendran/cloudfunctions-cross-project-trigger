

Step1:
Create a function in cf-source: attach a custom service account to cf. Deploy the cf in a vpc. Turn on serverless VPC access.
explore vpc connector.

Step2: 
Serverless VPC access:
projects/cf-source-416306/locations/us-central1/connectors/function-b-network

Step3:
attach the serverless VPC access to the cloud function.

Step4:
execute the function:

Step5:
Create a ilb for the cloud function, chosse internal loadbalancer.


Step6:
Testing connectivity to ilb via VM instance. In managed project

```
monishdev@testvm:~$ curl -m 70 -X POST http://10.12.0.2:80/function-b -H "Authorization: bearer $(gcloud auth print-identity-token)" -H "Content-Type: application/json" -d '{
  "name": "Hello World"
}'
Hello Hello World! (working with ilb)

```


Step7: test connectivity from trigger project via PSC enpoint ip

```
monishdev@trigger-project-vm-01:~$ curl -m 70 -X POST http://10.128.0.2:80/function-b -H "Authorization: bearer $(gcloud auth print-identity-token)" -H "Content-Type: application/json" -d '{
  "name": "Hello World"
}'
Hello Hello World! (working in trigger project)

monishdev@trigger-project-vm-01:~$
```



Step8: Create a Event ARC for pubsub.

And then run then publishing a message to test.

```
❯ REGION=us-central1


❯ gcloud eventarc triggers list --location=us-central1
NAME  TYPE                                           DESTINATION                                     ACTIVE  LOCATION
ad    google.cloud.pubsub.topic.v1.messagePublished  HTTP endpoint: http://10.128.0.2:80/function-b  Yes     us-central1


❯ MY_TOPIC=$(gcloud eventarc triggers describe ad \
    --location=$REGION \
    --format='value(transport.pubsub.topic)')


❯ echo $MY_TOPIC
projects/cf-triggers/topics/eventarc-us-central1-ad-155


❯ gcloud pubsub topics publish $MY_TOPIC --message "Sending Data from cf-trigger which is a project to another project called cf-source where the cloudfunction is hosted"
messageIds:
- '10662482951164189'
```

------------------------------------------------


login to cf-trigger project, Go to event arc.

Create a eventArc, And attach/create a service account to it. Provide permissions for the service account.

This trigger needs the role roles/eventarc.eventReceiver granted to service account cf-functionb-trigger@cf-triggers.iam.gserviceaccount.com to receive events via Google sources.
This trigger needs the role roles/pubsub.publisher granted to service account service-407580361807@gs-project-accounts.iam.gserviceaccount.com to receive events via Cloud Storage.

