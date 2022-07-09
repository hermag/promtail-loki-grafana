## Deployment of Promtail, Loki and Grafana
For the deployment of the Promtail, Grafana and Loki, I am using the following approach (take into account that this is just PoC - Proof of Concept Approach and if you would like to use it in production, security hardning and identity management configuration will be required.)

![](/screenshots/promtaillokigrafanaeks.png).


### K8S (EKS)
We asume that you already have working kubeconfig file and able to communicate with K8S using the ```kubectl``` command. 
First of all create the grafana namespace:
```kubectl apply -f grafana-ns.yaml```

Afterwards we need to create the grafana and loki storage classes and PV claims, we need that for storage allocation.
```
kubectl apply -f grafana-storage-class.yaml
kubectl apply -f loki-storage-class.yaml
```
Check if the claims are working, mounted and active
```
kubectl get pvc -n grafana
```

#### Grafana Loki
To deploy the Grafana loki and Load Balancer service, which will allow to communicate with it from outside, we need to step by step apply these ```yaml``` files.
```
kubectl apply -f loki01.yaml
kubectl apply -f loki02.yaml
kubectl apply -f loki03.yaml
kubectl apply -f loki04.yaml
```
Thus we will have four deployments of ```Loki``` with corresponding Load Balaner services to reach them. Take into account that, DNS records update takes about a minute or two, so, you will be able to access the Load Balancers on port 3100 only after few minutes.

#### Grafana

To deploy the grafana dashboard and the corresponding Load Balancer service we need to do the following
```
kubectl apply -f grafana.yaml
```

To add the Loki services as sources of data, we need to go to ```Configuration --> Data Sources ``` and select Loki. After we need to specify the URL http://loki0:3100, which corresponds to the service ```loki0```, which is running on port 3100. Since all the deployments are in the same namespace, all these services will be visible just by the name and on the port 3100. After writing the URL, one need to click **Save & test** button to make sure that source is appended. 

One can add Organisations plugin (from the ```Server Admin --> Plugins``` section). After that you can create multiple organizations (for example as many orgs as many Loki deployments you have), and assign each Loki deployment as a data source to each organization.

To visualize the data all the services (promtail, loki and of course Grafana) should be running. One need to switch to the organisation ![](/screenshots/switchingorganizations.png).

After switching the organization, one should go to the **Explore** section and  make sure that Loki selected as a data source ![](/screenshots/lokidatasource.png).

After selecting the **Loki** as a data source, if data is pushed from the PromtTail, you should be able to see the option for selecting the files.
![](/screenshots/logsbrowser.png)

Once you will be able to see these files, you need to select one of them and browse the data, pay attention that in my case there are csv files, this is because I have configured Promtail to push the CSV files to the Loki.

#### Promtail
In this scenario, Promtail services are deployed on a standalone EC2 instance as docker containers. To deploy several instances of promtail containers, have a look to [docker-compose.yml](https://github.com/hermag/promtail-loki-grafana/blob/main/eks/dockerized-promtail/docker-compose.yml) file. Before starting the containers one need to specify in ```dockerized-promtail/promtail0/promtail-config.yml, dockerized-promtail/promtail1/promtail-config.yml, dockerized-promtail/promtail2/promtail-config.yml``` and ```dockerized-promtail/promtail3/promtail-config.yml``` files the addresses of the Loki load balancers, also don't forget to check and make sure that volumes are mounted and there are text files. For more details have a [here](https://github.com/hermag/promtail-loki-grafana/tree/main/docker).