# Pre presentation cleanup
Don't forget to:
1. start crc (AL beforehand, JP at presentation start)
2. remove existing github repos that we are going to create during presentation (AL only)
3. remove services, builds, projects, every openshift object that might exist
1. check crc config...

# Introduction

## Objective
The objective of this presentation is to showcase a set of modern tools that significantly accelerate the development and deployment of applications. By leveraging integrated development environments, automation, and container platforms, developers can streamline their workflows and deliver faster, more efficiently.

Demo App: Loty
To demonstrate these tools in action, we'll build a simple application called Loty - Linkarean of the year. The backend will be developed using Quarkus, a lightweight, Kubernetes-native Java framework, while the frontend will be created with Vue, a progressive JavaScript framework. This demo will focus on simplicity and speed to highlight the power of these tools.

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
> ./crc-linux/crc config set consent-telemetry no
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
  Password: efI6e-Wyzg6-RUJPm-VSqSa

Log in as user:
  Username: developer
  Password: developer

Use the 'oc' command line interface:
  $ eval $(crc oc-env)
  $ oc login -u developer https://api.crc.testing:6443
'''

So you can start by executing 
eval $(./crc-linux/crc oc-env)
oc login -u developer https://api.crc.testing:6443 (password is "developer")

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
>oc new-project mlga

<mark>=================== by AL =====================</mark>

# 2 create apps 

We start by installing the Quarkus CLI, which makes it easier to scaffold and manage Quarkus applications. You can follow the official guide based on your OS.
https://quarkus.io/guides/cli-tooling


## 2.1 create backend app (quarkus cli) 
Now we’ll scaffold the backend app using the Quarkus CLI. I’m choosing Java 17, SQLite for simplicity, and adding extensions for REST, metrics, and health endpoints. 
The app is created with Maven as the build tool, and we also specify the package
We specifically specify that we don't want dockerfiles since we are going to take care of that later.

> quarkus create app com.linkare.loty:loty-be:1.0.0-SNAPSHOT --maven --java=17 --no-dockerfiles --package-name=com.linkare.loty.be -x quarkus-rest -x quarkus-jdbc-sqlite -x quarkus-smallrye-metrics -x quarkus-smallrye-health
> cd loty-be

Let’s spin up the backend in development mode.
> ./mvnw quarkus:dev 

http://localhost:8080/
This is the Dev UI, which helps us explore our endpoints, metrics, health checks, and more.

Once we're happy with development, we build the app...
> ./mvnw quarkus:build

and run it in production mode...
> ./mvnw quarkus:run

## 2.2 create frontend app (npm cli)
Now let’s create the frontend using Vue. This will scaffold a Vue app with Vite.

> npm create vue@latest loty-fe

> cd loty-be

> nvm use node 20.10.0 (I had to switch to the latest node...)

We install the dependencies...
> npm install 

 and run the dev server...
> npm run dev

Let’s simulate a production build
> npm run build 

and preview it locally
> npm run preview 

# 3. create github projects (github cli)

## 3.1 create supporting structures

Now we initialize a Git repo for the backend and commit the files
This helps with collaboration and version control.

> cd loty-be
> git init
> git add . 
> git commit -m"initial backend impl" 

We do the same for the frontend.
> cd loty-fe
> git init 
> git add .
> git commit -m"initial frontend impl" 

## 3.2 create github (public???) repositories 
the next step is to make our projects collaborative and that means pushing them to some source code management, in our case we are goint to use GitHub.

Instead of manually creating the repositories in the GitHub web interface, we’re using the GitHub CLI.

The gh CLI lets us manage repositories, issues, pull requests, and more, right from the terminal, which is great for automation and scripting.

Check https://github.com/cli/cli#installation 
The first time you need to authenticate with: 
gh auth login (it will spawn a browser for auth). I'm already authenticated so i don't need to do it now

So, we navigate into each project directory
> cd ../loty-be

then run gh repo create with the --push and --source options.
> gh repo create -d "Linkarean Of The Year - Backend" --public --push --remote loty-be --source .
 
This creates the repo remotely on GitHub, links it to our local directory, and pushes the code, all in one step.
We’re keeping the repo public for simplicity, so we don’t have to deal with secret management or authentication setup right now.

We do the same for the frontend
cd ../loty-fe
gh repo create -d "Linkarean Of The Year - Frontend" --public --push --remote loty-fe --source .
cd ../loty-be

To recap, we’ve created two separate applications:
A Quarkus backend with REST endpoints, health checks, and an embedded SQLite database.
A Vue frontend built with Vite.

Both apps are now version-controlled and hosted on GitHub.

Now that the repositories are initialized and both projects are scaffolded, it’s time to jump into actual development.

To show how we can accelerate that process even further, I’ll hand it over back to José Pedro, who will demonstrate how to use GitHub Copilot to rapidly build out both applications with AI-assisted coding.

<mark>=================== by JP =====================</mark>

## 3.3 use vscode to implement CRUD on quarkus and UI 
... Visual Studio code demo ... 


''' 
Create a rest CRUD endpoints on base path /api/v1/loty with quarkus-rest and over a database with sqlite. Loty entity has two attributes, name and year. Make sure the database gets initialized even if it doesn't exist at first access
'''

'''
please remove the demo content from this vue app

Please call the http://localhost:8080/api/v1/loty endpoint to get a list of Linkarean Of The Year in json format with id, name and year. For them, format a flowing set of cards with just the year as title with a bounding box in blue. Please make the cards clickable to a new route /{year} where you invoke the http://localhost:8080/api/v1/loty/year/{year} endpoint and you format the year as title and name in regular text unless it is the currrent year where you should format the name in strong.


Please change the detail page to format the name in h1 but keep the rule with strong for the currrent year. If it is the current year, please add some twinkling effect as well on the detail page.


reduce the year from h1 to just a greyed out text and smaller letter and also make the data bounded by a nice rounded card with blue borders

do set the background of the card to transparent instead

do set the background of the card to transparent instead on the detail page as well

in dev mode I want to run against localhost:8080 fetch, but in prod mode I want to run against a base address of loty-be.apps-crc.testing. 
'''

=================== by AL =====================
Because live builds and live demos don't always play nice, we're taking a page from cooking shows and using a pre-baked app, one we've already prepared earlier and pushed to GitHub.

This version was built using exactly the same steps we’re showing you here today. While some of the AI-generated code suggestions may vary slightly from one session to another, the overall structure and functionality are fully reproducible by following the same process.

# 4. deploy in crc (s2i)

## 4.1 deploy the backend 
Alright, now that the application is ready, let’s talk about deployment.

So, what do we usually need to do to deploy an app to Kubernetes or OpenShift in our case today?
Well… first, we’d need to containerize the app by writing a Dockerfile or Containerfile 
Then, we’d build the container image using a tool like docker build or podman build.
push it to a registry like Docker Hub or Quay,
Finally, we configure deployment manifests, services, and ingress rules, maybe even write a Helm chart to manage these resources.

That sounds like a lot of work... Thanks to OpenShift’s Source-to-Image (S2I) support, we can skip all of that. OpenShift takes our source code, uses a builder image, builds and deploys everything automatically, all with a single command:

> oc new-app openshift/java:openjdk-17-ubi8~https://github.com/sunnymoon/loty-be --name=loty-be --labels=app=loty --strategy=source --context-dir=/

We're using oc new-app to kick off a new build and deployment in OpenShift. This command uses the OpenShift-provided Java 17 image as the build image, pulls code from a public GitHub repo, and builds the app using the Source-to-Image strategy. We give the app a name, apply helpful labels, and define where in the repository the source code lives.

It’s a fast and developer-friendly way to go from code to a running app in minutes.

> oc get builds 
> oc get builds/loty-be-1
> oc logs -f builds/loty-be-1 

This method:
Automatically builds the app using the selected builder image (java:openjdk-17-ubi8).
Deploys the built image into a pod.
Creates the necessary OpenShift resources: BuildConfig, DeploymentConfig, Service, etc.
Avoids manual containerization, while still allowing you to customize the S2I process if needed.

lets see what we got in our openshift
> oc get all

Our service is running and available internally within OpenShift — that means other apps and components inside the cluster can talk to it just fine.
But to access it from outside the cluster, like from our browser or external users, we need to expose it externally.
For that, OpenShift provides a resource called a Route, which creates an external URL and handles the traffic routing to our service.
We can create this easily with the oc expose command.
> oc expose service/loty-be --hostname=loty-be.apps-crc.testing

If we didn’t specify a hostname, OpenShift would automatically generate a sensible default hostname based on the service name, project namespace, and cluster domain, in this case would be something like loty-be-mlga.apps-crc.testing.

Let’s test our app by fetching all existing entries.
> curl -s -k http://loty-be.apps-crc.testing/api/v1/loty | jq 

Now, let’s add a new entry with year 1960 and name "testing".
> curl -s -k -v -X POST --data '{ "year": 1960, "name": "testing"}' -H "Content-type: application/json"  http://loty-be.apps-crc.testing/api/v1/loty 

Finally, we’ll check that the entry we just added is stored correctly by querying it directly.
> curl -s -k http://loty-be.apps-crc.testing/api/v1/loty/year/1960 | jq 


## 4.2 deploy the frontend ... 
Lets quicky do the same for the frontend
> oc new-app openshift/nodejs:20-ubi9-minimal~https://github.com/sunnymoon/loty-fe --name=loty-fe --labels=app=loty --strategy=source --context-dir=/

> oc logs -f builds/loty-fe-1 

lets see what we got in our openshift
> oc get all

Lets expose it
oc expose service/loty-fe --hostname=loty.apps-crc.testing

We can check it out in the web interface
> crc console

## Conclusion
What we’ve shown today is a simplified, streamlined path, an example of how fast you can go from zero to source code to a running application using OpenShift, GitHub copilot, and modern developer tools. 

While this may not be a production-grade CI/CD pipeline not is it automated, it’s a powerful and fast way to prototype, test, or even run lightweight applications in a real Kubernetes environment.
In future sessions, we’ll possible explore tools like Tekton, Argo CD, and Backstage, as well as concepts like hooks and automation to take this workflow closer to production readiness.

One more note
Everything we did today was from the command line. No clicks no dashboards, no manual configuration (aside from viewing the end results).

And that’s not just for show. It highlights that everything we’ve done is scriptable, repeatable, and easily automatable.

From setting up the cluster, initializing projects, to building and deploying applications, every step can be integrated into CI/CD pipelines or developer portals to streamline onboarding and boost productivity.


## new linkarean of the year!
curl -s -k -v -X PUT --data @2024-reset.json -H "Content-type: application/json"  http://loty-be.apps-crc.testing/api/v1/loty/16  --output /dev/null

curl -s -k -v -X PUT --data @2024.json -H "Content-type: application/json"  http://loty-be.apps-crc.testing/api/v1/loty/16  --output /dev/null
