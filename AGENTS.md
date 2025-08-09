

# **Development & Contribution Protocol for geminiboard**

### **Preamble**

This document serves as the single source of truth and authoritative development protocol for the geminiboard project. It is designed to be parsed and executed by the AI development agent, jules.google. All contributions, whether from AI or human developers, must strictly adhere to these guidelines.

Analysis of the target repository at https://github.com/ClemensSchartmueller/geminiboard was not possible due to inaccessibility.1 Therefore, this document is

**prescriptive**, establishing the foundational rules and best practices for all new and future development on this project. It defines the "ideal state" to which the codebase must conform. Non-compliance with these directives is grounds for rejection of any contribution.

## **Section 1: Foundational Principles & Build Configuration**

This section establishes the non-negotiable bedrock of the project's technical stack and build system. Adherence is mandatory to ensure consistency, maintainability, and a modern development environment.

### **1.1. Core Technology Stack Mandate**

The technological foundation of geminiboard is defined to align with modern, official Android development standards, prioritizing developer productivity, code safety, and long-term maintainability.

* **Kotlin as the Sole Language:** All new source code for the geminiboard project must be written exclusively in Kotlin. Google has designated Kotlin as the official and dominant language for Android development, a decision supported by its concise syntax, null-safety features, and extensive support for functional programming concepts, which collectively reduce boilerplate and minimize common runtime errors.3 Java is considered a legacy language within the context of this project and its use in new code is prohibited.  
* **Jetpack Suite as Foundation:** The project must be built entirely upon the Jetpack suite of libraries. This strategic decision ensures the use of a cohesive, architecture-aware, and Google-supported set of components. Mandatory Jetpack libraries include, but are not limited to: Jetpack Compose for the UI layer, ViewModel for presentation layer logic, Room for local data persistence, and Navigation for managing screen flows.3 The adoption of the Jetpack suite standardizes the approach to common development challenges and guarantees compatibility with modern Android OS features.

### **1.2. Gradle Configuration Protocol: Kotlin DSL and Version Catalogs**

The project's build system is a critical component of its architecture and must be configured for maximum type safety, maintainability, and efficiency.

* **Kotlin DSL (.gradle.kts):** All Gradle build scripts, including settings.gradle.kts and module-level build.gradle.kts files, must use the Kotlin Domain-Specific Language (DSL). As of Android Studio Giraffe, Kotlin DSL is the default for new projects, offering a superior development experience compared to the traditional Groovy DSL.6 The benefits include compile-time type checking, enhanced IDE support such as autocompletion, content assistance, and navigation to source definitions, which significantly reduce configuration errors.6 All build files must use the  
  .gradle.kts extension.  
* **Mandatory Use of Version Catalogs (libs.versions.toml):** Dependency management for the entire project must be centralized within a single Version Catalog file located at gradle/libs.versions.toml. This approach provides a scalable and maintainable single source of truth for all project dependencies, including libraries and plugins.8 It enables type-safe access to dependencies in  
  .kts build scripts, simplifies version updates across all modules, and allows for easy sharing of dependency configurations. The use of older dependency management techniques, such as defining versions in ext blocks within Groovy scripts or using the buildSrc directory, is strictly forbidden. While buildSrc offers some organizational benefits, it introduces a significant performance drawback: any modification within the buildSrc directory invalidates the entire Gradle build cache, forcing a complete project rebuild and leading to longer development cycles.7 Version Catalogs provide the same benefits of centralization without this performance penalty, making them the mandated choice.8

An app/build.gradle.kts file must conform to the following structure, utilizing aliases from the version catalog:

Kotlin

plugins {  
    alias(libs.plugins.android.application)  
    alias(libs.plugins.kotlin.android)  
    alias(libs.plugins.hilt)  
    // Any other required plugins must be referenced via the version catalog.  
}

android {  
    namespace \= "com.clemensschartmueller.geminiboard"  
    compileSdk \= 35

    defaultConfig {  
        applicationId \= "com.clemensschartmueller.geminiboard"  
        minSdk \= 24  
        targetSdk \= 35  
        versionCode \= 1  
        versionName \= "1.0"

        testInstrumentationRunner \= "androidx.test.runner.AndroidJUnitRunner"  
        vectorDrawables {  
            useSupportLibrary \= true  
        }  
    }

    buildTypes {  
        release {  
            isMinifyEnabled \= true  
            isShrinkResources \= true  
            proguardFiles(  
                getDefaultProguardFile("proguard-android-optimize.txt"),  
                "proguard-rules.pro"  
            )  
        }  
    }  
    compileOptions {  
        sourceCompatibility \= JavaVersion.VERSION\_17  
        targetCompatibility \= JavaVersion.VERSION\_17  
    }  
    kotlinOptions {  
        jvmTarget \= "17"  
    }  
    buildFeatures {  
        compose \= true  
    }  
    composeOptions {  
        kotlinCompilerExtensionVersion \= libs.versions.composeCompiler.get()  
    }  
    packaging {  
        resources {  
            excludes \+= "/META-INF/{AL2.0,LGPL2.1}"  
        }  
    }  
}

dependencies {  
    // Dependencies are exclusively sourced from the version catalog.  
    implementation(libs.androidx.core.ktx)  
    implementation(libs.androidx.lifecycle.runtime.ktx)  
    implementation(libs.androidx.activity.compose)  
    implementation(platform(libs.androidx.compose.bom))  
    implementation(libs.androidx.ui)  
    implementation(libs.androidx.ui.graphics)  
    implementation(libs.androidx.ui.tooling.preview)  
    implementation(libs.androidx.material3)

    // Hilt for Dependency Injection  
    implementation(libs.hilt.android)  
    kapt(libs.hilt.compiler)

    // Testing Dependencies  
    testImplementation(libs.junit)  
    androidTestImplementation(libs.androidx.junit)  
    androidTestImplementation(libs.androidx.espresso.core)  
    androidTestImplementation(platform(libs.androidx.compose.bom))  
    androidTestImplementation(libs.androidx.ui.test.junit4)  
    debugImplementation(libs.androidx.ui.tooling)  
    debugImplementation(libs.androidx.ui.test.manifest)  
}

The evolution of Android's build system reflects a deliberate progression towards greater architectural rigor. Early Groovy scripts were dynamic and error-prone. The buildSrc module introduced better organization but at the cost of build performance. The current standard of Kotlin DSL with Version Catalogs represents a mature state where the build system itself becomes an active participant in maintaining code quality. By enforcing type-safe, centralized dependency management, the build system acts as the first line of defense against architectural decay. For an AI agent, this is a critical control mechanism; it transforms the task of adding a dependency from a string-based operation to a type-safe reference (e.g., libs.retrofit), where an invalid reference results in a compile-time failure, preventing the introduction of unapproved or version-conflicting libraries.

The following table provides the non-negotiable baseline for the project's foundational dependencies. This manifest must be reflected in the initial libs.versions.toml file.

**Table 1: Prescribed Core Dependencies and Versions**

| Library/Plugin ID | Version Catalog Alias (in \[versions\]) | Prescribed Version | Source |
| :---- | :---- | :---- | :---- |
| com.android.application | androidGradlePlugin | 8.7.2 | 9 |
| org.jetbrains.kotlin.android | kotlin | 1.9.23 | 10 |
| com.google.dagger.hilt.android | hilt | 2.56.2 | 11 |
| androidx.compose.compiler | composeCompiler | 1.5.11 | N/A |
| androidx.core:core-ktx | androidxCore | 1.13.1 | N/A |
| org.jetbrains.kotlinx.coroutines | coroutines | 1.8.0 | N/A |

### **1.3. API Key and Secrets Management Protocol**

The handling of sensitive information is critical for application security. All API keys, tokens, and other secrets must be managed securely and must never be committed to version control.

The project must use the([https://github.com/google/secrets-gradle-plugin](https://github.com/google/secrets-gradle-plugin)). This plugin provides a standardized and secure way to manage secrets.9 All sensitive keys must be stored in the

local.properties file at the project root. This file is included in the project's .gitignore by default and will not be checked into the repository. The plugin will then make these keys available as build configuration fields, which can be accessed securely within the application code without exposing them in the versioned source.

## **Section 2: Application Architecture Blueprint**

This section defines the mandatory architectural pattern for geminiboard. This blueprint is designed to enforce a strict separation of concerns, enhance testability, and ensure the application is scalable and maintainable over time. Deviation from this blueprint is not permitted.

### **2.1. Modularization Strategy**

The geminiboard project must be structured into multiple, logically distinct Gradle modules. A monolithic application structure, where all code resides in a single :app module, is strictly forbidden as it hinders scalability, slows down build times, and complicates code ownership and maintenance.3

The project must be divided into the following module types:

* **Application Module (:app):** This module serves as the entry point of the application. Its responsibilities are limited to high-level application setup, including the Application class annotated with @HiltAndroidApp, the primary navigation graph, and any other essential Android-specific bootstrapping code. The :app module will depend on all necessary feature modules to assemble the final application.  
* **Core Module (:core):** This module contains shared, domain-agnostic code that is utilized across multiple features. This includes base classes (e.g., BaseViewModel), common utility functions, Kotlin extension functions, and any models or constants that are not specific to a single feature.  
* **Data Module (:data):** This module is solely responsible for data retrieval, storage, and management. It will contain all Repository implementations, definitions for network services (e.g., Retrofit interfaces), local database schemas and DAOs (e.g., Room), and data source models (DTOs).  
* **Domain Module (:domain):** This optional but highly recommended module is for applications with complex business logic. It contains use cases (also known as interactors) and pure Kotlin domain models. This layer must be entirely independent of the Android framework; it should not contain any Android-specific imports or dependencies.  
* **Feature Modules (:feature\_\*):** Each distinct feature of the application must reside in its own dedicated module. For example, a feature for displaying the main board would be in :feature\_board, while settings would be in :feature\_settings. This enforces a strong separation between features, allowing for parallel development and the possibility of dynamic feature delivery.

### **2.2. The Clean Architecture Mandate**

The project must strictly adhere to the principles of Clean Architecture to ensure a robust and decoupled system.3 This architectural style enforces a clear separation of concerns by organizing the codebase into distinct layers with a strict dependency rule.

The layers of the application are defined as follows:

1. **UI Layer (Presentation):** This layer is responsible for displaying data on the screen and capturing user input. It is composed of Jetpack Compose UI elements, ViewModels that manage UI state, and UI-specific state holder classes. This layer resides within the feature modules.  
2. **Domain Layer:** This layer contains the core business logic of the application, encapsulated in use cases or interactors. It also defines the core business entities (domain models). The domain layer is pure Kotlin and must remain independent of any implementation details of the other layers.  
3. **Data Layer:** This layer is responsible for implementing the data interfaces (Repositories) defined by the domain layer. It orchestrates data from various sources, such as network APIs and local databases, and handles the mapping between data transfer objects (DTOs) and domain models.

The **Dependency Rule** is the cornerstone of Clean Architecture and must be enforced at all times: dependencies must only point inwards. The UI layer may depend on the Domain layer, and the Domain layer may depend on the Data layer's interfaces, but not its implementation. The Data layer must not have any knowledge of the UI layer, and the Domain layer must remain pure, with no dependencies on the UI or the specific implementation details of the Data layer.

### **2.3. MVVM for the Presentation Layer**

The Model-View-ViewModel (MVVM) architectural pattern is mandatory for the presentation layer within each feature module.3 This pattern effectively separates the UI from the business logic, enhancing testability and maintainability.

The responsibilities of each component are strictly defined:

* **View (Composable Function):** The View's sole responsibility is to render the UI based on the state provided by the ViewModel. It observes this state and forwards all user interactions (events) to the ViewModel for processing. The View should be as "dumb" and stateless as possible, containing no business logic.  
* **ViewModel (androidx.lifecycle.ViewModel):** The ViewModel acts as the intermediary between the Model (data layer) and the View. It owns and manages all UI-related state. It executes business logic, makes calls to repositories or use cases to fetch or update data, and exposes the resulting state to the UI. A ViewModel must never hold a direct reference to any Android framework classes like Context or View to prevent memory leaks and ensure testability.  
* **Model (UI State):** The Model in this context refers to the UI state. This is typically represented by a single, immutable Kotlin data class that encapsulates all the information required to render a screen at any given moment. This includes data lists, loading indicators, error messages, and user input states.

### **2.4. Package Structure & Naming Conventions**

To ensure code is organized logically and is easy to navigate, all modules must follow a standardized package structure. Code must be organized by feature, not by layer (e.g., a single package containing all ViewModels from different features is forbidden).

The following package structure must be used within each feature module (e.g., :feature\_board):

com.clemensschartmueller.geminiboard.board  
├── di/                // Hilt modules specific to this feature.  
│   └── BoardModule.kt  
├── ui/                // All UI-related classes for the board feature.  
│   ├── BoardScreen.kt   // The main composable entry point for the screen.  
│   ├── BoardViewModel.kt// The ViewModel responsible for the BoardScreen.  
│   ├── BoardState.kt    // The data class defining the UI state for the BoardScreen.  
│   └── components/      // Small, reusable composables specific to the board feature.  
│       └── BoardItem.kt  
└── data/              // (If feature-specific data logic exists)  
    └── BoardRepositoryImpl.kt

This feature-centric organization keeps all related code cohesive, making it easier for developers to understand the scope of a feature and locate relevant files.

## **Section 3: UI Layer Protocol: Jetpack Compose**

This section details the specific rules and mandatory practices for building the user interface with Jetpack Compose. The primary goals are to create a declarative, performant, and maintainable UI layer by strictly adhering to modern Compose principles.

### **3.1. Composable Design Principles**

All composable functions must be designed with reusability, testability, and performance in mind.

* **Stateless by Default:** Composable functions must be designed to be stateless whenever possible. A stateless composable does not manage its own state but instead receives state as parameters from its parent and exposes events via lambda callbacks (e.g., onClick: () \-\> Unit).12 This makes composables highly reusable and easier to test in isolation.  
* **Mandatory State Hoisting:** The practice of state hoisting is required. State that needs to be managed and mutated must be "hoisted" (lifted) up the composable tree to the nearest common ancestor, which is typically the screen-level composable that interacts with the ViewModel.12 A stateless composable that accepts hoisted state must follow the standard pattern of accepting a  
  value: T parameter for display and an onValueChange: (T) \-\> Unit lambda to notify the state owner of a requested change.12  
* **Single Responsibility:** Each composable function must have a single, well-defined purpose. Complex screens must be decomposed into smaller, more manageable, and reusable component functions. This improves readability and simplifies maintenance.  
* **Previews are Mandatory:** Every significant, non-trivial composable function must be accompanied by one or more @Preview functions. These previews are essential for rapid UI development, visual regression testing, and documenting the different states of a component.5

### **3.2. Unidirectional Data Flow (UDF) and State Management**

A strict Unidirectional Data Flow (UDF) architecture is mandatory for managing state in the UI layer. This ensures that the flow of data is predictable, easy to debug, and less prone to errors.

* **ViewModel as Single Source of Truth:** The ViewModel is the one and only source of truth for the UI's state.12 Composables must never define or manage their own business logic or state. State flows down from the ViewModel to the UI.  
* **State Exposure with StateFlow:** ViewModels must expose their UI state using kotlinx.coroutines.flow.StateFlow. A private MutableStateFlow must be used internally within the ViewModel to update the state, while an immutable, public StateFlow is exposed to the UI for observation.14 The use of  
  LiveData is discouraged for new features in favor of the more powerful and flexible Kotlin Flow API.  
* **Atomic UI State Class:** The entire state for a given screen must be encapsulated within a single, immutable Kotlin data class (e.g., BoardUiState). This practice ensures that state updates are atomic and consistent, preventing transient or invalid UI states.14  
* **Lifecycle-Aware State Collection:** The UI must collect the StateFlow from the ViewModel in a lifecycle-aware manner. This must be achieved by using the collectAsStateWithLifecycle() extension function from the androidx.lifecycle:lifecycle-runtime-compose library.12 This API automatically starts collecting the flow when the UI is visible and stops when it goes into the background, preventing unnecessary resource consumption and potential crashes.  
* **Event Handling:** User interactions are treated as events that flow up from the composable to the ViewModel via lambda function calls. The ViewModel processes these events, updates its internal state, and the new state is then automatically reflected back down to the UI. This strict separation of state flow (down) and event flow (up) is the core of the UDF pattern and must be followed without exception.13

### **3.3. Performance Optimization Directives**

Writing performant Compose code is not an optional refinement; it is a core requirement. The following directives must be followed to prevent common performance issues like excessive recomposition.

* **Minimize Expensive Calculations:** Complex or long-running calculations are forbidden within the body of a composable function. Composable functions can be executed as frequently as every frame during an animation. Any necessary calculation must be wrapped in a remember {... } block to cache the result across recompositions.15 The preferred approach, however, is to move all such calculations off the UI thread and into the  
  ViewModel or a background coroutine, providing only the final result to the composable.  
* **Use Lazy Layout Keys:** When implementing scrollable lists with LazyColumn, LazyRow, or similar lazy layouts, providing a stable and unique key for each item via the key \= {... } parameter is mandatory. This allows Compose's diffing algorithm to identify individual items that have changed, added, or moved, enabling it to perform minimal, targeted recompositions instead of redrawing the entire list.15  
* **Limit Recompositions with derivedStateOf:** In scenarios where a piece of state changes very frequently (e.g., scroll position), but the UI only needs to react to a derived condition (e.g., whether the first item is visible), derivedStateOf must be used. This creates a new State object that only triggers recomposition when its resulting value actually changes, effectively filtering out unnecessary updates.15  
* **Defer State Reads:** To optimize recomposition, state reads should be deferred as long as possible. When passing a frequently changing state variable to a modifier, the lambda version of that modifier must be used (e.g., Modifier.offset { IntOffset(offsetX.value, 0\) }). This defers the reading of offsetX.value to the layout phase, allowing Compose to potentially skip the entire composition phase if the state change only affects layout or drawing.15  
* **Avoid Backwards Writes:** A "backwards write" occurs when a composable function writes to a State object that has already been read during the same composition pass. This is a critical error that creates an infinite recomposition loop and severely degrades performance. This practice is strictly forbidden. State should only flow downwards, and events upwards.15

The principles of Compose performance are not a mere checklist of optimizations but are deeply interconnected with the application's architecture. A well-architected UI layer that strictly adheres to UDF and state hoisting naturally avoids many common performance pitfalls. For instance, hoisting state and creating small, focused composables that only accept the data they need directly limits the scope of recomposition. When a small piece of data changes, only the specific composable that depends on it will be recomposed. Therefore, the primary directive for writing performant Compose code is to first write architecturally sound code. The performance directives listed above serve as tools to fine-tune this already solid foundation.

## **Section 4: Domain & Data Layer Protocol**

This section governs the implementation of the application's backend logic, including data handling, business rules, and asynchronous operations. Adherence to these protocols is essential for creating a robust, decoupled, and testable system.

### **4.1. Repository Pattern Implementation**

The Repository pattern is the mandatory design pattern for the data layer. Repositories serve as the single source of truth for all application data, abstracting the data sources from the rest of the app.4

Each repository must expose a clean, high-level API to its consumers (typically ViewModels or Use Cases). For example, a repository might expose a function like getBoardItems(): Flow\<List\<BoardItem\>\>. Internally, the repository is responsible for managing the complexities of data retrieval, such as deciding whether to fetch data from a network API, a local database cache, or another source. This implementation detail must be completely hidden from the caller.4

### **4.2. Asynchronous Operations with Kotlin Coroutines**

All asynchronous operations, such as network requests, database access, or complex computations, must be executed using Kotlin Coroutines.4 Coroutines provide a modern, efficient, and structured way to manage concurrency.

* **Structured Concurrency:** Coroutines must be launched within a CoroutineScope that is tied to a specific lifecycle. For work initiated from the UI, this must be the viewModelScope provided by the androidx.lifecycle:lifecycle-viewmodel-ktx library.16 The  
  viewModelScope is automatically cancelled when the ViewModel is cleared, preventing memory leaks and orphaned work. The use of GlobalScope is strictly forbidden. GlobalScope creates top-level coroutines that are not bound to any job, making them difficult to manage, impossible to test reliably, and a common source of resource leaks.16  
* **Main-Safety:** All suspend functions exposed by the data and domain layers must be "main-safe." This is a critical principle meaning that they are safe to be called directly from the main thread without blocking it.16 If a function needs to perform a long-running or blocking operation (e.g., I/O), it is the responsibility of that function to switch its execution context to an appropriate background dispatcher using  
  withContext(Dispatchers.IO) for disk or network operations, or withContext(Dispatchers.Default) for CPU-intensive work.16 The caller, such as a  
  ViewModel, should not be burdened with managing thread pools or dispatchers.  
* **Dispatcher Injection:** To facilitate testing, CoroutineDispatchers must be injected into classes like Repositories via their constructor. Hardcoding dispatchers (e.g., withContext(Dispatchers.IO)) directly couples the class to a specific execution policy, making it difficult to test. By injecting the dispatcher, it can be replaced with a TestDispatcher in unit tests, allowing for precise control over coroutine execution and making tests deterministic and reliable.16

### **4.3. Reactive Data with Kotlin Flow**

For data streams that can change over time, Kotlin Flow must be used.

* Repositories must expose a kotlinx.coroutines.flow.Flow for any data that is observable. For example, a query to a Room database that should automatically update the UI when the underlying data changes must return a Flow\<List\<MyEntity\>\>.4  
* For one-shot operations that execute once and return a single result (e.g., making a POST request to a server), a simple suspend function is the appropriate choice.16 This distinction ensures that the UI layer can be built reactively, responding seamlessly to data changes without requiring manual refresh logic.

### **4.4. Dependency Injection with Hilt**

Hilt is the mandatory dependency injection (DI) framework for the geminiboard project. It reduces the boilerplate of manual DI and integrates seamlessly with Jetpack components.11

* **Core Setup:** The project's Application class must be annotated with @HiltAndroidApp. Any Android component (Activity, Fragment, View, Service) that requires dependency injection must be annotated with @AndroidEntryPoint. ViewModels have a specific annotation, @HiltViewModel, which must be used.11  
* **Constructor Injection:** Constructor injection is the preferred method for providing dependencies. Any class that Hilt should know how to create must have its primary constructor annotated with @Inject.11  
* **Hilt Modules:** For cases where constructor injection is not possible (e.g., providing an implementation for an interface, or creating instances of classes from external libraries like Retrofit or Room), a Hilt module must be used.  
  * **@Binds:** To provide an implementation for an interface, an abstract function annotated with @Binds must be used within a Hilt module. @Binds is more efficient and generates less code than @Provides, making it the preferred choice for interface bindings.11  
  * **@Provides:** To provide instances of external library classes or objects that require a complex initialization (e.g., using a builder pattern), a concrete function annotated with @Provides must be used.11  
* **Scoping:** Dependencies must be scoped correctly to manage their lifecycle and prevent memory leaks. Hilt provides several scope annotations (e.g., @Singleton, @ActivityRetainedScoped, @ViewModelScoped). A dependency should be given the narrowest scope possible. For example, an object that should live for the entire application lifecycle (like a Retrofit instance) can be scoped with @Singleton. An object that should only live as long as a specific ViewModel should be scoped with @ViewModelScoped. Overusing @Singleton for all dependencies is a common anti-pattern that leads to unnecessary memory consumption and is forbidden.11

## **Section 5: Quality Assurance Mandates**

This section defines the mandatory testing strategy and quality gates that all code contributed to the geminiboard project must pass. A commitment to high-quality, well-tested code is non-negotiable.

### **5.1. Unit Testing Protocol (Local Tests)**

The foundation of the testing strategy is a comprehensive suite of local unit tests that run on the JVM. These tests are fast, reliable, and essential for verifying business logic in isolation.

* **Scope and Location:** The majority of testing efforts must be focused on unit tests. These tests are located in the src/test/java directory of each module.20  
* **Primary Targets:** The primary targets for unit testing are ViewModels, Repositories, Use Cases, and any other class containing business logic. UI components (Composables) should be tested via UI tests.  
* **Frameworks and Structure:** Tests must be written using the JUnit 4 or JUnit 5 testing framework. Dependencies of the class under test must be replaced with test doubles (mocks or fakes). The MockK or Mockito library is the standard for creating mock objects.20 All test methods must follow the Arrange-Act-Assert (AAA) pattern for clarity and maintainability: first arrange the test conditions and mocks, then act by calling the method under test, and finally assert that the expected outcome occurred.23  
* **Testing Coroutines:** Asynchronous code involving coroutines requires a specific testing approach. All tests for suspend functions or Flows must use the runTest builder from the kotlinx-coroutines-test library. This builder provides a TestCoroutineScheduler that allows for precise control over virtual time, enabling immediate execution of delayed work and ensuring that tests are deterministic and fast. Injected CoroutineDispatchers must be replaced with a TestDispatcher (e.g., StandardTestDispatcher) in the test setup.16

### **5.2. UI Testing Protocol (Instrumentation Tests)**

Instrumentation tests run on a physical Android device or an emulator and are used to verify the correctness of the UI and its integration with the underlying logic.

* **Scope and Location:** Instrumentation tests are located in the src/androidTest/java directory.24 Because they are slower and more resource-intensive than unit tests, they must be used judiciously. Their use should be reserved for verifying critical user flows, complex UI interactions, and ensuring correct layout rendering.24  
* **Framework:** All UI tests for Jetpack Compose must be written using the official Compose testing APIs. A test rule must be created for each test class using either createComposeRule() for testing composables in isolation or createAndroidComposeRule\<MyActivity\>() when access to an Activity is required.24  
* **Methodology:** Tests must use semantic finders (e.g., onNodeWithText, onNodeWithTag) to locate UI elements in the semantics tree. After finding a node, the test can perform user actions on it (e.g., performClick(), performScrollTo()) and then make assertions about its state (e.g., assertIsDisplayed(), assertTextContains()) to verify that the UI has responded correctly to user input or state changes from the ViewModel.24

### **5.3. Test Coverage and Automation**

To ensure a consistent quality bar, all testing processes must be automated.

* **Continuous Integration (CI):** The project's CI/CD pipeline must be configured to run all unit and instrumentation tests automatically on every pull request. A pull request cannot be merged into the main branch unless all tests pass.  
* **Code Coverage Enforcement:** The CI pipeline must also be configured to measure and enforce a minimum code coverage threshold. A target of at least 70% line coverage for all ViewModels, Repositories, and Use Cases will be enforced. Pull requests that do not meet this standard will be blocked from merging.

## **Section 6: Contribution and Code Hygiene Protocol**

This section defines the standardized process for all contributions to the geminiboard codebase. These rules ensure clarity, consistency, and a high degree of automation in the development workflow.

### **6.1. Version Control: Conventional Commits**

To maintain a clean, understandable, and machine-readable version history, all git commit messages must strictly adhere to the Conventional Commits specification v1.0.0.25

* **Structure:** The commit message must follow the format: \<type\>(\<scope\>): \<description\>. For example: feat(board): add drag-and-drop reordering. The scope is optional but recommended for clarity in a multi-module project.  
* **Rationale:** This convention is not merely a stylistic choice. It enables powerful automation, including the automatic generation of CHANGELOG files and the automatic determination of semantic version bumps based on the types of commits merged.26 This is especially critical for a project with an AI contributor, as it provides a structured language for the AI to describe its changes.

The following table provides a quick-reference guide for the allowed commit types and their impact on semantic versioning. This table must be used to classify all changes.

**Table 2: Conventional Commit Type Reference**

| Type | Description | Impacts SemVer | Source |
| :---- | :---- | :---- | :---- |
| feat | A new feature for the user. | MINOR | 25 |
| fix | A bug fix for the user. | PATCH | 25 |
| perf | A code change that improves performance without changing functionality. | PATCH | 29 |
| build | Changes that affect the build system or external dependencies. | PATCH | 26 |
| ci | Changes to CI configuration files and scripts. | None | 29 |
| docs | Documentation only changes. | None | 29 |
| refactor | A code change that neither fixes a bug nor adds a feature. | None | 29 |
| style | Changes that do not affect the meaning of the code (formatting, etc.). | None | 29 |
| test | Adding missing tests or correcting existing tests. | None | 29 |
| chore | Other changes that don't modify src or test files. | None | 26 |
| BREAKING CHANGE | Indicated by a footer or \! after type/scope (e.g., refactor\!:). | MAJOR | 25 |

### **6.2. Pull Request (PR) Standards**

All code changes must be submitted via a pull request. To ensure PRs are clear, complete, and easy to review, the repository must contain a PULL\_REQUEST\_TEMPLATE.md file located in the .github/ directory.30

This template must include the following mandatory sections 30:

* **Context / Related Issue:** A mandatory link to the issue or task ticket that this PR addresses. This provides reviewers with the necessary background and rationale for the change.  
* **Description of Changes:** A clear and concise summary of the "what" and "why" behind the code changes. This should explain the approach taken and any significant architectural decisions.  
* **Testing Strategy:** A detailed description of how the changes were tested. This must include specifics, such as "Unit tests were added for NewFeatureViewModel" or "Manual testing was performed on a Pixel 8 emulator, API 34, covering the user registration flow."  
* **Screenshots / Videos:** For any PR that involves UI changes, screenshots or a short video demonstrating the changes are required. This provides essential visual context for reviewers and stakeholders.  
* **Pre-Submission Checklist:** A mandatory checklist that the author must complete before submitting the PR. This checklist will confirm that the code compiles, all tests pass locally, new code is documented with KDoc, and the PR adheres to all other project standards.

### **6.3. Code Documentation: KDoc Standards**

Comprehensive documentation is essential for long-term maintainability and for providing context to all developers, including the AI agent.

All public classes, methods, and properties must be documented using KDoc.34 The documentation must strictly follow the official AndroidX KDoc guidelines.34 Key requirements include:

* A summary description for every public API.  
* An explicit @param tag for every parameter in a function or constructor.  
* An explicit @return tag for every function that returns a non-Unit value.  
* The use of @sample to provide compilable code examples for complex or important APIs.  
* The use of Markdown for inline formatting (e.g., \[link\] for links, \`code\` for code spans) instead of Javadoc's HTML tags.

This documentation is not optional; it is a required part of the definition of "done" for any code change.

#### **Works cited**

1. accessed January 1, 1970, [https://github.com/ClemensSchartmueller/geminiboard](https://github.com/ClemensSchartmueller/geminiboard)  
2. accessed January 1, 1970, [https://github.com/ClemensSchartmueller/geminiboard/tree/main/app/src/main/java/com/clemensschartmueller/geminiboard](https://github.com/ClemensSchartmueller/geminiboard/tree/main/app/src/main/java/com/clemensschartmueller/geminiboard)  
3. Android App Development in 2025: What You Need to Know \- Jhavtech Studios, accessed August 9, 2025, [https://www.jhavtech.com.au/android-app-development-2025/](https://www.jhavtech.com.au/android-app-development-2025/)  
4. Roadmap to Android App Development in 2025 | by Sumeet Panchal \- Medium, accessed August 9, 2025, [https://sumeetpanchal-21.medium.com/roadmap-to-android-app-development-in-2025-bf32a32980bd](https://sumeetpanchal-21.medium.com/roadmap-to-android-app-development-in-2025-bf32a32980bd)  
5. Android App Development Tips and Tricks for 2025 \- AppIt Ventures, accessed August 9, 2025, [https://appitventures.com/blog/android-app-development-tips-tricks](https://appitventures.com/blog/android-app-development-tips-tricks)  
6. Migrate your build configuration from Groovy to Kotlin | Android Studio, accessed August 9, 2025, [https://developer.android.com/build/migrate-to-kotlin-dsl](https://developer.android.com/build/migrate-to-kotlin-dsl)  
7. Share your Gradle configuration with the Gradle Kotlin DSL \- A guide for Android projects, accessed August 9, 2025, [https://engineering.matchesfashion.com/share-your-gradle-configuration-with-the-gradle-kotlin-dsl-a-guide-for-android-projects-3ce6dc34ea75](https://engineering.matchesfashion.com/share-your-gradle-configuration-with-the-gradle-kotlin-dsl-a-guide-for-android-projects-3ce6dc34ea75)  
8. How to properly configure build.gradle files so that the project is built \- Help/Discuss, accessed August 9, 2025, [https://discuss.gradle.org/t/how-to-properly-configure-build-gradle-files-so-that-the-project-is-built/49847](https://discuss.gradle.org/t/how-to-properly-configure-build-gradle-files-so-that-the-project-is-built/49847)  
9. build.gradle.kts \- googlemaps/android-maps-rx \- GitHub, accessed August 9, 2025, [https://github.com/googlemaps/android-maps-rx/blob/main/build.gradle.kts](https://github.com/googlemaps/android-maps-rx/blob/main/build.gradle.kts)  
10. kotlin-samples/run/grpc-hello-world-gradle/build.gradle.kts at main \- GitHub, accessed August 9, 2025, [https://github.com/GoogleCloudPlatform/kotlin-samples/blob/main/run/grpc-hello-world-gradle/build.gradle.kts](https://github.com/GoogleCloudPlatform/kotlin-samples/blob/main/run/grpc-hello-world-gradle/build.gradle.kts)  
11. Dependency injection with Hilt | App architecture | Android Developers, accessed August 9, 2025, [https://developer.android.com/training/dependency-injection/hilt-android](https://developer.android.com/training/dependency-injection/hilt-android)  
12. State and Jetpack Compose | Android Developers, accessed August 9, 2025, [https://developer.android.com/develop/ui/compose/state](https://developer.android.com/develop/ui/compose/state)  
13. Jetpack Compose State Management: A Guide for Android Developers \- Bugfender, accessed August 9, 2025, [https://bugfender.com/blog/jetpack-compose-state-management/](https://bugfender.com/blog/jetpack-compose-state-management/)  
14. Handling State Efficiently with Jetpack Compose | by Gastón Saillén \- Medium, accessed August 9, 2025, [https://medium.com/@gsaillen95/handling-state-efficiently-with-jetpack-compose-252b6452748b](https://medium.com/@gsaillen95/handling-state-efficiently-with-jetpack-compose-252b6452748b)  
15. Follow best practices | Jetpack Compose \- Android Developers, accessed August 9, 2025, [https://developer.android.com/develop/ui/compose/performance/bestpractices](https://developer.android.com/develop/ui/compose/performance/bestpractices)  
16. Best practices for coroutines in Android | Kotlin | Android Developers, accessed August 9, 2025, [https://developer.android.com/kotlin/coroutines/coroutines-best-practices](https://developer.android.com/kotlin/coroutines/coroutines-best-practices)  
17. Best practices \- Kt. Academy, accessed August 9, 2025, [https://kt.academy/article/cc-best-practices](https://kt.academy/article/cc-best-practices)  
18. (Deprecated) Using Hilt in your Android app, accessed August 9, 2025, [https://developer.android.com/codelabs/android-hilt](https://developer.android.com/codelabs/android-hilt)  
19. Mastering Dependency Injection with Hilt in Android | by praveen sharma \- Medium, accessed August 9, 2025, [https://medium.com/@sharmapraveen91/mastering-dependency-injection-with-hilt-in-android-0d1f9ee5953c](https://medium.com/@sharmapraveen91/mastering-dependency-injection-with-hilt-in-android-0d1f9ee5953c)  
20. Build local unit tests | Test your app on Android | Android Developers, accessed August 9, 2025, [https://developer.android.com/training/testing/local-tests](https://developer.android.com/training/testing/local-tests)  
21. Unit Test in Android all the things you need to know | by Nine Pages Of My Life | Medium, accessed August 9, 2025, [https://medium.com/@niranjanky14/unit-test-in-android-all-the-things-you-need-to-know-18357f468126](https://medium.com/@niranjanky14/unit-test-in-android-all-the-things-you-need-to-know-18357f468126)  
22. Best Practices for Writing Unit Tests in Android : Basics | by KmDev | Medium, accessed August 9, 2025, [https://medium.com/@mkcode0323/best-practices-for-writing-unit-tests-in-android-basics-34eb77172f27](https://medium.com/@mkcode0323/best-practices-for-writing-unit-tests-in-android-basics-34eb77172f27)  
23. What is Android Unit Testing? | BrowserStack, accessed August 9, 2025, [https://www.browserstack.com/guide/android-unit-testing](https://www.browserstack.com/guide/android-unit-testing)  
24. Build instrumented tests | Test your app on Android | Android ..., accessed August 9, 2025, [https://developer.android.com/training/testing/instrumented-tests](https://developer.android.com/training/testing/instrumented-tests)  
25. Conventional Commits, accessed August 9, 2025, [https://www.conventionalcommits.org/en/v1.0.0/](https://www.conventionalcommits.org/en/v1.0.0/)  
26. Conventional Commit Specification | by Pranay Bathini \- Medium, accessed August 9, 2025, [https://pranaybathini.medium.com/conventional-commit-specification-ecd701b0bbb2](https://pranaybathini.medium.com/conventional-commit-specification-ecd701b0bbb2)  
27. Conventional Commits. A specification for adding human and… | by Cid Miranda | Medium, accessed August 9, 2025, [https://medium.com/@cidmiranda/conventional-commits-5c3b4bc7de08](https://medium.com/@cidmiranda/conventional-commits-5c3b4bc7de08)  
28. Conventional Commits \- Specification for Your Commit Messages \- DEV Community, accessed August 9, 2025, [https://dev.to/carlosazaustre/conventional-commits-specification-for-your-commit-messages-14lo](https://dev.to/carlosazaustre/conventional-commits-specification-for-your-commit-messages-14lo)  
29. From chaos to clarity: the power of Conventional Commits \- Eagerworks, accessed August 9, 2025, [https://eagerworks.com/blog/conventional-commits](https://eagerworks.com/blog/conventional-commits)  
30. GitHub pull request template | Axolo Blog, accessed August 9, 2025, [https://axolo.co/blog/p/part-3-github-pull-request-template](https://axolo.co/blog/p/part-3-github-pull-request-template)  
31. Comprehensive Checklist: GitHub PR Template \- Graphite, accessed August 9, 2025, [https://graphite.dev/guides/comprehensive-checklist-github-pr-template](https://graphite.dev/guides/comprehensive-checklist-github-pr-template)  
32. What do you think should be the standard for pull requests? : r/ExperiencedDevs \- Reddit, accessed August 9, 2025, [https://www.reddit.com/r/ExperiencedDevs/comments/142n5ea/what\_do\_you\_think\_should\_be\_the\_standard\_for\_pull/](https://www.reddit.com/r/ExperiencedDevs/comments/142n5ea/what_do_you_think_should_be_the_standard_for_pull/)  
33. Write better PR's with this template \- DEV Community, accessed August 9, 2025, [https://dev.to/nicolasmontielf/writing-a-good-pull-request-with-template-46pm](https://dev.to/nicolasmontielf/writing-a-good-pull-request-with-template-46pm)  
34. Kotlin documentation (KDoc) guidelines \- Android GoogleSource, accessed August 9, 2025, [https://android.googlesource.com/platform/frameworks/support/+/refs/heads/androidx-recyclerview-release/docs/kdoc\_guidelines.md](https://android.googlesource.com/platform/frameworks/support/+/refs/heads/androidx-recyclerview-release/docs/kdoc_guidelines.md)