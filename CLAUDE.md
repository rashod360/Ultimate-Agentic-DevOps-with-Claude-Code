# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A **static portfolio website** (HTML + CSS only) used as the **DMI (DevOps Micro Internship) Week 1** exercise. It's a Single-page site with sections for About, Services, Courses, Books, Community, and Contact. Will be deployed to aws using S3 and CloudFront, provisioned with Terraform.

## Key Conventions

- "CSS uses mobile-first approach. Breakpoints at 900px, 768px, and 600px."
- "No JavaScript in this project."
- "All images go in the images/ directory"

## Local Preview

Open `index.html` directly in a browser, or serve the directory statically:

```bash
python3 -m http.server 8080
```

The production target is Nginx on Ubuntu, served at `http://<public-ip>`.

## Mandatory Student Customization (DMI Rule)

Every student MUST edit the footer in `index.html` before deploying, replacing the default credit line with their ownership proof:

```html
<p><strong>Deployed by:</strong> DMI Cohort 2 | Your Name | Group # | Week 1 | DD-MM-YYYY</p>
```

This line must appear in the browser screenshot submitted as deployment proof.

## File Layout

- `index.html` — single-page portfolio: navbar, hero, about, services, courses, books, community, contact, footer
- `style.css` — flat stylesheet, no preprocessor
- `privacy.html` / `terms.html` — standalone legal pages linked from the footer
- `images/` — logo, profile photo, course thumbnails, signature

## Architecture

This is a pure static site with **Pure HTML5 plus CSS3. No JavaScript. No build step. No frameworks**. There is no build step, no test suite, and no linter configured.

- Navigation between sections is via anchor IDs on each `<section>` (`#home`, `#about`, `#services`, `#courses`, `#book`, `#community`, `#contact`).
- The navbar buttons and hamburger use inline `onclick` handlers (`goToSection('...')`, `toggleMenu()`) but **those JS functions are not currently defined anywhere in the repo** — there is no `<script>` block in `index.html`. If a student reports that nav clicks or the mobile menu do nothing, that is the cause; adding a small inline `<script>` defining those two helpers is the fix.
- The footer's `<span id="year">` is also empty and would normally be filled by JS — it currently renders blank.
