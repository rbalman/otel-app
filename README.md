# otel-app

Mark:
    - sbom
    - better static code analysis tools
    - caching
    - docker bake built tool


## Security
How would I secure this system in production is that. 

I usually like to think production security in 4 Layers commonly known as 4Cs. `Cloud`, `Cluster`, `Container`, `Code`. We should secure each 

Cloud
- CSPM tool or Config tracking tools like 
- CloudTrail logs and events tracking
- Only allowing CD to manage production account/infra
- Drift detection and reporting
- DR drills and Backup Restore mechanism
- IAM best practices, read only acess to the cloud envrionment
- Use Github OIDC to deploy changes
- Secure Cloud with SSO logins
- No static IAM credentails should be used
- Use proper VPC, subnet, NACLs, firewall practices
- Employ WAF, Rate limiting, DDOS protection
- For mission critical accounts we can even restrict all the IAM access with IP whitesliting which will prevent malacious actor to access AWS APIs even if credentails are leaked.
- Use SSO and RBAC for accessing tools like Arogcd
- Using VPC is not enough, organization has complex acess requirements, multiple roles, business needs, internal tools. Use zero trust mesh vpns like tailscale for security, auditing, speed and centralized access managmenet.
- Never expose internal toolings in public internet even if they have authentication in place. eg. grafana, prometheus, argocd endpoints should never have public endpoints.

Cluster
- Practice sound RBAC policies
- Use external secrets management services like hashicorp vault, secrets manager, 1password
- Use secrets management controllers like external secrets manager
- Logging and monitoring control plane components logs and metrics
- Whenever possible use managed/vetted amis for hosting workloads
- Use advance node management/autoscaling tools like karpenter for better recycling of amis/k8s version
- Timely ugprading of k8s version and controllers along with them
- Never use latest tag images, always prefer to use specific version
- Employ CNIs that provide better network policies or visibility like Cilium.
- Create and employ sound network policies


Container
- Use minimal images or distroless image to reduce attack surface and bloatware
- Always prefer to run container as non root user
- Use multi stage build to remove built generated files, folder and layers
- Use vetted base image like from chainguard if possible
- Never use latest tag and rather try to use specific version.

Code
- Shift left code scanning by introducing linters, security plugins during development phase for faster feedback and remediation before it lands on production
- Introduce Code and Dependency scanning as a part of CI
- Define Quality gate like unit test coverage, vulnerabiilty count/type..
- Minimizing dependency, write custom code whenever possible
- if critical may be pen testing
- Minimze the responsiblity and break as a separate micro-server if too much responsibility.