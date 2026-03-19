Provide your solution here:
## Architecture Diagram
![Architecture Diagram](./trading.png)

## Components
|Components|Role/Responsible|
|---|---|
| Components | Role/Responsible |
| WAF | Project the Cloudfront and Network Load balancer from internet facing |
| Cloudfront | Optimize the content delivery |
| Route53 | DNS Record configuring |
| ACM | Use for validating the public domain certificate (SSL) |
| Secret Manager | Store the secret application as well as database credentials, this could be |  |replace by AWS parameter store |
| S3 bucket | each of bucket use for store the platfom data and database snapshoot in case |  |need to recovery from failure |
| API Gateway | use both Rest API and web socket to enhance the real time trading experiences |
| Network loadbalancer | use for balance the traffic from client |
| EKS | Where the application handle the business logic, in hear, it could be handle the |  |certificate termination and routing base on request path. Or we can use an AWS Application |  |Load Balancer to do the similiar tasks |
| Redis | Handle caching to improve performance |
| RDS | Responsible for storing transaction data, business data |
| MSK | A message broker for comunication between services async |
| Transit Gateway | Use for control network between 2 or more VPCs |
| Observability Platform | this component can be many tools like Datadog, Dynatrace, LGTM to |  |make sure the SLO and SLI can be mesuarable and visualizable |