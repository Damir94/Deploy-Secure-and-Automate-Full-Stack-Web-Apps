# Deploy, Secure, and Automate Full-Stack Web Applications

*From cloud infrastructure provisioning to secure, automated production deployments.*

## Introduction

Deploying a web application involves more than writing code. It requires building reliable infrastructure, securing application access, automating deployments, and maintaining system performance in production.

This project focuses on understanding the fundamentals of application deployment rather than relying entirely on managed platforms that hide the underlying infrastructure. By working through each stage manually before introducing automation, we gain practical knowledge of how servers, networking, security, and deployment pipelines work together.

The project follows a hands-on approach inspired by real-world production environments, using a full-stack application consisting of a Vue.js frontend, a FastAPI backend, and a self-hosted Meilisearch search engine.

The techniques demonstrated are applicable across different environments, including cloud virtual machines, AWS EC2 instances, bare-metal servers, and local hardware.

Throughout this project, we explore infrastructure provisioning, server hardening, reverse proxy configuration, continuous integration and delivery (CI/CD), automated security scanning, reliable deployments, and rollback strategies.

**The goal is to build a production-ready application environment that is secure, automated, maintainable, and reliable.**

## The roadmap
This book is structured systematically to guide you from absolute zero to an advanced automation posture.

- Module 1: The foundation — Cloud infrastructure provisioning, setting up locked rootless user models, and establishing strict SSH cryptographic authentication.
- Module 2: The application runtime — Manual stack deployment, memory swap optimization for resource-constrained systems, Gunicorn process engineering, and Supervisor monitoring.
- Module 3: Data & search — Self-hosting and isolating internal database and indexing layers securely behind standard server firewalls.
- Module 4: Global delivery & app security — DNS configuration, edge proxy routing, end-to-end SSL/TLS encryption, and robust Nginx web-layer hardening against soft 404 vulnerabilities and automated scanners.
- Module 5: The automation pipeline — Shifting security left by constructing a multi-stage GitHub Actions CI/CD framework featuring dependency auditing (SCA), source code scanning (SAST), integration testing, and atomic zero-downtime deployments.
- Module 6: Optimization & maintenance — Native log-based traffic analytics with GoAccess, real-time terminal resource observation, dynamic frontend vulnerability scanning (DAST), and automated rolling maintenance.

# Module 1: The foundation

## Cloud setup & access

### Introduction
- Before you deploy any code, you need a secure environment. We use AWS EC2 here, but these concepts apply to DigitalOcean, Linode, or even a Raspberry Pi.
- By the end of this section, you will have a server running Ubuntu 24.04 LTS, accessed securely via SSH keys, with password authentication completely disabled.

### Create the project
