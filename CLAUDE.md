# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

### Dependencies
Install dependencies using Composer:
```bash
composer install
```

### Testing
Run all tests:
```bash
vendor/bin/phpunit
```

Run a specific test file:
```bash
vendor/bin/phpunit test/InflectorTest.php
```

Run tests with verbose output (shows skipped tests):
```bash
vendor/bin/phpunit --verbose
```

Run tests with specific database adapter:
```bash
vendor/bin/phpunit --adapter mysql
vendor/bin/phpunit --adapter pgsql
vendor/bin/phpunit --adapter sqlite
```

Enable slow tests:
```bash
vendor/bin/phpunit --slow-tests
```

### Database Setup for Tests
Tests support multiple database adapters. Default connection strings can be overridden with environment variables:
- `PHPAR_MYSQL` - MySQL connection (default: `mysql://test:test@127.0.0.1/test`)
- `PHPAR_PGSQL` - PostgreSQL connection (default: `pgsql://test:test@127.0.0.1/test`)
- `PHPAR_SQLITE` - SQLite connection (default: `sqlite://test.db`)
- `PHPAR_OCI` - Oracle connection (default: `oci://test:test@127.0.0.1/dev`)

## Architecture Overview

### Core Structure
PHP ActiveRecord is an ORM library implementing the Active Record pattern, inspired by Ruby on Rails' ActiveRecord.

**Main entry point:** `ActiveRecord.php` - loads all required classes and sets up autoloading

**Core components:**
- `lib/Model.php` - Base model class that all ActiveRecord models extend
- `lib/Config.php` - Configuration management (database connections, model directories)
- `lib/ConnectionManager.php` - Manages database connections
- `lib/Table.php` - Represents database table metadata and schema
- `lib/SQLBuilder.php` - Builds SQL queries
- `lib/Connection.php` - Database connection abstraction
- `lib/Relationship.php` - Handles model relationships (has_many, belongs_to, etc.)
- `lib/Validations.php` - Model validation framework
- `lib/CallBack.php` - Model lifecycle callbacks (before_save, after_create, etc.)

**Database adapters:** `lib/adapters/` contains database-specific implementations:
- `MysqlAdapter.php`
- `PgsqlAdapter.php` 
- `SqliteAdapter.php`
- `OciAdapter.php`

### Configuration Pattern
ActiveRecord uses a singleton Config class that can be initialized with a closure:

```php
ActiveRecord\Config::initialize(function($cfg) {
    $cfg->set_model_directory('/path/to/models');
    $cfg->set_connections([
        'development' => 'mysql://user:pass@host/db',
        'production' => 'mysql://user:pass@host/db'
    ]);
    $cfg->set_default_connection('development');
});
```

### Model Definition Pattern
Models extend `ActiveRecord\Model` and use static properties to define relationships and validations:

```php
class Person extends ActiveRecord\Model {
    static $has_many = [
        ['orders']
    ];
    
    static $validates_length_of = [
        ['first_name', 'within' => [1,50]]
    ];
}
```

### Testing Architecture
- Test bootstrap: `test/helpers/config.php`
- Test models: `test/models/` contains sample models for testing
- Database fixtures: `test/fixtures/` contains CSV data files
- SQL schemas: `test/sql/` contains database-specific schema files
- Custom test base class: `test/helpers/SnakeCase_PHPUnit_Framework_TestCase.php`

The test suite supports multiple database adapters and can be configured via command line arguments or environment variables.