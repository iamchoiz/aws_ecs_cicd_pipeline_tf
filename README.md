# aws_ecs_cicd_pipeline_tf

> Terraform으로 구성하는 **AWS ECS(EC2 Launch Type) + AWS CodeSeries CI/CD 파이프라인** 인프라 자동화 프로젝트

## Architecture

![Architecture](https://user-images.githubusercontent.com/77256060/147801679-6aba0538-6765-41d9-aed4-1ee9998176dd.png)

---

## Tech Stack

| Category | Tool |
|---|---|
| IaC | Terraform |
| Container Orchestration | AWS ECS (EC2 Launch Type) |
| CI/CD | AWS CodePipeline, CodeBuild, CodeDeploy |
| Source | GitHub / AWS CodeCommit |
| Networking | AWS VPC, ALB |

---

## 디렉토리 구조

```
.
├── vpc/                  # VPC, Subnet, IGW, NAT Gateway
├── ecs/                  # ECS Cluster, Task Definition, Service, ALB
├── ecs-gitlab/           # GitLab 연동 ECS 설정
└── aws-cicd-pipeline/    # CodeBuild, CodeDeploy, CodePipeline, IAM
```

---

## 사전 준비사항

- Terraform Cloud 또는 Terraform OSS
- AWS ECR Repository (컨테이너 이미지 저장소)
- [예제 컨테이너 이미지](https://github.com/cshift/example_ecs_service)
- AWS CLI 설정 완료

---

## Terraform Workspace 구성

> VPC → ECS → AWS-CICD/PIPELINE 순서로 적용하는 것을 권장합니다.

| # | Workspace | 생성 리소스 |
|---|---|---|
| 1 | **VPC** | VPC, Public/Private Subnet, IGW, NAT GW, Route Table |
| 2 | **ECS** | ECS Cluster, Task Definition, ECS Service, ALB |
| 3 | **AWS-CICD/PIPELINE** | CodeBuild, CodeDeploy, CodePipeline, CodeStar, CodeCommit |
| 4 | CodeBuild | CodeBuild 단독 프로비저닝 |
| 5 | CodeDeploy | CodeDeploy 단독 프로비저닝 |
| 6 | CodePipeline | CodePipeline 단독 프로비저닝 |

---

## CI/CD 파이프라인 동작 방식

**1. Source Stage**
- GitHub 또는 CodeCommit에서 코드 변경 감지
- 소스 선택: [`source_provider`](aws-cicd-pipeline/_terraform.auto.tfvars) 환경변수로 설정

**2. Build Stage**
- CodeBuild에서 Docker 이미지 빌드 및 ECR 푸시

**3. Deploy Stage** (두 가지 옵션 중 선택)
- **Option 1** : CodePipeline 자체 ECS 배포
- **Option 2** : CodeDeploy Blue/Green 배포 (권장)
- 배포 방식: [`deployment_controller`](ecs/terraform.auto.tfvars) 환경변수로 설정

---

## 사용 방법

```bash
# 1. 레포지토리 클론
git clone https://github.com/DACHANCHOI/aws_ecs_cicd_pipeline_tf.git
cd aws_ecs_cicd_pipeline_tf

# 2. VPC 프로비저닝
cd vpc && terraform init && terraform apply

# 3. ECS 프로비저닝
cd ../ecs && terraform init && terraform apply

# 4. CI/CD 파이프라인 프로비저닝
# _terraform.auto.tfvars에서 source_provider 및 deployment_controller 설정 후
cd ../aws-cicd-pipeline && terraform init && terraform apply
```
