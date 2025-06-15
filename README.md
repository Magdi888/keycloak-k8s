# Managing Access to Kubernetes with Keycloak

This repository provides a guide and resources for integrating Keycloak as an identity provider to manage access to your Kubernetes cluster. By using Keycloak, you can enable Single Sign-On (SSO), centralized user management, and fine-grained access control for Kubernetes.

## Overview
- **Keycloak**: Open-source Identity and Access Management (IAM) solution.
- **Kubernetes**: Container orchestration platform.
- **OIDC**: OpenID Connect, a protocol for authenticating users with Keycloak.

## Steps to Configure Keycloak with Kubernetes

### 1. Deploy Keycloak
- Deploy Keycloak using Docker, Kubernetes, or a managed service.
- Access the Keycloak admin console and create a new realm for your organization.

### 2. Configure a Client for Kubernetes
- In Keycloak, create a new client (e.g., `kubernetes`) with:
  - Client Protocol: `openid-connect`
  - Access Type: `confidential` or `public`
  - Valid Redirect URIs: `https://<k8s-api-server>/oauth2/callback`
- Note the client ID and secret (if confidential).

### 3. Create Users and Groups
- Add users and groups in Keycloak to represent your Kubernetes users and teams.
- Assign roles as needed.

### 4. Configure Kubernetes API Server
- Edit the Kubernetes API server configuration to enable OIDC authentication:
  ```
  --oidc-issuer-url=https://<keycloak-domain>/realms/<realm-name>
  --oidc-client-id=<client-id>
  --oidc-username-claim=preferred_username
  --oidc-groups-claim=groups
  --oidc-ca-file=/etc/kubernetes/pki/ca.crt
  ```
- Restart the API server after making changes.

### 5. Generate kubeconfig for Users
- Use `kubectl` or a script to generate kubeconfig files for users, referencing Keycloak as the OIDC provider.
- Example:
  ```yaml
  users:
  - name: keycloak-user
    user:
      auth-provider:
        name: oidc
        config:
          client-id: <client-id>
          client-secret: <client-secret>
          id-token: <user-id-token>
          idp-issuer-url: https://<keycloak-domain>/realms/<realm-name>
          refresh-token: <user-refresh-token>
  ```

### 6. RBAC in Kubernetes
- Use Kubernetes Role-Based Access Control (RBAC) to grant permissions to users and groups from Keycloak.
- Example:
  ```yaml
  kind: RoleBinding
  apiVersion: rbac.authorization.k8s.io/v1
  metadata:
    name: dev-access
    namespace: development
  subjects:
  - kind: Group
    name: dev-team
    apiGroup: rbac.authorization.k8s.io
  roleRef:
    kind: Role
    name: developer
    apiGroup: rbac.authorization.k8s.io
  ```

## References
- [Keycloak Documentation](https://www.keycloak.org/docs/)
- [Kubernetes OIDC Auth](https://kubernetes.io/docs/reference/access-authn-authz/authentication/#openid-connect-tokens)
- [Kubernetes RBAC](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)

---

Feel free to contribute improvements or additional resources!
