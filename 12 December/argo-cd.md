# Argo CD

Argo CD is a declarative, GitOps continuous delivery tool for Kubernetes. It automates the deployment of desired application states defined in Git repositories to Kubernetes clusters, ensuring that the deployed applications are always in sync with the source configurations.

## Table of Contents

- [What is Argo CD?](#what-is-argo-cd)
- [Argo CD Architecture](#argo-cd-architecture)
- [Core Components](#core-components)
- [RBAC (Role-Based Access Control)](#rbac-role-based-access-control)
- [Deploying Argo CD on Kubernetes](#deploying-argo-cd-on-kubernetes)
- [Installation Steps](#installation-steps)
- [Configuration Files](#configuration-files)
  - [AppProject Configuration](#appproject-configuration)
  - [Certificate Configuration](#certificate-configuration)
  - [ConfigMaps](#configmaps)
  - [Ingress Configuration](#ingress-configuration)
  - [RBAC Configuration](#rbac-configuration)
- [Understanding the Setup](#understanding-the-setup)
  - [Project Configuration](#project-configuration)
  - [TLS and Ingress](#tls-and-ingress)
  - [Azure AD Integration via OIDC](#azure-ad-integration-via-oidc)
  - [Managing Applications Outside Argocd Namespace](#managing-applications-outside-argocd-namespace)
    - [Best Practices](#best-practices)
- [Conclusion](#conclusion)

## What is Argo CD?

Argo CD is a declarative, GitOps-based continuous delivery tool specifically designed for Kubernetes. It continuously monitors Git repositories for changes and automatically applies them to Kubernetes clusters, ensuring that the cluster’s state matches the desired state defined in Git. Argo CD provides features such as:
• Declarative Setup: Define your application deployments in Git, version-controlled and auditable.
• Automated Syncing: Automatically or manually synchronize changes from Git to Kubernetes.
• Visibility: Offers a web UI and CLI for visualizing application states and deployment histories.
• RBAC: Fine-grained access control to manage who can deploy and modify applications.
• Integration: Supports various authentication mechanisms, including OAuth2 and OIDC for integrating with identity providers like Azure AD.

## Argo CD Architecture

Argo CD follows a client-server architecture with several core components working together to manage and deploy applications.

## Core Components

• argocd-server: Exposes the API and web UI for interacting with Argo CD.
• argocd-repo-server: Manages access to Git repositories and Helm charts.
• argocd-application-controller: Monitors the desired state in Git and ensures the Kubernetes cluster matches it.
• argocd-notifications-controller: Handles notifications and alerts based on application events.

## RBAC (Role-Based Access Control)

Argo CD uses RBAC to control access to its resources. Policies are defined in argocd-rbac-cm.yaml, specifying which roles have access to which operations. This ensures secure and controlled management of applications and cluster resources.

## Deploying Argo CD on Kubernetes

Deploying Argo CD involves installing it into your Kubernetes cluster, configuring it to communicate with your Git repositories, setting up RBAC, and securing it with TLS.

### Installation Steps

Create Namespace and Install Argo CD

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f <https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml>
```

Apply Configuration Files

```bash
kubectl apply -n argocd -f argocd/argocd-cm.yaml
kubectl apply -n argocd -f argocd/argocd-cmd-params-cm.yaml
kubectl apply -n argocd -f argocd/argocd-rbac-cm.yaml
```

Configure Azure AD Integration via OIDC. Follow the Argo CD Azure Login via OIDC guide to create an Azure AD app registration.

```bash
kubectl edit secret argocd-secret --namespace argocd
```

Convert your client secret to base64:

```bash
echo -n "<secret>" | base64
```

Add the encoded client secret to the data section:

```yaml
data:
  oidc.azure.argocd.clientSecret: <base64-encoded-clientsecret>
```

Apply TLS Certificate

```bash
kubectl apply -f argocd/argocd-cert.yaml --namespace argocd
```

Create Ingress for Argo CD

```bash
kubectl apply -f argocd/argocd-ingress.yaml --namespace argocd
```

Restart deployments to apply changes:

```bash
kubectl rollout restart --namespace argocd deployment argocd-server
kubectl rollout restart --namespace argocd statefulset argocd-application-controller
```

Allow Argo CD to Manage Applications Outside argocd Namespace

```bash
kubectl apply -k argocd/argocd-server-applications/
```

## Configuration Files

### AppProject Configuration

File: project.yaml

```yaml
apiVersion: argoproj.io/v1alpha1
kind: AppProject
metadata:
  name: foobar
  namespace: argocd

# Finalizer that ensures that project is not deleted until it is not referenced by any application

finalizers:
  - resources-finalizer.argocd.argoproj.io

spec:
  description: Foo Bar Projekt
  clusterResourceWhitelist:
    - group: ""
      kind: "Namespace"
    - group: ""
      kind: "PersistentVolume"
    - group: cert-manager.io
      kind: "Issuer"
    - group: cert-manager.io
      kind: "Certificate"
  sourceNamespaces:
    - foo-_
  sourceRepos:
    - "_"
  destinations:
    - namespace: "foo-_"
      server: <https://kubernetes.default.svc>
    - namespace: "argocd"
      server: <https://kubernetes.default.svc>
  syncWindows:
    - kind: deny
      schedule: "0 7 * * *"
      timeZone: "Europe/Zurich"
      duration: 5h
      namespaces:
        - foo-prod
      manualSync: true
    - kind: deny
      schedule: "0 13 * * *"
      timeZone: "Europe/Zurich"
      duration: 5h
      namespaces:
        - foo-prod
      manualSync: true
```

Explanation:

- kind AppProject: Defines a group of applications with specific access and deployment policies.
- finalizers: Ensures the project is not deleted until all referencing applications are removed.
- clusterResourceWhitelist: Specifies the Kubernetes resources that applications within the project are allowed to manage.
- sourceNamespaces & sourceRepos: Defines the namespaces and Git repositories from which applications can be sourced.
- destinations: Specifies the Kubernetes clusters and namespaces where applications can be deployed.
- syncWindows: Defines time windows during which automatic synchronization is denied, allowing for controlled deployments.

### Certificate Configuration

File: argocd-cert.yaml

```yaml
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: argocd-ingress
spec:
  secretName: argocd-ingress
  dnsNames:
    - argocd.foo.bar.ch
  usages:
    - digital signature
    - key encipherment
  issuerRef:
    group: cert-manager.io
    kind: ClusterIssuer
    name: letsencrypt
```

Explanation:

- kind Certificate: Requests a TLS certificate for Argo CD ingress.
- dnsNames: Specifies the domain for which the certificate is issued.
- usages: Defines the purposes for which the certificate can be used.
- issuerRef: References the ClusterIssuer to obtain the certificate from Let’s Encrypt.

### ConfigMaps

File: argocd-cm.yaml

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-cm
  namespace: argocd
  labels:
    app.kubernetes.io/name: argocd-cm
    app.kubernetes.io/part-of: argocd
data:
  application.resourceTrackingMethod: annotation

  # Argo CD's externally facing base URL (optional). Required when configuring SSO
  url: https://argocd.foo.bar.ch

  # OIDC configuration as an alternative to dex (optional)
  oidc.config: |
    name: Azure
    issuer: <issuer-endpoint>
    clientID: <client-id>
    clientSecret: $oidc.azure.argocd.clientSecret
    # Optional set of OIDC scopes to request. If omitted, defaults to: ["openid", "profile", "email", "groups"]
    requestedScopes:
      - openid
      - profile
      - email
    # Optional set of OIDC claims to request on the ID token.
    requestedIDTokenClaims:
      groups:
        essential: true

  # disables admin user. Admin is enabled by default
  admin.enabled: "true"
```

Explanation:

- application.resourceTrackingMethod: Defines how Argo CD tracks resources (using annotations in this case).
- url: Sets the external URL for Argo CD, essential for Single Sign-On (SSO) configurations.
- oidc.config: Configures OpenID Connect (OIDC) for Azure AD integration, enabling Azure-based authentication.
- admin.enabled: Enables the default admin user.

File: argocd-cmd-params-cm.yaml

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-cmd-params-cm
  labels:
    app.kubernetes.io/name: argocd-cmd-params-cm
    app.kubernetes.io/part-of: argocd
data:
  server.insecure: "true"
  application.namespaces: foo-*
```

Explanation:

- server.insecure: Configures the Argo CD server to run without TLS (useful for testing or internal deployments).
- application.namespaces: Specifies the namespaces Argo CD can manage, using a wildcard to include all namespaces starting with foo-.

File: argocd-rbac-cm.yaml

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-rbac-cm
  namespace: argocd
  labels:
    app.kubernetes.io/name: argocd-rbac-cm
    app.kubernetes.io/part-of: argocd
data:
  policy.csv: |
    p, role:org-admin, applications, _, _/_, allow
    p, role:org-admin, clusters, get, _, allow
    p, role:org-admin, repositories, get, _, allow
    p, role:org-admin, repositories, create, _, allow
    p, role:org-admin, repositories, update, _, allow
    p, role:org-admin, repositories, delete, _, allow
    g, "<random-string>", role:org-admin

    # group id from azure AD for AKS admins

  policy.default: role:readonly
scopes: "[groups, email]"
```

Explanation:

- policy.csv: Defines RBAC policies, granting the org-admin role extensive permissions over applications, clusters, and repositories.
- policy.default: Sets the default role to readonly for users without explicit permissions.
- scopes: Specifies the OIDC scopes to request from Azure AD.

### Ingress Configuration

File: argocd-ingress.yaml

```yaml
kind: Ingress
apiVersion: networking.k8s.io/v1
metadata:
  name: argocd-ingress
  annotations:
    nginx.ingress.kubernetes.io/force-ssl-redirect: "true"
    nginx.ingress.kubernetes.io/backend-protocol: "HTTP"
    nginx.ingress.kubernetes.io/proxy-buffering: "on"
    nginx.ingress.kubernetes.io/proxy-buffer-size: "128k" # authentication headers are larger than the default buffer
    nginx.ingress.kubernetes.io/proxy-buffers-number: "4"
spec:
  ingressClassName: nginx
  tls:
    - hosts:
        - argocd.foo.bar.ch
      secretName: argocd-ingress
  rules:
    - host: argocd.foo.bar.ch
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: argocd-server
                port:
                  name: http
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: argocd-server-grpc-ingress
  namespace: argocd
  annotations:
    nginx.ingress.kubernetes.io/backend-protocol: "GRPC"
spec:
  ingressClassName: nginx
  rules:
    - http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: argocd-server
                port:
                  name: https
        host: grpc.argocd.foo.bar.ch
  tls:
    - hosts:
        - grpc.argocd.foo.bar.ch
      secretName: argocd-ingress
```

Explanation:

- argocd-ingress:
  - annotations: Configure NGINX ingress controller settings, such as forcing SSL redirection and adjusting proxy buffer sizes to handle large authentication headers.
  - tls: Uses the argocd-ingress secret for TLS termination.
  - rules: Routes traffic from argocd.foo.bar.ch to the argocd-server service on the HTTP port.
- argocd-server-grpc-ingress:
  - annotations: Specifies the backend protocol as GRPC for gRPC communication.
  - rules: Routes traffic from grpc.argocd.foo.bar.ch to the argocd-server service on the HTTPS port.
  - tls: Shares the argocd-ingress secret for TLS termination.

### RBAC Configuration

Folder: argocd-server-application

File: argocd-notifications-controller-rbac-clusterrole.yaml

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  labels:
    app.kubernetes.io/name: argocd-notifications-controller-cluster-apps
    app.kubernetes.io/part-of: argocd
    app.kubernetes.io/component: notifications-controller
  name: argocd-notifications-controller-cluster-apps
rules:
  - apiGroups:
      - "argoproj.io"
    resources:
      - "applications"
    verbs:
      - get
      - list
      - watch
      - update
      - patch
  - apiGroups:
      - ""
    resources:
      - secrets
      - configmaps
    verbs:
      - get
      - list
      - watch
```

File: argocd-notifications-controller-rbac-clusterrolebinding.yaml

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  labels:
    app.kubernetes.io/name: argocd-notifications-controller-cluster-apps
    app.kubernetes.io/part-of: argocd
    app.kubernetes.io/component: notifications-controller
  name: argocd-notifications-controller-cluster-apps
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: argocd-notifications-controller-cluster-apps
subjects:
  - kind: ServiceAccount
    name: argocd-notifications-controller
    namespace: argocd

---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  labels:
    app.kubernetes.io/name: argocd-server-cluster-apps
    app.kubernetes.io/part-of: argocd
    app.kubernetes.io/component: server
  name: argocd-server-cluster-apps
rules:
  - apiGroups:
      - ""
    resources:
      - events
    verbs:
      - create
  - apiGroups:
      - "argoproj.io"
    resources:
      - "applications"
    verbs:
      - create
      - delete
      - update
      - patch
```

File: argocd-server-rbac-clusterrolebinding.yaml

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  labels:
    app.kubernetes.io/name: argocd-server-cluster-apps
    app.kubernetes.io/part-of: argocd
    app.kubernetes.io/component: server
  name: argocd-server-cluster-apps
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: argocd-server-cluster-apps
subjects:
  - kind: ServiceAccount
    name: argocd-server
    namespace: argocd

---
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - argocd-server-rbac-clusterrole.yaml
  - argocd-server-rbac-clusterrolebinding.yaml
  - argocd-notifications-controller-rbac-clusterrole.yaml
  - argocd-notifications-controller-rbac-clusterrolebinding.yaml
```

Explanation:

- ClusterRoles: Define permissions for the argocd-notifications-controller and argocd-server components, specifying the API groups, resources, and verbs they can access.
- ClusterRoleBindings: Bind the defined ClusterRoles to specific ServiceAccounts within the argocd namespace.
- Kustomization: Aggregates all RBAC-related YAML files for easier management and deployment using kubectl apply -k.

## Understanding the Setup

### Project Configuration

The project.yaml defines an AppProject named foobar within the argocd namespace. It sets up resource whitelists, source namespaces, source repositories, and deployment destinations. Additionally, it configures sync windows to control when automatic synchronizations are allowed, ensuring deployments do not occur during critical hours in the foo-prod namespace.

### TLS and Ingress

- Certificate Management: The argocd-cert.yaml requests a TLS certificate for the Argo CD ingress using Cert-Manager and Let’s Encrypt. This ensures secure HTTPS communication with the Argo CD web UI.
- Ingress Configuration: The argocd-ingress.yaml sets up NGINX Ingress rules for both HTTP and GRPC protocols. It enforces SSL redirection and adjusts proxy settings to handle large authentication headers, facilitating secure and reliable access to Argo CD services.

### Azure AD Integration via OIDC

Argo CD is configured to use Azure Active Directory (Azure AD) for authentication via OpenID Connect (OIDC). This integration allows users to authenticate using their Azure AD credentials, enabling Single Sign-On (SSO) and centralized user management.

Steps include:

1. App Registration: Create an Azure AD app registration following the Argo CD Azure Login via OIDC guide.
2. Secret Configuration: Store the Azure AD client secret in the argocd-secret Kubernetes Secret, encoded in base64.
3. ConfigMap Update: Update argocd-cm.yaml with the OIDC configuration details, linking to the Azure AD issuer and client credentials.

### Managing Applications Outside Argocd Namespace

To allow Argo CD to manage applications outside the argocd namespace, the setup applies additional RBAC configurations using Kustomize. This is achieved by applying the configurations found in the argocd-server-applications directory, which grants necessary permissions to Argo CD components to interact with resources in other namespaces matching the foo-\* pattern.

#### Best Practices

1. Version Pinning: Always specify exact versions for Helm charts and Kubernetes manifests to ensure deployment consistency and stability.
2. Secure Secrets Management: Avoid storing sensitive information in plain text. Use Kubernetes Secrets and ensure they are appropriately encrypted and access-controlled.
3. RBAC Granularity: Implement the principle of least privilege by granting only necessary permissions to roles and service accounts.
4. Monitoring and Alerts: Integrate Argo CD with monitoring tools like Prometheus and set up alerts for deployment failures or unauthorized access attempts.
5. Regular Backups: Backup Argo CD configurations and application manifests to prevent data loss and facilitate disaster recovery.
6. Automate Rollbacks: Configure automated rollback mechanisms to revert to previous stable states in case of deployment failures.
7. Use Sync Policies Wisely: Leverage automated synchronization with caution, especially in production environments. Consider manual approvals for critical deployments.

## Conclusion

Argo CD streamlines continuous delivery for Kubernetes by leveraging GitOps principles, ensuring that your cluster’s state is always in sync with your Git repositories. By following the structured setup process, integrating with Azure AD for secure authentication, and implementing robust RBAC policies, you can achieve a secure, scalable, and maintainable deployment pipeline. Utilizing tools like Cert-Manager for TLS management and NGINX Ingress for traffic routing further enhances the reliability and security of your Argo CD setup.

Citations

- Argo CD Official Documentation
- Cert-Manager Documentation
- Kubernetes RBAC Documentation
- NGINX Ingress Controller Annotations
- GitOps Principles
- Azure AD OIDC Integration with Argo CD
