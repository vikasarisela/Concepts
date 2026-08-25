# To enable your Kubernetes pods to access other AWS services

To enable your Kubernetes pods to access other AWS services using **IAM Roles for Service Accounts (IRSA)**, you must follow a 3-step lifecycle. Using `eksctl` automates the role creation, policy attachment, and service account annotation, but you must ensure your cluster's **OIDC provider** is enabled first. [1, 2]

Here is the exact step-by-step workflow to implement this.

**Step 1: Enable the OIDC Provider for your EKS Cluster**

Before creating the service account, your EKS cluster needs an associated **IAM OpenID Connect (OIDC) provider** to validate pod identity tokens against AWS IAM. [1, 2, 3]

Run this `eksctl` command to check or associate the OIDC provider: [1]

**bash**

```
eksctl utils associate-iam-oidc-provider --cluster=<your-cluster-name> --approve
```

Use code with caution.

**Step 2: Create the Service Account and Auto-Attach the Policy**

You can handle the Kubernetes Service Account creation, IAM Role creation, and policy attachment all in **one command** using `eksctl`. [1, 2]

Replace the placeholders and execute:

**bash**

```
eksctl create iamserviceaccount \
  --name <my-service-account-name> \
  --namespace <my-namespace> \
  --cluster <your-cluster-name> \
  --role-name <desired-iam-role-name> \
  --attach-policy-arn <arn:aws:iam::aws:policy/your-policy-arn> \
  --approve
```

Use code with caution.

- **What this does behind the scenes:** `eksctl` creates a CloudFormation stack that provisions an IAM Role with an OIDC trust policy, attaches your requested AWS IAM policy to it, creates the Kubernetes Service Account, and automatically annotates it with the newly created IAM Role ARN. [1, 2]

**Step 3: Configure your Pod to use the Service Account**

The final step is to tell your Kubernetes workload to execute using the identities you just configured. Add the `serviceAccountName` field to your Pod's deployment manifest spec. [1, 2]

**yaml**

```
apiVersion: apps/v1kind: Deploymentmetadata:name: my-aws-appnamespace: <my-namespace>spec:replicas:1selector:matchLabels:app: my-aws-apptemplate:metadata:labels:app: my-aws-appspec:serviceAccountName: <my-service-account-name># <--- Matches the SA created in Step 2containers:
      -name: app-containerimage: <your-app-image>
```

Use code with caution.

**Step 4: Verification**

To ensure the setup worked correctly, verify that your service account is properly linked to the IAM Role: [1, 2]

**bash**

```
kubectl get sa <my-service-account-name> -n <my-namespace> -o yaml
```

Use code with caution.

Look for the `eks.amazonaws.com/role-arn` metadata annotation in the output.