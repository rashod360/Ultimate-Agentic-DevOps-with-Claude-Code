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



