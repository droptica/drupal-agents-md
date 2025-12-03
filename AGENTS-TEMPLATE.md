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

Comprehensive testing ensures code quality and prevents regressions.

### PHPUnit (Unit and Kernel Tests)

**Purpose**: Test individual components and Drupal integration

```bash
# Run all PHPUnit tests
ddev exec vendor/bin/phpunit web/modules/custom

# Run specific module tests
ddev exec vendor/bin/phpunit web/modules/custom/[module_name]/tests

# Run specific test class
ddev exec vendor/bin/phpunit web/modules/custom/[module_name]/tests/src/Unit/MyTest.php

# Run with coverage report
ddev exec vendor/bin/phpunit --coverage-html coverage web/modules/custom
```

**Test Types**:
- **Unit Tests**: Test individual classes/methods in isolation (`tests/src/Unit/`)
- **Kernel Tests**: Test with minimal Drupal bootstrap (`tests/src/Kernel/`)
- **Functional Tests**: Full Drupal installation tests (`tests/src/Functional/`)
- **FunctionalJavaScript**: Tests with JavaScript support (`tests/src/FunctionalJavascript/`)

### Codeception (Acceptance/Functional Tests)

**Purpose**: End-to-end testing from user perspective

```bash
# Run all Codeception tests
ddev exec vendor/bin/codecept run

# Run specific suite
ddev exec vendor/bin/codecept run acceptance
ddev exec vendor/bin/codecept run functional
ddev exec vendor/bin/codecept run unit

# Run specific test
ddev exec vendor/bin/codecept run acceptance LoginCest

# Run with detailed output
ddev exec vendor/bin/codecept run --steps --debug

# Generate HTML report
ddev exec vendor/bin/codecept run --html
```

**Test Organization**:
```
tests/
├── acceptance/          # Browser-based acceptance tests
├── functional/          # Functional tests without JavaScript
├── unit/               # Unit tests
├── _support/           # Helper classes and support code
├── _data/              # Test fixtures and data
├── _output/            # Test results and screenshots
└── codeception.yml     # Configuration
```

### Test Best Practices

**Writing Tests**:
- Keep tests focused and atomic (one concept per test)
- Use descriptive test method names: `testUserCanLoginWithValidCredentials()`
- Add proper PHPDoc comments explaining what is being tested
- Follow AAA pattern: Arrange, Act, Assert
- Clean up test data in tearDown() methods
- Use data providers for testing multiple scenarios
- Test both positive (success) and negative (failure) scenarios

**Test Data**:
- Use fixtures for consistent test data
- Avoid dependencies between tests
- Reset state between tests
- Use factories or builders for complex test objects

**Assertions**:
```php
// PHPUnit assertions
$this->assertEquals($expected, $actual);
$this->assertTrue($condition);
$this->assertInstanceOf(ClassName::class, $object);
$this->assertCount(3, $array);
$this->assertStringContainsString('needle', $haystack);
```

**When to Run Tests**:
- Before committing code
- After fixing bugs (write test first)
- After refactoring
- Before deploying to production
- In CI/CD pipeline automatically

### Debugging Failed Tests

```bash
# Run single test with verbose output
ddev exec vendor/bin/phpunit --testdox --verbose [test-file]

# Codeception with debugging
ddev exec vendor/bin/codecept run --debug

# Generate fresh screenshots on failure
ddev exec vendor/bin/codecept run --steps
```

Check test output, logs, and screenshots in `tests/_output/` directory.

## Debugging

### Xdebug

**Enable/Disable**:
```bash
# Enable Xdebug
ddev xdebug on

# Disable Xdebug (improves performance)
ddev xdebug off

# Check Xdebug status
ddev exec php -v | grep Xdebug
```

**IDE Configuration**:
- PhpStorm: Configure PHP > Debug > Xdebug port 9003
- VS Code: Install PHP Debug extension
- Set breakpoints in your IDE
- Start listening for debug connections

**Performance Note**: Disable Xdebug when not actively debugging to improve performance.

### Container Access

```bash
# SSH into web container
ddev ssh

# SSH as root user
ddev ssh -s db

# Execute command without SSH
ddev exec [command]

# Run composer commands
ddev composer [command]
```

### Database Access

```bash
# MySQL CLI
ddev mysql

# Access specific database (multisite)
ddev mysql -D [database_name]

# Execute SQL query directly
ddev mysql -e "SELECT * FROM node_field_data LIMIT 5;"

# Export database
ddev export-db --file=backup.sql.gz

# Import database
ddev import-db --file=backup.sql.gz
```

### Logs and Monitoring

```bash
# View container logs
ddev logs

# Follow logs in real-time
ddev logs -f

# View Drupal watchdog logs
ddev drush watchdog:show

# Tail recent watchdog entries
ddev drush watchdog:show --count=50 --severity=Error

# View PHP error log
ddev exec tail -f /var/log/php/php-fpm.log
```

### Cache and State Debugging

```bash
# Clear specific cache bin
ddev drush cache:clear [bin_name]

# Rebuild cache (synonym for cr)
ddev drush cache:rebuild

# Clear Twig cache specifically
ddev drush twig:debug

# Get state value
ddev drush state:get [key]

# Set state value
ddev drush state:set [key] [value]

# Delete state value
ddev drush state:delete [key]
```

### Performance Profiling

```bash
# Check Redis status (if using Redis)
ddev redis-cli
# Then: INFO stats

# Monitor disk usage
ddev exec df -h

# Check memory usage
ddev exec free -m

# Top processes
ddev exec top

# Check PHP memory limit
ddev exec php -i | grep memory_limit
```

### Network Debugging

```bash
# Check network connectivity from container
ddev exec curl -I https://example.com

# Test DNS resolution
ddev exec nslookup example.com

# Check open ports
ddev exec netstat -tulpn
```

### Debugging Tips

- Use `ddev describe` to see all project URLs and services
- Check `.ddev/config.yaml` for project configuration
- Review `ddev logs` when containers fail to start
- Use `ddev debug` for DDEV-specific issues
- Enable Twig debugging in `development.services.yml`

## Performance and Monitoring

### Cache Management

```bash
# Clear all caches
ddev drush cache:rebuild
# Alias: ddev drush cr

# Clear specific cache bins
ddev drush cache:clear render
ddev drush cache:clear dynamic_page_cache
ddev drush cache:clear config

# List all cache bins
ddev drush cache:clear

# Disable caches for development (in development.services.yml)
# parameters:
#   twig.config:
#     debug: true
#     auto_reload: true
#     cache: false
```

### Redis Monitoring

```bash
# Access Redis CLI
ddev redis-cli

# Check Redis stats
ddev redis-cli INFO stats

# Check memory usage
ddev redis-cli INFO memory

# Monitor commands in real-time
ddev redis-cli MONITOR

# Check connected clients
ddev redis-cli CLIENT LIST

# Flush Redis cache
ddev redis-cli FLUSHALL
```

### Performance Metrics

```bash
# Check disk usage
ddev exec df -h

# Check specific directory size
ddev exec du -sh web/sites/default/files

# Check memory usage
ddev exec free -m

# Monitor CPU and memory in real-time
ddev exec top

# Check PHP-FPM status
ddev exec curl http://localhost/fpm-status

# Check PHP memory limit
ddev exec php -i | grep memory_limit
```

### Database Performance

```bash
# Check database size
ddev mysql -e "SELECT table_schema AS 'Database', ROUND(SUM(data_length + index_length) / 1024 / 1024, 2) AS 'Size (MB)' FROM information_schema.TABLES GROUP BY table_schema;"

# Show slow queries log
ddev mysql -e "SHOW VARIABLES LIKE 'slow_query%';"

# Optimize tables
ddev drush sql:query "OPTIMIZE TABLE cache_bootstrap, cache_config, cache_container, cache_data, cache_default, cache_discovery, cache_dynamic_page_cache, cache_entity, cache_menu, cache_render, cache_toolbar;"

# Show table sizes
ddev mysql -e "SELECT table_name AS 'Table', round(((data_length + index_length) / 1024 / 1024), 2) 'Size in MB' FROM information_schema.TABLES WHERE table_schema = DATABASE() ORDER BY (data_length + index_length) DESC;"
```

### Performance Monitoring Commands

```bash
# Check Drupal cron status
ddev drush core:cron

# View system status
ddev drush status

# Check for pending updates
ddev drush updatedb:status

# Profile page load with Drush
ddev drush php:eval "timer_start('test'); node_load(1); \$time = timer_stop('test'); print(\$time['time'] . ' ms');"
```

### Optimization Checklist

- **Caching**:
  - Enable page caching for anonymous users
  - Enable dynamic page cache
  - Use Redis/Memcache for backend caching
  - Configure appropriate cache max-age

- **Aggregation**:
  - Enable CSS aggregation
  - Enable JavaScript aggregation
  - Consider using AdvAgg module for advanced aggregation

- **Database**:
  - Regular cache table truncation
  - Optimize tables periodically
  - Monitor slow queries
  - Consider read replicas for high traffic

- **File System**:
  - Use CDN for static assets
  - Optimize images (use image styles)
  - Enable lazy loading for images
  - Clean up old files regularly

- **Code**:
  - Use dependency injection
  - Implement proper caching strategies
  - Avoid loading unnecessary entities
  - Use Queue API for heavy operations

## Code Standards

### Core Principles

- **SOLID**: Follow SOLID principles for object-oriented programming
- **DRY**: Extract repeated logic into reusable functions, methods, or classes
- **PHP Version**: PHP 8.1+ with strict typing: `declare(strict_types=1);`
- **Drupal Standards**: Follow Drupal coding standards (PSR-12 based)
- **Language**: All code comments and documentation must be in English

### Module Structure

**Custom modules** must be placed in `/web/modules/custom/` and follow this structure (see [Web Root Directory](#web-root-directory) for path variations):
```
[prefix]_[module_name]/
├── [prefix]_[module_name].info.yml      # Module definition
├── [prefix]_[module_name].module         # Module hooks and functions
├── [prefix]_[module_name].install        # Install/update hooks
├── [prefix]_[module_name].routing.yml    # Route definitions
├── [prefix]_[module_name].permissions.yml # Permission definitions
├── [prefix]_[module_name].services.yml   # Service definitions
├── [prefix]_[module_name].libraries.yml  # Asset library definitions
├── src/                          # PHP classes (PSR-4)
│   ├── Entity/                   # Entity classes
│   ├── Form/                     # Form classes
│   ├── Controller/               # Controller classes
│   └── Plugin/                   # Plugin implementations
├── templates/                    # Twig templates
├── css/                         # Stylesheets
└── js/                          # JavaScript files
```

**Module Naming**: Follow `[prefix]_[descriptive_name]` pattern (e.g., `myproject_`, `custom_`, `d_`)

**Why use a prefix?**
- Prevents naming conflicts with contributed modules
- Groups your custom modules together
- Makes it clear which modules are custom vs. contrib
- Recommended: `d_` as default prefix for Droptica projects

**PSR-4 Autoloading**:

Drupal follows PSR-4 autoloading standard for classes in the `src/` directory:

```
[prefix]_[module_name]/
├── src/
│   ├── Entity/              # Namespace: \Drupal\[prefix]_[module_name]\Entity
│   │   └── CustomEntity.php # Class: \Drupal\[prefix]_[module_name]\Entity\CustomEntity
│   ├── Form/                # Namespace: \Drupal\[prefix]_[module_name]\Form
│   │   └── SettingsForm.php # Class: \Drupal\[prefix]_[module_name]\Form\SettingsForm
│   ├── Controller/          # Namespace: \Drupal\[prefix]_[module_name]\Controller
│   │   └── PageController.php
│   ├── Plugin/              # Namespace: \Drupal\[prefix]_[module_name]\Plugin
│   │   ├── Block/          # \Drupal\[prefix]_[module_name]\Plugin\Block
│   │   └── Field/
│   │       └── FieldWidget/
│   └── Service/             # Custom services
│       └── CustomService.php
```

**Namespace Rules**:
- Base namespace: `Drupal\[module_name]`
- Subdirectory becomes part of namespace
- Class name must match filename
- One class per file

**Example**:
```php
<?php

namespace Drupal\my_module\Controller;

use Drupal\Core\Controller\ControllerBase;

class MyController extends ControllerBase {
  // Controller code
}
```

**Directory Purpose**:
- `Entity/` - Custom entity classes and interfaces
- `Form/` - Form classes (ConfigFormBase, FormBase)
- `Controller/` - Page controllers and route controllers
- `Plugin/` - Plugin implementations (Block, Field, etc.)
  - `Plugin/Block/` - Custom block plugins
  - `Plugin/Field/FieldWidget/` - Custom field widgets
  - `Plugin/Field/FieldFormatter/` - Custom field formatters
- `Service/` - Custom service classes
- `EventSubscriber/` - Event subscriber classes
- `Access/` - Access checkers
- `Routing/` - Route subscribers

### Entity Development Patterns

#### 1. Constants Instead of Magic Numbers

```php
// ✅ Good: Define constants in .module file
define('ENTITY_STATUS_DRAFT', 0);
define('ENTITY_STATUS_PUBLISHED', 1);
define('ENTITY_STATUS_ARCHIVED', 2);

// Use in field definitions
->setSettings([
  'allowed_values' => [
    ENTITY_STATUS_DRAFT => 'Draft',
    ENTITY_STATUS_PUBLISHED => 'Published',
    ENTITY_STATUS_ARCHIVED => 'Archived',
  ],
])

// Use in business logic
if ($entity->getStatus() == ENTITY_STATUS_PUBLISHED) {
  // Handle published entity
}
```

#### 2. Getter Methods Instead of Direct Field Access

```php
// ✅ Good: Create dedicated getter method
/**
 * Get the entity status.
 *
 * @return int
 *   The status value.
 */
public function getStatus(): int {
  return (int) $this->get('status')->value;
}

// Always declare in entity interface
public function getStatus(): int;
```

#### 3. Production-Safe Migrations

```php
// ✅ Good: Safe migration with backward compatibility
function [module]_update_XXXX() {
  $entity_definition_update_manager = \Drupal::entityDefinitionUpdateManager();

  $field_storage_definition = $entity_definition_update_manager
    ->getFieldStorageDefinition('field_name', 'entity_type');

  if ($field_storage_definition) {
    $new_definition = BaseFieldDefinition::create('field_type')
      ->setSettings([
        'allowed_values' => [
          CONSTANT_OLD => 'Old Value',
          CONSTANT_NEW => 'New Value', // New option
        ],
      ]);

    $entity_definition_update_manager->updateFieldStorageDefinition($new_definition);
    drupal_flush_all_caches();
    \Drupal::logger('module_name')->info('Migration completed successfully.');
  }
}

// Use >= comparison for backward compatibility
if ($entity->getFieldValue() >= CONSTANT_MIN_VALUE) {
  // Works with both old and new values
}
```

**Migration Safety Checklist**:
- ✅ Backup database before running migrations
- ✅ Test on staging first
- ✅ Ensure backward compatibility
- ✅ Include error handling and logging
- ✅ Clear caches after schema changes
- ✅ Have rollback plan

### Drupal Best Practices

Follow these Drupal-specific patterns for maintainable, secure code.

#### Database API

**Always use database API, never raw SQL**:

```php
// ✅ Good: Use database API
$query = \Drupal::database()->select('node_field_data', 'n')
  ->fields('n', ['nid', 'title'])
  ->condition('n.status', 1)
  ->condition('n.type', 'article')
  ->range(0, 10);
$results = $query->execute()->fetchAll();

// ✅ Good: Use placeholders for dynamic values
$results = \Drupal::database()->query(
  "SELECT nid, title FROM {node_field_data} WHERE status = :status",
  [':status' => 1]
)->fetchAll();

// ❌ Bad: Raw SQL without placeholders (SQL injection risk)
$results = \Drupal::database()->query(
  "SELECT nid, title FROM node_field_data WHERE status = " . $status
);
```

#### Service Container and Dependency Injection

**Use dependency injection in classes**:

```php
// ✅ Good: Dependency injection in service
namespace Drupal\my_module\Service;

use Drupal\Core\Entity\EntityTypeManagerInterface;
use Drupal\Core\Config\ConfigFactoryInterface;

class MyService {

  public function __construct(
    private readonly EntityTypeManagerInterface $entityTypeManager,
    private readonly ConfigFactoryInterface $configFactory,
  ) {}

  public function doSomething() {
    $storage = $this->entityTypeManager->getStorage('node');
    // Use $storage
  }
}

// ❌ Bad: Using \Drupal static calls in classes
class MyService {
  public function doSomething() {
    $storage = \Drupal::entityTypeManager()->getStorage('node');
  }
}
```

#### Caching API

**Implement proper caching strategies**:

```php
// ✅ Good: Use cache with tags and contexts
$cache_tags = ['node:' . $node->id()];
$cache_contexts = ['user', 'url.path'];

$build = [
  '#markup' => $content,
  '#cache' => [
    'tags' => $cache_tags,
    'contexts' => $cache_contexts,
    'max-age' => 3600,
  ],
];

// Invalidate specific cache tags
\Drupal\Core\Cache\Cache::invalidateTags(['node:' . $node->id()]);

// Get from cache
$cid = 'my_module:my_data:' . $key;
if ($cache = \Drupal::cache()->get($cid)) {
  return $cache->data;
}

// Set cache
\Drupal::cache()->set($cid, $data, time() + 3600, ['my_module']);
```

#### Queue API

**Use queues for heavy operations**:

```php
// Define queue in [module].services.yml
// services:
//   queue.my_module_processor:
//     class: Drupal\Core\Queue\DatabaseQueue
//     factory: ['@queue', 'get']
//     arguments: ['my_module_processor']

// Add item to queue
$queue = \Drupal::queue('my_module_processor');
$queue->createItem(['data' => $data]);

// Process queue in QueueWorker plugin
/**
 * @QueueWorker(
 *   id = "my_module_processor",
 *   title = @Translation("My Module Processor"),
 *   cron = {"time" = 60}
 * )
 */
class MyModuleProcessor extends QueueWorkerBase {
  public function processItem($data) {
    // Process item
  }
}
```

#### Entity System

**Use entity API properly**:

```php
// ✅ Good: Use entity type manager
$node_storage = \Drupal::entityTypeManager()->getStorage('node');

// Load entity
$node = $node_storage->load($nid);

// Load multiple entities
$nodes = $node_storage->loadMultiple([1, 2, 3]);

// Query entities
$query = $node_storage->getQuery()
  ->condition('type', 'article')
  ->condition('status', 1)
  ->accessCheck(TRUE)  // Always include access check
  ->sort('created', 'DESC')
  ->range(0, 10);
$nids = $query->execute();

// Create entity
$node = $node_storage->create([
  'type' => 'article',
  'title' => 'My Title',
  'field_my_field' => 'value',
]);
$node->save();
```

#### Form API

**Build forms with proper validation**:

```php
namespace Drupal\my_module\Form;

use Drupal\Core\Form\FormBase;
use Drupal\Core\Form\FormStateInterface;

class MyForm extends FormBase {

  public function getFormId() {
    return 'my_module_my_form';
  }

  public function buildForm(array $form, FormStateInterface $form_state) {
    $form['name'] = [
      '#type' => 'textfield',
      '#title' => $this->t('Name'),
      '#required' => TRUE,
    ];

    $form['email'] = [
      '#type' => 'email',
      '#title' => $this->t('Email'),
      '#required' => TRUE,
    ];

    $form['submit'] = [
      '#type' => 'submit',
      '#value' => $this->t('Submit'),
    ];

    return $form;
  }

  public function validateForm(array &$form, FormStateInterface $form_state) {
    $email = $form_state->getValue('email');
    if (!\Drupal::service('email.validator')->isValid($email)) {
      $form_state->setErrorByName('email', $this->t('Invalid email address.'));
    }
  }

  public function submitForm(array &$form, FormStateInterface $form_state) {
    $values = $form_state->getValues();
    // Process form submission
    \Drupal::messenger()->addStatus($this->t('Form submitted successfully.'));
  }
}
```

#### Translation API

**Make all user-facing strings translatable**:

```php
// ✅ Good: Use t() or TranslatableMarkup
use Drupal\Core\StringTranslation\TranslatableMarkup;

$message = $this->t('Hello @name', ['@name' => $name]);

// For complex strings
$markup = new TranslatableMarkup('Welcome to @site', [
  '@site' => $site_name,
]);

// In render arrays
$build['message'] = [
  '#markup' => $this->t('Welcome!'),
];

// ❌ Bad: Hardcoded English strings
$message = "Hello " . $name;
```

#### Configuration Management

**Use configuration API for settings**:

```php
// Get configuration
$config = \Drupal::config('my_module.settings');
$value = $config->get('my_setting');

// Set configuration (editable)
$config = \Drupal::configFactory()->getEditable('my_module.settings');
$config->set('my_setting', $value);
$config->save();

// In services with dependency injection
public function __construct(ConfigFactoryInterface $config_factory) {
  $this->config = $config_factory->get('my_module.settings');
}
```

### Form Development

- Use Drupal's Form API states system for conditional fields
- Implement proper form validation
- Follow Drupal's form building patterns
- Use field groups for organizing form elements
- Implement proper access checks for form operations

### Logging Standards

- Implement comprehensive logging for debugging
- Use Drupal's logger service: `\Drupal::logger('module_name')->notice()`
- Log important actions and state changes
- Include relevant data in log messages for debugging

### Code Style

- **Type Declarations**: Always use explicit return type declarations
- **Type Hints**: Use appropriate PHP type hints for method parameters
- **PHPDoc**: Provide complete documentation for classes, methods, properties
- **Array Alignment**: Always align `=>` in multi-line array declarations
- **Variable Alignment**: Always align `=` in sequential variable definitions
- **Controllers**: Should be final classes, use dependency injection, keep thin
- **Services**: Register in `services.yml`, keep focused on single responsibility

### Entity Updates

- All entity structure changes must include update hooks in module `.install` file
- Follow Drupal's update hook system
- Maintain backward compatibility

### Role Permissions Management

```php
// Grant permissions
user_role_grant_permissions($role_id, [$permission_array]);

// Revoke permissions
user_role_revoke_permissions($role_id, [$permission_array]);

// Check permissions
user_role_permissions($role_id);

// Common roles: [list project-specific roles]
// Built-in roles: Use constants like AccountInterface::AUTHENTICATED_ROLE for authenticated users
// Always include drupal_flush_all_caches() after permission changes
```

## Directory Structure

### Key Directories

- **Web Root**: `/web/` (or `/docroot/`, `/html/` - see "Web Root Directory" section)
- **Custom Modules**: `/web/modules/custom/`
- **Configuration**: `/config/sync/` (or `/config/[site.domain]/` for multisite)
- **Settings**: `/web/sites/default/settings.php`
- **Patches**: `/patches/` organized by core/contrib
- **Tests**: `/tests/`
- **Build Scripts**: `/build/` or `/scripts/`
- **Tools**: `/tools/`

### Finding Functionality

- **[Feature 1]**: `/web/modules/custom/[module]/`
- **[Feature 2]**: `/web/modules/custom/[module]/`
- **Site configuration**: `/config/sync/`
- **Custom theme**: `/web/themes/custom/[theme_name]/`
- **Testing**: `/tests/`

### Common Development Paths

- **Adding routes**: Module `routing.yml` files
- **Creating forms**: Module `src/Form/` directories
- **Entity definitions**: Module `src/Entity/` directories
- **Custom permissions**: Module `permissions.yml` files
- **Database updates**: Module `.install` files

## Multilingual Configuration

Quick reference for multilingual Drupal setup. For comprehensive guide, see [Drupal Multilingual Documentation](https://www.drupal.org/docs/multilingual-guide).

### Quick Start

**Enable Core Modules**:
```bash
ddev drush pm:enable language locale content_translation config_translation
ddev drush cr
```

**Add Languages**:
```bash
ddev drush language:add pl
ddev drush language:add es
# Or via UI: /admin/config/regional/language/add
```

**Check Current Setup**:
```bash
ddev drush language:list
ddev drush config:get system.site default_langcode
ddev drush config:get language.content_settings.node.*
```

### Language Detection

**Configure at**: `/admin/config/regional/language/detection`

**Recommended Order**:
1. **URL** - Use path prefix (`/en/`, `/pl/`) for SEO benefits
2. User preference
3. Session
4. Browser detection

**URL Prefix Configuration**:
```bash
# English: example.com/en/page
# Polish: example.com/pl/page
```

### Enable Content Translation

**Content Types**:
```bash
# Via Drush
ddev drush config:set language.content_settings.node.article third_party_settings.content_translation.enabled true

# Via UI: Structure > Content types > [Type] > Edit > Language settings
```

**Custom Entities**:
```php
/**
 * @ContentEntityType(
 *   translatable = TRUE,
 * )
 */
$fields['name'] = BaseFieldDefinition::create('string')
  ->setTranslatable(TRUE);
```

### Translation Workflows

**Manual Translation**:
1. Create content in default language
2. Edit content > Translate tab
3. Add translation for each language

**TMGMT Module** (bulk translation):
```bash
ddev composer require drupal/tmgmt
ddev drush pm:enable tmgmt tmgmt_content
# Configure at /admin/tmgmt
```

### Interface Translation

```bash
# Update translations
ddev drush locale:check
ddev drush locale:update

# Translate custom strings at /admin/config/regional/translate
```

### Language Switcher

**Enable Block**: Structure > Block layout > Place "Language switcher" block

**Custom Switcher**:
```php
function custom_language_switcher() {
  $languages = \Drupal::languageManager()->getLanguages();
  $links = [];
  foreach ($languages as $lang) {
    $links[$lang->getId()] = [
      'title' => $lang->getName(),
      'url' => \Drupal::url('<current>', [], ['language' => $lang]),
    ];
  }
  return ['#theme' => 'links', '#links' => $links];
}
```

### Best Practices

- ✅ Use URL prefix for SEO (`/en/`, `/pl/`)
- ✅ Enable translation for all content types, taxonomies, menus
- ✅ Implement hreflang tags (see SEO section)
- ✅ Use `t()` and `TranslatableMarkup` for all strings
- ✅ Test with `drush locale:check` regularly
- ✅ Enable language cache contexts

### Common Issues

**Missing translations**: `ddev drush locale:update`
**Content not translatable**: Check Language settings tab on content type
**Switcher not working**: Verify URL-based detection is enabled
**Fields not translatable**: Edit field > Enable translation

### Quick Reference

```php
// In code
$this->t('Hello @name', ['@name' => $name]);
new TranslatableMarkup('Welcome!');

// In Twig
{{ 'Hello World'|trans }}
{{ 'Hello @name'|trans({'@name': name}) }}
```

**Document your project languages in Project Overview section above.**

## Configuration Management

### Config Export/Import

```bash
# Export configuration
ddev drush cex

# Import configuration
ddev drush cim

# Show config differences
ddev drush config:status
```

### Config Split (environment-specific configuration)

<!--
Uncomment this section if using config_split module
-->

<!--
The project uses `config_split` module to manage environment-specific configuration.

**Config Split Structure**:
```
config/
├── sync/              # Shared configuration (all environments)
├── dev/               # Development-only configuration
├── staging/           # Staging-specific configuration
└── prod/              # Production-specific configuration
```

**Enable/Disable Config Splits**:
```bash
# Enable development split
ddev drush config-split:export dev

# Activate specific split
ddev drush csex dev

# Deactivate split
ddev drush config-split:deactivate dev
```

**Common Use Cases**:
- **Development split**: Development modules (devel, webprofiler, stage_file_proxy)
- **Production split**: Production-only modules (purge, cdn, shield)
- **Environment-specific settings**: Different API endpoints, debug settings

**Configuration**:
- Create config split at: `/admin/config/development/configuration/config-split`
- Add modules to blacklist (exclude from main config)
- Or add modules to graylist (override in split)
- Export configuration: `ddev drush cex`
-->

### Config Ignore (if using config_ignore module)

<!--
The project uses the `config_ignore` module to exclude certain configuration entities from being exported/imported. This is important for configurations that are site-specific or managed manually.

**Config Ignore Settings**:
- Location: `/config/sync/config_ignore.settings.yml`
- Common ignored patterns:
  - [List patterns specific to your project]
-->

## Security

### Security Best Practices

- Follow Drupal's security best practices
- Implement proper access control and permission checks
- Use Drupal's security features for form validation and CSRF protection
- HTTPS redirect must remain present
- Sanitize all user input
- Use Drupal's database abstraction layer to prevent SQL injection
- Never store sensitive data in configuration
- Use environment variables for API keys and secrets

### Security Update Commands

```bash
# Check for security updates
ddev drush pm:security

# Check for available updates
ddev drush pm:security-php

# Update Drupal core (security updates)
ddev composer update drupal/core-recommended --with-dependencies

# Update all modules (security only)
ddev composer update --security-only

# Update specific module with dependencies
ddev composer update drupal/[module_name] --with-dependencies

# After updates, run database updates
ddev drush updatedb

# Clear caches
ddev drush cr
```

### Security Audit

```bash
# Review enabled modules
ddev drush pm:list --status=enabled

# Check user permissions
ddev drush role:list

# View user roles and permissions
ddev drush role:perm:list

# Review users with admin access
ddev drush user:information --uid=1

# Check file permissions
ddev exec ls -la web/sites/default/

# Review security-related configuration
ddev drush config:get system.site page.403
ddev drush config:get system.site page.404
```

### Security Hardening

**File Permissions**:
- Settings files should be read-only: `chmod 444 settings.php`
- Files directory should not be executable: `chmod 755 sites/default/files`
- Never make web root writable by web server

**Configuration**:
- Disable PHP execution in files directory
- Configure .htaccess properly
- Enable security headers
- Use strong session settings
- Implement rate limiting for forms

**Code Security**:
- Always use parameter placeholders in database queries
- Escape output: Use `\Drupal\Component\Utility\Html::escape()`
- Validate input: Use Form API validation
- Check permissions: Use `$account->hasPermission()`
- Sanitize URLs: Use `\Drupal\Core\Url::fromUri()`

### Security Monitoring

```bash
# Review watchdog for security events
ddev drush watchdog:show --severity=Error --count=100

# Check failed login attempts
ddev drush sql:query "SELECT * FROM watchdog WHERE type='user' AND message LIKE '%Failed login%' ORDER BY wid DESC LIMIT 20;"

# Review access logs (if available)
ddev exec tail -n 100 /var/log/nginx/access.log
```

## Headless/API-First Development

### JSON:API (Core Module)

**JSON:API** is included in Drupal core and provides a complete REST API for all content entities.

**Enable JSON:API**:
```bash
# Enable module
ddev drush pm:enable jsonapi

# Optional: Enable extras module for better DX
ddev composer require drupal/jsonapi_extras
ddev drush pm:enable jsonapi_extras
```

**Basic Usage**:

**Get all articles**:
```bash
GET /jsonapi/node/article
```

**Get specific article**:
```bash
GET /jsonapi/node/article/{uuid}
```

**Get article with relationships**:
```bash
GET /jsonapi/node/article/{uuid}?include=field_image,uid
```

**Filter articles**:
```bash
GET /jsonapi/node/article?filter[status]=1&filter[field_category.name]=Technology
```

**Sort articles**:
```bash
GET /jsonapi/node/article?sort=-created
```

**Pagination**:
```bash
GET /jsonapi/node/article?page[limit]=10&page[offset]=20
```

**Create content via API**:
```bash
POST /jsonapi/node/article
Content-Type: application/vnd.api+json
Authorization: Bearer {token}

{
  "data": {
    "type": "node--article",
    "attributes": {
      "title": "New Article",
      "body": {
        "value": "Content here",
        "format": "basic_html"
      }
    }
  }
}
```

### GraphQL Integration

**Install GraphQL Module**:
```bash
ddev composer require drupal/graphql
ddev drush pm:enable graphql

# Install GraphQL Compose for automatic schema generation
ddev composer require drupal/graphql_compose
ddev drush pm:enable graphql_compose
```

**GraphQL Explorer**:
- Access at: `/admin/config/graphql`
- Built-in GraphiQL interface for testing queries

**Example Query**:
```graphql
query {
  nodeArticles(first: 10) {
    nodes {
      id
      title
      body {
        value
      }
      created
      author {
        name
      }
    }
  }
}
```

**Example Mutation**:
```graphql
mutation {
  createArticle(
    title: "New Article"
    body: "Article content"
  ) {
    entity {
      id
      title
    }
  }
}
```

### API Authentication

**Simple OAuth (Recommended)**:

```bash
# Install Simple OAuth
ddev composer require drupal/simple_oauth
ddev drush pm:enable simple_oauth

# Generate keys
mkdir -p keys
openssl genrsa -out keys/private.key 2048
openssl rsa -in keys/private.key -pubout -out keys/public.key
chmod 600 keys/*.key

# Configure at /admin/config/people/simple_oauth
```

**Get Access Token**:
```bash
POST /oauth/token
Content-Type: application/x-www-form-urlencoded

grant_type=password
&client_id={client_id}
&client_secret={client_secret}
&username={username}
&password={password}
```

**Use Token in Requests**:
```bash
GET /jsonapi/node/article
Authorization: Bearer {access_token}
```

### CORS Configuration

**Enable CORS** for headless frontend (in `services.yml`):

```yaml
cors.config:
  enabled: true
  allowedOrigins: ['http://localhost:3000', 'https://yourdomain.com']
  allowedMethods: ['GET', 'POST', 'PATCH', 'DELETE', 'OPTIONS']
  allowedHeaders: ['*']
  exposedHeaders: false
  maxAge: 1000
  supportsCredentials: true
```

**Or use CORS module**:
```bash
ddev composer require drupal/cors
ddev drush pm:enable cors
# Configure at /admin/config/services/cors
```

### Decoupled Architecture Patterns

**1. Fully Decoupled** (Headless):
- Drupal as pure API backend
- Separate frontend (React, Vue, Next.js)
- Complete design freedom
- Best for: Mobile apps, multiple frontends

**2. Progressively Decoupled**:
- Drupal renders some pages
- JavaScript framework for interactive components
- Best for: Existing sites adding modern UI

**3. Hybrid**:
- Mix of Drupal templates and API-driven sections
- Best for: Gradual migration

### Frontend Frameworks Integration

**Next.js (React)**:
```javascript
// Example: Fetch articles from Drupal
export async function getStaticProps() {
  const res = await fetch('https://drupal.site/jsonapi/node/article')
  const data = await res.json()

  return {
    props: {
      articles: data.data
    }
  }
}
```

**Nuxt.js (Vue)**:
```javascript
// Example: Fetch articles
export default {
  async asyncData({ $axios }) {
    const articles = await $axios.$get('/jsonapi/node/article')
    return { articles: articles.data }
  }
}
```

### API Performance Optimization

**Enable Caching**:
```bash
# Configure cache in settings.php
$settings['cache']['bins']['jsonapi_normalizations'] = 'cache.backend.database';

# Enable BigPipe for progressive rendering
$settings['container_yamls'][] = 'modules/contrib/big_pipe/big_pipe.services.yml';
```

**Use Subrequests Module**:
```bash
# Batch multiple API calls
ddev composer require drupal/subrequests
ddev drush pm:enable subrequests

# Make batch request
POST /subrequests?_format=json
[
  {"uri": "/jsonapi/node/article/123"},
  {"uri": "/jsonapi/node/article/456"}
]
```

### API Security Best Practices

- ✅ Use OAuth tokens, not basic auth
- ✅ Implement rate limiting (use `rate_limiter` module)
- ✅ Validate all input data
- ✅ Use HTTPS in production
- ✅ Limit API access with permissions
- ✅ Monitor API usage and errors
- ✅ Version your API endpoints
- ✅ Document API for consumers

### API Documentation

**Use OpenAPI/Swagger**:
```bash
# Install OpenAPI module
ddev composer require drupal/openapi
ddev drush pm:enable openapi openapi_ui

# View documentation
# Visit: /admin/config/services/openapi
```

**Or use Schemata**:
```bash
ddev composer require drupal/schemata
ddev drush pm:enable schemata schemata_json_schema

# Generate JSON schemas
# Visit: /schemata/[entity_type]/[bundle]
```

### Testing API Endpoints

```bash
# Test with curl
curl -X GET "https://drupal.site/jsonapi/node/article" \
  -H "Accept: application/vnd.api+json"

# Test authenticated endpoint
curl -X GET "https://drupal.site/jsonapi/node/article" \
  -H "Authorization: Bearer {token}" \
  -H "Accept: application/vnd.api+json"

# Test POST
curl -X POST "https://drupal.site/jsonapi/node/article" \
  -H "Authorization: Bearer {token}" \
  -H "Content-Type: application/vnd.api+json" \
  -d '{"data":{"type":"node--article","attributes":{"title":"Test"}}}'
```

## SEO & Structured Data

### Core SEO Modules

**Metatag Module** (Essential):
```bash
ddev composer require drupal/metatag
ddev drush pm:enable metatag metatag_open_graph metatag_twitter_cards
```

**Pathauto** (Clean URLs):
```bash
ddev composer require drupal/pathauto
ddev drush pm:enable pathauto
```

**Simple XML Sitemap**:
```bash
ddev composer require drupal/simple_sitemap
ddev drush pm:enable simple_sitemap

# Generate sitemap
ddev drush simple-sitemap:generate

# Access at: /sitemap.xml
```

**Redirect Module** (301/302 redirects):
```bash
ddev composer require drupal/redirect
ddev drush pm:enable redirect
```

### Schema.org Structured Data

**Schema.org Metatag Module**:
```bash
ddev composer require drupal/schema_metatag
ddev drush pm:enable schema_metatag schema_article schema_organization schema_web_page
```

**Configure Schema.org**:

1. **Organization Schema** (Site-wide):
   - Navigate to: `/admin/config/search/metatag/global`
   - Enable "Schema.org" tab
   - Add Organization schema:
   ```json
   {
     "@type": "Organization",
     "name": "[Site Name]",
     "url": "[Site URL]",
     "logo": "[Logo URL]",
     "sameAs": [
       "[Facebook URL]",
       "[Twitter URL]",
       "[LinkedIn URL]"
     ]
   }
   ```

2. **Article Schema** (Content type):
   - Navigate to: `/admin/structure/types/manage/article/fields`
   - Add Schema.org fields or use tokens:
   ```json
   {
     "@type": "Article",
     "headline": "[node:title]",
     "datePublished": "[node:created:html_datetime]",
     "dateModified": "[node:changed:html_datetime]",
     "author": {
       "@type": "Person",
       "name": "[node:author:display-name]"
     },
     "image": "[node:field_image:entity:url]",
     "publisher": {
       "@type": "Organization",
       "name": "[site:name]",
       "logo": "[site:logo]"
     }
   }
   ```

3. **Breadcrumb Schema**:
   ```json
   {
     "@type": "BreadcrumbList",
     "itemListElement": [
       {
         "@type": "ListItem",
         "position": 1,
         "name": "Home",
         "item": "[site:url]"
       }
     ]
   }
   ```

### Open Graph & Twitter Cards

**Open Graph Tags** (Facebook, LinkedIn):
```yaml
# Configure at /admin/structure/types/manage/[content_type]/fields

og:title: "[node:title]"
og:description: "[node:summary]"
og:image: "[node:field_image:entity:url]"
og:url: "[current-page:url]"
og:type: "article"
og:site_name: "[site:name]"
```

**Twitter Card Tags**:
```yaml
twitter:card: "summary_large_image"
twitter:site: "@yourtwitterhandle"
twitter:title: "[node:title]"
twitter:description: "[node:summary]"
twitter:image: "[node:field_image:entity:url]"
```

### Hreflang for Multilingual Sites

**Using Hreflang module**:
```bash
ddev composer require drupal/hreflang
ddev drush pm:enable hreflang

# Configure at /admin/config/search/hreflang
```

**Manual implementation in template**:
```twig
{# In html.html.twig #}
{% for langcode, language in languages %}
  <link rel="alternate" hreflang="{{ langcode }}" href="{{ path('<current>', {}, {'language': language}) }}" />
{% endfor %}
```

### Robots.txt & Robots Meta Tags

**Robots.txt**:
```bash
# Edit web/robots.txt
User-agent: *
Disallow: /admin/
Disallow: /user/
Disallow: /search/
Allow: /

Sitemap: https://yoursite.com/sitemap.xml
```

**RobotsTxt module** (manage via UI):
```bash
ddev composer require drupal/robotstxt
ddev drush pm:enable robotstxt

# Configure at /admin/config/search/robotstxt
```

**Robots Meta Tags**:
```yaml
# Per content type at /admin/structure/types/manage/[type]/fields

robots: "index, follow"
# or for no-index:
robots: "noindex, nofollow"
```

### Canonical URLs

**Configure Canonical**:
```yaml
# Via Metatag module
canonical_url: "[current-page:url:absolute]"

# For content types
canonical_url: "[node:url:absolute]"
```

**For Multilingual**:
- Ensure each language version has its own canonical
- Use hreflang to link language versions

### XML Sitemap Configuration

```bash
# Configure Simple XML Sitemap
# Visit: /admin/config/search/simplesitemap

# Settings to configure:
# - Base URL
# - Include content types
# - Update frequency
# - Priority

# Generate sitemap
ddev drush simple-sitemap:generate

# Rebuild sitemap
ddev drush simple-sitemap:rebuild-queue
```

**Multi-site Sitemaps**:
```yaml
# config/sync/simple_sitemap.settings.yml
engines:
  google: true
  bing: true
variants:
  default:
    label: 'Default sitemap'
    types:
      node:
        - article
        - page
```

### SEO-Friendly URLs

**Pathauto Patterns**:
```yaml
# Configure at /admin/config/search/path/patterns

# Articles: /blog/[node:title]
# Pages: /[node:title]
# Terms: /category/[term:name]

# Bulk generate paths
ddev drush pathauto:generate

# Update existing paths
ddev drush pathauto:update-all
```

### Performance for SEO

**Image Optimization**:
- Use WebP format when possible
- Implement lazy loading
- Use responsive images (image styles)
- Add proper alt attributes

**Page Speed**:
```bash
# Enable caching
ddev drush config:set system.performance css.preprocess 1
ddev drush config:set system.performance js.preprocess 1

# Enable BigPipe
ddev drush pm:enable big_pipe

# Check Core Web Vitals
# Use Google PageSpeed Insights
# Use Lighthouse in Chrome DevTools
```

### SEO Testing & Validation

**Test Structured Data**:
- Google Rich Results Test: https://search.google.com/test/rich-results
- Schema.org Validator: https://validator.schema.org/

**Test Open Graph**:
- Facebook Sharing Debugger: https://developers.facebook.com/tools/debug/
- LinkedIn Post Inspector: https://www.linkedin.com/post-inspector/

**Test Twitter Cards**:
- Twitter Card Validator: https://cards-dev.twitter.com/validator

**Check Sitemaps**:
```bash
# Validate sitemap XML
curl https://yoursite.com/sitemap.xml

# Submit to Google Search Console
# Submit to Bing Webmaster Tools
```

### SEO Checklist

**On-Page SEO**:
- [ ] Title tags optimized (50-60 characters)
- [ ] Meta descriptions (150-160 characters)
- [ ] H1 tag present and unique per page
- [ ] URL structure clean and descriptive
- [ ] Images have alt attributes
- [ ] Internal linking implemented
- [ ] Content is unique and valuable
- [ ] Mobile-friendly responsive design

**Technical SEO**:
- [ ] XML sitemap generated and submitted
- [ ] Robots.txt configured
- [ ] Canonical URLs set
- [ ] Schema.org markup implemented
- [ ] HTTPS enabled
- [ ] Page speed optimized (Core Web Vitals)
- [ ] 404 pages handled properly
- [ ] Redirects (301) for changed URLs

**Multilingual SEO**:
- [ ] Hreflang tags implemented
- [ ] Language-specific sitemaps
- [ ] Canonical URLs per language
- [ ] Localized content (not just translated)

### Monitoring SEO Performance

**Google Search Console**:
```bash
# Add site property
# Verify ownership (via DNS or HTML file)
# Submit sitemap
# Monitor:
#   - Index coverage
#   - Performance (clicks, impressions)
#   - Core Web Vitals
#   - Mobile usability
```

**Google Analytics 4**:
```bash
# Install Google Analytics module
ddev composer require drupal/google_analytics
ddev drush pm:enable google_analytics

# Configure at /admin/config/system/google-analytics
```

**SEO Audit Tools**:
- Screaming Frog SEO Spider
- Ahrefs Site Audit
- SEMrush Site Audit
- Moz Pro

## JavaScript and CSS

### JavaScript Aggregation

**Common Issues**:
- Missing dependencies in `.libraries.yml` files
- Incorrect loading order causing undefined objects/functions
- `drupalSettings` not available when needed

**Solutions**:
- Add proper dependencies: `core/jquery`, `core/drupal`, `core/drupalSettings`, `core/once`
- Use error checking: `if (typeof $.fn.pluginName === 'function')`
- Use modern `once()` function instead of jQuery `.once()`
- Always test with JS aggregation enabled: `admin/config/development/performance`

### CSS Organization

- Follow BEM naming convention (or project-specific convention)
- Use SCSS/SASS if applicable
- Organize stylesheets by component/feature
- Use single directory compontents if applicable

## Frontend Development

<!--
===========================================
HOW TO DISCOVER THEME STRUCTURE - GUIDE FOR AI/LLM
===========================================

Use this guide to discover the theme structure of a Drupal project.
Priority order: 1) Config + theme info file, 2) File system exploration, 3) Drush/PHP commands

**Theme File Types Reference:**

| File Type | Location | Purpose |
|-----------|----------|---------|
| Theme info | `[theme].info.yml` | Theme definition, regions, libraries |
| Libraries | `[theme].libraries.yml` | CSS/JS asset definitions |
| Theme file | `[theme].theme` | Preprocess functions, hooks |
| Twig templates | `templates/` | HTML templates |
| Components (SDC) | `components/` | Single Directory Components |
| Stylesheets | `css/` or `scss/` | Compiled/source styles |
| JavaScript | `js/` | JavaScript files |

### 1. FIND ACTIVE THEME

```bash
# Check config for default theme
cat config/sync/system.theme.yml

# Via Drush
ddev drush config:get system.theme default

# List all themes
ddev drush pm:list --type=theme

# Get active theme info via PHP
ddev drush php:eval "
\$theme = \Drupal::theme()->getActiveTheme();
echo 'Name: ' . \$theme->getName() . PHP_EOL;
echo 'Path: ' . \$theme->getPath() . PHP_EOL;
echo 'Base themes: ' . implode(', ', array_keys(\$theme->getBaseThemeExtensions())) . PHP_EOL;
"
```

### 2. THEME FILE STRUCTURE DISCOVERY

```bash
# List theme files
ls -la web/themes/custom/[theme_name]/

# Find all Twig templates
find web/themes/custom/[theme_name] -name "*.twig" -type f

# Find all SCSS/CSS files
find web/themes/custom/[theme_name] -name "*.scss" -o -name "*.css" | head -20

# Find all JS files
find web/themes/custom/[theme_name] -name "*.js" -type f

# Check theme info file
cat web/themes/custom/[theme_name]/[theme_name].info.yml

# Check libraries definition
cat web/themes/custom/[theme_name]/[theme_name].libraries.yml

# Check for Single Directory Components
ls web/themes/custom/[theme_name]/components/
```

### 3. TWIG TEMPLATE DISCOVERY

```bash
# Enable Twig debugging (in development.services.yml)
# Then check HTML source for template suggestions

# Find template source via PHP
ddev drush php:eval "
\$theme = \Drupal::theme()->getActiveTheme();
echo 'Templates path: ' . \$theme->getPath() . '/templates' . PHP_EOL;
"

# List Twig namespaces
ddev drush php:eval "
\$loader = \Drupal::service('twig.loader.filesystem');
\$namespaces = \$loader->getNamespaces();
foreach (\$namespaces as \$ns) {
  echo \$ns . ': ' . implode(', ', \$loader->getPaths(\$ns)) . PHP_EOL;
}
"
```

### 4. PREPROCESS FUNCTIONS

```bash
# Check theme file for preprocess functions
grep -n "function.*_preprocess" web/themes/custom/[theme_name]/[theme_name].theme

# Common patterns:
# [theme_name]_preprocess_node()
# [theme_name]_preprocess_paragraph()
# [theme_name]_preprocess_page()
# [theme_name]_theme_suggestions_*_alter()
```

### 5. SINGLE DIRECTORY COMPONENTS (SDC)

```bash
# Check if SDC module is enabled
ddev drush pm:list | grep sdc

# List components
ls web/themes/custom/[theme_name]/components/

# Component structure:
# components/
#   card/
#     card.component.yml    # Component definition
#     card.twig             # Template
#     card.css              # Styles (optional)
#     card.js               # Scripts (optional)

# Read component definition
cat web/themes/custom/[theme_name]/components/[component]/[component].component.yml
```

### RECOMMENDED WORKFLOW

1. **First**: Check `config/sync/system.theme.yml` for active theme name
2. **Second**: Read `[theme].info.yml` for theme configuration and base themes
3. **Third**: Explore `templates/` directory for Twig templates
4. **Fourth**: Check `[theme].theme` file for preprocess functions
5. **Fifth**: Check `components/` directory if using SDC

-->

### Theme Development Setup

**Theme location**: `/web/themes/custom/[theme_name]/` (see [Web Root Directory](#web-root-directory) for path variations)

**Initial Setup**:
```bash
# Navigate to theme directory
cd web/themes/custom/[theme_name]

# Install Node.js dependencies
npm install
# or
yarn install
```

### Build Commands

```bash
# Development build (non-minified, with source maps)
npm run dev
# or
gulp

# Production build (minified, optimized)
npm run build
# or
gulp dist

# Watch mode (auto-rebuild on file changes)
npm run watch
# or
gulp watch
```

### Common Frontend Tasks

**Working with SCSS**:
```bash
# Compile SCSS to CSS
gulp sass

# Lint SCSS files
gulp lint:scss

# Watch SCSS changes
gulp watch:scss
```

**Working with JavaScript**:
```bash
# Compile/transpile JavaScript
gulp js

# Lint JavaScript
gulp lint:js

# Watch JavaScript changes
gulp watch:js
```

**Asset Optimization**:
```bash
# Optimize images
gulp images

# Generate icon fonts
gulp fonts

# Process all assets
gulp assets
```

### Development Workflow

1. **Start watch mode**:
   ```bash
   cd web/themes/custom/[theme_name]
   npm run watch
   ```

2. **Make changes** to SCSS/JS files

3. **View changes** in browser (auto-reload if configured)

4. **Before committing**:
   ```bash
   npm run build  # Production build
   ```

### Theme File Structure

```
[theme_name]/
├── package.json           # Node dependencies
├── gulpfile.js           # Build configuration
├── webpack.config.js     # Webpack config (if using)
├── scss/                 # SCSS source files
│   ├── components/       # Component styles
│   ├── base/            # Base styles
│   ├── layout/          # Layout styles
│   └── style.scss       # Main SCSS file
├── js/                   # JavaScript source files
│   └── global.js        # Main JS file
├── css/                  # Compiled CSS (generated)
├── templates/            # Twig templates
├── images/              # Source images
└── dist/                # Compiled assets (generated)
```

### Asset Libraries

Define in `[theme_name].libraries.yml`:
```yaml
global:
  css:
    theme:
      css/style.css: {}
  js:
    js/global.js: {}
  dependencies:
    - core/drupal
    - core/jquery
```

### Adding/Overriding Twig Templates

**Enable Twig debugging** (in `web/sites/default/development.services.yml`):
```yaml
parameters:
  twig.config:
    debug: true
    auto_reload: true
    cache: false
```

**Find template suggestions**:
1. Enable Twig debugging
2. View page source - look for HTML comments like:
   ```html
   <!-- THEME DEBUG -->
   <!-- THEME HOOK: 'node' -->
   <!-- FILE NAME SUGGESTIONS:
      * node--article--full.html.twig
      * node--article.html.twig
      * node--1.html.twig
      * node.html.twig
   -->
   ```
3. Copy original template from `core/themes/` or contrib module
4. Place in `templates/` directory with appropriate name
5. Clear cache: `ddev drush cr`

**Template naming conventions**:
- **Nodes**: `node--[type]--[view-mode].html.twig`
- **Paragraphs**: `paragraph--[type].html.twig`
- **Blocks**: `block--[block-type].html.twig`
- **Fields**: `field--[field-name]--[entity-type].html.twig`

**Standard template directory structure**:
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

### Preprocess Functions

Add to `[theme_name].theme` file:

```php
<?php

/**
 * Implements hook_preprocess_node().
 */
function [theme_name]_preprocess_node(&$variables) {
  $node = $variables['node'];

  // Add custom variables
  $variables['custom_var'] = 'value';

  // Add classes based on content type
  if ($node->bundle() == 'article') {
    $variables['attributes']['class'][] = 'article-content';
  }
}

/**
 * Implements hook_preprocess_paragraph().
 */
function [theme_name]_preprocess_paragraph(&$variables) {
  $paragraph = $variables['paragraph'];
  $variables['paragraph_type'] = $paragraph->bundle();
}

/**
 * Implements hook_preprocess_page().
 */
function [theme_name]_preprocess_page(&$variables) {
  // Add site-wide variables
  $variables['site_name'] = \Drupal::config('system.site')->get('name');
}

/**
 * Implements hook_theme_suggestions_node_alter().
 */
function [theme_name]_theme_suggestions_node_alter(array &$suggestions, array $variables) {
  // Add suggestion based on custom field
  $node = $variables['elements']['#node'];
  if ($node->hasField('field_layout') && !$node->get('field_layout')->isEmpty()) {
    $suggestions[] = 'node__' . $node->bundle() . '__' . $node->get('field_layout')->value;
  }
}

/**
 * Implements hook_form_alter().
 */
function [theme_name]_form_alter(&$form, \Drupal\Core\Form\FormStateInterface $form_state, $form_id) {
  // Customize forms
  if (strpos($form_id, 'search') !== FALSE) {
    $form['keys']['#attributes']['placeholder'] = t('Search...');
  }
}
```

### Single Directory Components (SDC)

**Requirements**: Drupal 10.1+ or SDC contrib module for Drupal 10.0

**Enable SDC**:
```bash
# Core SDC (Drupal 10.1+)
ddev drush pm:enable sdc

# Or contrib for Drupal 10.0
ddev composer require drupal/sdc
ddev drush pm:enable sdc
```

**Create a new component**:

1. Create directory: `web/themes/custom/[theme_name]/components/card/`

2. Create `card.component.yml`:
```yaml
name: Card
status: stable
props:
  type: object
  properties:
    title:
      type: string
      title: Title
    description:
      type: string
      title: Description
    image_url:
      type: string
      title: Image URL
    link:
      type: string
      title: Link URL
slots:
  content:
    title: Content
```

3. Create `card.twig`:
```twig
<article class="card">
  {% if image_url %}
    <img src="{{ image_url }}" alt="{{ title }}" class="card__image">
  {% endif %}
  <div class="card__content">
    <h3 class="card__title">{{ title }}</h3>
    {% if description %}
      <p class="card__description">{{ description }}</p>
    {% endif %}
    {{ content }}
    {% if link %}
      <a href="{{ link }}" class="card__link">Read more</a>
    {% endif %}
  </div>
</article>
```

4. Create `card.css`:
```css
.card {
  border: 1px solid #ddd;
  border-radius: 8px;
  overflow: hidden;
}
.card__image {
  width: 100%;
  height: auto;
}
.card__content {
  padding: 1rem;
}
```

**Use component in templates**:
```twig
{# Include component #}
{% include '[theme_name]:card' with {
  title: node.label,
  description: content.body|render|striptags|slice(0, 150) ~ '...',
  image_url: file_url(node.field_image.entity.uri.value),
  link: path('entity.node.canonical', {'node': node.id})
} %}

{# Embed with slots #}
{% embed '[theme_name]:card' with { title: 'My Card' } %}
  {% block content %}
    <p>Custom slot content here</p>
  {% endblock %}
{% endembed %}
```

**List available components**:
```bash
ddev drush sdc:list
```

### Theme Hook Suggestions

Add custom template suggestions:

```php
/**
 * Implements hook_theme_suggestions_page_alter().
 */
function [theme_name]_theme_suggestions_page_alter(array &$suggestions, array $variables) {
  // Add suggestion based on node type
  if ($node = \Drupal::routeMatch()->getParameter('node')) {
    $suggestions[] = 'page__node__' . $node->bundle();
  }

  // Add suggestion based on path
  $current_path = \Drupal::service('path.current')->getPath();
  $path_alias = \Drupal::service('path_alias.manager')->getAliasByPath($current_path);
  $path_alias = ltrim($path_alias, '/');
  $suggestions[] = 'page__' . str_replace('/', '__', $path_alias);
}

/**
 * Implements hook_theme_suggestions_views_view_alter().
 */
function [theme_name]_theme_suggestions_views_view_alter(array &$suggestions, array $variables) {
  $suggestions[] = 'views_view__' . $variables['view']->id();
  $suggestions[] = 'views_view__' . $variables['view']->id() . '__' . $variables['view']->current_display;
}
```

### Troubleshooting Frontend Builds

```bash
# Clear node_modules and reinstall
rm -rf node_modules package-lock.json
npm install

# Clear build cache
rm -rf dist/ css/ js/compiled/

# Check Node/NPM versions
node -v
npm -v

# Update dependencies
npm update
```

### Performance Optimization

- Minify CSS and JavaScript for production
- Use image optimization (imagemin)
- Implement critical CSS loading
- Use font-display: swap for web fonts
- Enable Drupal CSS/JS aggregation
- Consider using AdvAgg module

## Environment Indicators

- **Visual verification**: Check indicators display correctly on all pages
- **Color scheme**: GREEN (Local), BLUE (DEV), ORANGE (STG), RED (PROD)
- **Never commit "LOCAL"** as value in `environment_indicator.indicator.yml` for production! Always use "PROD" and red color.

## Documentation

**MANDATORY**: Document all work in the "Tasks and Problems" section below.

- Update **Tasks and Problems** section after every development session
- Document all significant changes and improvements
- Record common problems and their solutions
- **CRITICAL**: Use real system date (run `date` command first)
- Keep entries concise and actionable

**What to Document**:
- New modules created or modified
- Bug fixes and their solutions
- Configuration changes
- Performance optimizations
- Integration implementations
- Development workflow improvements
- Common problems encountered
- Security updates and patches

**When to Update**:
- After adding new features
- After fixing bugs
- After implementing improvements
- After encountering new problems
- After finding solutions to existing problems
- End of each development session

See **Tasks and Problems Log** section at the end of this document.

## Common Tasks

### Adding New Module

1. Create directory: `/web/modules/custom/[prefix]_[module_name]/`
2. Create `[prefix]_[module_name].info.yml` with module definition:
   ```yaml
   name: 'Module Name'
   type: module
   description: 'Module description'
   core_version_requirement: ^10 || ^11
   package: Custom
   ```
3. Create `[prefix]_[module_name].module` for hooks (if needed)
4. Add routing, permissions, services as needed
5. Follow module structure pattern above
6. Enable module: `ddev drush pm:enable [prefix]_[module_name]`
7. Clear caches: `ddev drush cr`

### Updating Drupal Core

1. Check module compatibility: `ddev composer outdated 'drupal/*'`
2. Backup database: `ddev export-db --file=backup-pre-update.sql.gz`
3. Update core:
   ```bash
   ddev composer update drupal/core-recommended --with-dependencies
   ddev composer update drupal/core-composer-scaffold --with-dependencies
   ```
4. Run database updates: `ddev drush updb`
5. Clear caches: `ddev drush cr`
6. Test thoroughly and check logs
7. Document in Tasks and Problems section

### Creating Database Migration

1. Add update hook to module `.install` file:
   ```php
   function [module_name]_update_10001() {
     // Update logic here
   }
   ```
2. Use `EntityDefinitionUpdateManager` for field changes
3. Check if field exists before updating
4. Update form/view displays if needed
5. Clear caches and log completion:
   ```php
   drupal_flush_all_caches();
   \Drupal::logger('module_name')->info('Migration completed.');
   ```
6. Test backward compatibility
7. Test on staging before production

### Adding Tests

1. Determine test type (unit, functional, integration, acceptance)
2. Create test file in appropriate directory:
   - Unit: `tests/src/Unit/`
   - Kernel: `tests/src/Kernel/`
   - Functional: `tests/src/Functional/`
   - Acceptance: `tests/acceptance/`
3. Follow naming convention: `*Test.php` for PHPUnit, `*Cest.php` for Codeception
4. Write test following best practices
5. Run tests to verify: `ddev exec vendor/bin/phpunit [test-file]`
6. Add to CI/CD pipeline

### Running Tests

1. PHPUnit tests:
   ```bash
   ddev exec vendor/bin/phpunit web/modules/custom/[module]/tests
   ```
2. Codeception tests:
   ```bash
   ddev exec vendor/bin/codecept run
   ```
3. Check results in `tests/_output/`

### Debugging Issues

1. Enable Xdebug: `ddev xdebug on`
2. Set breakpoints in IDE
3. Run code/request
4. Step through code
5. Disable when done: `ddev xdebug off`

See **Debugging** section for more detailed debugging commands.

### Managing Permissions

1. Grant permissions to role:
   ```php
   user_role_grant_permissions($role_id, ['permission1', 'permission2']);
   drupal_flush_all_caches();
   ```
2. Via UI: `/admin/people/permissions`
3. Check current permissions: `ddev drush role:perm:list [role_name]`

### Exporting Configuration

1. Make configuration changes in UI
2. Export: `ddev drush cex`
3. Review changes: `git diff config/`
4. Commit exported configuration
5. On other environments: `ddev drush cim`

## Troubleshooting

### JavaScript Errors with Aggregation

**Problem**: JavaScript works in development but breaks with aggregation enabled.

**Diagnosis**:
1. Check `.libraries.yml` for missing dependencies
2. Verify loading order in Network tab (browser DevTools)
3. Check browser console for errors
4. Enable aggregation: `/admin/config/development/performance`

**Solutions**:
```yaml
# Fix: Add proper dependencies in [module].libraries.yml
my_library:
  js:
    js/script.js: {}
  dependencies:
    - core/drupal
    - core/jquery
    - core/drupalSettings
    - core/once
```

```javascript
// Fix: Add safety checks in JavaScript
(function (Drupal, once) {
  'use strict';

  Drupal.behaviors.myBehavior = {
    attach: function (context, settings) {
      // Check if dependencies exist
      if (typeof once === 'undefined' || typeof settings.myModule === 'undefined') {
        return;
      }

      // Safe to use
      once('my-behavior', '.my-selector', context).forEach(function (element) {
        // Your code
      });
    }
  };
})(Drupal, once);
```

### Drush Issues

**Problem**: Drush commands fail or produce unexpected results.

**Common Causes & Solutions**:

1. **Drush version incompatibility**:
   ```bash
   # Check Drush version
   ddev drush --version

   # Update Drush
   ddev composer update drush/drush
   ```

2. **Memory exhaustion**:
   ```bash
   # Increase PHP memory limit temporarily
   ddev exec php -d memory_limit=512M vendor/bin/drush [command]
   ```

3. **Permission issues**:
   ```bash
   # Fix file permissions
   ddev exec chmod -R 755 web/sites/default/files
   ddev exec chmod 444 web/sites/default/settings.php
   ```

4. **Bootstrap errors**:
   ```bash
   # Clear caches
   ddev drush cr

   # Rebuild registry
   ddev drush php:eval "\$kernel = \Drupal::service('kernel'); \$kernel->invalidateContainer();"
   ```

### Cache Issues

**Problem**: Changes not appearing or stale content showing.

**Solutions**:
```bash
# Clear all caches
ddev drush cr

# Clear specific cache bin
ddev drush cache:clear render
ddev drush cache:clear dynamic_page_cache

# Clear Twig cache and rebuild
ddev drush twig:debug
rm -rf web/sites/default/files/php/twig/*

# Disable caches for development (development.services.yml)
# Then: ddev drush cr

# Nuclear option: Clear everything
ddev drush sql:query "TRUNCATE cache_bootstrap; TRUNCATE cache_config; TRUNCATE cache_container; TRUNCATE cache_data; TRUNCATE cache_default; TRUNCATE cache_discovery; TRUNCATE cache_dynamic_page_cache; TRUNCATE cache_entity; TRUNCATE cache_menu; TRUNCATE cache_render;"
ddev drush cr
```

### Database Issues

**Problem**: Database corruption, pending updates, or connection errors.

**Solutions**:
```bash
# Check database connection
ddev drush sql:cli
# Type: SELECT 1; (should return 1)

# Run pending updates
ddev drush updatedb
# Alias: ddev drush updb

# Rebuild entity schema
ddev drush entity:updates

# Repair tables
ddev mysql -e "REPAIR TABLE [table_name];"

# Check for crashed tables
ddev mysql -e "CHECK TABLE [table_name];"

# Reset to known good state
ddev db-restore  # If you have backup
```

### DDEV Container Issues

**Problem**: Containers won't start or behave unexpectedly.

**Solutions**:
```bash
# Restart DDEV
ddev restart

# Stop and start (full restart)
ddev stop
ddev start

# Remove and recreate containers
ddev delete -O
ddev start

# Check Docker resources
docker ps
docker stats

# View DDEV logs
ddev logs

# Check for port conflicts
ddev describe
```

### Module Installation Issues

**Problem**: Module won't install or enable.

**Solutions**:
```bash
# Check module dependencies
ddev composer why-not drupal/[module_name]

# Install with dependencies
ddev composer require drupal/[module_name]

# Enable module
ddev drush pm:enable [module_name]

# Check for conflicts
ddev drush pm:list | grep [module_name]

# If schema issues
ddev drush updatedb
ddev drush entity:updates
```

### Permission Denied Errors

**Problem**: Cannot write files or access directories.

**Solutions**:
```bash
# Fix files directory permissions
ddev exec chmod -R 775 web/sites/default/files
ddev exec chown -R $(id -u):$(id -g) web/sites/default/files

# Make settings.php writable temporarily
ddev exec chmod 644 web/sites/default/settings.php
# After changes:
ddev exec chmod 444 web/sites/default/settings.php

# Fix Twig cache permissions
ddev exec chmod -R 775 web/sites/default/files/php
```

### White Screen of Death (WSOD)

**Problem**: Blank white page with no error message.

**Solutions**:
```bash
# Enable error reporting temporarily
ddev drush config:set system.logging error_level verbose

# Check PHP error log
ddev logs

# Check watchdog
ddev drush watchdog:show --count=50

# Check for fatal errors
ddev exec tail -f /var/log/php/php-fpm.log

# Disable recently enabled modules
ddev drush pm:uninstall [module_name]
```

### Configuration Import Failures

**Problem**: `ddev drush cim` fails with errors.

**Solutions**:
```bash
# Check what would be imported
ddev drush config:status

# Import with override
ddev drush cim -y

# Import specific configuration
ddev drush config:import --partial --source=/path/to/config

# If UUID mismatch
ddev drush config:set system.site uuid [correct-uuid]

# Skip validation (use with caution)
ddev drush cim --skip-modules
```

### Memory Limit Issues

**Problem**: PHP memory exhausted errors.

**Solutions**:
```bash
# Check current limit
ddev exec php -i | grep memory_limit

# Increase in .ddev/php/php.ini
echo "memory_limit = 512M" >> .ddev/php/php.ini
ddev restart

# Or for single command
ddev exec php -d memory_limit=1G vendor/bin/drush [command]
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
