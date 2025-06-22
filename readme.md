# Introduction

## Objective
The objective of this presentation is to showcase a set of modern tools that significantly accelerate the development and deployment of applications. By leveraging integrated development environments, automation, and container platforms, developers can streamline their workflows and deliver faster, more efficiently.

Demo App: Loty
To demonstrate these tools in action, we'll build a simple application called Loty - Linkarean of the year. The backend will be developed using Quarkus, a lightweight, Kubernetes-native Java framework—while the frontend will be created with Vue.js, a progressive JavaScript framework. This demo will focus on simplicity and speed to highlight the power of these tools.

## Presentation Structure – 4 Key Parts

**CRC Installation (OpenShift Local)**

We'll begin by setting up CRC (CodeReady Containers), a local OpenShift environment. This allows us to run a full OpenShift cluster on a developer machine, providing a realistic platform for testing and deploying applications.

**Project and Repository Setup**

Next, we'll create two GitHub repositories and initialize both projects structure, the Quarkus backend and Vue frontend. 

**Development with GitHub Copilot**

We'll then use GitHub Copilot, an AI pair programmer, to assist in writing code for both the frontend and backend. This step demonstrates how AI can accelerate development by suggesting boilerplate code, functions, and even entire logic blocks.

**Deployment Using OpenShift S2I (Source-to-Image)**

Finally, we’ll deploy the application using OpenShift’s S2I (Source-to-Image) process via the oc new-app command. This tool simplifies container image creation directly from source code, enabling fast, repeatable deployments.



<mark>=================== BY JP =====================</mark>

# 1. Install crc 
<mark>NOTA! START THE CRC BEFORE ANYTHING ELSE</mark>

## 1.1 Download the binaries and pull secret
Access https://console.redhat.com/openshift/create/local 
Note: You might have to register (free) for a redhat account, otherwise, log in with your redhat account

> Download what you need to get started 
> Choose: Linux!!! :) x86_64 and you have two buttons 
> Download Openshift Local  -> we will not do this in the presentation because it takes long 
> Download Pull Secret -> store it in file pull-secret.txt on the same path as the crc binary 

## 1.2 Unpack the binary 
tar -zxvf crc-linux-amd64.tgz
(check the proper name of your dir)
ln -s crc-linux-2.46.0-amd64 crc-linux


## 1.3 installing OS requirements
> sudo dnf install libvirt NetworkManager
(depending on your OS, you may need qemu-kvm if not already installed)

## 1.4 Avoid telemetry (optional - adjust for your own machine)
> ./crc-linux/crc config set consent-telemetry
> ./crc-linux/crc config set memory 20480
> ./crc-linux/crc config set cpus 6
> ./crc-linux/crc config set disk-size 200

(check with crc config view)
./crc-linux/crc config view
consent-telemetry                     : no
cpus                                  : 6
disk-size                             : 200
memory                                : 20480
(this is stored in ~/.crc/crc.json) 

## 1.5 check your firewall rules 
sudo iptables -L FORWARD -v
(and if they are closed... open!)
sudo iptables -P FORWARD ACCEPT

or if you use firewalld, much better this way (check your own NICs with sudo ip a or sudo ifconfig or nmcli dev and adapt accordingly): 
firewall-cmd --permanent --direct --add-rule ipv4 filter FORWARD 1 -i crc -j ACCEPT -m comment --comment "crc exiting subnet"
firewall-cmd --permanent --direct --add-rule ipv4 filter FORWARD 1 -o crc -j ACCEPT -m comment --comment "crc  subnet"
firewall-cmd --permanent --direct --add-rule ipv4 filter FORWARD 1 -o docker0 -j ACCEPT -m comment --comment "docker subnet"
firewall-cmd --permanent --direct --add-rule ipv4 filter FORWARD 1 -o vnet0 -j ACCEPT -m comment --comment "vnet0 exiting subnet"

## 1.6 setup and start
> ./crc-linux/crc setup 
(this command will take long if never installed befora - or if you delete the cache!)
(o bundle é guardado em: $HOME/.crc/cache)
(DON'T IGNORE ERRORS and FIX THEM IF THEY SHOW UP... We had issues with DNS resolve and firewalld as above stated)

> ./crc-linux/crc start --pull-secret-file pull-secret.txt --disable-update-check


At the end of this command, it will show you some info... like

'''bash
Started the OpenShift cluster.

The server is accessible via web console at:
  https://console-openshift-console.apps-crc.testing

Log in as administrator:
  Username: kubeadmin
  Password: EPgz4-bBjs5-PUgzI-GprNB

Log in as user:
  Username: developer
  Password: developer

Use the 'oc' command line interface:
  $ eval $(crc oc-env)
  $ oc login -u developer https://api.crc.testing:6443  
'''

So you can start by executing 
eval $(./crc-linux/crc oc-env)
oc  -u developer https://api.crc.testing:6443 (password is "developer")

Which will show you something like 

'''bash
Logged into "https://api.crc.testing:6443" as "developer" using existing credentials.

You don't have any projects. You can try to create a new project, by running

    oc new-project <projectname>
'''

If you need to destroy, use 
./crc-linux/crc delete (just deletes the cluster instance, does not cleanup your OS of other settings)
./crc-linux/crc cleanup (doesn't delete the cluster, but cleans up your OS settings)

<mark>both need to create the project</mark>

lets start to create the project mlga
>oc new project mlga

<mark>=================== by AL =====================</mark>

# 2 create apps 

Install quarkus cli first... : https://quarkus.io/guides/cli-tooling

## 2.1 create backend app (quarkus cli) 
quarkus create app com.linkare.loty:loty-be:1.0.0-SNAPSHOT --maven --java=17 --no-dockerfiles --package-name=com.linkare.loty.be -x quarkus-rest -x quarkus-jdbc-sqlite -x quarkus-smallrye-metrics -x quarkus-smallrye-health

Give it a spin 
./mvnw quarkus:dev 

http://localhost:8080/
check dev-ui and dashbard and other helping tools 

Give it a build 
./mvnw quarkus:build

Give it a run 
./mvnw quarkus:run

## 2.2 create frontend app (npm cli)
npm create vue loty-fe

nvm use node 20.10.0 (I had to switch to the latest node...)
npm install 

(Give it a spin)
npm dev

npm build 
npm preview 

# 3. create github projects (github cli)

## 3.1 create supporting structures
cd loty-be
git init
git add . 
git commit -m"initial backend impl" 
git push

cd loty-fe
git init 
git add .
git commit -m"initial frontend impl" 
git push

## 3.2 create github (public???) repositories 

Check https://github.com/cli/cli#installation and then do 
gh auth login (it will spawn a browser for auth)

cd ../loty-be
gh repo create -d "Linkarean Of The Year - Backend" --public --push -r loty-be -s .
cd ../loty-fe
gh repo create -d "Linkarean Of The Year - Frontend" --public --push -r loty-fe -s .
cd ../loty-be


Now that we’ve set up our GitHub repository and initialized both the Quarkus backend and Vue frontend projects, it's time to dive into the actual development.
To show how we can speed things up even more, I’ll hand it over again to José Pedro, who will demonstrate the usage of GitHub Copilot to rapidly build out both application with AI-assisted coding.

<mark>=================== by JP =====================</mark>

## 3.3 use vscode to implement CRUD on quarkus 
... Visual Studio code demo ... 


''' 
Create a rest CRUD endpoints on base path /api/v1/loty with quarkus-rest and over a database with sqlite. Loty entity has two attributes, name and year. Make sure the database gets initialized even if it doesn't exist at first access
'''

'''
please remove the demo content from this vue app

Please call the http://localhost:8080/api/v1/loty endpoint to get a list of Linkarean Of The Year in json format with id, name and year. For them, format a flowing set of cards with just the year as titlewith a bounding box in blue. Please make the cards are clickable to a new route /{year} where you invoke the http://localhost:8080/api/v1/loty/year/{year} endpoint and you format the year as title and name in regular text unless it is the currrent year where you should format the name in strong.


Please change the detail page to format the name in h1 but keep the rule with strong for the currrent year. If it is the current year, please add some twinkling effect as well on the detail page.


reduce the year from h1 to just a greyed out text and smaller letter and also make the data bounded by a nice rounded card with blue borders

do set the background of the card to transparent instead

do set the background of the card to transparent instead on the detail page as well

in dev mode I want to run against localhost:8080 fetch, but in prod mode I want to run against a base address of loty-be.apps-crc.testing. 
'''

=================== by AL =====================
TODO: explicar pre-baked app

# 4. deploy in crc (s2i)

## 4.1 deploy the backend 
Alright, now that the application is ready, let’s talk about deployment.

So, what do we usually need to do to deploy an app to Kubernetes or OpenShift?
Well… first, we’d need to containerize the app by writing a Dockerfile,
then build the image with docker build,
push it to a registry like Docker Hub or Quay,
then maybe write a Helm chart, configure deployments, services, routes…

Just kidding!

Thanks to OpenShift’s Source-to-Image (S2I) support, we can skip all of that and deploy directly from our source code with a single command:

oc new-app openshift/java:openjdk-17-ubi8~https://github.com/sunnymoon/loty-be --name=loty-be --labels=app=loty --strategy=source --context-dir=/

TODO: explain the options...
TODO: com base na image build X (what is an imagem build?)
TODO: The --strategy=source flag tells OpenShift to use the S2I build process, ....

It’s a fast and developer-friendly way to go from code to running app in minutes.

oc get builds 
oc get builds/loty-be-1
oc logs -f builds/loty-be-1 

oc expose service/loty-be --hostname=loty-be.apps-crc.testing

curl -s -k http://loty-be.apps-crc.testing/api/v1/loty | jq 

curl -s -k -v -X POST --data '{ "year": 1960, "name": "testing"}' -H "Content-type: application/json"  http://loty-be.apps-crc.testing/api/v1/loty 

curl -s -k http://loty-be.apps-crc.testing/api/v1/loty/year/1960 | jq 


## 4.2 deploy the frontend ... 

oc new-app openshift/nodejs:20-ubi9-minimal~https://github.com/sunnymoon/loty-fe --name=loty-fe --labels=app=loty --strategy=source --context-dir=/

oc get builds 
oc get builds/loty-fe-1
oc describe builds/loty-fe-1
oc logs -f builds/loty-fe-1 

oc expose service/loty-fe --hostname=loty.apps-crc.testing


curl -s -k -v -X PUT --data @2024-reset.json -H "Content-type: application/json"  http://loty-be.apps-crc.testing/api/v1/loty/16  --output /dev/null

curl -s -k -v -X PUT --data @2024.json -H "Content-type: application/json"  http://loty-be.apps-crc.testing/api/v1/loty/16  --output /dev/null


## Conclusion
What we’ve shown today is a simplified, streamlined path, an example of how fast you can go from source code to a running application using tools like OpenShift S2I, GitHub, CRC, and copilot.

While it’s not production-grade CI/CD, it’s a powerful way to prototype, test, or even run lightweight apps quickly in a real Kubernetes environment.
In other oportunities we will for sure talk about tools like Tekton, Argo CD, or Backstage. And the usage of hooks for instance...

One more important thing:
Everything we did today was from the command line, no clicks, no dashboards, no manual configs.

That’s not just for show, it means everything we’ve done is scriptable and easily automatable.

Whether it’s setting up the cluster, initializing the project, or building and deploying the apps, every step can be integrated into scripts, pipelines, to build developer portals.

It’s a great foundation for building toward real automation and CI/CD.


TODO! remove repos before presentation.
explain - everything from command line. close to automation. we can quicky automate everything like this