# openshift-consul
A repository using RedHat codeready containers to provision a K8s cluster with Consul Service-mesh and a Mesh Gateway

# How to use

## Prerequisites:

- RedHat CodeReady Containers. Register a free, personal RedHat account and navigate to the [console](https://console.redhat.com/openshift/create/local), then download the runtime for your specific OS, along with the pull secret. Use it when asked during installation.
- Follow the second half of the latter webpage to setup (run `crc setup`) and start (run `crc start`) to Install CodeReady Containers
- Install [Helm](https://helm.sh/docs/intro/install/)
- Install [kubectl](https://kubernetes.io/docs/tasks/tools/)

## Setup

### Prepare the Consul environment

#### Start-up the CodeReady Containers platform. Take note of the credentials::

    [krastin:~/workspace/ … und/openshift] % crc start
    INFO Checking if running as non-root
    INFO Checking if crc-admin-helper executable is cached
    INFO Checking for obsolete admin-helper executable
    INFO Checking if running on a supported CPU architecture
    INFO Checking minimum RAM requirements
    INFO Checking if running emulated on a M1 CPU
    INFO Checking if HyperKit is installed
    INFO Checking if qcow-tool is installed
    INFO Checking if crc-driver-hyperkit is installed
    INFO Starting CodeReady Containers VM for OpenShift 4.9.0...
    INFO CodeReady Containers instance is running with IP 127.0.0.1
    INFO CodeReady Containers VM is running
    INFO Check internal and public DNS query...
    INFO Check DNS query from host...
    INFO Verifying validity of the kubelet certificates...
    INFO Starting OpenShift kubelet service
    INFO Waiting for kube-apiserver availability... [takes around 2min]
    INFO Waiting for user's pull secret part of instance disk...
    INFO Starting OpenShift cluster... [waiting for the cluster to stabilize]
    INFO All operators are available. Ensuring stability...
    INFO Operators are stable (2/3)...
    INFO Operators are stable (3/3)...
    INFO Adding crc-admin and crc-developer contexts to kubeconfig...
    Started the OpenShift cluster.

    The server is accessible via web console at:
    https://console-openshift-console.apps-crc.testing

    Log in as administrator:
    Username: kubeadmin
    Password: SAMPLEPASSWORDHERE

    Log in as user:
    Username: developer
    Password: developer

    Use the 'oc' command line interface:
    $ eval $(crc oc-env)
    $ oc login -u developer https://api.crc.testing:6443
    [krastin:~/workspace/ … und/openshift] %

#### Setup your environment and login as `kubeadmin`::

    [krastin:~/workspace/ … und/openshift] % eval $(crc oc-env)
    [krastin:~/workspace/ … und/openshift] % oc login -u kubeadmin https://api.crc.testing:6443
    Logged into "https://api.crc.testing:6443" as "kubeadmin" using existing credentials.

    You have access to 65 projects, the list has been suppressed. You can list all projects with 'oc projects'

    Using project "default".
    [krastin:~/workspace/ … und/openshift] % [krastin:~/workspace/ … und/openshift] % 

#### Add The HashiCorp Helm Chart Repository

    [krastin:~/workspace/ … und/openshift] % eval $(crc oc-env)
    helm repo add hashicorp https://helm.releases.hashicorp.com

    "hashicorp" has been added to your repositories

#### Create a Consul project

    [krastin:~/workspace/ … und/openshift] % oc new-project consul
    Now using project "consul" on server "https://api.crc.testing:6443".

    You can add applications to this project with the 'new-app' command. For example, try:

        oc new-app rails-postgresql-example

    to build a new example application in Ruby. Or use kubectl to deploy a simple Kubernetes application:

        kubectl create deployment hello-node --image=k8s.gcr.io/serve_hostname

#### Install Consul via Helm chart

helm install -f dc1-values.yaml consul hashicorp/consul --version "0.32.1" --wait

#### Set mesh gateway mode

Mesh gateways can be configured per datacenter in one of three modes: local, remote, or none. The [official documentation](https://www.consul.io/docs/connect/gateways/mesh-gateway#modes-of-operation) has full details on the implications of running in each mode. Use the proxy-defaults.yaml file that contains a CRD specification that globally configures all proxies to run in local mode.

