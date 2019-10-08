---
title: "1. Exploring the templates"
weight: 1
draft: false
---

Please collapse the **Source CloudFormation Template** file, You can use
any text editor to explore the different elements of VPC mentioned in
the template. To make it easy for you can copy and paste the entire template into any YAML friendly text editor and collapse / expand the section.

Some Recommendations :

* Notepad++ (https://notepad-plus-plus.org/downloads/)
* VisualStudioCode. (https://code.visualstudio.com/)


**Source CloudFormation Template**

Copy this template for the next section.

<details><summary>CLICK HERE</summary>
<p>


```
AWSTemplateFormatVersion: '2010-09-09'
Parameters:
  vpccidr:
    Type: String
    Default: 10.20.0.0/16
  psharedacidr:
    Type: String
    Default: 10.20.0.0/24
  psharedbcidr:
    Type: String
    Default: 10.20.1.0/24
  pvsharedacidr:
    Type: String
    Default: 10.20.2.0/24
  pvsharedbcidr:
    Type: String
    Default: 10.20.3.0/24

Resources:
  VPC:
    Type: "AWS::EC2::VPC"
    Properties:
      CidrBlock: !Ref vpccidr
  IGW:
    Type: "AWS::EC2::InternetGateway"
  S3AppBucket:
    Type: "AWS::S3::Bucket"
    Properties:
      AccessControl: PublicRead
      WebsiteConfiguration:
        ErrorDocument: index.html
        IndexDocument: index.html
  BucketPolicyApp:
    Type: "AWS::S3::BucketPolicy"
    Properties:
      Bucket: !Ref S3AppBucket
      PolicyDocument:
        Statement:
          -
            Sid: "ABC123"
            Action:
              - "s3:GetObject"
            Effect: Allow
            Resource: !Join ["", ["arn:aws:s3:::", !Ref S3AppBucket, "/*"]]
            Principal:
              AWS:
                - "*"
  GatewayAttach:
    Type: "AWS::EC2::VPCGatewayAttachment"
    Properties:
      InternetGatewayId: !Ref IGW
      VpcId: !Ref VPC
  SubnetPublicSharedA:
    Type: "AWS::EC2::Subnet"
    Properties:
      AvailabilityZone: !Select [0, !GetAZs ]
      CidrBlock: !Ref psharedacidr
      MapPublicIpOnLaunch: true
      VpcId: !Ref VPC
  SubnetPublicSharedB:
    Type: "AWS::EC2::Subnet"
    Properties:
      AvailabilityZone: !Select [1, !GetAZs ]
      CidrBlock: !Ref psharedbcidr
      MapPublicIpOnLaunch: true
      VpcId: !Ref VPC
  SubnetRouteTableAssociatePublicA:
    Type: "AWS::EC2::SubnetRouteTableAssociation"
    Properties:
      RouteTableId: !Ref RouteTablePublic
      SubnetId: !Ref SubnetPublicSharedA
  SubnetRouteTableAssociatePublicB:
    Type: "AWS::EC2::SubnetRouteTableAssociation"
    Properties:
      RouteTableId: !Ref RouteTablePublic
      SubnetId: !Ref SubnetPublicSharedB
  SubnetPrivateSharedA:
    Type: "AWS::EC2::Subnet"
    Properties:
      AvailabilityZone: !Select [0, !GetAZs ]
      CidrBlock: !Ref pvsharedacidr
      MapPublicIpOnLaunch: false
      VpcId: !Ref VPC
  SubnetPrivateSharedB:
    Type: "AWS::EC2::Subnet"
    Properties:
      AvailabilityZone: !Select [1, !GetAZs ]
      CidrBlock: !Ref pvsharedbcidr
      MapPublicIpOnLaunch: false
      VpcId: !Ref VPC
  SubnetRouteTableAssociatePrivateA:
    Type: "AWS::EC2::SubnetRouteTableAssociation"
    Properties:
      RouteTableId: !Ref RouteTablePublic
      SubnetId: !Ref SubnetPrivateSharedA
  SubnetRouteTableAssociatePrivateB:
    Type: "AWS::EC2::SubnetRouteTableAssociation"
    Properties:
      RouteTableId: !Ref RouteTablePublic
      SubnetId: !Ref SubnetPrivateSharedB
  RouteDefaultPublic:
    Type: "AWS::EC2::Route"
    DependsOn: GatewayAttach
    Properties:
      DestinationCidrBlock: 0.0.0.0/0
      GatewayId: !Ref IGW
      RouteTableId: !Ref RouteTablePublic
  RouteDefaultPrivateA:
    Type: "AWS::EC2::Route"
    Properties:
      DestinationCidrBlock: 0.0.0.0/0
      NatGatewayId: !Ref NatGatewayA
      RouteTableId: !Ref RouteTablePrivateA
  RouteDefaultPrivateB:
    Type: "AWS::EC2::Route"
    Properties:
      DestinationCidrBlock: 0.0.0.0/0
      NatGatewayId: !Ref NatGatewayB
      RouteTableId: !Ref RouteTablePrivateB
  RouteTablePublic:
    Type: "AWS::EC2::RouteTable"
    Properties:
      VpcId: !Ref VPC
  RouteTablePrivateA:
    Type: "AWS::EC2::RouteTable"
    Properties:
      VpcId: !Ref VPC
  RouteTablePrivateB:
    Type: "AWS::EC2::RouteTable"
    Properties:
      VpcId: !Ref VPC
  EIPNatGWA:
    DependsOn: GatewayAttach
    Type: "AWS::EC2::EIP"
    Properties:
      Domain: vpc
  EIPNatGWB:
    DependsOn: GatewayAttach
    Type: "AWS::EC2::EIP"
    Properties:
      Domain: vpc
  NatGatewayA:
    Type: "AWS::EC2::NatGateway"
    Properties:
      AllocationId: !GetAtt EIPNatGWA.AllocationId
      SubnetId: !Ref SubnetPublicSharedA
  NatGatewayB:
    Type: "AWS::EC2::NatGateway"
    Properties:
      AllocationId: !GetAtt EIPNatGWB.AllocationId
      SubnetId: !Ref SubnetPublicSharedB

```


</p>
</details>



You will notice following resources in Initial AWS CloudFormation
Template:

-   VPC

    ```
    ...
    VPC:
      Type: "AWS::EC2::VPC"
      Properties:
        CidrBlock: !Ref vpccidr
    ...
    ```


-   Internet Gateway

    ```
    ...
    IGW:
      Type: "AWS::EC2::InternetGateway"
      
    ...

    GatewayAttach:
      Type: "AWS::EC2::VPCGatewayAttachment"
      Properties:
        InternetGatewayId: !Ref IGW
        VpcId: !Ref VPC
    ...
    ```


-   S3 bucket

    ```
    ...
      S3AppBucket:
        Type: "AWS::S3::Bucket"
        Properties:
        AccessControl: PublicRead
        WebsiteConfiguration:
            ErrorDocument: index.html
            IndexDocument: index.html
    ...
    ```

-   Two public subnets with corresponding route tables
    ```
    ...
    SubnetPublicSharedA:
        Type: "AWS::EC2::Subnet"
        Properties:
        AvailabilityZone: !Select [0, !GetAZs ]
        CidrBlock: !Ref psharedacidr
        MapPublicIpOnLaunch: true
        VpcId: !Ref VPC
    SubnetPublicSharedB:
        Type: "AWS::EC2::Subnet"
        Properties:
        AvailabilityZone: !Select [1, !GetAZs ]
        CidrBlock: !Ref psharedbcidr
        MapPublicIpOnLaunch: true
        VpcId: !Ref VPC
    ...

    ...
    SubnetRouteTableAssociatePublicA:
        Type: "AWS::EC2::SubnetRouteTableAssociation"
        Properties:
        RouteTableId: !Ref RouteTablePublic
        SubnetId: !Ref SubnetPublicSharedA
    SubnetRouteTableAssociatePublicB:
        Type: "AWS::EC2::SubnetRouteTableAssociation"
        Properties:
        RouteTableId: !Ref RouteTablePublic
        SubnetId: !Ref SubnetPublicSharedB
    RouteDefaultPublic:
        Type: "AWS::EC2::Route"
        DependsOn: GatewayAttach
        Properties:
        DestinationCidrBlock: 0.0.0.0/0
        GatewayId: !Ref IGW
        RouteTableId: !Ref RouteTablePublic
    ...

    ```

-   Two private subnets with corresponding route tables
```
    ...
    SubnetPrivateSharedA:
        Type: "AWS::EC2::Subnet"
        Properties:
        AvailabilityZone: !Select [0, !GetAZs ]
        CidrBlock: !Ref psharedacidr
        MapPublicIpOnLaunch: false
        VpcId: !Ref VPC
    SubnetPrivateSharedB:
        Type: "AWS::EC2::Subnet"
        Properties:
        AvailabilityZone: !Select [1, !GetAZs ]
        CidrBlock: !Ref psharedbcidr
        MapPublicIpOnLaunch: false
        VpcId: !Ref VPC
    SubnetRouteTableAssociatePrivateA:
        Type: "AWS::EC2::SubnetRouteTableAssociation"
        Properties:
        RouteTableId: !Ref RouteTablePrivateA
        SubnetId: !Ref SubnetPrivateSharedA
    SubnetRouteTableAssociatePrivateB:
        Type: "AWS::EC2::SubnetRouteTableAssociation"
        Properties:
        RouteTableId: !Ref RouteTablePrivateB
        SubnetId: !Ref SubnetPrivateSharedB
    RouteTablePrivateA:
        Type: "AWS::EC2::RouteTable"
        Properties:
        VpcId: !Ref VPC
    RouteTablePrivateB:
        Type: "AWS::EC2::RouteTable"
        Properties:
        VpcId: !Ref VPC
    RouteDefaultPrivateA:
        Type: "AWS::EC2::Route"
        Properties:
        DestinationCidrBlock: 0.0.0.0/0
        NatGatewayId: !Ref NatGatewayA
        RouteTableId: !Ref RouteTablePrivateA
    RouteDefaultPrivateB:
        Type: "AWS::EC2::Route"
        Properties:
        DestinationCidrBlock: 0.0.0.0/0
        NatGatewayId: !Ref NatGatewayB
        RouteTableId: !Ref RouteTablePrivateB
    ...
```


-   Two Elastic IP
    ```
    ...
    EIPNatGWA:
        DependsOn: GatewayAttach
        Type: "AWS::EC2::EIP"
        Properties:
        Domain: vpc
    EIPNatGWB:
        DependsOn: GatewayAttach
        Type: "AWS::EC2::EIP"
        Properties:
        Domain: vpc
    ...
    ```
-   Two NAT Gateway
    ```
    ...
    NatGatewayA:
        Type: "AWS::EC2::NatGateway"
        Properties:
        AllocationId: !GetAtt EIPNatGWA.AllocationId
        SubnetId: !Ref SubnetPublicSharedA
    NatGatewayB:
        Type: "AWS::EC2::NatGateway"
        Properties:
        AllocationId: !GetAtt EIPNatGWB.AllocationId
        SubnetId: !Ref SubnetPublicSharedB
    ...
    ```