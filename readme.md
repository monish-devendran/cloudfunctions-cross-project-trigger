
Here we are going to create a function in project cf-source and a trigger project in cf-trigger


Permissions for trigger:

Pubsub:
Service account(s) might not have enough permissions to deploy the function with selected trigger. To mitigate potential errors, you can grant them the following roles:
roles/iam.serviceAccountTokenCreator to service-1031278184125@gcp-sa-pubsub.iam.gserviceaccount.com


![EVENT ARC](./img/eventarc_api.jpg)



DEMO STARTS HERE, But its for Gen 1 only. Gen2 work is inprogress.

cmd:

gcloud functions deploy my_pubsub_function --region=us-east1 --runtime="python311" --trigger-topic projects/cf-triggers/topics/send-to-project-cf-source --entry-point=hello_pubsub --source="gs://cf-source-416306/pubsub_trigger/function-source.zip"


Project A: cf-source -> project used to host cloud function
Project B: cf-trigger -> project hosts triggers


Project A ( cf-source : https://us-central1-cf-source-416306.cloudfunctions.net/FunctionA)
Add Project B svc account to Project A

switch to Project B (cf-triggers)

create a topic ( projects/cf-triggers/topics/send-to-project-cf-source)

gsutil notification create -t projects/cf-triggers/topics/send-to-project-cf-source -f json gs://cloudfunction-trigger/
>Created notif  ication config projects/_/buckets/cloudfunction-trigger/notificationConfigs/1

create a subcription (projects/cf-triggers/subscriptions/trigger-subscription)
set the push target at https://us-central1-cf-source-416306.cloudfunctions.net/FunctionA
enable authentication and set it a service account that you added to project A (cf-triggers)

Make sure you provide a service account for that function alone, use 
```
❯ gcloud functions add-iam-policy-binding  function-b --region=us-central1 --member=serviceAccount:cf-functionb-trigger@cf-triggers.iam.gserviceaccount.com --role=roles/cloudfunctions.invoker


WARNING: The role [roles/cloudfunctions.invoker] was successfully bound to member [serviceAccount:cf-functionb-trigger@cf-triggers.iam.gserviceaccount.com] but this does not grant the member permission to invoke 2nd gen function [function-b]. Instead, the role [roles/run.invoker] must be granted on the underlying Cloud Run service. This can be done by running the `gcloud functions add-invoker-policy-binding` command.

Would you like to run this command and additionally grant [serviceAccount:cf-functionb-trigger@cf-triggers.iam.gserviceaccount.com] permission to invoke function [function-b] (Y/n)?  y

The role [roles/run.invoker] was successfully bound to the underlying Cloud Run service. You can view its IAM policy by running:
gcloud run services get-iam-policy projects/cf-source-416306/locations/us-central1/services/function-b

bindings:
- members:
  - serviceAccount:cf-functionb-trigger@cf-triggers.iam.gserviceaccount.com
  role: roles/cloudfunctions.invoker
etag: BwYTjpJjfmM=
version: 1
```



Sending a file to project B (cf-triggers bucket)
❯ gsutil cp ./temp  gs://cloudfunction-trigger/
Copying file://./temp [Content-Type=application/octet-stream]...
/ [1 files][   16.0 B/   16.0 B]                                                
Operation completed over 1 objects/16.0 B.  


Check logs in project A (cf-trigger):

![Logs](image.png)




---


❯ gcloud iam service-accounts add-iam-policy-binding svc-foo-cloudfunction@cf-triggers.iam.gserviceaccount.com --member user:monish.devendran@gmail.com --role roles/iam.serviceAccountUser
Updated IAM policy for serviceAccount [svc-foo-cloudfunction@cf-triggers.iam.gserviceaccount.com].
bindings:
- members:
  - serviceAccount:svc-foo-cloudfunction@cf-triggers.iam.gserviceaccount.com
  - user:monish.devendran@gmail.com
  role: roles/iam.serviceAccountUser
etag: BwYTMRkOnOo=
version: 1


---



cross project : servicAccount https://stackoverflow.com/questions/71340529/adding-cross-project-service-account-in-gcp-cloud-functions

