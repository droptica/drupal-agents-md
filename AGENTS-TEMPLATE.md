# AGENTS.md

AI coding guide for [PROJECT_NAME] Drupal project.

## AI Response Requirements

**Communication style:**
- Code over explanations - provide implementations, not descriptions
- Be direct, skip preambles
- Assume Drupal expertise - no over-explaining basics
- Suggest better approaches with code
- Show only changed code sections with minimal context
- Complete answers in one response when possible
- Use Drupal APIs, not generic PHP
- Ask if requirements are ambiguous

**Response format:**
- Production-ready code with imports/dependencies
- Inline comments explain WHY, not WHAT
- Include full file paths
- Proper markdown code blocks

## Project Overview

- **Platform**: Drupal [VERSION] [single site | multisite]
- **Context**: [Brief description]
- **Architecture**: [Architecture description]
- **Security**: [Security level]
- **Languages**: [single | multilingual: list languages]
- **Custom Entities**: [List if any]
- **Role System**: [Describe roles]
- **Use Cases**: [Main use cases]

<!-- MULTISITE (uncomment if applicable)
### Multi-site Setup
Sites: [site1.domain], [site2.domain], [site3.domain]
- Separate databases per site
- Shared codebase
- Config in `/config/[site.domain]/`
-->

<!-- AI INTEGRATION (uncomment if applicable)
### AI Integration
Provider: [OpenAI | Other] | Modules: [List modules]
Uses: content generation, translation, summarization
API keys in `.ddev/.env`
-->

<!-- COMMERCE (uncomment if applicable)
### E-commerce
Platform: Drupal Commerce [VERSION]
Gateways: [Stripe, PayPal, TPay]
Features: catalog, cart, checkout, orders, payments
API keys in environment variables
-->

## Date Verification Rule

**CRITICAL**: Before writing dates to `.md` files, run `date` command first.
Never use example dates (e.g., "2024-01-01") - always use actual system date.

## Git Workflow

**Branches**: `main` (prod), `staging` (test), `feature/*`, `bugfix/*`, `hotfix/*`, `release/*`

**Flow**: `staging` → `feature/name` → PR → merge to `staging` → eventually `main`

**Hotfix**: `main` → `hotfix/name` → merge to `main` + `staging`, tag release

**Commit format**: `[type]: description` (max 50 chars)
Types: `feat`, `fix`, `docs`, `style`, `refactor`, `perf`, `test`, `chore`, `config`

**Before PR**: Run `phpcs`, `phpstan`, tests, `drush cex`

**Don'ts**: No direct commits to main/staging, no `--force` on shared branches, no credentials in code

**.gitignore essentials**:
```gitignore
web/core/
web/modules/contrib/
web/themes/contrib/
vendor/
web/sites/*/files/
web/sites/*/settings.local.php
.ddev/
node_modules/
.env
```

## Development Environment

**Web root**: `web/` (or `docroot/`, `html/` - check for dir with `index.php` and `core/`)

### Setup
```bash
git clone [REPOSITORY_URL] && cd [PROJECT_DIR]
ddev start && ddev [BUILD_COMMAND]
```

**DDEV**: `ddev ssh` (container), `ddev describe` (info), `ddev drush [cmd]`

### Custom DDEV Commands
Location: `.ddev/commands/host/[name]`
**WARNING**: Don't use `## #ddev-generated` comments - they break command recognition.

### Drush Commands

```bash
# Core commands
ddev drush status                    # Status check
ddev drush cr                        # Cache rebuild
ddev drush cex                       # Config export
ddev drush cim                       # Config import
ddev drush updb                      # Database updates

# Database & PHP eval
ddev drush sql:query "SELECT * FROM node_field_data LIMIT 5;"
ddev drush php:eval "echo 'Hello World';"

# Test services and entities
ddev drush php:eval "var_dump(\Drupal::hasService('entity_type.manager'));"
ddev drush php:eval "\$node = \Drupal::entityTypeManager()->getStorage('node')->load(1); var_dump(\$node->getTitle());"
ddev drush php:eval "var_dump(\Drupal::config('system.site')->get('name'));"
ddev drush php:eval "var_dump(\Drupal::service('custom_module.service_name'));"

# Quick setup (pull from platform)
ddev pull platform -y && ddev drush cim -y && ddev drush cr && ddev drush uli
```

<!-- MULTISITE: Use `ddev drush -l [site.ddev.site] [cmd]` -->

### Composer
```bash
ddev composer outdated 'drupal/*'                    # Check updates
ddev composer update drupal/[module] --with-deps     # Update module
ddev composer require drupal/core:X.Y.Z drupal/core-recommended:X.Y.Z --update-with-all-dependencies  # Core update
```

**Scripts** in `composer.json`: `build`, `deploy`, `test`, `phpcs`, `phpstan`

### Environment Variables
Store in `.ddev/.env` (gitignored). Access: `$_ENV['VAR']`. Restart DDEV after changes.

### Patches
Structure: `./patches/{core,contrib/[module],custom}/`

In `composer.json` → `extra.patches`:
```json
"drupal/module": {"#123 Fix": "patches/contrib/module/fix.patch"}
```
Sources: local files, Drupal.org issue queue, GitHub PRs
Always include issue numbers in descriptions. Monitor upstream for merged patches.

## Code Quality Tools

```bash
# PHPStan - static analysis
ddev exec vendor/bin/phpstan analyze web/modules/custom --level=1

# PHPCS - coding standards check
ddev exec vendor/bin/phpcs --standard=Drupal web/modules/custom/

# PHPCBF - auto-fix coding standards
ddev exec vendor/bin/phpcbf --standard=Drupal web/modules/custom/

# Rector - code modernization (run in container)
ddev ssh && vendor/bin/rector process web/modules/custom --dry-run

# Upgrade Status - Drupal compatibility check
ddev drush upgrade_status:analyze --all
```

**Config files**: `phpstan.neon`, `phpcs.xml`, `rector.php`
**Run before**: commits, PRs, Drupal upgrades

## Testing

```bash
# PHPUnit
ddev exec vendor/bin/phpunit web/modules/custom
ddev exec vendor/bin/phpunit web/modules/custom/[module]/tests/src/Unit/MyTest.php
ddev exec vendor/bin/phpunit --coverage-html coverage web/modules/custom

# Codeception
ddev exec vendor/bin/codecept run [acceptance|functional|unit]
ddev exec vendor/bin/codecept run --steps --debug --html

# Debug failed tests
ddev exec vendor/bin/phpunit --testdox --verbose [test-file]
```

**Drupal test types** (in `tests/src/`): `Unit/` (isolated), `Kernel/` (minimal bootstrap), `Functional/` (full Drupal), `FunctionalJavascript/`

**Codeception structure**: `tests/{acceptance,functional,unit,_support,_data,_output}/`, config: `codeception.yml`

**Test Best Practices**:
- Keep tests focused and atomic (one concept per test)
- Use descriptive names: `testUserCanLoginWithValidCredentials()`
- Follow AAA pattern: Arrange, Act, Assert
- Clean up in `tearDown()`, use data providers for multiple scenarios
- Test both positive and negative scenarios
- Use fixtures, avoid dependencies between tests, reset state

## Debugging

```bash
# Xdebug
ddev xdebug on|off                              # Toggle (disable when not debugging for perf)

# Container & DB access
ddev ssh                                         # Web container
ddev mysql                                       # MySQL CLI
ddev mysql -e "SELECT..."                        # Direct query
ddev export-db --file=backup.sql.gz             # Export
ddev import-db --file=backup.sql.gz             # Import

# Logs
ddev logs -f                                     # Container logs (follow)
ddev drush watchdog:show --count=50 --severity=Error

# State
ddev drush state:get|set|delete [key] [value]
```

**IDE**: PhpStorm (port 9003), VS Code (PHP Debug extension)
**Tips**: `ddev describe` (URLs/services), `ddev debug` (DDEV issues), Twig debug in `development.services.yml`

## Performance

```bash
# Cache
ddev drush cr                                    # Rebuild all
ddev drush cache:clear [render|dynamic_page_cache|config]

# Redis (if enabled)
ddev redis-cli INFO stats|memory
ddev redis-cli FLUSHALL                          # Clear Redis

# DB performance
ddev mysql -e "SELECT table_name, round(((data_length+index_length)/1024/1024),2) 'MB' FROM information_schema.TABLES WHERE table_schema=DATABASE() ORDER BY (data_length+index_length) DESC;"
ddev mysql -e "SHOW VARIABLES LIKE 'slow_query%';"
ddev drush sql:query "OPTIMIZE TABLE cache_bootstrap, cache_config, cache_data, cache_default, cache_discovery, cache_dynamic_page_cache, cache_entity, cache_menu, cache_render;"
```

**Optimization**: Enable page cache + dynamic page cache, CSS/JS aggregation, Redis/Memcache, CDN for assets, image styles with lazy loading

## Code Standards

### Core Principles

- **SOLID/DRY**: Follow SOLID principles, extract repeated logic
- **PHP 8.1+**: Use strict typing: `declare(strict_types=1);`
- **Drupal Standards**: PSR-12 based, English comments only

### Module Structure

Location: `/web/modules/custom/[prefix]_[module_name]/`
Naming: `[prefix]_[descriptive_name]` (e.g., `d_`, `custom_`) - prevents conflicts with contrib

```
[prefix]_[module_name]/
├── [prefix]_[module_name].{info.yml,module,install,routing.yml,permissions.yml,services.yml,libraries.yml}
├── src/                          # PSR-4: \Drupal\[module_name]\[Subdir]\ClassName
│   ├── Entity/                   # Custom entities
│   ├── Form/                     # Forms (ConfigFormBase, FormBase)
│   ├── Controller/               # Route controllers
│   ├── Plugin/{Block,Field/FieldWidget,Field/FieldFormatter}/
│   ├── Service/                  # Custom services
│   └── EventSubscriber/          # Event subscribers
├── templates/                    # Twig templates
├── css/ & js/                    # Assets
```

**PSR-4**: `src/Form/MyForm.php` → `\Drupal\my_module\Form\MyForm`

### Entity Development Patterns

```php
// 1. Constants instead of magic numbers (in .module)
define('ENTITY_STATUS_DRAFT', 0);
define('ENTITY_STATUS_PUBLISHED', 1);

// 2. Getter methods instead of direct field access
public function getStatus(): int {
  return (int) $this->get('status')->value;
}

// 3. Safe migrations with backward compatibility
function [module]_update_XXXX() {
  $manager = \Drupal::entityDefinitionUpdateManager();
  $field = $manager->getFieldStorageDefinition('field_name', 'entity_type');
  if ($field) {
    $new_def = BaseFieldDefinition::create('field_type')->setSettings([...]);
    $manager->updateFieldStorageDefinition($new_def);
    drupal_flush_all_caches();
    \Drupal::logger('module')->info('Migration completed.');
  }
}
```

**Migration safety**: Backup DB, test on staging, ensure backward compatibility, log changes, have rollback plan.

### Drupal Best Practices

```php
// Database API - always use placeholders, never raw SQL
$query = \Drupal::database()->select('node_field_data', 'n')
  ->fields('n', ['nid', 'title'])->condition('status', 1)->range(0, 10);
$results = $query->execute()->fetchAll();

// Dependency Injection - avoid \Drupal:: static calls in classes
class MyService {
  public function __construct(
    private readonly EntityTypeManagerInterface $entityTypeManager,
  ) {}
}

// Caching - use tags and contexts
$build = [
  '#markup' => $content,
  '#cache' => ['tags' => ['node:' . $nid], 'contexts' => ['user'], 'max-age' => 3600],
];
\Drupal\Core\Cache\Cache::invalidateTags(['node:' . $nid]);
\Drupal::cache()->set($cid, $data, time() + 3600, ['my_module']);
```

```php
// Queue API - for heavy operations
$queue = \Drupal::queue('my_module_processor');
$queue->createItem(['data' => $data]);
// QueueWorker plugin: @QueueWorker(id="...", cron={"time"=60})

// Entity System - always use entity type manager
$storage = \Drupal::entityTypeManager()->getStorage('node');
$node = $storage->load($nid);
$query = $storage->getQuery()
  ->condition('type', 'article')->condition('status', 1)
  ->accessCheck(TRUE)->sort('created', 'DESC')->range(0, 10);
$nids = $query->execute();

// Form API - extend FormBase, implement getFormId(), buildForm(), validateForm(), submitForm()
$form['field'] = ['#type' => 'textfield', '#title' => $this->t('Name'), '#required' => TRUE];
$form_state->setErrorByName('field', $this->t('Error message.'));

// Translation - always use t() for user-facing strings
$this->t('Hello @name', ['@name' => $name]);

// Config API
$config = \Drupal::config('my_module.settings')->get('key');
\Drupal::configFactory()->getEditable('my_module.settings')->set('key', $value)->save();

// Permissions
user_role_grant_permissions($role_id, ['permission']);
user_role_revoke_permissions($role_id, ['permission']);
```

### Code Style

- Type declarations/hints required, PHPDoc for classes/methods
- Align `=>` in arrays, `=` in variable definitions
- Controllers: final classes, DI, keep thin
- Services: register in `services.yml`, single responsibility
- Logging: `\Drupal::logger('module')->notice('message')`
- Entity updates: always use update hooks in `.install`, maintain backward compatibility

## Directory Structure

**Key paths**: `/web/` (or `docroot/`), `/web/modules/custom/`, `/config/sync/`, `/web/sites/default/settings.php`, `/patches/`, `/tests/`

**Development paths**: routes → `routing.yml`, forms → `src/Form/`, entities → `src/Entity/`, permissions → `permissions.yml`, updates → `.install`

## Multilingual Configuration

```bash
# Setup
ddev drush pm:enable language locale content_translation config_translation
ddev drush language:add pl && ddev drush language:add es
ddev drush locale:check && ddev drush locale:update

# Enable content translation
ddev drush config:set language.content_settings.node.article third_party_settings.content_translation.enabled true
```

**Detection**: Configure at `/admin/config/regional/language/detection` - use URL prefix (`/en/`, `/pl/`) for SEO

**Custom entities**: Add `translatable = TRUE` to `@ContentEntityType`, use `->setTranslatable(TRUE)` on fields

**In code**: `$this->t('Hello @name', ['@name' => $name])` | **In Twig**: `{{ 'Hello'|trans }}`

**Common issues**: Missing translations → `locale:update`, content not translatable → check Language settings tab

## Configuration Management

```bash
ddev drush cex                    # Export config
ddev drush cim                    # Import config
ddev drush config:status          # Show differences
```

<!-- CONFIG SPLIT (uncomment if using)
Structure: config/{sync,dev,staging,prod}/
Commands: `ddev drush config-split:export dev`, `ddev drush csex dev`
Use cases: dev modules (devel), prod modules (purge, cdn), env-specific settings
-->

<!-- CONFIG IGNORE (uncomment if using)
Location: /config/sync/config_ignore.settings.yml
-->

## Security

**Principles**: HTTPS required, sanitize input, use DB abstraction (no raw SQL), env vars for secrets, proper access checks

```bash
# Security updates
ddev drush pm:security
ddev composer update drupal/core-recommended --with-dependencies
ddev composer update --security-only

# Audit
ddev drush role:perm:list
ddev drush watchdog:show --severity=Error --count=100
```

**Hardening**: `chmod 444 settings.php`, `chmod 755 sites/default/files`, disable PHP in files dir

**Code**: Use placeholders in queries, `Html::escape()` for output, `$account->hasPermission()` for access, Form API for validation

## Headless/API-First Development

### JSON:API (Core)

```bash
ddev drush pm:enable jsonapi
# Optional: ddev composer require drupal/jsonapi_extras
```

**Endpoints**:
```
GET  /jsonapi/node/article                                    # List all
GET  /jsonapi/node/article/{uuid}?include=field_image,uid    # With relations
GET  /jsonapi/node/article?filter[status]=1&sort=-created&page[limit]=10
POST /jsonapi/node/article  (Content-Type: application/vnd.api+json, Authorization: Bearer {token})
```

### GraphQL

```bash
ddev composer require drupal/graphql drupal/graphql_compose
ddev drush pm:enable graphql graphql_compose
# Explorer at /admin/config/graphql
```

### Authentication (Simple OAuth)

```bash
ddev composer require drupal/simple_oauth && ddev drush pm:enable simple_oauth
openssl genrsa -out keys/private.key 2048 && openssl rsa -in keys/private.key -pubout -out keys/public.key
# POST /oauth/token with grant_type, client_id, client_secret, username, password
# Use: Authorization: Bearer {access_token}
```

### CORS (in services.yml)

```yaml
cors.config:
  enabled: true
  allowedOrigins: ['http://localhost:3000']
  allowedMethods: ['GET', 'POST', 'PATCH', 'DELETE', 'OPTIONS']
  allowedHeaders: ['*']
  supportsCredentials: true
```

### Architecture Patterns

- **Fully Decoupled**: Drupal API + React/Vue/Next.js frontend
- **Progressively Decoupled**: Drupal pages + JS framework for interactive components
- **Hybrid**: Mix of Drupal templates and API-driven sections

### API Best Practices

OAuth tokens (not basic auth), rate limiting, HTTPS, validate input, API documentation (`drupal/openapi`)

## SEO & Structured Data

### Core Modules

```bash
ddev composer require drupal/metatag drupal/pathauto drupal/simple_sitemap drupal/redirect drupal/schema_metatag
ddev drush pm:enable metatag metatag_open_graph metatag_twitter_cards pathauto simple_sitemap redirect schema_metatag
ddev drush simple-sitemap:generate    # Generate sitemap at /sitemap.xml
ddev drush pathauto:generate          # Generate URL aliases
```

### Schema.org & Open Graph

Configure at `/admin/config/search/metatag/global`:
- **Organization**: `@type: Organization`, name, url, logo, sameAs
- **Article**: `@type: Article`, headline `[node:title]`, datePublished, author, image
- **Open Graph**: og:title, og:description, og:image, og:url
- **Twitter Cards**: twitter:card `summary_large_image`, twitter:title, twitter:image

### Multilingual SEO

```bash
ddev composer require drupal/hreflang && ddev drush pm:enable hreflang
```
Twig: `{% for lang in languages %}<link rel="alternate" hreflang="{{ lang.id }}" href="..."/>{% endfor %}`

### Performance

CSS/JS aggregation, BigPipe, WebP images, lazy loading, responsive image styles

**Testing**: Google Rich Results Test, Facebook Sharing Debugger, PageSpeed Insights

### SEO Checklist

On-page: title tags (50-60 chars), meta descriptions (150-160), H1 unique, clean URLs, alt attributes
Technical: sitemap submitted, robots.txt, canonical URLs, Schema.org, HTTPS, Core Web Vitals
Multilingual: hreflang tags, language-specific sitemaps, canonical per language

**SEO Audit Tools**:
- Screaming Frog SEO Spider
- Ahrefs Site Audit
- SEMrush Site Audit
- Moz Pro

## JavaScript and CSS

**JS Aggregation Issues**: Missing `.libraries.yml` dependencies, wrong load order, `drupalSettings` unavailable
**Solutions**: Add deps (`core/jquery`, `core/drupal`, `core/drupalSettings`, `core/once`), use `once()` not `.once()`, test with aggregation enabled
**CSS**: BEM naming, SCSS/SASS, organize by component, use SDC when applicable

## Frontend Development

<!--
THEME DISCOVERY FOR AI/LLM:
1. cat config/sync/system.theme.yml → active theme
2. cat web/themes/custom/[theme]/[theme].info.yml → config, base themes
3. find web/themes/custom/[theme] -name "*.twig" → templates
4. grep "function.*_preprocess" [theme].theme → preprocess hooks
5. ls components/ → SDC components
6. ddev drush sdc:list → list all components

Theme files: [theme].info.yml (definition), [theme].libraries.yml (assets), [theme].theme (hooks), templates/ (Twig), components/ (SDC)
-->

**Theme location**: `/web/themes/custom/[theme_name]/`

### Build Commands

```bash
cd web/themes/custom/[theme_name]
npm install                    # Setup
npm run dev|build|watch        # Dev/prod/watch (or gulp/gulp dist/gulp watch)
gulp sass|js|images|fonts      # Individual tasks
gulp lint:scss|lint:js         # Linting
```

### Libraries (`[theme].libraries.yml`)

```yaml
global:
  css: { theme: { css/style.css: {} } }
  js: { js/global.js: {} }
  dependencies: [core/drupal, core/jquery, core/drupalSettings, core/once]
```

### Twig Templates

**Enable debugging** (`development.services.yml`): `twig.config: { debug: true, auto_reload: true, cache: false }`

**Naming**: `node--[type]--[view-mode].html.twig`, `paragraph--[type].html.twig`, `block--[type].html.twig`, `field--[name]--[entity].html.twig`

**Override**: Enable debug → view source for suggestions → copy from core/themes → place in templates/ → `ddev drush cr`

**Template directory structure**:
```
templates/
├── block/           # Block templates
├── content/         # Node templates
├── field/           # Field templates
├── form/            # Form element templates
├── layout/          # Layout templates
├── misc/            # Miscellaneous templates
├── navigation/      # Menu and navigation
├── paragraph/       # Paragraph templates
└── views/           # Views templates
```

### Preprocess Functions (`[theme].theme`)

```php
function [theme]_preprocess_node(&$variables) {
  $variables['custom_var'] = $variables['node']->bundle();
}
function [theme]_preprocess_paragraph(&$variables) {
  $variables['type'] = $variables['paragraph']->bundle();
}
function [theme]_theme_suggestions_node_alter(array &$suggestions, array $variables) {
  $node = $variables['elements']['#node'];
  if ($node->hasField('field_layout') && !$node->get('field_layout')->isEmpty()) {
    $suggestions[] = 'node__' . $node->bundle() . '__' . $node->get('field_layout')->value;
  }
}
```

### Single Directory Components (SDC)

Drupal 10.1+ core, or `ddev composer require drupal/sdc` for 10.0

**Structure**: `components/[name]/` with `[name].component.yml`, `[name].twig`, optional `.css`/`.js`

**component.yml**:
```yaml
name: Card
props: { type: object, properties: { title: { type: string }, link: { type: string } } }
slots: { content: { title: Content } }
```

**Usage**: `{% include '[theme]:card' with { title: node.label, link: url } %}` or `{% embed %}` for slots

**Commands**: `ddev drush sdc:list`, `ddev drush pm:enable sdc`

### Troubleshooting

```bash
rm -rf node_modules package-lock.json && npm install   # Reset deps
rm -rf dist/ css/ js/compiled/                          # Clear build cache
```

**Performance**: Minify for prod, imagemin, critical CSS, font-display:swap, CSS/JS aggregation, AdvAgg module

## Environment Indicators

- **Visual verification**: Check indicators display correctly on all pages
- **Color scheme**: GREEN (Local), BLUE (DEV), ORANGE (STG), RED (PROD)
- **Never commit "LOCAL"** as value in `environment_indicator.indicator.yml` for production! Always use "PROD" and red color.

## Documentation

**MANDATORY**: Document work in "Tasks and Problems" section. Use real date (`date` command). Document: modules, fixes, config changes, optimizations, problems/solutions.

## Common Tasks

### New Module
```bash
# Create /web/modules/custom/[prefix]_[name]/ with:
# - [prefix]_[name].info.yml (name, type:module, core_version_requirement:^10||^11, package:Custom)
# - [prefix]_[name].module (hooks), .routing.yml, .permissions.yml, .services.yml as needed
ddev drush pm:enable [prefix]_[name] && ddev drush cr
```

### Update Core
```bash
ddev export-db --file=backup.sql.gz                                    # Backup
ddev composer update drupal/core-recommended drupal/core-composer-scaffold --with-dependencies
ddev drush updb && ddev drush cr                                       # Updates + cache
```

### Database Migration
```php
// In [module].install
function [module]_update_10001() {
  // Use EntityDefinitionUpdateManager for field changes
  // Check field exists, update displays, log completion
  drupal_flush_all_caches();
}
```

### Tests
```bash
ddev exec vendor/bin/phpunit web/modules/custom/[module]/tests   # PHPUnit
ddev exec vendor/bin/codecept run                                 # Codeception
# Dirs: tests/src/Unit/, Kernel/, Functional/; tests/acceptance/
```

### Permissions
```bash
ddev drush role:perm:list [role]                                  # List
# PHP: user_role_grant_permissions($role_id, ['perm1']); drupal_flush_all_caches();
```

### Config Export/Import
```bash
ddev drush cex && git diff config/ && git commit                  # Export
ddev drush cim -y                                                 # Import
```

## Troubleshooting

### Quick Fixes
```bash
ddev drush cr                                                     # Clear cache
ddev restart                                                      # Restart containers
ddev xdebug on|off                                               # Debug mode
ddev drush watchdog:show --count=50                              # Check logs
```

### JS Aggregation Issues
Add deps in `.libraries.yml`: `core/drupal`, `core/jquery`, `core/drupalSettings`, `core/once`
Use `once()` function, check browser console, test with aggregation enabled

### Cache Not Clearing
```bash
ddev drush cr                                                     # Standard
rm -rf web/sites/default/files/php/twig/* && ddev drush cr       # Twig
ddev drush sql:query "TRUNCATE cache_render;" && ddev drush cr   # Nuclear
```

### Database Issues
```bash
ddev drush sql:cli                     # Check connection (SELECT 1;)
ddev drush updb && ddev drush entity:updates   # Pending updates
ddev mysql -e "REPAIR TABLE [name];"   # Repair table
```

### DDEV Issues
```bash
ddev restart                           # Soft restart
ddev stop && ddev start                # Full restart
ddev delete -O && ddev start           # Recreate containers
ddev logs                              # View logs
```

### Module Installation
```bash
ddev composer why-not drupal/[module]  # Check deps
ddev composer require drupal/[module] && ddev drush pm:enable [module]
ddev drush updb && ddev drush entity:updates   # Schema issues
```

### Permissions
```bash
ddev exec chmod -R 775 web/sites/default/files
ddev exec chmod 444 web/sites/default/settings.php
```

### WSOD (White Screen)
```bash
ddev drush config:set system.logging error_level verbose
ddev logs && ddev drush watchdog:show --count=50
ddev exec tail -f /var/log/php/php-fpm.log    # Check fatal errors
```

### Config Import Fails
```bash
ddev drush config:status              # Check status
ddev drush config:set system.site uuid [correct-uuid]  # UUID mismatch
```

### Memory Issues
```bash
echo "memory_limit = 512M" >> .ddev/php/php.ini && ddev restart
# Or: ddev exec php -d memory_limit=1G vendor/bin/drush [cmd]
```

## Additional Resources

- **Project Documentation**: `.cursor/TASKS_AND_PROBLEMS.md`
- **Drupal Documentation**: https://www.drupal.org/docs
- **DDEV Documentation**: https://ddev.readthedocs.io/

---

<!--
===========================================
PROJECT-SPECIFIC SECTIONS BELOW
Add sections specific to your project here
===========================================
-->

<!--
===========================================
HOW TO DISCOVER FULL ENTITY STRUCTURE - GUIDE FOR AI/LLM
===========================================

Use this guide to discover the complete entity structure of a Drupal project.
Priority order: 1) Config YAML files (most complete), 2) Drush commands, 3) PHP evaluation

### 1. CONFIG YAML FILES (Primary Source)

Default config directory: `./config/sync/` (see "Directory Structure" section for verification commands and multisite paths)

**Entity Type Config File Patterns:**

| Entity Type | Config File Pattern | Example |
|-------------|---------------------|---------|
| Content Types | `node.type.*.yml` | `node.type.article.yml` |
| Field Storage | `field.storage.*.yml` | `field.storage.node.field_image.yml` |
| Field Instance | `field.field.*.yml` | `field.field.node.article.field_image.yml` |
| Paragraph Types | `paragraphs.paragraphs_type.*.yml` | `paragraphs.paragraphs_type.text.yml` |
| Media Types | `media.type.*.yml` | `media.type.image.yml` |
| Taxonomy Vocabularies | `taxonomy.vocabulary.*.yml` | `taxonomy.vocabulary.tags.yml` |
| View Modes | `core.entity_view_mode.*.yml` | `core.entity_view_mode.node.teaser.yml` |
| Form Modes | `core.entity_form_mode.*.yml` | `core.entity_form_mode.node.default.yml` |
| View Display | `core.entity_view_display.*.yml` | `core.entity_view_display.node.article.default.yml` |
| Form Display | `core.entity_form_display.*.yml` | `core.entity_form_display.node.article.default.yml` |

**Commands to list config files:**
```bash
# List all content type configs
ls config/sync/node.type.*.yml

# List all field storage configs
ls config/sync/field.storage.*.yml

# List all paragraph type configs
ls config/sync/paragraphs.paragraphs_type.*.yml

# Read specific config file
cat config/sync/node.type.article.yml
```

### 2. DRUSH COMMANDS

```bash
# List all entity types
ddev drush entity:info

# List bundles for entity type
ddev drush entity:bundle-info node
ddev drush entity:bundle-info paragraph
ddev drush entity:bundle-info media
ddev drush entity:bundle-info taxonomy_term

# List fields for entity type and bundle
ddev drush field:list node article
ddev drush field:list paragraph text

# Get field info
ddev drush field:info node article field_image

# Export all config
ddev drush config:export

# Get specific config
ddev drush config:get node.type.article

# List all config
ddev drush config:list | grep node.type
```

### 3. PHP/DRUSH PHP:EVAL

For programmatic access to field definitions:

```bash
# Get all fields for content type
ddev drush php:eval "
\$fields = \Drupal::service('entity_field.manager')->getFieldDefinitions('node', 'article');
foreach (\$fields as \$name => \$field) {
  echo \$name . ' - ' . \$field->getLabel() . ' (' . \$field->getType() . ')' . PHP_EOL;
}
"

# Get field settings
ddev drush php:eval "
\$field = \Drupal\field\Entity\FieldConfig::loadByName('node', 'article', 'field_image');
print_r(\$field->getSettings());
"

# Get all bundles for entity type
ddev drush php:eval "
\$bundles = \Drupal::service('entity_type.bundle.info')->getBundleInfo('node');
foreach (\$bundles as \$id => \$info) {
  echo \$id . ' - ' . \$info['label'] . PHP_EOL;
}
"

# Export full entity structure as JSON
ddev drush php:eval "
\$entity_types = ['node', 'paragraph', 'media', 'taxonomy_term'];
\$result = [];
foreach (\$entity_types as \$entity_type) {
  \$bundles = \Drupal::service('entity_type.bundle.info')->getBundleInfo(\$entity_type);
  foreach (\$bundles as \$bundle_id => \$bundle_info) {
    \$fields = \Drupal::service('entity_field.manager')->getFieldDefinitions(\$entity_type, \$bundle_id);
    \$field_list = [];
    foreach (\$fields as \$name => \$field) {
      \$field_list[\$name] = [
        'label' => (string) \$field->getLabel(),
        'type' => \$field->getType(),
        'required' => \$field->isRequired(),
      ];
    }
    \$result[\$entity_type][\$bundle_id] = [
      'label' => \$bundle_info['label'],
      'fields' => \$field_list,
    ];
  }
}
echo json_encode(\$result, JSON_PRETTY_PRINT);
"
```

### RECOMMENDED WORKFLOW

1. **First**: Check if `config/sync/` directory exists and list YAML files
2. **Second**: Use `ddev drush entity:bundle-info [type]` for quick overview
3. **Third**: Use `ddev drush field:list [type] [bundle]` for field details
4. **Fourth**: Use `php:eval` for complex queries or full export

-->

## Drupal Entities Structure

Complete reference of content types, media types, taxonomies, and custom entities in this project.

### Content Types (Node Bundles)

<!--
Document all content types in the project. Update this section during customization.

Example format:
-->

```toon
content_types[2]{machine_name,label,description,features,key_fields}:
  article,Article,News and blog posts,"revisions,menu_ui,content_translation","body,field_image,field_tags,field_category"
  page,Basic Page,Static pages,"revisions,menu_ui","body,field_sections"
```

**Discovery**: See "HOW TO DISCOVER FULL ENTITY STRUCTURE" guide above. Quick commands:
```bash
ddev drush entity:bundle-info node        # List all content types
ls config/sync/node.type.*.yml            # Config files (PRIMARY SOURCE)
ddev drush field:list node article        # Fields for specific type
```

### Paragraph Types

<!--
Document paragraph types if using Paragraphs module.
Organize by category for better readability.
-->

**Discovery**: See "HOW TO DISCOVER FULL ENTITY STRUCTURE" guide above. Quick commands:
```bash
ddev drush entity:bundle-info paragraph       # List all paragraph types
ls config/sync/paragraphs.paragraphs_type.*.yml  # Config files (PRIMARY SOURCE)
ddev drush field:list paragraph text          # Fields for specific type
```

```toon
paragraph_types:
  layout_paragraphs[3]: banner,two_column_layout,accordion
  content_paragraphs[3]: text_block,quote,call_to_action
  media_paragraphs[3]: image_gallery,video_embed,carousel
```

### Media Types

**Discovery**: See "HOW TO DISCOVER FULL ENTITY STRUCTURE" guide above. Quick commands:
```bash
ddev drush entity:bundle-info media    # List all media types
ls config/sync/media.type.*.yml        # Config files (PRIMARY SOURCE)
ddev drush field:list media image      # Fields for specific type
```

```toon
media_types[3]{machine_name,label,source,source_field}:
  image,Image,image,field_media_image
  document,Document,file,field_media_document
  remote_video,Remote Video,oembed:video,field_media_oembed_video
```

### Taxonomy Vocabularies

**Discovery**: See "HOW TO DISCOVER FULL ENTITY STRUCTURE" guide above. Quick commands:
```bash
ddev drush entity:bundle-info taxonomy_term   # List all vocabularies
ls config/sync/taxonomy.vocabulary.*.yml      # Config files (PRIMARY SOURCE)
ddev drush field:list taxonomy_term tags      # Fields for specific vocabulary
```

```toon
taxonomies[2]{machine_name,label,description,hierarchy}:
  tags,Tags,Content tagging,false
  categories,Categories,Content categorization,true
```

### Custom Entities

<!--
Document custom content entities here if project has any.
-->

**Discovery**: See "HOW TO DISCOVER FULL ENTITY STRUCTURE" guide above. Quick commands:
```bash
ddev drush entity:info                        # List ALL entity types
ls web/modules/custom/*/src/Entity/*.php      # Find custom entity classes
```

```toon
custom_entities:
  custom_entity_name:
    type: content_entity
    base_table: custom_entity
    entity_keys: id:id,uuid:uuid,label:name
    fields[6]{name,type}:
      id,integer
      uuid,uuid
      name,string
      status,boolean
      created,timestamp
      changed,timestamp
```

### Entity Relationships

<!--
Document key entity relationships.
Use this section to explain how entities connect.
-->

**Examples**:
- **Article** → **Tags** (many-to-many via `field_tags`)
- **Article** → **Author** (many-to-one via `uid`)
- **Page** → **Paragraphs** (one-to-many via `field_sections`)
- **Custom Entity** → **Node** (reference via `field_node_ref`)

### Field Patterns

**Common field naming patterns in this project**:

- `field_[name]` - Standard field prefix
- `field_[prefix]_[name]` - Module-specific fields (e.g., `field_meta_tags`)
- Base fields: `title`, `body`, `created`, `changed`, `uid`, `status`

**Key Field Types**:
- Reference fields: `entity_reference`, `entity_reference_revisions`
- Text fields: `string`, `text_long`, `text_with_summary`
- Date fields: `datetime`, `daterange`, `timestamp`
- Media: `image`, `file`
- Structured: `link`, `address`, `telephone`

### View Modes

**Discovery**: See "HOW TO DISCOVER FULL ENTITY STRUCTURE" guide above. Quick commands:
```bash
ls config/sync/core.entity_view_mode.*.yml       # View mode definitions
ls config/sync/core.entity_view_display.node.*.yml  # Node view displays
```

**Node View Modes**:
- `full` - Full content display
- `teaser` - Summary/card display
- `search_result` - Search results display
- Custom: `[document_custom_view_modes]`

**Media View Modes**:
- `full` - Full media display
- `media_library` - Media library thumbnail
- Custom: `[document_custom_view_modes]`

### Entity Constants

<!--
Document entity status values and other constants used in the project.
-->

```php
// Example: Content workflow states
define('ENTITY_STATUS_DRAFT', 0);
define('ENTITY_STATUS_PUBLISHED', 1);
define('ENTITY_STATUS_ARCHIVED', 2);

// Custom entity states
define('CUSTOM_ENTITY_PENDING', 0);
define('CUSTOM_ENTITY_APPROVED', 1);
define('CUSTOM_ENTITY_REJECTED', 2);
```

### Entity Access Patterns

**Common access patterns**:
- View published content: `access content` permission
- Edit own content: `edit own [type] content` permission
- Delete own content: `delete own [type] content` permission
- Administer content: `administer [type] content` permission

### Entity Queries

**Common query patterns for this project**:

```php
// Query nodes
$query = \Drupal::entityTypeManager()->getStorage('node')->getQuery()
  ->condition('type', 'article')
  ->condition('status', 1)
  ->accessCheck(TRUE)
  ->sort('created', 'DESC')
  ->range(0, 10);
$nids = $query->execute();

// Query with relationships
$query = \Drupal::entityTypeManager()->getStorage('node')->getQuery()
  ->condition('type', 'article')
  ->condition('field_category.entity.name', 'Technology')
  ->accessCheck(TRUE);
$nids = $query->execute();

// Query custom entities
$query = \Drupal::entityTypeManager()->getStorage('custom_entity')->getQuery()
  ->condition('status', CUSTOM_ENTITY_APPROVED)
  ->accessCheck(TRUE);
$ids = $query->execute();
```

### Migration Patterns

If project uses migrations, document source to destination mappings:

```yaml
# Example migration mapping
source:
  entity_type: legacy_node
  bundle: legacy_article

destination:
  entity_type: node
  bundle: article

field_mapping:
  legacy_title → title
  legacy_body → body
  legacy_image → field_image
  legacy_category → field_category
```

## Project-Specific Features

<!--
Add documentation for project-specific features here.
Examples:
- Custom entity workflows
- Integration with external services
- PDF generation
- Email notifications
- API endpoints
-->

## Development Workflow

- Document all significant changes in "Tasks and Problems" section below
- Follow the format and examples provided
- Review existing entries before making architectural changes
- Always run `date` command to get current date before adding entries

---

## Tasks and Problems Log

**MANDATORY**: Update this section after completing tasks or solving problems during each development session.

**Format**: `YYYY-MM-DD | Brief description of task/problem and solution`

**Instructions**:
1. Run `date` command to get current date before adding entries
2. Add new entries at the top (most recent first)
3. Keep entries concise and actionable
4. Include relevant file paths, module names, or configuration keys
5. Document both what was done and why it was necessary

### Completed Tasks

```
[Add completed tasks here following the format]

Example:
2024-01-15 | Created custom module d_custom_feature for handling special workflow
2024-01-14 | Updated Drupal core from 10.1.0 to 10.2.0, ran database updates successfully
2024-01-13 | Fixed JavaScript aggregation issue by adding missing core/once dependency
```

### Problems and Solutions

```
[Add problems and solutions here]

Example format:
2024-01-15 | PROBLEM: Configuration import failing with UUID mismatch
          | SOLUTION: Updated system.site UUID using drush config:set system.site uuid [correct-uuid]

2024-01-14 | PROBLEM: Drush commands timing out with memory exhausted error
          | SOLUTION: Increased PHP memory limit in .ddev/php/php.ini to 512M, restarted DDEV

2024-01-13 | PROBLEM: Custom module not appearing in module list
          | SOLUTION: Fixed .info.yml syntax error (missing space after colon), cleared cache
```

### Configuration Changes

```
[Document significant configuration changes]

Example:
2024-01-15 | Enabled Redis cache backend in settings.php
2024-01-14 | Configured TPay payment gateway with sandbox credentials
2024-01-13 | Updated environment indicator color scheme (dev=blue, staging=orange, prod=red)
```

### Performance Optimizations

```
[Document performance improvements]

Example:
2024-01-15 | Enabled CSS/JS aggregation and AdvAgg module
2024-01-14 | Optimized database by truncating old cache tables
2024-01-13 | Implemented lazy loading for images using native loading="lazy"
```

### Security Updates

```
[Document security-related changes]

Example:
2024-01-15 | Applied security update for Drupal core 10.1.8
2024-01-14 | Updated webform module to address SA-CONTRIB-2024-001
2024-01-13 | Hardened file permissions on settings.php (chmod 444)
```

### Development Notes

```
[General development notes and observations]

Example:
2024-01-15 | Note: Custom entity queries must include ->accessCheck(TRUE/FALSE)
2024-01-14 | Note: When using Config Split, remember to activate splits in settings.php
2024-01-13 | Note: DDEV custom commands must NOT use ## #ddev-generated comments
```

---

<!--
===========================================
TEMPLATE CUSTOMIZATION CHECKLIST
===========================================

For the full customization checklist with phases, see README.md in the repository:
https://github.com/droptica/drupal-agents-md

The checklist includes:
- Phase 1: Project Discovery (composer.json, web root, modules, theme, entities)
- Phase 2: Replace Placeholders ([PROJECT_NAME], [VERSION], web/, etc.)
- Phase 3: Fill Project-Specific Sections (entities, modules, languages)
- Phase 4: Cleanup (remove inapplicable commented sections)
- Phase 5: Finalize (save as AGENTS.md, summary)

Required placeholder replacements:
- [PROJECT_NAME], [VERSION], [PHP_VERSION], [REPOSITORY_URL]
- [PROJECT_DIR], [BUILD_COMMAND], [prefix], [theme_name]
- web/ (if your web root is different: docroot/, html/, etc.)

Optional sections to uncomment if applicable:
- Multisite, AI Integration, Commerce/Payment, Config Split/Ignore

---

**Last Updated**: [Add date when customization completed]
**Customized By**: [Add your name]
**Project Start Date**: [Add project start date]
-->
