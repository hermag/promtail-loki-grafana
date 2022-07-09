<div align="center">
<img src="https://img.shields.io/static/v1?label=%F0%9F%8C%9F&message=If%20Useful&style=style=flat&color=BC4E99" alt="Star Badge"/>
<br>
<a href="https://github.com/hermag/promtail-loki-grafana/stargazers"><img src="https://img.shields.io/github/stars/hermag/promtail-loki-grafana" alt="Stars Badge"/></a>
<a href="https://github.com/hermag/promtail-loki-grafana/network/members"><img src="https://img.shields.io/github/forks/hermag/promtail-loki-grafana" alt="Forks Badge"/></a>
<a href="https://github.com/hermag/promtail-loki-grafana/pulls"><img src="https://img.shields.io/github/issues-pr/hermag/promtail-loki-grafana" alt="Pull Requests Badge"/></a>
<a href="https://github.com/hermag/promtail-loki-grafana/issues"><img src="https://img.shields.io/github/issues/hermag/promtail-loki-grafana" alt="Issues Badge"/></a>
<a href="https://github.com/hermag/promtail-loki-grafana/graphs/contributors"><img alt="GitHub contributors" src="https://img.shields.io/github/contributors/hermag/promtail-loki-grafana?color=2b9348"></a>
<a href="https://github.com/hermag/promtail-loki-grafana/blob/master/LICENSE.md"><img src="https://img.shields.io/github/license/hermag/promtail-loki-grafana?color=2b9348" alt="License Badge"/></a>
</div>

# Deployment of [Promtail](https://grafana.com/docs/loki/latest/clients/promtail/), [Loki](https://grafana.com/oss/loki/) and [Grafana](https://grafana.com/)
Goal of this project is to deploy several (4) Promtail log parsing services and equal number of Loki services to get these logs into one single instance of Grafana dashboard. 
With multiple Loki services we will be able to separate logs as we like. In particular, we will add Grafana Organizations Plugin to our Grafana deployment and add single Loki instance to a single organization, in this way each organization will have it's own Loki instance as a data source. Thus, we will be able to isolate logs per organization, on other hand, Organizations plugin allows to add users in it, hence it will be possible to give access to the specific users only to the specific set of data.

All these is deployed in two ways, using Docker and K8S, you can check details here:

* [Docker](https://github.com/hermag/promtail-loki-grafana/tree/main/docker)
* [K8S (EKS)](https://github.com/hermag/promtail-loki-grafana/tree/main/eks)