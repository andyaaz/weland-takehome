WeLand takehome

Tasks:
- Use Terraform or Ansible or CloudFormation to automate the following tasks against any cloud provider platform, e.g. AWS, GCP, Aliyun.
- Provision a new VPC and any networking related configurations.
- In this environment provision a virtual machine instance, with an OS of your choice.
- Apply any security hardening (OS, firewall, etc..) you see fit for the VM instance.
- Install Docker CE on that VM instance.
- Deploy/Start an Nginx container on that VM instance.
- Demonstrate how you would test the healthiness of the Nginx container.
- Expose the Nginx container to the public web on port 80.
- Fetch the output of the Nginx container’s default welcome page.
- Excluding any HTML/JS/CSS tags and symbols, output the words and their frequency count for the words that occurred the most times on the default welcome page.
- Demonstrate how you would log the resource usage of the containers every 10 seconds.

Bonus Points:
- Replace VM instance(s) with K8S cluster
- Use AWS as cloud provider platform
- Use Terraform instead of Ansible or CloudFormation
- Visualize monitoring with metrics query language
- Submit your solution with git and README

Solution:
1. Apply `vpc.cf.yaml` to create the VPC resources
2. Update and Apply `eksctl.clusterconfig.yaml` to create EKS cluster
3. Install helm, switch to corresponding cluster with `kubectl` and run the following
  a. `helm repo add bitnami https://charts.bitnami.com/bitnami`
  b. `helm install nginx-web-server bitnami/nginx -f nginx-web-server.values.yaml` (Assuming it's ok to install in default namespace. Otherwise better to create a namespace first)
  c. optional a WAF can be created for protecting service from OWASP security risks
4. To check healthiness of the service, NodePing can be used setting up health check and alert. Checking container health directly might not too helpful in k8s context as containers get destroyed/recreated/restarted all the time.
5. To fetch the output, run `curl http://<service-url>:80`
6. To output words with out htlm tags and css `curl http://<service-url>:80 | paste -d" " -s - |  sed -e 's/<style>.*<\/style>//g' | sed -e 's/<[^>]*>//g'`
7. To count word frequency, 
```bash
# get html
curl -s http://localhost:80 |\ 
  # put everything on a single line so sed would work later
  paste -d" " -s - |\ 
  # remove <style> tag
  sed -e 's/<style>.*<\/style>//g' |\ 
  # strip html tags
  sed -e 's/<[^>]*>//g' |\ 
  # replace period with a space for word counting
  sed -e 's/\./ /g' |\ 
  # remove extra spaces between words
  awk '$1=$1' |\ # re
  # put every word on a new line
  tr ' ' '\n' |\
  sort |\
  # count unique words
  uniq -c
```
8. For resource usage logging, AWS CloudWatch Container Insights can be used
