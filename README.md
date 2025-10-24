# monitoring

![Version](https://img.shields.io/github/v/tag/grevyrincon/monitoring?label=version&style=flat-square)

A repository for configuring and deploying the **monitoring** stack for containers and services.  
It includes custom dashboards, configuration files, and a CI/CD pipeline powered by **Jenkins**.

---

## 📁 Project Structure

```bash
.
├── CHANGELOG.md              # Version change log
├── Jenkinsfile               # Jenkins pipeline for automated deployment
├── README.md                 # Project documentation
├── dashboard/
│   └── container-dashboard.json   # Grafana dashboard for container monitoring
└── values-monitoring.yaml    # Monitoring configuration values (Helm/Prometheus)


## dashboard
![screenshot](images/image1.png)
![screenshot](images/image2.png)
![screenshot](images/image3.png)
