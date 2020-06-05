# IBM Multicloud Manager Tutorial

This is a tutorial which explains how to use MCM and how to :
- deploy a simple application to a managed cluster
- Modify some parameters in the fly
- deploy a simple policy by using the UI
- deploy a policy with the command line. 

NB : if you use RedHat Advanced Cluster Manager, you will need to modify the api versions in the yaml files. 



## The application

A simple guestbook. You type your text and it is displayed and record. 
[Here is a printscreen of the application](https://github.com/rodolphe-fontaine/guestbook_mcm/blob/master/images/guestbook.png)
<img src="https://raw.githubusercontent.com/rodolphe-fontaine/guestbook_mcm/master/images/guestbook.png"></img> 
## The architecture

The application has been a little bit complexified by placing the frontend and the backend in 2 differents namespaces to properly illustrate the MCM capabilities
[Here is the architecture schema](https://github.com/rodolphe-fontaine/guestbook_mcm/blob/master/images/archi.png)
<img src="https://raw.githubusercontent.com/rodolphe-fontaine/guestbook_mcm/master/images/archi.png"></img> 

## The scenario

 1. The guestbook application is deployed on managed cluster. The managed cluster has to be tagged as a "prod" cluster.
 2. The communication between the namespace Team1 & Team2 is blocked by using a rule
 3. Only the Redis port is opened. 


## Let's go and do it

### Deploy the channel, subscriptions, placementrule

    oc apply -f channel.yaml
    oc apply -f application.yaml
    oc apply -f subscription-frontend.yaml
    oc apply -f subscription-backend.yaml
    oc apply -f placement-rules.yaml

Your subscription is created. It is github channel. All the yaml files are available to make the application works. 

Let's have a look at the subscription-frontend.yaml file : 
<img src="https://raw.githubusercontent.com/rodolphe-fontaine/guestbook_mcm/master/images/subscription.png"></img> 

Notice 2 things : 

 1. The `packageFilter` section : only the yaml file with the `annotation` tier: frontend will be deployed by this subscription
 2. The `clusterOverrides`section will override values when deploying yaml file in the managed cluster. In this case, the yaml files will be created in a namespace called `team1`

 You are ready to test your application. 

### Let's secure the communication between namespaces.
It is a best practice to close all the communication between to namespaces. 
<img src="https://raw.githubusercontent.com/rodolphe-fontaine/guestbook_mcm/master/images/denyall.png"></img> 

Let's do it with the UI
<img src="https://raw.githubusercontent.com/rodolphe-fontaine/guestbook_mcm/master/images/createdenyall.png"></img> 

You can notice if you test your application, that when you type text, it is not anymore displayed. The communication with your backend does not work anymore. 

### Let's open the port 6379

We will create a network policy on your target cluster to allow the communication between the frontend and the backend. 
<img src="https://raw.githubusercontent.com/rodolphe-fontaine/guestbook_mcm/master/images/allow6379.png"></img>

    oc apply -f ./policy/policy-networkpolicy-gb.yaml

The port is open and your application works fine.

Connect to the managed cluster and try to delete the network policy :

    oc delete from-backend-to-frontend -n team2
Notice that it is impossible to delete it. MCM is monitoring your cluster and keep consistency regarding the governance you have configured in your hub. 







