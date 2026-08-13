# Secure Payments Kubernetes Deployment

## Overview

This project implements a secure, modular Helm-based Kubernetes deployment
for three microservices:

- web-frontend
- order-processor
- payment-gateway

The solution applies DevSecOps and Kubernetes security best practices,
including container hardening, network segmentation, resource controls,
secret management and automated secret synchronization.

## Architecture

```text
                    Internet
                       |
                       v
              +----------------+
              |  web-frontend  |
              |    NodePort    |
              +----------------+
                       |
                 NetworkPolicy
                       |
                       v
              +------------------+
              | order-processor  |
              |    ClusterIP     |
              +------------------+
                       |
                 NetworkPolicy
                       |
                       v
              +------------------+
              | payment-gateway |
              |    ClusterIP     |
              +------------------+

External Secrets Operator
          |
          v
      SecretStore
          |
          v
     ExternalSecret
          |
          v
       db-secret
          |
          v
   order-processor
