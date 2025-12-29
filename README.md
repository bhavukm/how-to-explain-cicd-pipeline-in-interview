# How to explain an End-To-End DevOps CI/CD project in an interview?

**YouTube Video URL Part 2:** https://youtu.be/HQN7eYdfK1w

**Part 1 Video:** https://youtu.be/vHd18cwh-iE

**How to explain an End-To-End DevOps CI/CD project in an interview?**

**Interview Question:** Explain the workflow of an End-To-End DevOps CI/CD Pipeline Project that you have managed.

**Answer:-**

**Explain your company’s business.** For example, Critical Alert Systems, Application for Lawyers, an e-commerce website, a travel portal, etc.

**Application programming language:** Java Springboot framework, Python, Drupal, etc.

**Application Layer being managed by the project:** Backend

**Tools Stack and Methodology Used:**

We use **GitLab SaaS CI/CD** to build, validate, scan, and promote **Java Spring Boot** applications using a shift-left DevSecOps approach.

Container images are built with Maven and Docker, stored in **Amazon ECR**, and deployed to **AWS EKS** using a **GitOps model with ArgoCD**, without direct cluster access from the pipeline.

Security, policy checks, and controlled promotions ensure only scanned and approved artifacts reach the cluster.

Shift-left DevSecOps means **checking security, quality, and policies early in the pipeline instead of waiting until after deployment**.

**DevOps Stages explanation:**

<img width="1628" height="365" alt="image" src="https://github.com/user-attachments/assets/f4509305-169b-4264-acd4-7a4e3c7274f8" />

## 1. **setup**

### **ecr-repo-check**

This step checks whether the required Amazon ECR repository already exists.

If the repository is missing, it is created so later image build and push steps do not fail.

### **eks-check**

This validates that the target EKS cluster is reachable and correctly configured.

It verifies AWS credentials, kubeconfig access, and basic cluster connectivity.

## 2. **validate**

### **dockerfile_lint_check**

This scans the Dockerfile for bad practices like using `latest` tags, running as root, or inefficient layers.

The goal is to catch security and performance issues before the image is built.

### **k8s_policy_check**

This validates Kubernetes manifests against cluster-level security and governance policies.

It ensures only compliant workloads are allowed to move further in the pipeline.

### **k8s_deprecated_apis_check**

This checks Kubernetes YAML files for APIs that are deprecated or removed in newer Kubernetes versions.

It helps prevent future deployment failures during EKS cluster upgrades.

### **k8s_syntax_check**

This performs syntax and schema validation of Kubernetes manifests.

It ensures YAML files are structurally correct and won’t fail at deployment time.

## 3. **build**

### **build_with_maven**

This runs `mvn clean package` using the `pom.xml` to compile and test the Spring Boot application.

The output is a versioned JAR file that will be used to build the container image.

## 4. **package**

### **build_dockerfile**

This builds the Docker image using the compiled Spring Boot JAR and Dockerfile.

The image is tagged with a commit SHA or build number for traceability.

### **build_helm_chart**

This packages the Helm chart that defines how the application runs on Kubernetes.

Environment-specific values like image tag, replicas, and resources are templated.

### **build_image_meta**

This generates image metadata such as digest, tags, commit ID, and build timestamp.

This metadata is later used for promotion, auditing, and GitOps-based deployments.

## 5. **scan**

### **scan_image**

This performs a security scan on the container image to detect OS and library vulnerabilities.

It ensures vulnerable images never reach the Kubernetes cluster.

### **sign_helm_chart**

This cryptographically signs the Helm chart to prove its integrity and source.

Only trusted charts are allowed to be deployed downstream.

### **sign_image**

This signs the container image so the cluster can verify it was built by this pipeline.

It prevents tampered or untrusted images from being deployed.

### **trivy_pipeline_scan**

This runs Trivy against the image, filesystem, and configuration.

The pipeline fails if critical vulnerabilities exceed defined thresholds.

### **trivy_sbom_scan**

This generates and scans a Software Bill of Materials (SBOM).

It provides full dependency visibility for compliance and security audits.

## 6. **promote**

### **promote-to-dev-eks-cluster**

This logically promotes the image and Helm chart to the Dev environment.

The artifact is not rebuilt, ensuring the same immutable image is promoted.

### **tag_build**

This applies to environment-specific tags such as `dev` or `release-candidate`.

Tags make it easy to track exactly what version is running in each environment.

## 7. **deploy**

### **generate-deploy-config**

This updates GitOps configuration with the new image tag and Helm values.

The change is committed to the Git repository monitored by ArgoCD.

### **trigger-deploy**

Once the Git change is pushed, ArgoCD detects it and syncs the desired state to the cluster.

No direct `kubectl apply` is performed from the pipeline.

## 8. **deploy-batch-1**

### **appname-dev**

This deploys the application to the Dev namespace in controlled batches.

Batch-based deployment reduces risk by limiting the blast radius during rollout.

## 9. **post-service-test**

### **post-service-test (smoke / functional tests)**

This runs automated smoke and functional tests using a Python script against live endpoints.

It confirms the application is reachable and core functionality works as expected.

The folder structure of the repo will be something like this:

![image](https://github.com/user-attachments/assets/ea816416-3eb2-4d56-ae3f-3596f10758e5)


**Deployment and Approval Approach:**

We follow a progressive promotion model using the same pipeline and the same Helm chart version across environments. A new Helm chart version is first deployed to the Dev cluster, then promoted to QA, and 

finally to Production, ensuring the exact same artifact is used everywhere.

Before promoting from QA to Production, we enforce a manual approval step in the pipeline. This approval acts as a controlled gate where an authorized user reviews test results, validations, and release 

readiness before allowing the production deployment to proceed.

Technically, this is achieved by marking the production promotion step as manual in the same GitLab pipeline. Once approved, the pipeline continues with the existing deployment steps, updating GitOps 

configuration and allowing ArgoCD to deploy the already-approved Helm chart version to the Production cluster. This keeps the process consistent, auditable, and safe without introducing a separate pipeline.

**What do you still need to learn so that you can answer questions on this project?**

1. How do you use GitLab templates to deploy infrastructure with Terraform and AWS EKS?

2. How do we authenticate AWS ECR repositories with ArgoCD?

3. How do we replicate AWS ECR Repositories across accounts?

4. How to set up a single GitLab server to run pipelines across different AWS accounts?

5. How do you manage secrets in CI/CD pipelines?

6. How do you remediate security vulnerabilities found while running the pipeline?

7. How do you manage the Terraform state file?
