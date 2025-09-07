# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Part-DB is an open-source electronic component inventory management system built with Symfony 7 and PHP 8.2+. It features a hierarchical component organization, barcode/label generation, user management with fine-grained permissions, project/BOM management, and API integration with component suppliers.

## Development Commands

### Backend (PHP/Symfony)
- `composer install` - Install PHP dependencies
- `composer install -o --no-dev` - Production install (optimized)
- `php bin/console doctrine:migrations:migrate` - Run database migrations
- `php bin/console cache:clear` - Clear Symfony cache
- `php bin/console cache:warmup` - Warm up cache for production
- `vendor/bin/phpunit` - Run PHPUnit tests
- `vendor/bin/phpstan analyse src --level 5 --memory-limit 1G` - Static analysis
- `vendor/bin/ecs check` - Code style checks
- `vendor/bin/ecs check --fix` - Fix code style issues

### Frontend (Node.js/Webpack)
- `yarn install` - Install frontend dependencies
- `yarn dev` - Build assets for development
- `yarn watch` - Build assets and watch for changes
- `yarn build` - Build optimized assets for production
- `yarn dev-server` - Start webpack dev server

### Database & Testing
- `php bin/console doctrine:fixtures:load` - Load test fixtures
- `php bin/console doctrine:schema:validate` - Validate database schema
- `APP_ENV=test php bin/console doctrine:database:create` - Create test database
- `APP_ENV=test php bin/console doctrine:migrations:migrate -n` - Run test migrations

## Architecture Overview

### Core Domain Model
The application follows Domain-Driven Design principles with these key concepts:

**Structural Entities (Hierarchical Tree Structure):**
- `Category` - Organizes parts into hierarchical categories
- `StorageLocation` - Physical storage locations in hierarchical structure
- `Footprint` - Component footprints/packages
- `Manufacturer` - Component manufacturers
- `Supplier` - Component suppliers
- `MeasurementUnit` - Units for measurements
- `Currency` - Supported currencies

**Core Entities:**
- `Part` - Central entity representing electronic components with lots, parameters, attachments
- `PartLot` - Inventory tracking for specific quantities of parts
- `PartParameter` - Key-value specifications for parts
- `Attachment` - Files associated with entities (datasheets, images, etc.)

**Project System:**
- `Project` - Bill of Materials management
- `ProjectBOMEntry` - Individual entries in project BOMs

**User System:**
- `User` - System users with group-based permissions
- `Group` - User groups with hierarchical permissions
- `ApiToken` - API authentication tokens

### Key Architectural Patterns

**Entity Hierarchy:**
- `AbstractDBElement` - Base for all entities with ID, timestamps
- `AbstractNamedDBElement` - Adds name/description to entities  
- `AbstractStructuralDBElement` - Adds hierarchical tree structure
- `AbstractPartsContainingDBElement` - For entities that contain parts

**Trait System:**
- `TimestampTrait` - Created/modified timestamps
- `ParametersTrait` - Adds parameter support to entities
- `MasterAttachmentTrait` - Adds attachment support

**Service Layer Organization:**
- `Services/` - Business logic services
- `Repository/` - Data access layer
- `Controller/` - Web controllers and API endpoints
- `DataTables/` - Server-side DataTables integration
- `Form/` - Symfony form types
- `Security/` - Authentication, authorization, and voters

### Frontend Architecture
- **Stimulus Controllers** - Modular frontend behavior in `assets/controllers/`
- **Twig Templates** - Server-side rendering with component macros
- **Webpack Encore** - Asset compilation with theme support
- **Bootstrap 5** - UI framework with Bootswatch theme variants

## Development Guidelines

### Code Organization
- Follow PSR-12 coding standards enforced by ECS
- Use strict types (`declare(strict_types=1);`) in all PHP files
- Organize services by domain in `src/Services/`
- Place entity-specific repositories in matching namespace structure
- Group related forms in `src/Form/` by feature area

### Entity Development
- Extend appropriate base classes (`AbstractDBElement`, `AbstractStructuralDBElement`)
- Use Doctrine attributes for ORM mapping
- Implement `__toString()` methods for entities used in forms
- Add API Platform attributes for API exposure when needed

### Database Considerations
- Supports MySQL/MariaDB, PostgreSQL, and SQLite
- Use migrations for all schema changes
- Test migrations on all supported databases
- Consider performance implications of hierarchical queries

### Security Implementation
- Use Symfony voters for authorization (`src/Security/Voter/`)
- Implement fine-grained permissions via `PermissionManager`
- All user actions should be logged via `EventLogger`
- CSRF protection is enabled globally

### Frontend Guidelines
- Use Stimulus controllers for interactive behavior
- Follow existing naming patterns for CSS classes
- Leverage Bootstrap 5 utility classes
- Support all available themes (defined in `webpack.config.js`)

### Testing Strategy
- Unit tests for business logic services
- Integration tests for repositories and database interactions
- Functional tests for controllers and web interfaces
- Use fixtures for consistent test data

### API Development
- API Platform provides REST and GraphQL APIs
- Use custom filters in `src/ApiPlatform/Filter/`
- Implement proper serialization groups
- Document API changes in entity annotations

## Environment Configuration

### Required Environment Variables
- `APP_ENV` - Environment (dev/prod/test)
- `DATABASE_URL` - Database connection string
- `APP_SECRET` - Symfony application secret

### Optional Configuration
- `INITIAL_ADMIN_PW` - Set admin password on first install
- `TRUSTED_PROXIES` - For reverse proxy setups
- Various info provider API keys for component data integration

## File Structure Notes

### Key Directories
- `src/Entity/` - Doctrine entities organized by domain
- `src/Services/` - Business logic services
- `src/Controller/` - Web controllers
- `templates/` - Twig templates
- `assets/` - Frontend assets (JS, CSS, images)
- `migrations/` - Database migrations
- `config/` - Symfony configuration
- `tests/` - PHPUnit tests

### Important Files
- `composer.json` - PHP dependencies and scripts
- `package.json` - Node.js dependencies and build scripts
- `webpack.config.js` - Asset compilation configuration
- `phpunit.xml.dist` - PHPUnit configuration
- `phpstan.dist.neon` - Static analysis configuration
- `ecs.php` - Coding standard configuration

## Common Patterns

### Creating New Structural Entities
1. Extend `AbstractStructuralDBElement`
2. Add specific repository extending `StructuralDBElementRepository`
3. Create admin controller extending `BaseAdminController`
4. Add admin form type
5. Create attachment type if needed
6. Add permissions configuration
7. Update tree navigation if applicable

### Adding New Services
1. Create service class in appropriate `Services/` subdirectory
2. Use dependency injection with constructor injection
3. Add interface if service will have multiple implementations
4. Register service tags if needed (e.g., for providers)
5. Write unit tests

### Working with Permissions
- Check permissions using `PermissionManager` service
- Use voters for entity-level authorization
- Add new permissions to `config/permissions.yaml`
- Test permission inheritance in hierarchical structures

This codebase emphasizes maintainability, security, and extensibility with a clean separation between domain logic, data access, and presentation layers.