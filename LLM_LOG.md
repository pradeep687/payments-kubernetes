# LLM Collaboration Log

## 1. Purpose

I used an LLM (ChatGPT) as an engineering assistant during the assessment to accelerate Helm templating, Kubernetes security hardening, NetworkPolicy design, and External Secrets configuration.

All generated configurations were reviewed, validated, and tested manually before being used.

---

## 2. Prompt Strategy

### Prompt 1 – Helm Chart Structure

I provided the legacy Kubernetes Deployment manifests and asked the LLM to convert them into a reusable Helm chart with:

* `values.yaml` for configurable values
* Deployment and Service templates
* `_helpers.tpl`
* NodePort for the frontend
* ClusterIP for internal services
* Liveness and readiness probes
* Resource requests and limits

The generated output was then reviewed and adapted to the requirements of the assessment.

### Prompt 2 – Kubernetes Security Hardening

I provided the insecure `order-processor` manifest and asked the LLM to harden it according to Kubernetes security best practices.

The required controls included:

* Non-root execution
* UID 10001
* `allowPrivilegeEscalation: false`
* `privileged: false`
* `readOnlyRootFilesystem: true`
* Dropping Linux capabilities
* Resource requests and limits
* Kubernetes Secret instead of a hardcoded password

I manually verified the generated security context and adjusted the YAML where required.

### Prompt 3 – NetworkPolicy

I asked the LLM to create Kubernetes NetworkPolicies implementing the following communication flow:

```text
web-frontend
     |
     v
order-processor
     |
     v
payment-gateway
```

The generated policies were reviewed to ensure that the payment gateway only accepts traffic from the order processor.

### Prompt 4 – External Secrets and Secret Rotation

I asked the LLM to design a GitOps-compatible secret management approach using the External Secrets Operator.

The design synchronizes an external credential into a Kubernetes Secret named `db-secret`.

I also used a Helm checksum annotation to detect changes to the Secret and trigger a Deployment rolling update when the checksum changes.

---

## 3. AI Error Catching and Verification

I did not blindly apply the LLM-generated configuration.

I validated the generated Kubernetes manifests using Helm and Kubernetes tooling.

### Helm validation

```bash
helm lint .
```

This was used to identify YAML syntax and Helm template errors.

### Rendered manifest validation

```bash
helm template payments .
```

The rendered output was reviewed to verify that Helm expressions were generating valid Kubernetes manifests.

### Kubernetes validation

The rendered resources were checked before deployment using Kubernetes validation/dry-run commands where applicable.

### Issues identified during review

During the implementation, I specifically reviewed the following areas because LLM-generated Kubernetes configurations can contain subtle issues:

* YAML indentation
* Kubernetes API versions
* SecurityContext placement
* NetworkPolicy selectors
* Secret references
* Helm template syntax
* Probe configuration
* Resource configuration
* ExternalSecret API version
* Secret checksum/reload logic

The final configuration was manually reviewed and validated instead of relying solely on the LLM output.

---

## 4. Security Decisions

The original manifests contained several security issues.

### Privileged container

The original configuration used:

```yaml
privileged: true
```

This was removed.

The hardened configuration uses:

```yaml
runAsNonRoot: true
runAsUser: 10001
allowPrivilegeEscalation: false
privileged: false
readOnlyRootFilesystem: true
```

Linux capabilities were also dropped:

```yaml
capabilities:
  drop:
    - ALL
```

### Hardcoded credential

The original manifest contained a hardcoded database password.

This was removed and replaced with:

```yaml
env:
  - name: DB_PASSWORD
    valueFrom:
      secretKeyRef:
        name: db-secret
        key: DB_PASSWORD
```

### Network segmentation

NetworkPolicies were added to restrict communication between services.

The intended traffic flow is:

```text
web-frontend -> order-processor -> payment-gateway
```

The payment gateway does not accept ingress traffic from unrelated pods.

---

## 5. Helm Architecture Decision

I used a single modular Helm chart rather than maintaining three independent charts.

The structure is:

```text
helm/payments/
├── Chart.yaml
├── values.yaml
└── templates/
    ├── _helpers.tpl
    ├── web-frontend-deployment.yaml
    ├── web-frontend-service.yaml
    ├── order-processor-deployment.yaml
    ├── order-processor-service.yaml
    ├── payment-gateway-deployment.yaml
    ├── payment-gateway-service.yaml
    ├── networkpolicy.yaml
    ├── externalsecret.yaml
    ├── secret-store.yaml
    └── secret-rbac.yaml
```

This approach keeps the three microservices independently configurable while maintaining a single deployment package for the assessment.

Environment-specific configuration such as image versions, replica counts, service configuration, and resource limits is maintained in `values.yaml`.

---

## 6. Final Verification

After making the changes, I:

1. Ran `helm lint`.
2. Rendered the Helm templates.
3. Installed the chart into the Kubernetes cluster.
4. Verified the Pods and Services.
5. Verified the frontend NodePort.
6. Tested the application endpoint using `curl`.
7. Reviewed the generated Kubernetes security configuration.
8. Verified the Secret and ExternalSecret resources.

The frontend endpoint successfully returned the Nginx welcome page.

---

## 7. LLM Usage Summary

The LLM was used as an engineering assistant rather than as an automated deployment mechanism.

The final implementation was reviewed, corrected, rendered, validated, and tested manually.

This helped accelerate the implementation while maintaining responsibility for Kubernetes correctness and security decisions.
