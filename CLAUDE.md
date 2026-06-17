# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A **static portfolio website** (HTML + CSS only) used as the **DMI (DevOps Micro Internship) Week 1** exercise. It's a Single-page site with sections for About, Services, Courses, Books, Community, and Contact. Will be deployed to aws using S3 and CloudFront, provisioned with Terraform.

## Architecture

### Application (Static Site)
- **index.html** - single-page portfolio: navbar, hero, about, services, courses, books, community, contact, footer
- **style.css** - flat stylesheet, no preprocessor, uses mobile-first approach. (Breakpoints at 900px, 768px, and 600px)
- **privacy.html / terms.html** - standalone legal pages linked from the footer with inline styles
- **images** - Static assets (logo, profile photo, course thumbnails, signature, hero background)
- Pure HTML5 plus CSS3, No JavaScript, No build step, No frameworks, no test suite, and no linter configured.

### Infrastructure ('terraform/')
- AWS S3 bucket for static site hosting (private OAC-based)
- CloudFront distribution as CDN with S3 origin
- GitHub OIDC provider + IAM role for keyless CI/CD auth
- Terraform state stored in S3 backend with DynamoDB locking
- All resources tagged with 'Project' and 'Environment'

### CI/CD ('.github/workflows/')
- GitHub Actions workflow triggers on push to 'main'
- Syncs site files to S3, then invalidates CloudFront cache
- Uses OIDC for AWS authentication (no long-lived keys)

## MCP Servers (`.mcp.json`)

Two MCP servers are configured for Claude Code:
- **aws** (`awslabs.aws-api-mcp-server`) — Direct AWS API access for querying and managing resources
- **terraform** (`hashicorp/terraform-mcp-server`) — Terraform operations via Docker, workspace mounted at `/workspace`

AWS credentials and region are configured in `.claude/settings.local.json` (gitignored), not in `.mcp.json`. This keeps secrets out of version control and provides a single source of truth for all tools.

## Custom Agents (`.claude/agents/`)

This project has 4 specialized subagents. Use them by name when delegating tasks:
- **tf-writer** — generates Terraform code (has Write access + project memory)
- **security-auditor** — audits TF for security issues (Read-only, Sonnet)
- **cost-optimizer** — reviews infra cost (Read-only, Haiku)
- **drift-detector** — detects state drift (Bash, Haiku)

## Skills (`.claude/skills/`)

All infrastructure and deployment tasks are handled via skills. Do not write Terraform or CI/CD code manually — use the appropriate skill. Action skills have `disable-model-invocation: true` (manual only). The `project-scope` skill has `user-invocable: false` (auto-loaded by Claude as background knowledge).

```
/scaffold-terraform [region] [name]  → Generate all Terraform files (uses tf-writer agent)
/scaffold-cicd [aws-account-id]      → Generate GitHub Actions + OIDC IAM role
/tf-plan                             → Run terraform plan + risk analysis
/tf-apply                            → Run terraform apply + verify
/deploy                              → Sync S3 + invalidate CloudFront
/infra-status                        → Health dashboard of all resources
/infra-audit                         → Parallel security + cost + drift audit (forked context)
/setup-gh-actions [create|validate]  → Create or validate CI workflow
/tf-destroy                          → Safe destroy with confirmation
project-scope                        → Background knowledge: AWS service constraints (auto-loaded)
/commit                              → Auto-generate commit message (built-in)
/compact                             → Compress long conversation context (built-in)
```

## Commands

```bash
# Terraform
cd terraform && terraform init
cd terraform && terraform plan
cd terraform && terraform apply

# Local preview
open index.html

# Manual S3 sync (CI does this automatically)
aws s3 sync . s3://$BUCKET_NAME --exclude "terraform/*" --exclude ".git/*" --exclude ".github/*" --exclude "*.md" --exclude ".claude/*"
```

## Safety Layers
1. **UserPromptSubmit hook** — catches destructive intent ("delete all", "nuke", "wipe") before Claude starts
2. **PreToolUse hook** — blocks dangerous commands (terraform destroy, aws s3 rm) at execution time
3. **Permissions** — auto-allows safe reads, blocks IAM and rm -rf
4. **PostToolUse hook** — logs all terraform apply executions to `.claude/deploy.log`

## Conventions
- Terraform files use `terraform/` directory with standard layout (main.tf, variables.tf, outputs.tf)
- GitHub Actions uses OIDC — no stored AWS access keys
- All infrastructure changes go through Terraform — never modify AWS resources manually
- Site content changes deploy automatically via GitHub Actions on push to main


