# 18 - Glosario de Términos AWS

## A

**ACL** - Access Control List  
**ACU** - Aurora Capacity Unit  
**ALB** - Application Load Balancer  
**AMI** - Amazon Machine Image  
**ARN** - Amazon Resource Name  
**ASG** - Auto Scaling Group  
**AZ** - Availability Zone  

## B

**Bucket** - Container para objects en S3  
**Burstable** - Instancias que pueden burst CPU (T family)  

## C

**CDN** - Content Delivery Network  
**CIDR** - Classless Inter-Domain Routing  
**CLB** - Classic Load Balancer  
**CloudFront** - CDN de AWS  
**CloudTrail** - Audit log de API calls  
**CloudWatch** - Monitoring y logging  
**CMK** - Customer Master Key (KMS)  
**CORS** - Cross-Origin Resource Sharing  
**CRR** - Cross-Region Replication (S3)  

## D

**DLM** - Data Lifecycle Manager  
**DR** - Disaster Recovery  
**DynamoDB** - Base de datos NoSQL  

## E

**EBS** - Elastic Block Store  
**EC2** - Elastic Compute Cloud  
**ECR** - Elastic Container Registry  
**ECS** - Elastic Container Service  
**EFS** - Elastic File System  
**EKS** - Elastic Kubernetes Service  
**ELB** - Elastic Load Balancing  
**EMR** - Elastic MapReduce  
**ENI** - Elastic Network Interface  

## F

**Fargate** - Serverless compute para containers  
**FIFO** - First In First Out  
**FSx** - Managed file systems  

## G

**Glacier** - S3 storage class para archival  
**gp2/gp3** - General Purpose SSD volumes  
**GWLB** - Gateway Load Balancer  

## H

**HA** - High Availability  
**HDD** - Hard Disk Drive  
**Hosted Zone** - Container para DNS records (Route 53)  

## I

**IA** - Infrequent Access (S3 storage class)  
**IAM** - Identity and Access Management  
**IOPS** - Input/Output Operations Per Second  
**io1/io2** - Provisioned IOPS SSD volumes  

## K

**KMS** - Key Management Service  

## L

**Lambda** - Serverless compute  

## M

**MFA** - Multi-Factor Authentication  
**Multi-AZ** - Deployment across multiple AZs  

## N

**NAT** - Network Address Translation  
**NLB** - Network Load Balancer  
**NFS** - Network File System  

## O

**OAI** - Origin Access Identity (CloudFront)  

## P

**PaaS** - Platform as a Service  
**Policy** - Documento JSON de permisos (IAM)  
**Pre-signed URL** - URL temporal para S3 objects  

## R

**RDS** - Relational Database Service  
**Region** - Ubicación geográfica AWS  
**Reserved Instance** - Instancia con compromiso 1-3 años  
**Role** - Permisos para servicios AWS (IAM)  
**RPO** - Recovery Point Objective  
**RTO** - Recovery Time Objective  

## S

**S3** - Simple Storage Service  
**SES** - Simple Email Service  
**SG** - Security Group  
**Snapshot** - Backup point-in-time (EBS)  
**SNS** - Simple Notification Service  
**Spot Instance** - Instancia con descuento pero interrumpible  
**SQS** - Simple Queue Service  
**SSD** - Solid State Drive  
**SSE** - Server-Side Encryption  
**STS** - Security Token Service  

## T

**Target Group** - Grupo de targets para Load Balancer  
**TTL** - Time To Live  

## V

**VPC** - Virtual Private Cloud  
**VPN** - Virtual Private Network  

## W

**WAF** - Web Application Firewall  
**WORM** - Write Once Read Many  

---

## Conceptos Importantes

**Least Privilege**: Dar solo los permisos mínimos necesarios

**Multi-AZ**: Deployment en múltiples Availability Zones para HA

**Serverless**: No gestionas servidores (Lambda, Fargate, Aurora Serverless)

**Pay-as-you-go**: Pagas solo por lo que usas

**Reserved**: Compromiso 1-3 años para descuento

**Spot**: Instancias con descuento pero AWS puede quitártelas

**On-Demand**: Sin compromiso, precio completo

**Storage Classes**: Diferentes niveles de S3 según frecuencia de acceso

**Lifecycle Policy**: Automatización de transiciones entre storage classes

**Edge Location**: Punto de presencia para CloudFront

**Replication**: Copia automática de data (RDS, S3, Aurora)

**Encryption at-rest**: Data cifrada en disco

**Encryption in-transit**: Data cifrada durante transferencia

---

**Fin del Curso**

¡Felicidades por completar el curso AWS!

Próximos pasos:
1. Practica con AWS Free Tier
2. Implementa arquitectura para tu proyecto
3. Considera certificación AWS (Solutions Architect Associate)
4. Sigue aprendiendo con AWS re:Invent videos
