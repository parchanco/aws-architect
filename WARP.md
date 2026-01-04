# WARP.md

This file provides guidance to WARP (warp.dev) when working with code in this repository.

## Repository Overview

This is a comprehensive AWS training course documentation repository, designed as modular Markdown files optimized for import into Notion. The course covers AWS fundamentals through advanced topics including serverless, machine learning, security, and disaster recovery, with a specific focus on Odoo deployment scenarios.

**Target Audience**: The content is structured for Spanish-speaking developers working with Odoo, but applicable to general AWS learning and certification preparation (AWS Solutions Architect Associate).

## Repository Structure

The repository contains 29 numbered markdown files (00-28) representing course modules:

### Core Organization Pattern
- `00-indice.md`: Master index with learning paths and module organization
- `01-18.md`: Fundamental AWS services and best practices
- `19-28.md`: Advanced topics (serverless, ML, security, VPC, DR)
- `README.md`: Import instructions and course metadata

### Content Categories
1. **Fundamentals** (01-02): AWS intro, IAM
2. **Compute & Storage** (03-04): EC2, EBS, EFS
3. **High Availability** (05-08): ELB, ASG, RDS, ElastiCache, Route 53
4. **Storage Services** (09-13): S3, CloudFront, Snow Family
5. **Integration** (14-15): Messaging, containers
6. **Operations** (16-18): Best practices, CLI commands, glossary
7. **Advanced** (19-28): Serverless, databases, analytics, ML, security, networking, DR

## Key Content Patterns

### Documentation Style
All markdown files follow a consistent structure:
- Headers use Spanish terminology with code examples in English
- Code blocks are extensively used for AWS CLI commands, architecture diagrams (ASCII), and configuration examples
- Real-world Odoo deployment scenarios are integrated throughout
- Navigation links at the bottom of each file (e.g., `**Siguiente**: [XX - Title]`)

### Technical Examples
When working with examples in this repository:
- AWS CLI commands use standard AWS CLI v2 syntax
- Region references default to `eu-west-1` (Ireland) for European context
- Architecture diagrams use ASCII art with arrow notation (→, ├─, └─)
- Code examples include Python (Lambda), bash (CLI), and JSON (policies, configs)

### Odoo-Specific Context
The course includes production architecture recommendations specifically for Odoo:
- Multi-AZ RDS PostgreSQL deployments
- ElastiCache Redis for session management
- EFS for shared filestore
- S3 for attachments and backups
- ALB with Auto Scaling Groups
- See `16-best-practices.md` for complete architecture diagram

## Common Operations

### Viewing Course Content
Read specific modules directly:
```bash
# View a specific topic
cat 03-ec2.md

# Search across all course content
grep -r "Lambda" *.md

# View the index
cat 00-indice.md
```

### Finding Information
The repository structure makes it easy to locate specific AWS services:
- IAM: `02-iam.md` (basic), `25-iam-advanced.md` (advanced)
- EC2: `03-ec2.md` (instances), `04-ec2-storage.md` (storage)
- S3: `09-s3-intro.md`, `10-s3-advanced.md`, `11-s3-security.md`
- Serverless: `19-serverless.md`, `20-serverless-architectures.md`
- Networking: `08-route53.md`, `27-vpc.md`
- Security: `26-security.md`

Use `00-indice.md` or `README.md` for complete file mapping.

### AWS CLI Reference
`17-cli-commands.md` contains a comprehensive collection of practical AWS CLI commands organized by service (IAM, EC2, S3, RDS, CloudWatch, Auto Scaling, Route 53, ELB, SSM, Cost Explorer).

## Content Maintenance Guidelines

### Editing Course Material
When modifying course content:
- Maintain numbered file naming convention (XX-topic.md)
- Preserve Spanish headers with English code examples
- Keep ASCII diagrams aligned with 4-space or similar indentation
- Update `00-indice.md` if adding/removing modules
- Maintain navigation links at file endings
- Use consistent code block formatting with language identifiers

### Adding New Modules
If extending the course:
1. Follow sequential numbering (29+)
2. Add entry to `00-indice.md` under appropriate module category
3. Update learning paths in `00-indice.md` if relevant
4. Include navigation footer pointing to previous/next modules
5. Update `README.md` structure section if adding major new category

### AWS Service Documentation Pattern
Each service file typically includes:
- Brief service description
- Key characteristics (pricing, limits, features)
- Practical code examples (CLI commands, Python snippets)
- Best practices section (✅/❌ format)
- Real-world use cases
- Integration with other AWS services

## Important Notes

### No Build System
This is a pure documentation repository with no build, test, or deployment commands. There are no package managers, dependencies, or compilation steps.

### Notion Import Optimization
The markdown files are specifically formatted for Notion import:
- Clean markdown without complex tables
- ASCII art for diagrams (Notion-compatible)
- Modular single-file structure for independent imports
- Use backticks for code blocks rather than images

### Version Control
Repository uses Git but is essentially a snapshot (single initial commit). When making changes, commit messages should be descriptive about which module was modified.

### Language Context
Content is in Spanish with technical terms in English (standard AWS terminology). When generating new content or examples, maintain this bilingual pattern.

## Learning Paths (from Index)

- **Odoo Development Essential**: Modules 1-8, 16-17
- **Production Complete**: Modules 1-17
- **AWS Solutions Architect Certification**: All modules (1-28)
- **Serverless/Modern Apps**: Modules 1-3, 6-7, 9-11, 14, 19-20
- **Data Engineering**: Modules 6, 9-11, 21-22

## Reference Architecture (from best-practices.md)

The recommended production Odoo architecture on AWS:
```
Route 53 → CloudFront → WAF → ALB (Multi-AZ) 
    → Auto Scaling Group (EC2 in Private Subnets)
    → RDS PostgreSQL Multi-AZ
    → ElastiCache Redis
    → EFS (shared filestore)
    → S3 (attachments, backups)
    → CloudWatch + SNS (monitoring/alerts)
```

## Cost Optimization Context

The course emphasizes AWS cost optimization throughout:
- Reserved Instances for production (40-75% savings)
- Spot Instances for batch jobs (70-90% savings)
- S3 Lifecycle policies (Standard → IA → Glacier)
- Right-sizing EC2 instances
- Free Tier maximization for learning

See `16-best-practices.md` for comprehensive cost optimization strategies.
