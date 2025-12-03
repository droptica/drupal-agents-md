# Drupal AGENTS.md Template

**Make AI code 10x better on your Drupal project.**

Production-ready [AGENTS.md](https://agents.md) template that gives AI coding assistants deep understanding of your Drupal architecture, coding standards, and workflows. Works with Cursor, GitHub Copilot, Claude Code, and 20+ AI tools.

Instead of explaining your project structure every time, AI reads AGENTS.md once and knows:
- Your Drupal version, modules, entities, and custom code
- Development workflow, Git strategy, testing approach
- Code quality standards, security requirements, performance optimization
- Multilingual setup, SEO configuration, API architecture

**Result:** AI provides accurate, project-specific code from day one.

**Note:** This template assumes DDEV-based development environment.

## Quick Start

**Step 1:** Download the template to your project root:

```bash
curl -o AGENTS-TEMPLATE.md https://raw.githubusercontent.com/droptica/drupal-agents-md/main/AGENTS-TEMPLATE.md
```

**Step 2:** Copy this prompt to your AI tool (Cursor, Copilot, etc.):

```
I need you to customize AGENTS-TEMPLATE.md for my Drupal project and save it as AGENTS.md.

Please:
1. Before starting, create a temporary file named `./tmp/agents-md-tasks.md` and copy the following checklist into it. Use this file to track your progress by marking tasks as `[x]` when completed. Update this file after each major step.

--- CHECKLIST START (copy to ./tmp/agents-md-tasks.md) ---

# AGENTS.md Customization Checklist

## Phase 1: Project Discovery
- [ ] Check composer.json for project name, Drupal version, PHP version
- [ ] Check if DDEV is used (.ddev/config.yaml exists)
- [ ] Check web root directory name (web/, docroot/, html/, or other)
- [ ] Check if multisite (web/sites/ or docroot/sites/ has multiple directories)
- [ ] Check languages (config/sync/language.entity.*.yml or via drush if available)
- [ ] Check if Commerce is installed (composer.json has drupal/commerce)
- [ ] Check if JSON:API or GraphQL is used (composer.json has drupal/jsonapi_extras or drupal/graphql)
- [ ] List all custom modules in web/modules/custom/ or docroot/modules/custom/
- [ ] Check theme name (web/themes/custom/ or docroot/themes/custom/)
- [ ] Discover theme structure (active theme, file organization, template suggestions)
- [ ] Check if theme uses Single Directory Components (SDC)
- [ ] List preprocess functions in the active theme
- [ ] List content types from config/sync/node.type.*.yml
- [ ] Discover other entity types (paragraphs, media, taxonomies, custom entities)

## Phase 2: Replace Placeholders
- [ ] Replace [PROJECT_NAME] - from composer.json
- [ ] Replace [VERSION] - Drupal version from composer.json
- [ ] Replace [PHP_VERSION] - from composer.json require
- [ ] Replace [REPOSITORY_URL] - leave empty or check .git/config
- [ ] Replace web/ - replace with actual web root directory name throughout entire file
- [ ] Replace [PREFIX] - determine custom module prefix (e.g., myproject_, custom_)
- [ ] Replace [THEME_NAME] - actual theme directory name

## Phase 3: Fill Project-Specific Sections
- [ ] Fill "Current Setup" section based on findings
- [ ] List installed custom modules
- [ ] Document multisite configuration if exists
- [ ] List configured languages
- [ ] Note if Commerce, AI modules, or other integrations are present
- [ ] Document content types and other entities in "Drupal Entities Structure" section

## Phase 4: Cleanup
- [ ] Remove commented-out sections if not applicable (multisite)
- [ ] Remove commented-out sections if not applicable (commerce)
- [ ] Remove commented-out sections if not applicable (headless)
- [ ] Remove commented-out sections if not applicable (AI integration)
- [ ] Remove commented-out sections if not applicable (config split/ignore)

## Phase 5: Finalize
- [ ] Save as AGENTS.md in project root
- [ ] Show what was found, replaced/customized, and removed/kept commented
- [ ] Delete the temporary `./tmp/agents-md-tasks.md` file

--- CHECKLIST END ---

2. Read AGENTS-TEMPLATE.md and follow all HTML comment guides (HOW TO DISCOVER...)
3. Execute all discovery steps from Phase 1
4. **STOP after Phase 1** - show me what you found and wait for my confirmation before proceeding
5. If you're unsure about ANY value (prefix, theme name, etc.) - ASK, don't guess
6. Replace all placeholders from Phase 2
7. Fill sections from Phase 3
8. Cleanup from Phase 4
9. Finalize from Phase 5

After customization, show me:
- What you found in the project
- What you replaced/customized
- Any sections you removed or kept commented
```

**Step 3:** Review the generated AGENTS.md and start coding!

## What This Template Includes

### Development Environment
- **DDEV Setup** - Container-based development with custom commands
- **Git Workflow** - Branching strategy, commit standards, code review process
- **Composer Management** - Dependencies, patches, custom scripts

### Code Quality & Testing
- **Code Quality Tools** - PHPStan, PHP CodeSniffer, Rector, Upgrade Status
- **Testing** - PHPUnit (Unit, Kernel, Functional), Codeception (Acceptance)
- **Debugging** - Xdebug, logs, performance profiling

### Drupal Development
- **Code Standards** - SOLID principles, PSR-4, Drupal best practices
- **Entity System** - Content types, paragraphs, media, taxonomies, custom entities
- **Module Development** - Structure, patterns, dependency injection
- **Form API** - Building, validation, submission handlers
- **Database API** - Queries, migrations, performance

### Modern Drupal Features
- **Headless/API-First** - JSON:API, GraphQL, OAuth, CORS configuration
- **SEO & Structured Data** - Schema.org, Open Graph, sitemaps, hreflang
- **Multilingual** - Translation workflows, language detection, i18n best practices

### Frontend & Theme
- **Theme Development** - SCSS/Gulp workflow, asset libraries, responsive design
- **JavaScript & CSS** - Aggregation, dependencies, optimization
- **Performance** - Caching (Redis, BigPipe), CDN, Core Web Vitals

### Operations
- **Configuration Management** - Config export/import, Config Split
- **Security** - Best practices, updates, hardening, monitoring
- **Performance Monitoring** - Cache management, database optimization
- **Troubleshooting** - Common issues and solutions

### Documentation
- **Tasks & Problems Log** - Integrated documentation of changes and issues
- **Entity Documentation** - JSON-based entity structure reference
- **Customization Checklist** - Step-by-step template adaptation guide

## Manual Customization (Optional)

If you prefer to customize manually, follow the checklist at the end of AGENTS-TEMPLATE.md.

## Compatible AI Tools

Cursor • VS Code + Copilot • Claude Code • Codex • Aider • Jules • Gemini CLI • Roo Code • Zed • Devin

See [agents.md](https://agents.md) for full list.

## FAQ / Common Issues

*This section will be updated based on community feedback and common questions.*

*Have a question or issue? [Open an issue](https://github.com/droptica/drupal-agents-md/issues) and we'll add it here.*

## Contributing

Found a bug? Have a suggestion? Open an issue or PR!

## License

MIT License - see [LICENSE](LICENSE)

---

## About Droptica

**Built with ❤️  by [Droptica](https://www.droptica.com) 🇵🇱**

Solid Open Source solutions for ambitious companies.

**What we do:**

- **Create:** Open Intranet, Droopler CMS, Druscan
- **AI Development:** AI chatbots (RAG), autonomous agents, OpenAI/Claude integrations, custom AI models, CMS content automation & translation, workflow automation
- **Customize:** Drupal, Mautic, Sylius, Symfony
- **Support & maintain:** Security, updates, training, monitoring 24/7

**Trusted by:** Corporations • SMEs • Startups • Universities • Government
