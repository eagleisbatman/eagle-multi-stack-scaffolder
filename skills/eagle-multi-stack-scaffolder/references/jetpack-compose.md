# Jetpack Compose Reference

## Research Queries
- "Jetpack Compose best practices 2025 2026"
- "Android app architecture guide official"
- "Jetpack Compose project structure clean architecture"
- "Android version catalogs dependency management"

## Package Manager
**Gradle Version Catalogs** - Required for modern Android projects.

```toml
# gradle/libs.versions.toml
[versions]
kotlin = "1.9.22"
compose-bom = "2025.01.00"
hilt = "2.50"

[libraries]
compose-bom = { group = "androidx.compose", name = "compose-bom", version.ref = "compose-bom" }
compose-ui = { group = "androidx.compose.ui", name = "ui" }
compose-material3 = { group = "androidx.compose.material3", name = "material3" }
hilt-android = { group = "com.google.dagger", name = "hilt-android", version.ref = "hilt" }

[plugins]
android-application = { id = "com.android.application", version = "8.2.2" }
kotlin-android = { id = "org.jetbrains.kotlin.android", version.ref = "kotlin" }
```

## Project Structure

```
app/src/main/java/com/example/{app}/
├── MainActivity.kt
├── {App}Application.kt
├── core/
│   ├── di/
│   ├── network/
│   └── database/
├── data/
│   ├── model/
│   ├── repository/
│   └── mapper/
├── domain/
│   ├── model/
│   ├── repository/
│   └── usecase/
└── presentation/
    ├── navigation/
    ├── theme/
    ├── components/
    └── screens/
        └── home/
            ├── HomeScreen.kt
            ├── HomeViewModel.kt
            └── HomeUiState.kt
```

## Architecture: Clean + MVVM

```kotlin
@HiltViewModel
class HomeViewModel @Inject constructor(
    private val getUsersUseCase: GetUsersUseCase
) : ViewModel() {
    private val _uiState = MutableStateFlow(HomeUiState())
    val uiState: StateFlow<HomeUiState> = _uiState.asStateFlow()

    fun loadUsers() {
        viewModelScope.launch {
            _uiState.update { it.copy(isLoading = true) }
            getUsersUseCase()
                .onSuccess { users ->
                    _uiState.update { it.copy(isLoading = false, users = users) }
                }
        }
    }
}

@Composable
fun HomeScreen(viewModel: HomeViewModel = hiltViewModel()) {
    val uiState by viewModel.uiState.collectAsStateWithLifecycle()
    HomeContent(uiState = uiState)
}
```

## Essential Libraries

| Category | Package | Notes |
|----------|---------|-------|
| DI | Hilt | Google recommended |
| Networking | Retrofit + OkHttp | Industry standard |
| Images | Coil | Kotlin-first, Compose support |
| Database | Room | Official |
| Navigation | Navigation Compose | Official |

## Setup Commands

```bash
# Via Android Studio:
# File > New > New Project > Empty Activity (Compose)

# After creation, add version catalog:
mkdir -p gradle
touch gradle/libs.versions.toml
```

## Conventions

- Composables: `PascalCase` - `UserProfileCard`
- ViewModels: `{Feature}ViewModel`
- UI State: `{Feature}UiState`
- Use `collectAsStateWithLifecycle()` for flows
- Stateless composables preferred
- Modifier always last parameter with default
