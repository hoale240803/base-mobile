# Mobile Base Project - Development Conventions

## 📋 Table of Contents

- [Project Structure](#project-structure)
- [Naming Conventions](#naming-conventions)
- [Code Style Guidelines](#code-style-guidelines)
- [Architecture Patterns](#architecture-patterns)
- [Testing Conventions](#testing-conventions)
- [Git Workflow](#git-workflow)
- [Documentation Standards](#documentation-standards)

## 🏗️ Project Structure

### Root Directory Structure

```
base-mobile/
├── docs/                           # Project documentation
├── core/                          # Core shared functionality
│   ├── domain/                    # Business logic and entities
│   │   ├── entities/             # Domain models
│   │   ├── usecases/             # Business use cases
│   │   └── repositories/         # Repository interfaces
│   ├── data/                     # Data layer implementation
│   │   ├── repositories/         # Repository implementations
│   │   ├── datasources/          # Data sources (local/remote)
│   │   └── models/               # Data transfer objects
│   └── presentation/             # Presentation layer
│       ├── viewmodels/           # ViewModels
│       ├── states/               # UI state definitions
│       └── events/               # UI events
├── platform/                     # Platform-specific implementations
│   ├── android/                  # Android-specific code
│   ├── ios/                      # iOS-specific code
│   └── shared/                   # Cross-platform shared code
├── features/                     # Feature modules
│   └── [feature-name]/
│       ├── domain/
│       ├── data/
│       └── presentation/
├── tools/                        # Development tools and scripts
├── tests/                        # Test utilities and shared test code
└── assets/                       # Shared assets (images, fonts, etc.)
```

### Feature Module Structure

```
feature-name/
├── domain/
│   ├── entities/
│   ├── usecases/
│   └── repositories/
├── data/
│   ├── repositories/
│   ├── datasources/
│   │   ├── local/
│   │   └── remote/
│   └── models/
└── presentation/
    ├── viewmodels/
    ├── screens/
    ├── components/
    └── states/
```

## 📝 Naming Conventions

### File Naming

#### General Rules

- Use PascalCase for class files: `UserRepository.kt`, `ProductViewModel.swift`
- Use kebab-case for directories: `user-management/`, `product-catalog/`
- Use camelCase for utility files: `networkUtils.ts`, `dateHelper.kt`
- Use SCREAMING_SNAKE_CASE for constants files: `API_CONSTANTS.kt`

#### Platform-Specific Extensions

- **Android Kotlin**: `.kt`
- **iOS Swift**: `.swift`
- **Cross-Platform TypeScript**: `.ts`
- **Dart (Flutter)**: `.dart`
- **Test files**: `.test.kt`, `.test.swift`, `.test.ts`

### Class and Interface Naming

#### Domain Layer

- **Entities**: `User`, `Product`, `Order`, `Account`
- **Use Cases**: `GetUserUseCase`, `CreateOrderUseCase`, `UpdateProfileUseCase`
- **Repository Interfaces**: `UserRepository`, `ProductRepository`

#### Data Layer

- **Repository Implementations**: `UserRepositoryImpl`, `ProductRepositoryImpl`
- **Data Sources**: `UserRemoteDataSource`, `UserLocalDataSource`
- **Data Models/DTOs**: `UserDto`, `ProductResponse`, `OrderRequest`

#### Presentation Layer

- **ViewModels**: `UserViewModel`, `ProductListViewModel`, `OrderDetailViewModel`
- **UI States**: `UserState`, `ProductListState`, `LoadingState`
- **UI Events**: `UserEvent`, `NavigationEvent`, `ErrorEvent`

#### Platform Layer

- **Platform Implementations**: `AndroidUserRepository`, `IOSLocationService`
- **Adapters**: `NetworkAdapter`, `StorageAdapter`
- **Bridges**: `PlatformBridge`, `NativeModuleBridge`

### Variable and Function Naming

#### Variables

- Use camelCase: `userName`, `isLoggedIn`, `userProfileData`
- Boolean variables: start with `is`, `has`, `can`, `should`
  - `isVisible`, `hasPermission`, `canEdit`, `shouldUpdate`

#### Functions

- Use camelCase: `getUserData()`, `validateInput()`, `navigateToHome()`
- Use verbs for actions: `get`, `set`, `create`, `update`, `delete`, `validate`
- Use descriptive names: `fetchUserProfileData()` vs `getUserData()`

#### Constants

- Use SCREAMING_SNAKE_CASE: `MAX_RETRY_COUNT`, `DEFAULT_TIMEOUT`, `API_BASE_URL`

## 🎨 Code Style Guidelines

### Kotlin (Android)

```kotlin
// Class declaration
class UserRepositoryImpl(
    private val remoteDataSource: UserRemoteDataSource,
    private val localDataSource: UserLocalDataSource
) : UserRepository {

    // Function with proper spacing and naming
    override suspend fun getUserById(userId: String): Result<User> {
        return try {
            val userDto = remoteDataSource.fetchUser(userId)
            val user = userDto.toDomain()
            localDataSource.saveUser(user)
            Result.success(user)
        } catch (exception: Exception) {
            Result.failure(exception)
        }
    }
}
```

### Swift (iOS)

```swift
// Class declaration with proper access control
final class UserRepositoryImpl: UserRepository {
    private let remoteDataSource: UserRemoteDataSource
    private let localDataSource: UserLocalDataSource

    init(
        remoteDataSource: UserRemoteDataSource,
        localDataSource: UserLocalDataSource
    ) {
        self.remoteDataSource = remoteDataSource
        self.localDataSource = localDataSource
    }

    // Function with proper async/await and error handling
    func getUserById(_ userId: String) async throws -> User {
        do {
            let userDto = try await remoteDataSource.fetchUser(userId)
            let user = userDto.toDomain()
            try await localDataSource.saveUser(user)
            return user
        } catch {
            throw RepositoryError.fetchFailed(error)
        }
    }
}
```

### TypeScript (Cross-Platform)

```typescript
// Interface definition
interface UserRepository {
  getUserById(userId: string): Promise<Result<User, Error>>;
}

// Implementation with proper typing
export class UserRepositoryImpl implements UserRepository {
  constructor(
    private readonly remoteDataSource: UserRemoteDataSource,
    private readonly localDataSource: UserLocalDataSource
  ) {}

  async getUserById(userId: string): Promise<Result<User, Error>> {
    try {
      const userDto = await this.remoteDataSource.fetchUser(userId);
      const user = userDto.toDomain();
      await this.localDataSource.saveUser(user);
      return Result.success(user);
    } catch (error) {
      return Result.failure(error as Error);
    }
  }
}
```

## 🏛️ Architecture Patterns

### MVVM Implementation

```
View ←→ ViewModel ←→ UseCase ←→ Repository ←→ DataSource
```

#### Responsibilities

- **View**: UI rendering, user input handling, navigation
- **ViewModel**: UI state management, business logic coordination
- **UseCase**: Single business operation, validation, orchestration
- **Repository**: Data abstraction, caching strategy
- **DataSource**: Raw data access (API, Database, File System)

### Clean Architecture Layers

```
┌─────────────────────┐
│   Presentation      │ ← ViewModels, UI Components
├─────────────────────┤
│   Domain           │ ← Entities, Use Cases, Repository Interfaces
├─────────────────────┤
│   Data             │ ← Repository Implementations, Data Sources
├─────────────────────┤
│   Infrastructure   │ ← External APIs, Database, File System
└─────────────────────┘
```

### Dependency Injection Structure

```kotlin
// Module organization
interface CoreModule {
    // Core dependencies
}

interface DataModule {
    // Data layer dependencies
}

interface DomainModule {
    // Domain layer dependencies
}

interface PresentationModule {
    // Presentation layer dependencies
}
```

## 🧪 Testing Conventions

### Test File Organization

```
tests/
├── unit/
│   ├── domain/
│   ├── data/
│   └── presentation/
├── integration/
│   ├── repositories/
│   └── usecases/
├── ui/
│   ├── screens/
│   └── components/
└── e2e/
    └── flows/
```

### Test Naming Convention

- **Pattern**: `should_[expected_result]_when_[condition]`
- **Examples**:
  - `should_return_user_when_valid_id_provided()`
  - `should_throw_exception_when_network_unavailable()`
  - `should_update_ui_state_when_data_changes()`

### Test Structure (AAA Pattern)

```kotlin
@Test
fun should_return_user_when_valid_id_provided() {
    // Arrange
    val userId = "123"
    val expectedUser = createTestUser(userId)
    whenever(dataSource.getUser(userId)).thenReturn(expectedUser)

    // Act
    val result = repository.getUserById(userId)

    // Assert
    assertThat(result).isSuccess()
    assertThat(result.getOrNull()).isEqualTo(expectedUser)
}
```

## 🔄 Git Workflow

### Branch Naming Convention

```
main                    # Production-ready code
develop                 # Integration branch
feature/[ticket-id]-[description]  # New features
bugfix/[ticket-id]-[description]   # Bug fixes
hotfix/[description]               # Critical production fixes
release/[version]                  # Release preparation
```

### Commit Message Convention

Follow [Conventional Commits](https://www.conventionalcommits.org/):

```
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

#### Types

- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation changes
- `style`: Code style changes (formatting)
- `refactor`: Code refactoring
- `perf`: Performance improvements
- `test`: Adding or updating tests
- `chore`: Maintenance tasks

#### Examples

```
feat(auth): implement biometric authentication
fix(navigation): resolve memory leak in tab navigation
docs(readme): update installation instructions
refactor(network): improve error handling in HTTP client
test(user): add integration tests for user repository
```

## 📚 Documentation Standards

### Code Documentation

#### Class Documentation

```kotlin
/**
 * Repository implementation for managing user data.
 *
 * This class handles user data operations including fetching from remote API,
 * caching locally, and providing offline support.
 *
 * @param remoteDataSource Remote API data source for user operations
 * @param localDataSource Local storage data source for caching
 *
 * @since 1.0.0
 * @author Development Team
 */
class UserRepositoryImpl(
    private val remoteDataSource: UserRemoteDataSource,
    private val localDataSource: UserLocalDataSource
) : UserRepository
```

#### Function Documentation

```kotlin
/**
 * Retrieves user information by ID with caching support.
 *
 * This method first attempts to fetch user data from the remote API.
 * On success, the data is cached locally for offline access.
 * If the remote fetch fails, it attempts to retrieve cached data.
 *
 * @param userId Unique identifier for the user
 * @return Result containing User object on success or error on failure
 *
 * @throws NetworkException when network is unavailable
 * @throws ValidationException when userId is invalid
 *
 * @since 1.0.0
 */
suspend fun getUserById(userId: String): Result<User>
```

### README Structure for Features

```markdown
# Feature Name

## Overview

Brief description of the feature and its purpose.

## Architecture

Explanation of the architectural approach used.

## Usage

Code examples showing how to use the feature.

## Configuration

Any configuration options available.

## Testing

How to run tests for this feature.

## Dependencies

List of external dependencies.
```

### API Documentation

- Use OpenAPI/Swagger for REST APIs
- Include request/response examples
- Document error codes and messages
- Provide authentication requirements

## 🔒 Security Conventions

### Data Protection

- Always encrypt sensitive data at rest
- Use HTTPS for all network communications
- Implement certificate pinning for critical APIs
- Validate all user inputs

### Authentication & Authorization

- Use industry-standard authentication protocols (OAuth 2.0, OpenID Connect)
- Implement proper session management
- Use secure token storage mechanisms
- Implement proper logout functionality

### Code Security

- Never commit sensitive information (API keys, passwords)
- Use environment variables for configuration
- Implement proper error handling without exposing sensitive information
- Regular security audits and dependency updates

## 📊 Performance Conventions

### Memory Management

- Use weak references for delegates and callbacks
- Implement proper cleanup in lifecycle methods
- Monitor memory usage and detect leaks
- Use object pools for frequently created objects

### Network Optimization

- Implement request/response caching
- Use compression for large payloads
- Implement request queuing and prioritization
- Handle offline scenarios gracefully

### Battery Optimization

- Minimize background processing
- Use efficient polling strategies
- Implement proper wake lock management
- Optimize for different power states

## 🎯 Quality Standards

### Code Quality Metrics

- **Test Coverage**: Minimum 80% for business logic
- **Cyclomatic Complexity**: Maximum 10 per method
- **Class Size**: Maximum 500 lines per class
- **Method Size**: Maximum 50 lines per method

### Performance Benchmarks

- **App Launch Time**: Maximum 3 seconds cold start
- **Screen Transition**: Maximum 300ms
- **API Response**: Maximum 2 seconds for critical operations
- **Memory Usage**: Monitor and optimize for target devices

### Accessibility Standards

- Support screen readers
- Provide proper content descriptions
- Ensure sufficient color contrast
- Support keyboard navigation where applicable
