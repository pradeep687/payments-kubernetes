# Payments Kubernetes Assignment

## Overview

This project demonstrates deploying three applications on Kubernetes:

- Web Frontend
- Payment Gateway
- Order Processor

Each application is deployed using a Kubernetes Deployment and exposed using a NodePort Service.

## Architecture

User
 |
 v
Web Frontend
 |
 v
Payment Gateway
 |
 v
Order Processor

## Components

| Application | Deployment | Service | NodePort |
|---|---|---|---|
| Web Frontend | web-frontend | web-frontend | 31263 |
| Payment Gateway | payment-gateway | payment-gateway | 31972 |
| Order Processor | order-processor | order-processor | 31343 |

## Deployment

Apply the deployments:

```bash
kubectl apply -f web-frontend.yaml
kubectl apply -f payment-gateway.yaml
kubectl apply -f order-processor.yaml
