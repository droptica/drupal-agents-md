# Drupal AGENTS.md Template

[AGENTS.md](https://agents.md) template for Drupal projects. AI reads it once and knows your architecture, standards, and workflows.

**Assumes DDEV-based development.**

## Quick Start

**1.** Download template:

```bash
curl -o AGENTS-TEMPLATE.md https://raw.githubusercontent.com/droptica/drupal-agents-md/main/AGENTS-TEMPLATE.md
```

**2.** Copy this prompt to your AI tool:

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
- [ ] Remove commented-out sections if not applicable (multisite, commerce, headless, AI, config split)
- [ ] Remove entire sections that don't apply to project (e.g., no SCSS? remove SCSS workflow)
- [ ] Remove tools/patterns not used in project

## Phase 5: Propose Additions
- [ ] List project-specific things found during discovery that are NOT in template
- [ ] Present as numbered list for user to choose (e.g., "1. Add Solr search section", "2. Add custom queue workers docs")
- [ ] Wait for user to respond with numbers to add
- [ ] Add selected items to AGENTS.md

## Phase 6: Finalize
- [ ] Save as AGENTS.md in project root
- [ ] Show summary: found / replaced / removed / added
- [ ] Delete the temporary `./tmp/agents-md-tasks.md` file

--- CHECKLIST END ---

2. Read AGENTS-TEMPLATE.md and follow all HTML comment guides (HOW TO DISCOVER...)
3. Execute all discovery steps from Phase 1
4. **STOP after Phase 1** - show me what you found and wait for my confirmation before proceeding
5. If you're unsure about ANY value (prefix, theme name, etc.) - ASK, don't guess
6. Replace all placeholders from Phase 2
7. Fill sections from Phase 3
8. Cleanup from Phase 4
9. **STOP at Phase 5** - show numbered list of proposed additions, wait for my response
10. Add selected items, then finalize (Phase 6)

After customization, show me:
- What you found in the project
- What you replaced/customized
- What you removed
- What you added (from proposals)
```

**3.** Review generated AGENTS.md and start coding!

## Template Covers

- **Environment:** DDEV, Git workflow, Composer
- **Quality:** PHPStan, PHPCS, PHPUnit, Codeception, Xdebug
- **Development:** Code standards, entities, modules, forms, database
- **Modern Drupal:** Headless/API, SEO, multilingual
- **Frontend:** Themes, SCSS, JS/CSS optimization, caching
- **Operations:** Config management, security, performance, troubleshooting

## Manual Customization

Follow checklist at end of AGENTS-TEMPLATE.md.

## Compatible AI Tools

Cursor • Copilot • Claude Code • Codex • Aider • Gemini CLI • Roo Code • Zed • Devin • [more](https://agents.md)

## Contributing

Issues and PRs welcome.

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
