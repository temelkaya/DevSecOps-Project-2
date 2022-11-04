# DevSecOps-Project-2
# VPC_CFT

AWS Cloud Formation template is **a declaration of the AWS resources that make up a stack**. **Resources** section the required section in the template.

<img width="694" alt="Screen Shot 2022-11-03 at 2 29 32 PM" src="https://user-images.githubusercontent.com/86754468/199842528-66ddac3e-6140-4092-80c6-8146a5074a33.png">

The steps that needs to be followed to create a stack for the diagram above.
- <a href="https://github.com/githabocal/cft#creating-vpc" target="_blank">*VPC*</a>
- <a href="https://github.com/githabocal/cft#creating-subnets" target="_blank">*Creating Subnets*</a>
- <a href="https://github.com/githabocal/cft#creating-internet-gateway" target="_blank">*Creating Internet Gateway*</a>
- <a href="https://github.com/githabocal/cft#creating-route-tables" target="_blank">*Creating Route Tables*</a>
- <a href="https://github.com/githabocal/cft#creating-natgateway" target="_blank">*Creating NatGateway*</a>
- <a href="https://github.com/githabocal/cft#creating-elastic-ip" target="_blank">*Creating Elastic IP*</a>
- <a href="https://github.com/githabocal/cft#creating-security-group" target="_blank">*Creating Security Group*</a>
- <a href="https://github.com/githabocal/cft#creating-nacl" target="_blank">*Creating NACL*</a>
- <a href="https://github.com/githabocal/cft#creating-load-balancer" target="_blank">*Creating Load Balancer*</a>
- <a href="https://github.com/githabocal/cft#creating-target-group" target="_blank">*Creating Target Group*</a>
- <a href="https://github.com/githabocal/cft#launch-configuration" target="_blank">*Launch Configuration*</a>
- <a href="https://github.com/githabocal/cft#creating-auto-scaling-group" target="_blank">*Creating Auto Scaling Group*</a>
- <a href="https://github.com/githabocal/cft#creating-instance" target="_blank">*Creating Instance*</a>
- <a href="https://github.com/githabocal/cft#github-actions" target="_blank">*GitHub Actions*</a> 

# <h3>Creating VPC:
```
        "VPC": {
            "Type" : "AWS::EC2::VPC",
            "Properties" : {
                "CidrBlock" : "10.0.0.0/16",
                "EnableDnsHostnames" : true,
                "EnableDnsSupport" : false,
                "InstanceTenancy" : "default",
                "Tags" : [
                    {
                        "Key": "Name",
                        "Value": "test-vpc-cft"
                    }
                ]
            }
        }
```

<a href="#top">Back to top</a>

# <h3>Creating Subnets:
```
        "PublicSubnet1":{
            "Type" : "AWS::EC2::Subnet",
            "Properties" : {
                "AvailabilityZone" : {
                    "Fn::Select" : [ 
                      "0", 
                      { 
                        "Fn::GetAZs" : "" 
                      } 
                    ]
                  },
                "CidrBlock" : "10.0.1.0/28",
                "MapPublicIpOnLaunch" : true,
                "Tags" : [
                    {
                        "Key": "Name",
                        "Value": "public-subnet-1"
                    }
                ],
                "VpcId" : {"Ref": "VPC"}
            }
        },
        "PublicSubnet2":{
            "Type" : "AWS::EC2::Subnet",
            "Properties" : {
                "AvailabilityZone" : {
                    "Fn::Select" : [ 
                      "1", 
                      { 
                        "Fn::GetAZs" : "" 
                      } 
                    ]
                  },
                "CidrBlock" : "10.0.2.0/28",
                "MapPublicIpOnLaunch" : true,
                "Tags" : [
                    {
                        "Key": "Name",
                        "Value": "public-subnet-2"
                    }
                ],
                "VpcId" : {"Ref": "VPC"}
            }
        },
        "PrivateSubnet1":{
            "Type" : "AWS::EC2::Subnet",
            "Properties" : {
                "AvailabilityZone" : {
                    "Fn::Select" : [ 
                      "0", 
                      { 
                        "Fn::GetAZs" : "" 
                      } 
                    ]
                  },
                "CidrBlock" : "10.0.3.0/28",
                "MapPublicIpOnLaunch" : false,
                "Tags" : [
                    {
                        "Key": "Name",
                        "Value": "private-subnet-1"
                    }
                ],
                "VpcId" : {"Ref": "VPC"}
            }
        },
        "PrivateSubnet2":{
            "Type" : "AWS::EC2::Subnet",
            "Properties" : {
                "AvailabilityZone" : {
                    "Fn::Select" : [ 
                      "1", 
                      { 
                        "Fn::GetAZs" : "" 
                      } 
                    ]
                  },
                "CidrBlock" : "10.0.4.0/28",
                "MapPublicIpOnLaunch" : false,
                "Tags" : [
                    {
                        "Key": "Name",
                        "Value": "private-subnet-2"
                    }
                ],
                "VpcId" : {"Ref": "VPC"}
            }
        }
```

<a href="#top">Back to top</a>

# <h3>Creating Internet Gateway:
```
        "internetGateway" : {
            "Type" : "AWS::EC2::InternetGateway",
            "Properties" : {
                "Tags" : [ 
                    {
                        "Key" : "Name", 
                        "Value" : "test-cft-internet-gateway"
                    }
                ]
            }
        },
        "AttachGateway" : {
            "Type" : "AWS::EC2::VPCGatewayAttachment",
            "Properties" : {
               "VpcId" : { "Ref" : "VPC" },
               "InternetGatewayId" : { "Ref" :"internetGateway" }
             }
         }
```
<a href="#top">Back to top</a>

# <h3>Creating Route Tables:
```
        "PublicRouteTable" : {
            "Type" : "AWS::EC2::RouteTable",
            "Properties" : {
               "VpcId" : { "Ref" : "VPC" },
               "Tags" : [ 
                    {
                        "Key" : "Name", 
                        "Value" : "public-RouteTable" 
                    } 
                ]
            }
        },
        "PublicRouteRouteConfiguration" : {
            "Type" : "AWS::EC2::Route",
            "DependsOn" : "AttachGateway",
            "Properties" : {
                "RouteTableId" : { "Ref" : "PublicRouteTable" },
                "DestinationCidrBlock" : "0.0.0.0/0",
                "GatewayId" : { "Ref" : "internetGateway" }
            }
        },
        "RouteTableAssociationPublicSubnet1" : {
            "Type" : "AWS::EC2::SubnetRouteTableAssociation",
            "Properties" : {
                "SubnetId" : { "Ref" : "PublicSubnet1" },
                "RouteTableId" : { "Ref" : "PublicRouteTable" }
            }
        },
        "RouteTableAssociationPublicSubnet2" : {
            "Type" : "AWS::EC2::SubnetRouteTableAssociation",
            "Properties" : {
                "SubnetId" : { "Ref" : "PublicSubnet2" },
                "RouteTableId" : { "Ref" : "PublicRouteTable" }
            }
        },
        "PrivateRouteTable" : {
            "Type" : "AWS::EC2::RouteTable",
            "Properties" : {
               "VpcId" : { "Ref" : "VPC" },
               "Tags" : [ 
                    {
                        "Key" : "Name", 
                        "Value" : "private-RouteTable" 
                    } 
                ]
            }
        },
        "NATGatewayRoute" : {
            "Type" : "AWS::EC2::Route",
            "DependsOn" : "AttachGateway",
            "Properties" : {
                "RouteTableId" : { "Ref" : "PrivateRouteTable" },
                "DestinationCidrBlock" : "0.0.0.0/0",
                "NatGatewayId" : { "Ref" : "NATGatewayPublicSubnet2" }
            }
        },
        "AppLayer1PrivateSubnetAssociation" : {
            "Type" : "AWS::EC2::SubnetRouteTableAssociation",
            "Properties" : {
                "SubnetId" : { "Ref" : "PrivateSubnet1" },
                "RouteTableId" : { "Ref" : "PrivateRouteTable" }
            }
        },
        "AppLayer2PrivateSubnetAssociation" : {
            "Type" : "AWS::EC2::SubnetRouteTableAssociation",
            "Properties" : {
                "SubnetId" : { "Ref" : "PrivateSubnet2" },
                "RouteTableId" : { "Ref" : "PrivateRouteTable" }
            }
        }
```
<a href="#top">Back to top</a>
# <h3>Creating NatGateway:
```
        "NATGatewayPublicSubnet2" : {
            "Type" : "AWS::EC2::NatGateway",
            "Properties" : {
                "AllocationId" : { "Fn::GetAtt" : ["NATGatewayElasticIP", "AllocationId"] },
                "ConnectivityType": "public",
                "SubnetId" : {"Ref" : "PublicSubnet2"},
                "Tags" : [ 
                   {
                        "Key" : "Name", 
                        "Value" : "cft-nat-gateway" 
                    } 
                ]
            }
         }
 ```        
 <a href="#top">Back to top</a>
# <h3>Creating Elastic IP:
```
         "NATGatewayElasticIP" : {
            "Type" : "AWS::EC2::EIP",
            "DependsOn": "AttachGateway",
            "Properties" : {
               "Domain" : "vpc",
               "Tags":[
                    {
                        "Key":"Name",
                        "Value": "Elastic-IP"
                    }
               ]
            }
         }
```

<a href="#top">Back to top</a>
# <h3>Creating Security Group:
```
        "ALBASG": {
            "Type" : "AWS::EC2::SecurityGroup",
            "Properties" : {
                "GroupDescription" : "Wordpress-ALB",
                "GroupName" : "LoadBalancer-SG",
                "VpcId" : {"Ref": "VPC"},
                "SecurityGroupIngress" : [
                    {
                        "CidrIp" : "0.0.0.0/0",
                        "FromPort" : 80,
                        "IpProtocol" : "tcp",
                        "ToPort" : 80
                    },
                    {
                        "CidrIp" : "0.0.0.0/0",
                        "FromPort" : 443,
                        "IpProtocol" : "tcp",
                        "ToPort" : 443 
                    }
                ],
                "Tags" : [
                    {
                        "Key": "Name",
                        "Value": "LoadBalancer-SG"
                    }
                ]
            }
        },
        "BastionHostSG": {
            "Type" : "AWS::EC2::SecurityGroup",
            "Properties" : {
                "GroupDescription" : "Wordpress-Bastion-SG",
                "GroupName" : "Bastion-SG",
                "VpcId" : {"Ref": "VPC"},
                "SecurityGroupIngress" : [
                    {
                        "CidrIp" : "0.0.0.0/0",
                        "FromPort" : 22,
                        "IpProtocol" : "tcp",
                        "ToPort" : 22
                    }
                ],
                "Tags" : [
                    {
                        "Key": "Name",
                        "Value": "BastionHost-SG"
                    }
                ]
            }
        },
        "AppServerSG": {
            "Type" : "AWS::EC2::SecurityGroup",
            "Properties" : {
                "GroupDescription" : "AppLayer-SG",
                "GroupName" : "Wordpress-SG",
                "VpcId" : {"Ref": "VPC"},
                "SecurityGroupIngress" : [
                    {
                        "SourceSecurityGroupId" : {"Ref": "BastionHostSG"},
                        "FromPort" : 22,
                        "IpProtocol" : "tcp",
                        "ToPort" : 22
                    },
                    {
                        "SourceSecurityGroupId" : {"Ref": "ALBASG"},
                        "FromPort" : 80,
                        "IpProtocol" : "tcp",
                        "ToPort" : 80 
                    }
                ],
                "Tags" : [
                    {
                        "Key": "Name",
                        "Value": "AppLayer-SG"
                    }
                ]
            }
        }
```

<a href="#top">Back to top</a>
# <h3>Creating NACL:
```
        "DMZPublicNACL": {
            "Type" : "AWS::EC2::NetworkAcl",
            "Properties" : {
                "Tags" : [
                    {
                        "Key": "Name",
                        "Value": "DMZ-Public-Subnet-NACL"
                    }
                ],
                "VpcId" : {"Ref": "VPC"}
            }
        },
        "DMZ1NACLAssociation": {
            "Type" : "AWS::EC2::SubnetNetworkAclAssociation",
            "DependsOn": "DMZPublicNACL", 
            "Properties" : {
                "NetworkAclId" : {"Ref": "DMZPublicNACL"},
                "SubnetId" : {"Ref": "PublicSubnet1"}
            }
        },
        "DMZ2NACLAssociation": {
            "Type" : "AWS::EC2::SubnetNetworkAclAssociation",
            "DependsOn": "DMZPublicNACL", 
            "Properties" : {
                "NetworkAclId" : {"Ref": "DMZPublicNACL"},
                "SubnetId" : {"Ref": "PublicSubnet2"}
            }
        },
        "DMZPublicSubnetNACLIngress100": {
            "Type" : "AWS::EC2::NetworkAclEntry",
            "Properties" : {
                "CidrBlock" : "0.0.0.0/0",
                "Egress" : false,
                "NetworkAclId" : {"Ref": "DMZPublicNACL"},
                "PortRange" : {"From": "22", "To": "22"},
                "Protocol" : "6",
                "RuleAction" : "allow",
                "RuleNumber" : 100
            }
        },
        "DMZPublicSubnetNACLIngress110": {
            "Type" : "AWS::EC2::NetworkAclEntry",
            "Properties" : {
                "CidrBlock" : "0.0.0.0/0",
                "Egress" : false,
                "NetworkAclId" : {"Ref": "DMZPublicNACL"},
                "PortRange" : {"From": "80", "To": "80"},
                "Protocol" : "6",
                "RuleAction" : "allow",
                "RuleNumber" : 110
            }
        },
        "DMZPublicSubnetNACLIngress120": {
            "Type" : "AWS::EC2::NetworkAclEntry",
            "Properties" : {
                "CidrBlock" : "0.0.0.0/0",
                "Egress" : false,
                "NetworkAclId" : {"Ref": "DMZPublicNACL"},
                "PortRange" : {"From": "443", "To": "443"},
                "Protocol" : "6",
                "RuleAction" : "allow",
                "RuleNumber" : 120
            }
        },
        "DMZPublicSubnetNACLIngress130": {
            "Type" : "AWS::EC2::NetworkAclEntry",
            "Properties" : {
                "CidrBlock" : "0.0.0.0/0",
                "Egress" : false,
                "NetworkAclId" : {"Ref": "DMZPublicNACL"},
                "PortRange" : {"From": "1024", "To": "65535"},
                "Protocol" : "6",
                "RuleAction" : "allow",
                "RuleNumber" : 130
            }
        },
        "DMZPublicSubnetNACLEgress100": {
            "Type" : "AWS::EC2::NetworkAclEntry",
            "Properties" : {
                "CidrBlock" : "0.0.0.0/0",
                "Egress" : true,
                "NetworkAclId" : {"Ref": "DMZPublicNACL"},
                "PortRange" : {"From": "22", "To": "22"},
                "Protocol" : "6",
                "RuleAction" : "allow",
                "RuleNumber" : 100
            }
        },
        "DMZPublicSubnetNACLEgress110": {
            "Type" : "AWS::EC2::NetworkAclEntry",
            "Properties" : {
                "CidrBlock" : "0.0.0.0/0",
                "Egress" : true,
                "NetworkAclId" : {"Ref": "DMZPublicNACL"},
                "PortRange" : {"From": "80", "To": "80"},
                "Protocol" : "6",
                "RuleAction" : "allow",
                "RuleNumber" : 110
            }
        },
        "DMZPublicSubnetNACLEgress120": {
            "Type" : "AWS::EC2::NetworkAclEntry",
            "Properties" : {
                "CidrBlock" : "0.0.0.0/0",
                "Egress" : true,
                "NetworkAclId" : {"Ref": "DMZPublicNACL"},
                "PortRange" : {"From": "443", "To": "443"},
                "Protocol" : "6",
                "RuleAction" : "allow",
                "RuleNumber" : 120
            }
        },
        "DMZPublicSubnetNACLEgress130": {
            "Type" : "AWS::EC2::NetworkAclEntry",
            "Properties" : {
                "CidrBlock" : "0.0.0.0/0",
                "Egress" : true,
                "NetworkAclId" : {"Ref": "DMZPublicNACL"},
                "PortRange" : {"From": "1024", "To": "65535"},
                "Protocol" : "6",
                "RuleAction" : "allow",
                "RuleNumber" : 130
            }
        },
        "AppLayerPrivateNACL": {
            "Type" : "AWS::EC2::NetworkAcl",
            "Properties" : {
                "Tags" : [
                    {
                        "Key": "Name",
                        "Value": "DMZ-Private-Subnet-NACL"
                    }
                ],
                "VpcId" : {"Ref": "VPC"}
            }
        },
        "AppLayer1NACLAssociation": {
            "Type" : "AWS::EC2::SubnetNetworkAclAssociation",
            "DependsOn": "AppLayerPrivateNACL", 
            "Properties" : {
                "NetworkAclId" : {"Ref": "AppLayerPrivateNACL"},
                "SubnetId" : {"Ref": "PrivateSubnet1"}
            }
        },
        "AppLayer2NACLAssociation": {
            "Type" : "AWS::EC2::SubnetNetworkAclAssociation",
            "DependsOn": "AppLayerPrivateNACL", 
            "Properties" : {
                "NetworkAclId" : {"Ref": "AppLayerPrivateNACL"},
                "SubnetId" : {"Ref": "PrivateSubnet2"}
            }
        },
        "AppLayerPrivateSubnetNACLIngress100": {
            "Type" : "AWS::EC2::NetworkAclEntry",
            "Properties" : {
                "CidrBlock" : {"Fn::GetAtt": ["VPC","CidrBlock"]},
                "Egress" : false,
                "NetworkAclId" : {"Ref": "AppLayerPrivateNACL"},
                "PortRange" : {"From": "22", "To": "22"},
                "Protocol" : "6",
                "RuleAction" : "allow",
                "RuleNumber" : 100
            }
        },
        "AppLayerPrivateSubnetNACLIngress110": {
            "Type" : "AWS::EC2::NetworkAclEntry",
            "Properties" : {
                "CidrBlock" : {"Fn::GetAtt": ["VPC","CidrBlock"]},
                "Egress" : false,
                "NetworkAclId" : {"Ref": "AppLayerPrivateNACL"},
                "PortRange" : {"From": "80", "To": "80"},
                "Protocol" : "6",
                "RuleAction" : "allow",
                "RuleNumber" : 110
            }
        },
        "AppLayerPrivateSubnetNACLIngress120": {
            "Type" : "AWS::EC2::NetworkAclEntry",
            "Properties" : {
                "CidrBlock" : {"Fn::GetAtt": ["VPC","CidrBlock"]},
                "Egress" : false,
                "NetworkAclId" : {"Ref": "AppLayerPrivateNACL"},
                "PortRange" : {"From": "443", "To": "443"},
                "Protocol" : "6",
                "RuleAction" : "allow",
                "RuleNumber" : 120
            }
        },
        "AppLayerPrivateSubnetNACLIngress130": {
            "Type" : "AWS::EC2::NetworkAclEntry",
            "Properties" : {
                "CidrBlock" : "0.0.0.0/0",
                "Egress" : false,
                "NetworkAclId" : {"Ref": "AppLayerPrivateNACL"},
                "PortRange" : {"From": "1024", "To": "65535"},
                "Protocol" : "6",
                "RuleAction" : "allow",
                "RuleNumber" : 130
            }
        },
        "AppLayerPrivateSubnetNACLEgress100": {
            "Type" : "AWS::EC2::NetworkAclEntry",
            "Properties" : {
                "CidrBlock" : "0.0.0.0/0",
                "Egress" : true,
                "NetworkAclId" : {"Ref": "AppLayerPrivateNACL"},
                "PortRange" : {"From": "22", "To": "22"},
                "Protocol" : "6",
                "RuleAction" : "allow",
                "RuleNumber" : 100
            }
        },
        "AppLayerPrivateSubnetNACLEgress110": {
            "Type" : "AWS::EC2::NetworkAclEntry",
            "Properties" : {
                "CidrBlock" : "0.0.0.0/0",
                "Egress" : true,
                "NetworkAclId" : {"Ref": "AppLayerPrivateNACL"},
                "PortRange" : {"From": "80", "To": "80"},
                "Protocol" : "6",
                "RuleAction" : "allow",
                "RuleNumber" : 110
            }
        },
        "AppLayerPrivateSubnetNACLEgress120": {
            "Type" : "AWS::EC2::NetworkAclEntry",
            "Properties" : {
                "CidrBlock" : "0.0.0.0/0",
                "Egress" : true,
                "NetworkAclId" : {"Ref": "AppLayerPrivateNACL"},
                "PortRange" : {"From": "443", "To": "443"},
                "Protocol" : "6",
                "RuleAction" : "allow",
                "RuleNumber" : 120
            }
        },
        "AppLayerPrivateSubnetNACLEgress130": {
            "Type" : "AWS::EC2::NetworkAclEntry",
            "Properties" : {
                "CidrBlock" : {"Fn::GetAtt": ["VPC","CidrBlock"]},
                "Egress" : true,
                "NetworkAclId" : {"Ref": "AppLayerPrivateNACL"},
                "PortRange" : {"From": "1024", "To": "65535"},
                "Protocol" : "6",
                "RuleAction" : "allow",
                "RuleNumber" : 130
            }
        }
```
<a href="#top">Back to top</a>
# <h3>Creating Load Balancer:
```
         "ALB": {
            "Type" : "AWS::ElasticLoadBalancingV2::LoadBalancer",
            "Properties" : {
                "IpAddressType" : "ipv4",
                "Name" : "Application-Wordpress",
                "Scheme" : "internet-facing",
                "SecurityGroups" : [{"Ref":"ALBASG"}],
                "Subnets" : [ {"Ref": "PublicSubnet1"}, {"Ref": "PublicSubnet2"}],
                "Type" : "application"
            }
        },
        "Listener": {
            "Type" : "AWS::ElasticLoadBalancingV2::Listener",
            "Properties" : {
                "DefaultActions" : [
                    {
                        "Type": "forward",
                        "TargetGroupArn" : {"Ref": "TargetGroup"}
                    }
                ],
                "LoadBalancerArn" : {"Ref": "ALB"},
                "Port" : 80,
                "Protocol" : "HTTP"
            }
        }
```
<a href="#top">Back to top</a>
# <h3>Creating Target Group:
```
        "TargetGroup": {
            "Type" : "AWS::ElasticLoadBalancingV2::TargetGroup",
            "Properties" : {
                "HealthCheckEnabled" : true,
                "HealthCheckIntervalSeconds" : 10,
                "HealthCheckPath" : "/",
                "HealthCheckPort" : "80",
                "HealthCheckProtocol" : "HTTP",
                "HealthyThresholdCount" : 2,
                "Name" : "TargetGroup-Wordpress",
                "Port" : 80,
                "Protocol" : "HTTP",
                "Tags" : [
                    {
                        "Key": "Name",
                        "Value": "TargetGroup-Wordpress"
                    }
                ],
                "TargetType" : "instance",
                "UnhealthyThresholdCount" : 2,
                "VpcId" : {"Ref": "VPC"}
            }
        }
```
<a href="#top">Back to top</a>
# <h3>Launch Configuration:
```
        "LaunchConfiguration": {
            "Type" : "AWS::AutoScaling::LaunchConfiguration",
            "Properties" : {
                "AssociatePublicIpAddress" : false,
                "ImageId" : "ami-09d3b3274b6c5d4aa",
                "InstanceMonitoring" : true,
                "InstanceType" : "t2.micro",
                "KeyName" : {"Ref": "WebServerKey"},
                "PlacementTenancy" : "default",
                "SecurityGroups" : [ {"Ref": "AppServerSG"} ],
                "UserData" : {
                    "Fn::Base64": {
                        "Fn::Join": [
                            "",
                            [
                            "#!/bin/bash\n",
                            "yum update -y\n",
                            "wget http://dev.mysql.com/get/mysql57-community-release-el7-8.noarch.rpm\n",
                            "yum localinstall -y mysql57-community-release-el7-8.rpm\n",
                            "yum install -y mysql-community-server\n",
                            "yum install -y httpd php php-mysqlnd git\n",
                            "cd /var/www/html\n",
                            "wget https://wordpress.org/wordpress-5.1.1.tar.gz\n",
                            "tar -xzf wordpress-5.1.1.tar.gz\n",
                            "cp wordpress/wp-config-sample.php wp-config.php\n",
                            "echo '<?php phpinfo(); ?>' > /var/www/html/phpinfo.php\n",
                            "echo 'Welcome' > /var/www/html/index.html\n",
                            "groupadd www\n",
                            "usermod -aG www ec2-user\n",
                            "chown -R root:www /var/www\n",
                            "service httpd start\n",
                            "chkconfig httpd on",
                            "service "
                            ]
                        ]
                        
                    }
                }
            }
        }
```
<a href="#top">Back to top</a>
# <h3>Creating Auto-Scaling Group:
```
        "AutoScalingGroup": {
            "Type" : "AWS::AutoScaling::AutoScalingGroup",
            "DependsOn": "ALB",
            "Properties" : {
                "AutoScalingGroupName" : "AppLayer-ASC",
                "Cooldown" : "300",
                "DesiredCapacity" : "3",
                "HealthCheckGracePeriod" : 300,
                "HealthCheckType" : "ELB",
                "LaunchConfigurationName" : {"Ref": "LaunchConfiguration"},
                "MaxSize" : "5",
                "MinSize" : "2",
                "Tags" : [ 
                    {
                        "PropagateAtLaunch": true,
                        "Key": "Name",
                        "Value": "ASG-wordpress"
                    }
                 ],
                "TargetGroupARNs" : [ 
                    {
                        "Ref": "TargetGroup"
                    }
                 ],
                "VPCZoneIdentifier" : [ 
                    {
                        "Ref": "PrivateSubnet1"
                    },
                    {
                        "Ref": "PrivateSubnet2"
                    }
                 ]
            }
        },
        "ASGPolicy": {
            "Type" : "AWS::AutoScaling::ScalingPolicy",
            "Properties" : {
                "AdjustmentType" : "PercentChangeInCapacity",
                "AutoScalingGroupName" : {"Ref": "AutoScalingGroup"},
                "PolicyType" : "TargetTrackingScaling",
                "TargetTrackingConfiguration" : {
                    "TargetValue": 40.0,
                    "PredefinedMetricSpecification": {
                        "PredefinedMetricType": "ASGAverageCPUUtilization"
                    }
                }
            }
        }
```
<a href="#top">Back to top</a>
# <h3>Creating Instance:
```
        "BastionInstance": {
            "Type" : "AWS::EC2::Instance",
            "Properties" : {
                "ImageId" : "ami-09d3b3274b6c5d4aa",
                "InstanceType" : "t2.micro",
                "KeyName" : {"Ref": "WebServerKey"},
                "NetworkInterfaces" : [ 
                    {
                        "GroupSet": [
                            {"Ref": "BastionHostSG"}
                        ],
                        "AssociatePublicIpAddress": true,
                        "DeviceIndex": "0",
                        "DeleteOnTermination": true,
                        "SubnetId": {"Ref": "PublicSubnet1"}
                    }
                 ],
                "Tags" : [
                    {
                        "Key": "Name",
                        "Value": "Bastion"
                    }
                ]
            }
        }
```

# <h3>GitHub Actions:

![Screen Shot 2022-11-03 at 6 31 14 PM](https://user-images.githubusercontent.com/86754468/199846805-fb677dc5-d050-4888-a817-f1deb3e70bae.png)
