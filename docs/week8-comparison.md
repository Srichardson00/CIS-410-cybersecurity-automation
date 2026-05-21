### Step 2: Complete the Comparison Table

| Dimension | On-Premise Docker (Wks 3–5) | Cloud Run (Week 8) |
| :--- | :--- | :--- |
| **Infrastructure setup** | 3 VMs created, Docker installed on each | Managed serverless environment, zero server provisioning |

| **Deployment command** | *SSH -> docker build -> docker run* | OIDC auth -> Docker build -> docker push -> gcloud run deploy |

| **TLS / HTTPS** | *Not configured* | Automatically provisioned and managed by Google Cloud |

| **Scaling approach** | *Manual — redeploy or add VMs* | Automatic autoscaling based on needs |

| **Port management** | *Ports 5000/5001/5002 per environment* | Cloud Run handles routing; No port management |

| **Cost when idle** | *VM running 24/7 regardless of traffic* | Scales down to zero when not in use; No activity, no charges |

| **Rollback** | *Re-deploy previous image manually* | Can easily rollback to previous revision with zero downtime | 

| **Secrets management** | *GitHub Secrets -> env vars in workflow* | Integration with Google Secret Manager |


Reflection Questions

Q1: Which approach required more manual steps from push to live URL? List the specific steps that were eliminated by Cloud Run.

The on-prem approach required significantly more manual steps to accomplish the task. Some of the steps we eliminated when running the Cloud Run were
•	Provisioning VMs
•	Manually initiate SSH connection to run deployment commands
•	Eliminate installing and running Docker on multiple VMs
•	No host port mapping or management


Q2: A security audit asks how you know which version of the code is currently running in production. How would you answer for on-premise Docker vs. Cloud Run with commit SHA tagging?

In an on prem setup you would need to SSH into the host VM and find the image tag fro the container you were looking for. In a cloud run enviroment every deployment tags a built container image with a SHA tag. So you would be able to see what version your running on the cloud run dashboard.

Q3: Your on-premise VMs run 24/7 even when no students are using the app. Cloud Run scales to zero. What is the security advantage of scale-to-zero beyond cost savings?


The security advantage of a scale-to-zero model is that when the container is inactive, Cloud Run tears down the running container. This limits the attack surface of the container to only when it is in use. In an on-prem setup, that machine is potentially vulnerable 24/7. Also, any memory-based attack is limited in scale to a zero model. Once the container is torn down, all memory is wiped out, so any malicious scripts living in memory are deleted when the machine is turned off. 


Q4: The OIDC workflow replaced the SSH key secrets from Weeks 3–5. What attack surface was eliminated?

The attack surface that was eliminated when we moved to the OIDC workflow over SSH keys is that we no longer use long-lasting static credentials. This is something that can be leaked, and if that happens, it can compromise the integrity of a system. OIDC uses keyless authentication that is dynamically generated. No static credentials to be stolen or leaked. 