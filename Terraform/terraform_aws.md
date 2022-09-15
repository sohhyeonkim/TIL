> ### 공식문서 

# Write Configuration

## Terraform Block

terraform 블록{}은 테라폼 관련 설정들(provider, version)이 작성되어있다. 이를 토대로 테라폼은 인프라스트럭쳐를 구성할 것이다. provider의 source 속성(attribute)은 옵셔널한 hostname, namespace, 그리고 다른 provider 타입을 정의한다. 테라폼은 Terraform Registry에서 provider를 티폴트로 설치한다. 아래의 예시 설정에서는, aws provider가 hashicorp/aws(registry.terraform.io/hashicorp/aws 단축어)로 정의되어 있다.

또한, 버전 설정도 각 provider의 required_providers 블록에 정의할 수 있다. 버전 속성은 옵셔널이지만, 테라폼이 설정환경에서 동작하지 않는 버전의 provider를 설치하는 것을 방지하기 위해 버전 옵션을 작성하는 것을 권장한다. 만약 버전 옵션을 명시하지 않으면, 테라폼은 자동으로 가장 최신버전을 설치할 것이다.

## Providers

provider 블록{}은 특정 provider 설정을 작성한다. 아래의 예시는 aws이다. provider는 테라폼이 리소스를 생성하고 관리하는 플러그인이다.

테라폼 설정에 여러개의, 그리고 서로 다른 provider 블록들을 작성할 수 있다. 

## Resources

resource 블록{}은 구성하려는 인프라스트럭쳐의 컴포넌트들을 정의한다. 리소스는 물리적인 혹은 EC2 인스턴스와 같은 가상의 컴포넌트이거나 Heroku 애플리케이션과 같은 logical resource일 수 있다. 

리소스 블록은 {} 시작 전에 두 개의 문자열이 있는데, 이는 각각 리소스 타입과 리소스 이름이다. 아래의 예시는 리소스 타입이 'aws_instance'이고, 리소스 이름이 'app_server'이다. 리소스 타입의 접두어(aws_instance의 aws)는 provider와 매핑된다. 리소스 타입과 리소스 이름을 조합해 리소스의 unique ID를 만든다. 이 경우 EC2 인스턴스의 ID는 'aws_instance.app_server'이다. 

resource 블록은 리소스에 대한 두 개의 인자를 포함하는데, 인자들로는 머신 사이즈, 디스크 이미지 이름, 또는 VPC ID 등이 포함될 수 있다. 아래의 예시에서는 AMI ID를 우분투 이미지로 설정하고, 인스턴스 타입을 t2.micro(프리티어)로 설정한다. 또한 인스턴스에 태그도 설정할 수 있다.

# Initialize the directory

```java
// 설정 폴더를 초기화하여 설정 파일에 정의된 provider들을 설치한다.
terraform init 
```

위의 명령어를 실행하면 테라폼이 aws provider를 다운로드하고 설치된 provider의 버전을 출력한다. 또한, 현재 디렉토리에 .terraform이라는 숨겨진 서브 디렉토리를 설치하고, .terraform.lock.hcl이라는 lock file을 생성하는데, 이는 provider의 버전을 명시하여 프로젝트의 provider를 업데이트하고 싶을때, 컨트롤 할 수 있다. 

```java
// 실행결과
Initializing the backend...

Initializing provider plugins...
- Finding hashicorp/aws versions matching "~> 4.16"...
- Installing hashicorp/aws v4.30.0...
- Installed hashicorp/aws v4.30.0 (signed by HashiCorp)

Terraform has created a lock file .terraform.lock.hcl to record the provider
selections it made above. Include this file in your version control repository
so that Terraform can guarantee to make the same selections by default when
you run "terraform init" in the future.

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.
```

# Format and validate the configuration

모든 설정 파일에 대해 일관성있는 포맷을 사용하는 것을 권장한다. 다음 명령어는 현재 디렉토리에 있는 설정들을 가독성과 일관성있게 자동 업데이트해준다. 실행 결과로는 변경된 파일들의 이름을 출력하는데, 만약 모든 설정 파일들이 올바르게 포맷팅 되어있었다면, 변경된 파일이 없으므로 아무것도 출력하지 않는다. 

```java
terraform fmt
```

또한, 문법적으로 유효한지 확인하는 명령어도 있다. 

```java
terraform validate
```

# Create Infrastructure

다음 명령어를 실행하면, 설정 파일들을 적용하고, 

```java
terraform apply
```

```java
// 실행결과
Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the
following symbols:
  + create

Terraform will perform the following actions:

  # aws_instance.app_server will be created
  + resource "aws_instance" "app_server" {
      + ami                                  = "ami-830c94e3"
      + arn                                  = (known after apply)
      + associate_public_ip_address          = (known after apply)
      + availability_zone                    = (known after apply)
      + cpu_core_count                       = (known after apply)
      + cpu_threads_per_core                 = (known after apply)
      + disable_api_stop                     = (known after apply)
      + disable_api_termination              = (known after apply)
      + ebs_optimized                        = (known after apply)
      + get_password_data                    = false
      + host_id                              = (known after apply)
      + host_resource_group_arn              = (known after apply)
      + id                                   = (known after apply)
      + instance_initiated_shutdown_behavior = (known after apply)
      + instance_state                       = (known after apply)
      + instance_type                        = "t2.micro"
      + ipv6_address_count                   = (known after apply)
      + ipv6_addresses                       = (known after apply)
      + key_name                             = (known after apply)
      + monitoring                           = (known after apply)
      + outpost_arn                          = (known after apply)
      + password_data                        = (known after apply)
      + placement_group                      = (known after apply)
      + placement_partition_number           = (known after apply)
      + primary_network_interface_id         = (known after apply)
      + private_dns                          = (known after apply)
      + private_ip                           = (known after apply)
      + public_dns                           = (known after apply)
      + public_ip                            = (known after apply)
      + secondary_private_ips                = (known after apply)
      + security_groups                      = (known after apply)
      + source_dest_check                    = true
      + subnet_id                            = (known after apply)
      + tags                                 = {
          + "Name" = "ExampleAppServerInstance"
        }
      // 생략
    }

Plan: 1 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.
```

변경사항을 적용하기 전에, 테라폼은 execution plan을 출력하는데, 이는 테라폼이 인프라스트럭처를 설정파일에 매칭되게 변경할 액션들을 설명한다. 여기서 (known after apply)는 리소스가 생성되기 전까지는 알 수 없음을 의미한다. 

플랜이 허용가능하다면, 'yes'를 입력해 계속 진행한다. 

# Inspect state

```java
terraform show
```

설정 파일 적용이 끝나면, 테라폼은 terraform.tfstate라는 파일을 생성하는데, 여기에는 리소스 ID들과 속성들이 저장되어 있다. 테라폼 상태(state) 파일은 테라폼이 관리할 수 있는 리소스들을 파악하는 유일한 방법이다. 이 파일에는 민감하나 정보들이 포함되어있을 수 있게 때문에 이 파일은 팀원들간만 공유할 수 있도록 접근을 제한해야한다. 