# DotNet Core Container to Fargate 

1 - TODO Steps
```
1. Create a sample CDK application
2. Add some Nuget packages
3. Add a VPC
4. Add a Cluster
5. Add a load balanced Fargate service
6. Create a smaple dotnet application
7. Create a Docker File
8. Replace Docker image with Task Definition
9. Add a certificate
10. Delete Everything
```

2 - Make a directory:

```
mkdir demoDotNetApplication
```

3 - Go into that directory:

```
cd demoDotNetApplication
```

4 - Use CDK

```
cdk init sample-app --language csharp
```

5 - Go back to the correct folder and open in Rider

```
cd ../

```

6 - Add the following nuget packages

```
using Amazon.CDK.AWS.EC2;
using Amazon.CDK.AWS.ECS;
using Amazon.CDK.AWS.ECS.Patterns;

```

7 - Check the build

```
dotnet build src cdk synth
```

8 - Add VPC
```
var vpc = new Vpc(this, "MyVpc", new VpcProps
{
    MaxAzs = 3 // Default is all AZs in region
});
```

9 - Add a Cluster
```
var cluster = new Cluster(this, "MyCluster", new ClusterProps
{
    Vpc = vpc
});
```
10 - Add a load balanced Fargate service
```
var serviceProps = new ApplicationLoadBalancedFargateServiceProps()
{
    Cluster = cluster,          // Required
    DesiredCount = 6,           // Default is 1
    TaskImageOptions = new ApplicationLoadBalancedTaskImageOptions
    {
        Image = ContainerImage.FromRegistry("amazon/amazon-ecs-sample")
    },
    MemoryLimitMiB = 2048,      // Default is 256
    PublicLoadBalancer = true   // Default is false
};

// Create a load-balanced Fargate service and make it public
new ApplicationLoadBalancedFargateService(this, "MyFargateService",
    serviceProps);
```

11 - Deploy and show the website working
```
cdk deploy
```

12 - Explain that PHP is not what I want. I'd like DotNet.

```
dotnet new webapp -o Website 
```

13 - Go into Folder

```
cd Website
```

14 - Run the website 

```
dotnet run
```

15 - Add Docker File

```
touch Dockerfile
```

16 - Add to Docker file

```
FROM mcr.microsoft.com/dotnet/core/sdk:3.1 AS build
WORKDIR /build
COPY . .
RUN dotnet restore
RUN dotnet publish -c Release -o /app

FROM mcr.microsoft.com/dotnet/core/aspnet:3.1 AS final
WORKDIR /app
COPY --from=build /app .
ENTRYPOINT [ "dotnet", "Website.dll" ]
```

17 - Create a task definition
```
var taskDefinition = new FargateTaskDefinition(this,"FargateTaskDefinition");
```
18 - Get a container from a folder
```
var containerOptions = new ContainerDefinitionOptions
{
    Image = ContainerImage.FromAsset("Website")
};
```
19 - Add Port Mapping
```
var portMapping = new PortMapping()
{
    ContainerPort = 80,
    HostPort = 80
};
```
20 - Add the container to the definition and add the port mapping
```
taskDefinition
    .AddContainer("Container", containerOptions)
    .AddPortMappings(portMapping);
```
21 - Replace TaskImageOptions with TaskDefinition
```
TaskDefinition = taskDefinition,
```
22 - Add a certificate
```
Certificate = Certificate.FromCertificateArn(this, "MyCertificate", "arn:aws:acm:eu-west-1:365489315573:certificate/2141211b-630e-4fb5-a8ac-fff9cbcd65f3"),
                
```

23 - Add a domain

```
DomainName = "fargate.thebeebs.net",
DomainZone = new HostedZone(this,"myHostedZone",new HostedZoneProps
{
    ZoneName = "thebeebs.net"
})
```

24 - Delete the lot

```
cdk destroy
```

