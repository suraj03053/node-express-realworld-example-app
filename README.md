# RealWorld Backend — Node/Express/Prisma on AWS Lambda

## Application Overview
A REST API backend for the RealWorld (Conduit) application — a Medium.com clone. Built with Node.js, Express, TypeScript, and Prisma ORM.

## Architecture and Deployment
- **Backend**: AWS Lambda + API Gateway (serverless)
- **Frontend**: AWS S3 + Static Website Hosting
- **Database**: Neon PostgreSQL (serverless)
- **CI/CD**: GitHub Actions
- **Monitoring**: AWS CloudWatch

### Why these choices
Lambda was chosen because it scales to zero when idle, meaning zero cost during inactivity. Neon was chosen as it is a serverless PostgreSQL provider compatible with Prisma ORM — DynamoDB was initially considered but is incompatible with Prisma's SQL-based query engine.

## Deployment Cost
All services used are on free tiers:
- AWS Lambda: 1M free requests/month
- AWS S3: 5GB free storage
- Neon: 0.5GB free database
- GitHub Actions: 2000 free minutes/month

Estimated monthly cost: $0

## Prerequisites
- Node.js 18+
- AWS CLI configured
- Serverless Framework v3
- PostgreSQL database URL (Neon recommended)

## Deployment Steps
1. Clone the repo
2. Run `npm install`
3. Create `.env` with `DATABASE_URL` and `JWT_SECRET`
4. Run `npx prisma generate && npx prisma db push && npx prisma db seed`
5. Run `serverless deploy`

## CI/CD Workflow
Every push to `master` triggers GitHub Actions which:
1. Checks out code
2. Installs dependencies
3. Builds TypeScript
4. Runs Snyk security scan
5. Deploys to AWS Lambda via Serverless Framework

## Versioning Strategy
Git tags are used for versioning (e.g. v1.0.0, v1.0.1). Every deployment is tied to a specific commit SHA visible in GitHub Actions logs.

## Security Approach
- Snyk scans dependencies on every push
- JWT authentication for protected routes
- IAM roles with least-privilege access
- Environment variables stored as GitHub Secrets, never in code

## Monitoring and Observability
- AWS CloudWatch collects Lambda logs and metrics
- CloudWatch alarm triggers email alert when Lambda errors exceed threshold
- Logs viewable via: `aws logs tail /aws/lambda/realworld-backend-dev-api --region us-east-2`

## Challenges Encountered
1. **Old frontend (Node 16)**: Used nvm to pin Node version for the legacy React app
2. **Prisma + Lambda**: Added `rhel-openssl-1.0.x` binary target to schema.prisma for Linux compatibility
3. **Package size**: Reduced Lambda package from 71MB to 1.8MB using serverless patterns to exclude unnecessary files
4. **Port conflict**: Backend configured to use PORT environment variable to avoid EADDRINUSE errors