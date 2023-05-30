# Hands-on Lab: Monitoring in Action with Grafana 

## <img src="./images/intr.png" width='60'> Introduction
Welcome to the Monitoring with Grafana lab. In this lab you will learn to use Grafana as a visualization tool and dashboard for Prometheus.

## <img src="./images/objectives.png" width='60'> Learning Objectives

After completing this exercise, you should be able to:
- Deploy Prometheus to OpenShift
- Deploy Grafana to OpenShift
- Connect Prometheus as a datasource for Grafana
- Create a dashboard with Grafana
## Prerequisites

<style>
        .sp{
                color:orange; 
                border-style: solid; 
                border-width: thin; 
                border-color:black
        }
</style>

There is some setup to do before you can proceed with the lab. In this prerequisite step you will <span class="sp">clone</span> the GitHub repository that contains the Kubernetes manifests needed to deploy Prometheus and Grafana to an OpenShift clutser in the lab environment.


## The Task

1. First, if you do not already have a terminal open, open a terminal using the top menu item  <span class="sp">Terminal</span> ->  <span class="sp">New Terminal</span> and  <span class="sp">cd</span> (change directory) into the default  <span class="sp">/home/project</span> folder:
![new_terminal](./images/new_terminal.png)
>>> cd /home/project

2. Then use the git clone command to clone this repository:
>>> git clone https://github.com/ibm-developer-skills-network/ondaw-prometheus-grafana-lab.git

3. Finally, change into the ondaw-prometheus-grafana-lab folder:
>>> cd ondaw-prometheus-grafana-lab

4. (Optional) Make your command prompt shorter with this <span class="sp">bash</span> command:

>>> export PS1="\[\033[01;32m\]\u\[\033[00m\]:\[\033[01;34m\]\W\[\033[00m\]\$ "

## <img src="./images/step-1.png" width='60'> Deploy node exporters:
In this step, you will deploy 3 Node Exporters, which will be used as monitoring targets.

## The Task
1. Use the <span class="sp">oc create deployment</span> command to deploy 3 node exporters, named <span class="sp">node-exporter1</span>, <span class="sp">node-exporter2</span>, and <span class="sp">node-exporter3</span>, all listening on port <span class="sp">9100</span>.

>>> oc create deployment node-exporter1 --port=9100 --image=bitnami/node-exporter:latest
oc create deployment node-exporter2 --port=9100 --image=bitnami/node-exporter:latest
oc create deployment node-exporter3 --port=9100 --image=bitnami/node-exporter:latest

2. Next, use the <span class="sp">oc expose</span> command to create services that expose the 3 node exporters so that Prometheus can communicate with them:

>>> oc expose deploy node-exporter1 --port=9100 --type=ClusterIP
oc expose deploy node-exporter2 --port=9100 --type=ClusterIP
oc expose deploy node-exporter3 --port=9100 --type=ClusterIP


3. Check that the <span class="sp">pods</span> are up and running with the following oc command:

>>> oc get pods

## Results
You should see that all of the pods for the three <span class="sp">node-exporters</span> are in a <span class="sp">running</span> state like the image below:

![grafana_node_exporter_running](./images/grafana_node_exporter_running.png)

## <img src="./images/step-2.png" width='60'> Deploy Prometheus:
n this step, you will confiugure and deploy Prometheus. While normally you would modify configuration files to configure Prometheus, this is not the case for Kubernetes. For a Kubernetes environment the proper appoach is to use a <span class="sp">ConfigMap</span>. This makes it easy to change the configuraton later. You will find the configuration files from which to make the <span class="sp">ConfigMap</span> in a folder named <span class="sp">./config</span>.

You will also need Kubernetes manifests to describe the Prometheus deployment and to link the <span class="sp">ConfigMap</span> with the Prometheus. These manifests can be found in the <span class="sp">./deploy </span> folder.

## The Task
Use the following commands to create a <span class="sp">ConfigMap</span> and deploy Prometheus:

1. First, create a ConfigMap called <span class="sp">prometheus-config</span> that is needed by Prometheus from the <span class="sp">prometheus.yml</span> and the <span class="sp">alerts.yml</span> file in the <span class="sp">./deploy</span> folder.

>>> oc create configmap prometheus-config \
   --from-file=prometheus=./config/prometheus.yml \
   --from-file=prometheus-alerts=./config/alerts.yml

You should see a message that the <span class="sp">prometheus-config</span> was created.

2. Now deploy Prometheus using the <span class="sp">deploy/prometheus-deployment.yaml</span> deployment manifest.

>>> oc apply -f deploy/prometheus-deployment.yaml

You should see a message that the <span class="sp">prometheus-volume-claim</span>, <span class="sp">deployment.apps/prometheus</span>, and <span class="sp">service/prometheus</span> were all created.

3. Finally, use the <span class="sp">oc</span> command to check that the <span class="sp">prometheus</span> pod is running.

>>> oc get pods -l app=prometheus

## Results
You should see that the <span class="sp">prometheus</span> pod is in a running state.

![grafana_prometheus_running](./images/grafana_prometheus_running.png)

## <img src="./images/step-3.png" width='60'> Deploy Grafana:
Now that you have 3 node exporters to emit metrics, and Prometheus to collect them, it is time to add Grafana for dashboarding. You will deploy Grafana into OpenShift and create a <span class="sp">route</span> which will allow you to open up the Grafana web UI and work with it.

## The Task
1. First, deploy Grafana using the <span class="sp">deploy/grafana-deployment.yaml</span> deployment manifest.
>>> oc apply -f deploy/grafana-deployment.yaml

You should see a message that both the grafana deployment and service have been created.

2. Next, use the  <span class="sp">oc expose</span> command to expose the  <span class="sp">grafana</span> service with an OpenShift  <span class="sp">route</span>. Routes are a special feature of OpenShift that makes it easier to use for developers.
>>> oc expose svc grafana

Your should see the message: “route.route.openshift.io/prometheus exposed”


3. Use the following <span class="sp">oc patch</span> command to enable TLS and the <span class="sp">https://</span> protocol for the route.
>>> oc patch route grafana -p '{"spec":{"tls":{"termination":"edge","insecureEdgeTerminationPolicy":"Redirect"}}}'

4. Then use the <span class="sp">oc get routes</span> command to check the URL that the route was assigned. You will be able to interact with Grafana using this URL.

>>> oc get routes

5. Finally, use the <span class="sp">oc</span> command to check that the <span class="sp">grafana</span> pod is running.
>>> oc get pods -l app=grafana

## Results
You should see that the grafana pod is in a running state.
![grafana_pod_running](./images/grafana_pod_running.png)
You should also see the URL of the route that you created.

![grafana_route.png](./images/grafana_route.png)

You are now ready to configure Grafana.

## <img src="./images/step4.png" width='60'>Log in to Grafana:

Now that Prometheus and Grafana are deployed and running, it is time to configure Grafana. In order to do this, you will need the URL from the <span class="sp">route</span> that you created in the last step.

Note: You do not need a Grafana account in order to complete this step.

## The Task

1. Use the <span class="sp">oc describe</span> command along with <span class="sp">grep</span> to extract the URL of the Requested Host for the <span class="sp">grafana</span> <span class="sp">route</span>:

>>> oc describe route grafana | grep "Requested Host:"

You will see the words “Requested Host:” in red followed by a URL. It will start with the word “grafana-“ followed by the rest of the URL.

2. Copy the URL after the words “Requested Host:” to the clipboard and paste it into a new web browser window outside of the lab environment.
> You should see the Grafana log in page:
 
>> grafana-sn-labs-ahmedabdoami.labs-prod-openshift-san-a45631dc5778dc6371c67d206ba9ae5c-0000.us-east.containers.appdomain.cloud

![grafana_login.png](./images/grafana_login.png)

3. From the Grafana login screen, log in with the default userid <span class="sp">admin</span> and default password <span class="sp">admin</span>, and then click the [Log in] button. You will be prompted to change your password. You can press the <span class="sp">skip</span> link to bypass that for now.

## Results
You should see the Grafana home page:
![grafana_homepage.png](./images/grafana_homepage.png)



## <img src="./images/step-5.png" width='60'> Configure Grafana:

Once you have logged in to Grafana, you can configure Prometheus as your datasource.

## The Task
1. At the home page, select the <span class="sp">data sources</span> icon.

![grafana_home_datasources.png](./images/grafana_home_datasources.png)

2. At the <span class="sp">Add data source</span> page select <span class="sp">Prometheus</span>.
![grafana_select_prometheus.png](./images/grafana_select_prometheus.png)

3. On the <span class="sp">Prometheus</span> configuration page, set the Prometheus URL to <span class="sp">http://prometheus:9090</span> which is the name and port of the Prometheus service in OpenShift.

![grafana_prometheus_url.png](./images/grafana_prometheus_url.png)

4. Scroll down to the bottom of the screen and click the <span class="sp">Save & test</span> button.

![grafana_save_test.png](./images/grafana_save_test.png)

## Results
You should see a message, confirming that the data source is working:

![grafana_datasource_working.png](./images/grafana_datasource_working.png)

## <img src="./images/step-6.png" width='60'> Create a Dashboard:
Now you can create your first dashboard. For this lab, you will use a precreated template provided by Grafana Dashboard. This template is identified by the id <span class="sp">1860</span>.

## The Task
1. On the Grafana homepage, click the <span class="sp">Dashboards</span> icon, and select <span class="sp">+ Import</span> from the menu to start creating the dashboard.

![import_data_to_dashboard](./images/import_data_to_dashboard.png)

2. Next,enter the template identifier id <span class="sp">1860</span> in the space provided and click the <span class="sp">[Load]</span> button to import that dashboard from Grafana.

![choose_template](./images/choose_template.png)

3. You will see the default name for the template, Node Exporter Full, displayed. You are allowed to change it if you want to. The change will be local and valid only for your instance.

![choose_prometheus_source](./images/choose_prometheus_source.png)

4. At the bottom of the page, choose <span class="sp">Prometheus (Default)</span> as the data source and click <span class="sp">Import</span>.

![import_from_prometheus](./images/import_from_prometheus.png)

This completes importing a dashboard. Next you will see what the dashboard allows you to monitor.


## <img src="./images/Step-7.png" width='60'> View the Dashboard: 

You should now see the General / Node Exporter Full dashboard which shows CPU Busy, Sys Load, RAM used and other information. This dashboard will allow you to get a broad overview of how the system is performing, as well as drilling down into individual nodes to see how they are performing. Now, try a few things out.


## Things to try
1. You can hover over the graph to get specific information at a given time.
![hover_over_graph](./images/hover_over_graph.png)

2. Choose a different host to observe the details about that host.
![switch_target](./images/switch_target.png)

3. Choose to visualize the metrics over different time ranges.
![change_log_period](./images/change_log_period.png)

As you can imagine, this could be monitoring an application that you have deployed into Kubernetes or OpenShift so that you can see the CPU, Memory, and other metrics to determine the overall health of your application.