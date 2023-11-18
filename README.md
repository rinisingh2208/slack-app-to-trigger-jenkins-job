# slack-app-to-trigger-jenkins-job
this repo contains jenkins and slack side configuration

**Slack App To Trigger Jenkins Job**

         From jenkins end changes

1) Make sure that IP is public facing otherwise slack slash command ( will discuss in further section ) will give invalid_url error.
  For example we made internal url : http://172.10.10.111:8080/, public url  http://60.70.125.11/



2) Add API token by going to user-> configure , token can be used later by slack slash command to save it


3) For jenkins job which you want to trigger via slack, add under Build Trigger section check on Trigger build remotely and add Authentication token of choice



4) For jenkins job to process the input sent by slack , we need to add inject environment variables under Build Environment and add text variable under parameterized string variable also which you can use inside your code for that particular input



        From slack end changes 

1) Can refer to app, by clicking apps in slack which will redirect you to man page, where you can select build and create app
https://app.slack.com/app-settings/

2) Set interactive payload request url with jenkins job with public url user authentication token and authentication token set at job level.

For example: http://admin:<authentication-token-generated-at-jenkins-admin>@<public-ip-set-for-job>/job/<job-name>/buildWithParameters?token=<token-generated-at-job-level>

Manifest will look as below for incoming webhook, set name in display information section and bot user display name:
_metadata:
    major_version: 1
    minor_version: 1
display_information:
    name: < app-name>
features:
    bot_user:
       display_name: <bot-name>
       always_online: false
          oauth_config:
    scopes:
      bot:
         - incoming-webhook
          settings:
  interactivity:
    is_enabled: true
    request_url: http://admin:<authentication-token-generated-at-jenkins-admin>@<public-ip-set-for-job>/job/<job-name>/buildWithParameters?token=<token-generated-at-job-level>
  org_deploy_enabled: false
  socket_mode_enabled: false
  token_rotation_enabled: false


2) Click on Install app under Distribution section and add channel where you want to add bot action, copy the webhook URL , same you can use to send notifications
          


Use Case:
Send user notification about their aws cluster which are in scale up state to ask them whether they want to retain the cluster or want to scale down in , in case of retain again user will be notified in next 2 hrs and in case of scale down option , scale down jenkins job will be triggered for cluster and user will be notified that there input is under process and earlier interactive notification will be modified so that for one notification there will be one input.
Solution:
We have created a job name as Slack Integrates on public facing jenkins which will be triggered every 2 hrs.
Functionality of Job:
i) In case of no user input provided in the last 2 hrs of last notification , auto scale down the cluster and that data will be taken from the csv file where we are storing cluster name, timestamp of last notification on channel and user input.
ii) It will send notification to owner for scaled up aws cluster and will store cluster name and notification timestamp to same csv file we discussed in functionality.

http://172.16.20.221:8080/job/Slack_integrates/

We have created a scale down job which is an asynchronous job and will be triggered only when the user taps into Scale Down. On the job end we are taking payload of response sent via slack having user input, add payload as part of string parameter to job also.
           Functionality of Job:
           i) Parse the slack payload and get cluster name, response url and action clicked by user
           ii) Send an acknowledgement message to the user about their request being processed and edit the original message too. Block below which doing the same in code is:

curl -X POST $response_url -H 'Content-Type: application/json' -d '{
    "replace_original": "true",
    "blocks": [
                      {
                          "type": "section",
                          "text": {
                              "type": "mrkdwn",
                              "text": "'"$message"'"
                        
                          }
                      }
              ]
}'

           iii) Store user input to the same csv file which can be used to check if there is no user input taken and auto scale down by Slack Integrates Job.
          iv) If action is scale Down trigger the scale down job logic.
        
   http://172.16.20.221:8080/job/EKS_Cluster_Scale_Down

Notes: 
To scale up the cluster use job: http://172.16.100.91:8080/job/EKS_Cluster_Scale_UP/

 Manual trigger of job by passing cluster name also possible via job : http://172.16.20.221:8080/job/EKS_Cluster_Scale_Down

We are storing scale up nodegroups and cluster count in 172.16.100.93 directory /var/lib/jenkins/workspace/EKS_Cluster_Scale_Down

 References:

https://www.excellarate.com/blogs/integrate-and-trigger-a-jenkins-job-from-slack/
https://stackoverflow.com/questions/59820320/error-in-triggering-jenkins-job-from-slack
https://api.slack.com/reference/surfaces/formatting#mentioning-users
https://api.slack.com/interactivity/handling#message_responses
https://api.slack.com/interactivity/handling
https://api.slack.com/interactivity/handling

Slack App To Trigger Jenkins Job

From jenkins end changes

Make sure that IP is public facing otherwise slack slash command ( will discuss in further section ) will give invalid_url error.
For example we made internal url : http://172.16.20.221:8080/, public url  http://64.74.135.12/
Second we disabled Allow anonymous read access as shown below in snap 1 with any project based authentication or any other as shown in Snap 2


Snap 1





Snap 2




Add API token by going to user-> configure , token can be used later by slack slash command to save it


For jenkins job which you want to trigger via slack, add under Build Trigger check on Trigger build remotely and add Authentication token of choice



For jenkins job to process the input sent by slack , we need to add inject environment variables under Build Environment and add text variable under parameterized string variable also which you can use inside your code for that particular input



From slack end changes 

Can refer to app created by us
https://app.slack.com/app-settings/T02TAQP5R/A02RMR4F954/general

Set interactive payload request url with jenkins job with public url user authentication token and authentication token set at job level.

For example: http://admin:1184ca0bba08d2b75965ebecdc69cec88f@64.74.135.12/job/EKS_Cluster_Scale_Down/buildWithParameters?token=453324678

Manifest will look as below for incoming webhook, set name in display information section and bot user display name:
           _metadata:
    major_version: 1
    minor_version: 1
display_information:
    name: aws-cluster-actions
features:
    bot_user:
       display_name: aws-cluster-actions
       always_online: false
          oauth_config:
    scopes:
      bot:
         - incoming-webhook
          settings:
  interactivity:
    is_enabled: true
    request_url: http://admin:1184ca0bba08d2b75965ebecdc69cec88f@64.74.135.12/job/EKS_Cluster_Scale_Down/buildWithParameters?token=453324678
  org_deploy_enabled: false
  socket_mode_enabled: false
  token_rotation_enabled: false

Click on Install app under Distribution section and add channel where you want to add bot action, copy the webhook URL , same you can use to send notifications
          

E2E Slack jenkins integration in exemplar use case
Use Case:
Send user notification about their aws cluster which are in scale up state to ask them whether they want to retain the cluster or want to scale down in , in case of retain again user will be notified in next 2 hrs and in case of scale down option , scale down jenkins job will be triggered for cluster and user will be notified that there input is under process and earlier interactive notification will be modified so that for one notification there will be one input.
Solution:
We have created a job name as Slack Integrates on public facing jenkins which will be triggered every 2 hrs.
Functionality of Job:
i) In case of no user input provided in the last 2 hrs of last notification , auto scale down the cluster and that data will be taken from the csv file where we are storing cluster name, timestamp of last notification on channel and user input.
ii) It will send notification to owner for scaled up aws cluster and will store cluster name and notification timestamp to same csv file we discussed in functionality.

http://172.16.20.221:8080/job/Slack_integrates/

We have created a scale down job which is an asynchronous job and will be triggered only when the user taps into Scale Down. On the job end we are taking payload of response sent via slack having user input, add payload as part of string parameter to job also.
           Functionality of Job:
           i) Parse the slack payload and get cluster name, response url and action clicked by user
           ii) Send an acknowledgement message to the user about their request being processed and edit the original message too. Block below which doing the same in code is:

curl -X POST $response_url -H 'Content-Type: application/json' -d '{
    "replace_original": "true",
    "blocks": [
                      {
                          "type": "section",
                          "text": {
                              "type": "mrkdwn",
                              "text": "'"$message"'"
                        
                          }
                      }
              ]
}'

           iii) Store user input to the same csv file which can be used to check if there is no user input taken and auto scale down by Slack Integrates Job.
          iv) If action is scale Down trigger the scale down job logic.
        
   http://172.16.20.221:8080/job/EKS_Cluster_Scale_Down

Notes: 
To scale up the cluster use job: http://172.16.100.91:8080/job/EKS_Cluster_Scale_UP/

 Manual trigger of job by passing cluster name also possible via job : http://172.16.20.221:8080/job/EKS_Cluster_Scale_Down

We are storing scale up nodegroups and cluster count in 172.16.100.93 directory /var/lib/jenkins/workspace/EKS_Cluster_Scale_Down

 References:

https://www.excellarate.com/blogs/integrate-and-trigger-a-jenkins-job-from-slack/
https://stackoverflow.com/questions/59820320/error-in-triggering-jenkins-job-from-slack
https://api.slack.com/reference/surfaces/formatting#mentioning-users
https://api.slack.com/interactivity/handling#message_responses
https://api.slack.com/interactivity/handling
https://api.slack.com/interactivity/handling



From jenkins end changes

Make sure that IP is public facing otherwise slack slash command ( will discuss in further section ) will give invalid_url error.
For example we made internal url : http://172.16.20.221:8080/, public url  http://64.74.135.12/
Second we disabled Allow anonymous read access as shown below in snap 1 with any project based authentication or any other as shown in Snap 2


Snap 1





Snap 2




Add API token by going to user-> configure , token can be used later by slack slash command to save it


For jenkins job which you want to trigger via slack, add under Build Trigger check on Trigger build remotely and add Authentication token of choice



For jenkins job to process the input sent by slack , we need to add inject environment variables under Build Environment and add text variable under parameterized string variable also which you can use inside your code for that particular input



From slack end changes 

Can refer to app created by us
https://app.slack.com/app-settings/T02TAQP5R/A02RMR4F954/general

Set interactive payload request url with jenkins job with public url user authentication token and authentication token set at job level.

For example: http://admin:1184ca0bba08d2b75965ebecdc69cec88f@64.74.135.12/job/EKS_Cluster_Scale_Down/buildWithParameters?token=453324678

Manifest will look as below for incoming webhook, set name in display information section and bot user display name:
           _metadata:
    major_version: 1
    minor_version: 1
display_information:
    name: aws-cluster-actions
features:
    bot_user:
       display_name: aws-cluster-actions
       always_online: false
          oauth_config:
    scopes:
      bot:
         - incoming-webhook
          settings:
  interactivity:
    is_enabled: true
    request_url: http://admin:1184ca0bba08d2b75965ebecdc69cec88f@64.74.135.12/job/EKS_Cluster_Scale_Down/buildWithParameters?token=453324678
  org_deploy_enabled: false
  socket_mode_enabled: false
  token_rotation_enabled: false

Click on Install app under Distribution section and add channel where you want to add bot action, copy the webhook URL , same you can use to send notifications
          

E2E Slack jenkins integration in exemplar use case
Use Case:
Send user notification about their aws cluster which are in scale up state to ask them whether they want to retain the cluster or want to scale down in , in case of retain again user will be notified in next 2 hrs and in case of scale down option , scale down jenkins job will be triggered for cluster and user will be notified that there input is under process and earlier interactive notification will be modified so that for one notification there will be one input.
Solution:
We have created a job name as Slack Integrates on public facing jenkins which will be triggered every 2 hrs.
Functionality of Job:
i) In case of no user input provided in the last 2 hrs of last notification , auto scale down the cluster and that data will be taken from the csv file where we are storing cluster name, timestamp of last notification on channel and user input.
ii) It will send notification to owner for scaled up aws cluster and will store cluster name and notification timestamp to same csv file we discussed in functionality.

http://172.16.20.221:8080/job/Slack_integrates/

We have created a scale down job which is an asynchronous job and will be triggered only when the user taps into Scale Down. On the job end we are taking payload of response sent via slack having user input, add payload as part of string parameter to job also.
           Functionality of Job:
           i) Parse the slack payload and get cluster name, response url and action clicked by user
           ii) Send an acknowledgement message to the user about their request being processed and edit the original message too. Block below which doing the same in code is:

curl -X POST $response_url -H 'Content-Type: application/json' -d '{
    "replace_original": "true",
    "blocks": [
                      {
                          "type": "section",
                          "text": {
                              "type": "mrkdwn",
                              "text": "'"$message"'"
                        
                          }
                      }
              ]
}'

           iii) Store user input to the same csv file which can be used to check if there is no user input taken and auto scale down by Slack Integrates Job.
          iv) If action is scale Down trigger the scale down job logic.
        
   http://172.16.20.221:8080/job/EKS_Cluster_Scale_Down

Notes: 
To scale up the cluster use job: http://172.16.100.91:8080/job/EKS_Cluster_Scale_UP/

 Manual trigger of job by passing cluster name also possible via job : http://172.16.20.221:8080/job/EKS_Cluster_Scale_Down

We are storing scale up nodegroups and cluster count in 172.16.100.93 directory /var/lib/jenkins/workspace/EKS_Cluster_Scale_Down

 References:

https://www.excellarate.com/blogs/integrate-and-trigger-a-jenkins-job-from-slack/
https://stackoverflow.com/questions/59820320/error-in-triggering-jenkins-job-from-slack
https://api.slack.com/reference/surfaces/formatting#mentioning-users
https://api.slack.com/interactivity/handling#message_responses
https://api.slack.com/interactivity/handling
https://api.slack.com/interactivity/handling


