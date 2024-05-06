AWSTemplateFormatVersion: 2010-09-09

Description:  This template deploys a VPC, with a pair of public and private subnets spread
  across two Availability Zones. It deploys an internet gateway, with a default
  route on the public subnets. It deploys a pair of NAT gateways (one in each AZ),
  and default routes for them in the private subnets. It deploys a pair of security groups as well.

Parameters:
  EnvironmentName:
    Description: An environment name that is prefixed to resource names
    Type: String
    Default: Development
    
  VpcCIDR:
    Default: 10.1.0.0/16
    Description: Please enter the IP range (CIDR notation) for this VPC
    Type: String

  PublicSubnet1CIDR:
    Default: 10.1.1.0/20
    Description: Please enter the IP range (CIDR notation) for the public subnet 1
    Type: String

  PublicSubnet2CIDR:
    Default: 10.1.20.0/20
    Description: Please enter the IP range (CIDR notation) for the public subnet 2
    Type: String

  PublicSubnet3CIDR:
    Default: 10.1.40.0/20
    Description: Please enter the IP range (CIDR notation) for the public subnet 3
    Type: String

  # PrivateSubnet1CIDR:
  #   Default: 10.1.60.0/20
  #   Description: Please enter the IP range (CIDR notation) for the private subnet 1
  #   Type: String

  # PrivateSubnet2CIDR:
  #   Default: 10.1.80.0/20
  #   Description: Please enter the IP range (CIDR notation) for the private subnet 2
  #   Type: String

Resources:
  VPC:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: !Ref VpcCIDR
      EnableDnsSupport: true
      EnableDnsHostnames: true
      Tags:
        - Key: Name
          Value: !Ref EnvironmentName

  InternetGateway:
    Type: AWS::EC2::InternetGateway
    Properties:
      Tags:
        - Key: Name
          Value: !Ref EnvironmentName

  InternetGatewayAttachment:
    Type: AWS::EC2::VPCGatewayAttachment
    Properties:
      InternetGatewayId: !Ref InternetGateway
      VpcId: !Ref VPC

  PublicSubnet1:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref VPC
      AvailabilityZone: !Select [ 0, !GetAZs '' ]
      CidrBlock: !Ref PublicSubnet1CIDR
      MapPublicIpOnLaunch: true
      Tags:
        - Key: Name
          Value: !Sub ${EnvironmentName} Public Subnet (AZ1)

  PublicSubnet2:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref VPC
      AvailabilityZone: !Select [ 1, !GetAZs  '' ]
      CidrBlock: !Ref PublicSubnet2CIDR
      MapPublicIpOnLaunch: true
      Tags:
        - Key: Name
          Value: !Sub ${EnvironmentName} Public Subnet (AZ2)
  
  PublicSubnet3:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref VPC
      AvailabilityZone: !Select [ 2, !GetAZs  '' ]
      CidrBlock: !Ref PublicSubnet3CIDR
      MapPublicIpOnLaunch: true
      Tags:
        - Key: Name
          Value: !Sub ${EnvironmentName} Public Subnet (AZ3)

  # PrivateSubnet1:
  #   Type: AWS::EC2::Subnet
  #   Properties:
  #     VpcId: !Ref VPC
  #     AvailabilityZone: !Select [ 0, !GetAZs  '' ]
  #     CidrBlock: !Ref PrivateSubnet1CIDR
  #     MapPublicIpOnLaunch: false
  #     Tags:
  #       - Key: Name
  #         Value: !Sub ${EnvironmentName} Private Subnet (AZ1)

  # PrivateSubnet2:
  #   Type: AWS::EC2::Subnet
  #   Properties:
  #     VpcId: !Ref VPC
  #     AvailabilityZone: !Select [ 1, !GetAZs  '' ]
  #     CidrBlock: !Ref PrivateSubnet2CIDR
  #     MapPublicIpOnLaunch: false
  #     Tags:
  #       - Key: Name
  #         Value: !Sub ${EnvironmentName} Private Subnet (AZ2)

  # NatGateway1EIP:
  #   Type: AWS::EC2::EIP
  #   DependsOn: InternetGatewayAttachment
  #   Properties:
  #     Domain: vpc

  # NatGateway2EIP:
  #   Type: AWS::EC2::EIP
  #   DependsOn: InternetGatewayAttachment
  #   Properties:
  #     Domain: vpc

  # NatGateway1:
  #   Type: AWS::EC2::NatGateway
  #   Properties:
  #     AllocationId: !GetAtt NatGateway1EIP.AllocationId
  #     SubnetId: !Ref PublicSubnet1

  # NatGateway2:
  #   Type: AWS::EC2::NatGateway
  #   Properties:
  #     AllocationId: !GetAtt NatGateway2EIP.AllocationId
  #     SubnetId: !Ref PublicSubnet2

  PublicRouteTable:
    Type: AWS::EC2::RouteTable
    Properties:
      VpcId: !Ref VPC
      Tags:
        - Key: Name
          Value: !Sub ${EnvironmentName} Public Routes

  DefaultPublicRoute:
    Type: AWS::EC2::Route
    DependsOn: InternetGatewayAttachment
    Properties:
      RouteTableId: !Ref PublicRouteTable
      DestinationCidrBlock: 0.0.0.0/0
      GatewayId: !Ref InternetGateway

  PublicSubnet1RouteTableAssociation:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      RouteTableId: !Ref PublicRouteTable
      SubnetId: !Ref PublicSubnet1

  PublicSubnet2RouteTableAssociation:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      RouteTableId: !Ref PublicRouteTable
      SubnetId: !Ref PublicSubnet2
  
  PublicSubnet3RouteTableAssociation:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      RouteTableId: !Ref PublicRouteTable
      SubnetId: !Ref PublicSubnet3

  # PrivateRouteTable1:
  #   Type: AWS::EC2::RouteTable
  #   Properties:
  #     VpcId: !Ref VPC
  #     Tags:
  #       - Key: Name
  #         Value: !Sub ${EnvironmentName} Private Routes (AZ1)

  # DefaultPrivateRoute1:
  #   Type: AWS::EC2::Route
  #   Properties:
  #     RouteTableId: !Ref PrivateRouteTable1
  #     DestinationCidrBlock: 0.0.0.0/0
  #     NatGatewayId: !Ref NatGateway1

  # PrivateSubnet1RouteTableAssociation:
  #   Type: AWS::EC2::SubnetRouteTableAssociation
  #   Properties:
  #     RouteTableId: !Ref PrivateRouteTable1
  #     SubnetId: !Ref PrivateSubnet1

  # PrivateRouteTable2:
  #   Type: AWS::EC2::RouteTable
  #   Properties:
  #     VpcId: !Ref VPC
  #     Tags:
  #       - Key: Name
  #         Value: !Sub ${EnvironmentName} Private Routes (AZ2)

  # DefaultPrivateRoute2:
  #   Type: AWS::EC2::Route
  #   Properties:
  #     RouteTableId: !Ref PrivateRouteTable2
  #     DestinationCidrBlock: 0.0.0.0/0
  #     NatGatewayId: !Ref NatGateway2

  # PrivateSubnet2RouteTableAssociation:
  #   Type: AWS::EC2::SubnetRouteTableAssociation
  #   Properties:
  #     RouteTableId: !Ref PrivateRouteTable2
  #     SubnetId: !Ref PrivateSubnet2
  
  VEDDEVSG:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupName: "VEDDEVSG"
      GroupDescription: "Security group with no ingress rule"
      VpcId: !Ref VPC

  VEDDEVSGEC2:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupName: "VEDDEVSGEC2"
      GroupDescription: "Security group with no ingress rule"
      VpcId: !Ref VPC

Outputs:
  VPC:
    Description: A reference to the created VPC
    Value: !Ref VPC

  PublicSubnets:
    Description: A list of the public subnets
    Value: !Join [ ",", [ !Ref PublicSubnet1, !Ref PublicSubnet2 ]]

  # PrivateSubnets:
  #   Description: A list of the private subnets
  #   Value: !Join [ ",", [ !Ref PrivateSubnet1, !Ref PrivateSubnet2 ]]

  PublicSubnet1:
    Description: A reference to the public subnet in the 1st Availability Zone
    Value: !Ref PublicSubnet1

  PublicSubnet2:
    Description: A reference to the public subnet in the 2nd Availability Zone
    Value: !Ref PublicSubnet2
  
  PublicSubnet3:
    Description: A reference to the public subnet in the 2nd Availability Zone
    Value: !Ref PublicSubnet3

  # PrivateSubnet1:
  #   Description: A reference to the private subnet in the 1st Availability Zone
  #   Value: !Ref PrivateSubnet1

  # PrivateSubnet2:
  #   Description: A reference to the private subnet in the 2nd Availability Zone
  #   Value: !Ref PrivateSubnet2

  VEDDEVSG:
    Description: Security group with no ingress rule
    Value: !Ref VEDDEVSG

  VEDDEVSGEC2:
    Description: Security group with no ingress rule
    Value: !Ref VEDDEVSGEC2