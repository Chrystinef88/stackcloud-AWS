# stackcloud-AWS
Desafio Stack
# 🚀 Desafio de Projeto DIO – Primeira Stack AWS com CloudFormation

## 🧩 Descrição do Projeto
Este projeto foi desenvolvido como parte do **Desafio de Projeto da Digital Innovation One (DIO)**, com o objetivo de implementar a **primeira stack na AWS** utilizando o **AWS CloudFormation**.

A proposta do desafio é compreender o funcionamento do conceito de **Infrastructure as Code (IaC)** na prática — criando, versionando e documentando recursos da AWS de forma automatizada e reprodutível.

---

## 🏗️ Objetivo
- Criar uma infraestrutura básica na AWS de forma automatizada;
- Utilizar **CloudFormation** com templates em **YAML**;
- Versionar o código em um repositório GitHub;
- Documentar os resultados e aprendizados obtidos durante o processo.

---

## 📂 Estrutura do Repositório

📁 aws-cloudformation-desafio
│
├── 📄 template.yaml # Template CloudFormation com os recursos da stack
├── 📄 README.md # Documentação do projeto
└── 📸 evidencias/ # Capturas de tela da execução e recursos criados

---

## 🧱 Recursos Criados na Stack
O template YAML cria uma infraestrutura simples composta por:

- **VPC** com sub-rede pública;  
- **Internet Gateway** e **tabela de rotas** configuradas;  
- **Security Group** com regras para SSH e HTTP;  
- **Instância EC2** com IP público;  
- **Outputs** com informações úteis (ID da instância, IP público, ID da VPC).  

---

## ⚙️ Execução do Template

### 1. Fazer login no Console AWS
Acesse: [https://console.aws.amazon.com/](https://console.aws.amazon.com/)

### 2. Abrir o serviço **CloudFormation**
No menu, selecione **CloudFormation → Create stack → With new resources (standard)**.

### 3. Enviar o arquivo YAML
- Escolha “Upload a template file”  
- Faça upload do arquivo `template.yaml`

### 4. Definir parâmetros e criar stack
- Nomeie a stack (ex: `MinhaPrimeiraStack`)
- Revise as configurações e clique em **Create stack**

### 5. Acompanhar o progresso
- Acompanhe o status em **CloudFormation → Stacks → Events**  
- Quando o status for **CREATE_COMPLETE**, a infraestrutura estará criada 🎉

---

## 🧾 Template CloudFormation (template.yaml)

```yaml
AWSTemplateFormatVersion: "2010-09-09"
Description: >
  Stack simples de exemplo para o Desafio de Projeto DIO.
  Cria uma VPC, Sub-rede, Internet Gateway, Security Group e uma instância EC2.

Parameters:
  KeyName:
    Description: Nome do par de chaves EC2 para acesso SSH
    Type: AWS::EC2::KeyPair::KeyName
    ConstraintDescription: Deve ser o nome de uma chave EC2 existente.

  InstanceType:
    Description: Tipo da instância EC2
    Type: String
    Default: t2.micro
    AllowedValues:
      - t2.micro
      - t3.micro
      - t3.small
    ConstraintDescription: Deve ser um tipo EC2 válido.

  SSHLocation:
    Description: Faixa de IP permitida para acesso SSH (ex: 0.0.0.0/0)
    Type: String
    MinLength: 9
    MaxLength: 18
    Default: 0.0.0.0/0
    AllowedPattern: "(\\d{1,3}\\.){3}\\d{1,3}/\\d{1,2}"
    ConstraintDescription: Deve ser um bloco CIDR válido (ex: 0.0.0.0/0).

Resources:

  MyVPC:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: 10.0.0.0/16
      EnableDnsSupport: true
      EnableDnsHostnames: true
      Tags:
        - Key: Name
          Value: DIO-VPC

  MySubnet:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref MyVPC
      CidrBlock: 10.0.1.0/24
      MapPublicIpOnLaunch: true
      AvailabilityZone: !Select [0, !GetAZs '']
      Tags:
        - Key: Name
          Value: DIO-Subnet

  MyInternetGateway:
    Type: AWS::EC2::InternetGateway
    Properties:
      Tags:
        - Key: Name
          Value: DIO-InternetGateway

  AttachGateway:
    Type: AWS::EC2::VPCGatewayAttachment
    Properties:
      VpcId: !Ref MyVPC
      InternetGatewayId: !Ref MyInternetGateway

  MyRouteTable:
    Type: AWS::EC2::RouteTable
    Properties:
      VpcId: !Ref MyVPC
      Tags:
        - Key: Name
          Value: DIO-RouteTable

  MyRoute:
    Type: AWS::EC2::Route
    DependsOn: AttachGateway
    Properties:
      RouteTableId: !Ref MyRouteTable
      DestinationCidrBlock: 0.0.0.0/0
      GatewayId: !Ref MyInternetGateway

  SubnetRouteTableAssociation:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      SubnetId: !Ref MySubnet
      RouteTableId: !Ref MyRouteTable

  MySecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Permitir acesso SSH e HTTP
      VpcId: !Ref MyVPC
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 22
          ToPort: 22
          CidrIp: !Ref SSHLocation
        - IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          CidrIp: 0.0.0.0/0
      Tags:
        - Key: Name
          Value: DIO-SecurityGroup

  MyEC2Instance:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: !Ref InstanceType
      KeyName: !Ref KeyName
      SubnetId: !Ref MySubnet
      SecurityGroupIds:
        - !Ref MySecurityGroup
      ImageId: ami-0c55b159cbfafe1f0 # Amazon Linux 2 (exemplo para us-east-1)
      Tags:
        - Key: Name
          Value: DIO-EC2-Instance

Outputs:
  InstanceId:
    Description: ID da instância EC2 criada
    Value: !Ref MyEC2Instance

  PublicIP:
    Description: Endereço IP público da instância
    Value: !GetAtt MyEC2Instance.PublicIp

  VPCId:
    Description: ID da VPC criada
    Value: !Ref MyVPC

