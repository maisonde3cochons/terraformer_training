

## 1. terraform 설치
```
wget https://releases.hashicorp.com/terraform/1.1.8/terraform_1.1.8_linux_amd64.zip
unzip terraform_1.1.8_linux_amd64.zip
sudo mv terraform /usr/bin/

terraform version
```



## 2. terraformer 설치
```
export PROVIDER={all,google,aws,kubernetes}
curl -LO https://github.com/GoogleCloudPlatform/terraformer/releases/download/$(curl -s https://api.github.com/repos/GoogleCloudPlatform/terraformer/releases/latest | grep tag_name | cut -d '"' -f 4)/terraformer-${PROVIDER}-linux-amd64 
curl -LO https://github.com/GoogleCloudPlatform/terraformer/releases/download/0.8.15/terraformer-${PROVIDER}-linux-amd64 
curl -LO https://github.com/GoogleCloudPlatform/terraformer/releases/download/0.8.15/terraformer-aws-linux-amd64 
chmod +x terraformer-${PROVIDER}-linux-amd64 
chmod +x terraformer-aws-linux-amd64 
sudo mv terraformer-${PROVIDER}-linux-amd64 /usr/local/bin/terraformer
sudo mv terraformer-aws-linux-amd64 /usr/local/bin/terraformer
```

#### Error: Invalid legacy provider address 에러 해결 방법
```
terraform state replace-provider -- -/aws hashicorp/aws
```



----------------------------------------------------------------------------
## [3. Terraform basic command]
```
terraform init
terraform plan
terraform apply
terraform destroy

terraform state list
```

#### tells Terraform to destroy and re-create a particular resource during the next apply, regardless of whether its resource arguments would normally require that.
```
terraform taint 
```

#### undoes a previous taint, or can preserve a resource that was automatically tainted due to failed provisioners.
```
terraform untaint 
```



## [4. AWS 내 Terraform Backend 저장소 생성]

#### 1. 로컬에 있는 Terraform 상태파일을 S3 Backend Bucket으로 전송
```
aws s3 cp terraform.state s3://<S3 Backend Bucket명>/<저장할 파일명>
```
#### 2. Terraform 상태파일 내 오브젝트 현황 확인
```
terraform state list
```
#### 3. Terraformer로 AWS 추출 가능한 대상 확인
##### - 참고 링크 :
> https://github.com/GoogleCloudPlatform/terraformer/blob/master/docs/aws.md



## [5. Terraformer를 활용한 기존 EKS Autoscaling 자원 대상 IaC 코드 추출]

#### 0. terraform init first
```
terraform init
```
#### 1. Terraformer를 활용한 기존 EKS Autoscaling 자원 대상 IaC 코드 추출
```
terraformer import aws --regions=<리전명> --resources=<자원명> --path-pattern=<추출한 파일 저장 디렉토리명>
terraformer import aws --regions=ap-northeast-2 --resources=ec2_instance --path-pattern=ec2_instance
```
#### 2. 추출된 Terraform 상태파일 내 오브젝트 현황 확인
```
terraform state list
```
#### 3. 추출 Terraform 상태파일을 기존 Terraform Backend 상태파일에 Import 방법
```
terraform state mv -state-out=<기존 Terraform Backend 상태파일 저장 경로> <추출Terraform Object명> <Import되서 저장될 Terraform Object명>
terraform state mv -state-out=<기존 Terraform Backend 상태파일 저장 경로> <terraform state list 조회한 녀석> <terraform state list 조회한 녀석>
```



## [6. 추출 IaC 코드를 활용한 2번째 EKS NodeGroup 생성 및 확인]

#### 1. 로컬에 있는 Terraform 상태파일을 S3 Backend Bucket으로 업로드
```
aws s3 cp terraform.state s3://<S3 Backend Bucket명>/<업로드할 파일명>
```
#### 2. 초기화 전 AWS Provider로 전환
```
terraform state replace-provider -auto-approve registry.terraform.io/-/awshashicorp/aws
```

#### 3. 프로비저닝을 통한 생성 및 확인
```
terraform init
terraform plan
terraform apply
```



## [7. Terraformer를 활용한 기존 AWS 네트워크 자원 대상 IaC 코드 추출]

#### 1. S3 Backend Bucket에 있는 Terraform 상태파일을 로컬로 다운로드
```
aws s3 cp s3://<S3 Backend Bucket명>/<저장된 파일명> <로컬의 다운로드 위치>
```
#### 2. Terraformer를 활용한 기존 EKS Autoscaling 자원 대상 IaC 코드 추출
```
terraformer import aws --regions=<리전명> --resources=<자원명> --path-pattern=<추출한 파일 저장 디렉토리명>
```
#### 3. 추출된 Terraform 상태파일 내 오브젝트 현황 확인
```
terraform state list
```
#### 4. 추출 Terraform 상태파일을 기존 Terraform Backend 상태파일에 Import 방법
```
terraform state mv -state-out=<기존 Terraform Backend 상태파일 저장 경로> <추출Terraform Object명> <Import되서 저장될 Terraform Object명>
```


## [8. 추출 IaC 코드를 활용한 Terraform 상태 및 형상 파일에 저장, 관리]

#### 1. 로컬에 있는 Terraform 상태파일을 S3 Backend Bucket으로 업로드
```
aws s3 cp terraform.state s3://<S3 Backend Bucket명>/<업로드할 파일명>
```
#### 2. 초기화 전 AWS Provider로 전환
```
terraform state replace-provider -auto-approve registry.terraform.io/-/awshashicorp/aws

```
