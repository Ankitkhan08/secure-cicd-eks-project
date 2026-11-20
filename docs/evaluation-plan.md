# Evaluation Plan

## 1. Objective
Compare a manual Kubernetes deployment (baseline) with an automated Jenkins CI/CD pipeline for deploying containerized applications on AWS EKS.

## 2. Baseline (Manual Path)
- Build Docker image locally or in a basic script.
- Push image to Amazon ECR.
- Use `kubectl set image` or `helm upgrade` to deploy to EKS.
- No automated security gate before deployment.

## 3. Automated Path (Jenkins Pipeline)
- Jenkins orchestrates: build → unit test → Docker image creation → Trivy image scan → push to ECR → `kubectl`/`helm` deploy to EKS.
- Trivy scan blocks the pipeline if CRITICAL vulnerabilities are found (configurable threshold).

## 4. Metrics and Data Collection
Each experiment will be repeated **three times**. The average value will be used for comparison.

| ID | Metric | Baseline Method | Automated Method | Unit |
|---|---|---|---|---|
| M1 | Deployment Lead Time | Time from code commit/trigger to Pod ready | Time from Jenkins trigger to Pod ready | Seconds |
| M2 | Failure / Rework Rate | Number of failed deployments requiring manual fix | Number of pipeline failures requiring re-run | Count per 3 runs |
| M3 | Security Gate Effectiveness | Vulnerable image deployed? (Yes/No) | Trivy blocks CRITICAL CVE? (Yes/No) | Binary |
| M4 | Rollback Time | `kubectl rollout undo` time to previous version | Time to re-run pipeline with previous image tag | Seconds |
| M5 | Infrastructure Reproducibility | Qualitative notes on manual steps needed | Qualitative notes on Terraform + pipeline steps | Notes |

## 5. Experimental Procedure
1. Provision EKS cluster with Terraform.
2. Deploy a sample application (e.g., a simple Python/Flask or Node.js app) manually three times; record M1–M4 for baseline.
3. Deploy the same application using the Jenkins pipeline three times; record M1–M4 for automated path.
4. For M3, push a known vulnerable image (e.g., `nginx:1.19`) and observe whether deployment is blocked in each path.
5. For M5, document the number of manual steps required to rebuild the environment from scratch in each path.

## 6. Expected Analysis
- Compare averaged M1–M4 values in a table and chart.
- Discuss trade-offs: automation reduces manual effort and failure rate but adds pipeline complexity; security gating increases lead time but prevents vulnerable deployments.
- Note limitations: single-node group, no production traffic, limited sample size.
