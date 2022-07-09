## Deployment of Promtail, Loki and Grafana
For the deployment of the Promtail, Grafana and Loki, I am using the following approach (take into account that this is just PoC - Proof of Concept Approach and if you would like to use it in production, security hardning and identity management configuration will be required.)

![](/screenshots/switchingorganizationspromtail-loki-grafana-eks.drawio.png).


### K8S in particular EKS version



#### Promtail


#### Grafana Loki


#### Grafana




The final step is to add the source of data, for that we need to go to ```Configuration --> Data Sources ``` and select Loki. After adding the Loki as a data source, we need to specify the URL http://loki0:3100, which corresponds to the service ```loki0```, which is running on port 3100. Since all the containers are in the same network loki, all these services will be visible just by the name. After writing the URL, one need to click **Save & test** button to make sure that source is appended. 

To visualize the data all the services (promtail, loki and of course Grafana) should be running. One need to switch to the organisation ![](/screenshots/switchingorganizations.png).

After switching the organization, one should go to the **Explore** section and  make sure that Loki selected as a data source ![](/screenshots/lokidatasource.png).

After selecting the **Loki** as a data source, if data is pushed from the PromtTail, you should be able to see the option for selecting the files.
![](/screenshots/logsbrowser.png)

Once you will be able to see these files, you need to select one of them and browse the data, pay attention that in my case there are csv files, this is because I have configured Promtail to push the CSV files to the Loki.