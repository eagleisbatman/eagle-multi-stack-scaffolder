# SwiftUI Reference

## Research Queries
- "SwiftUI best practices 2025 2026"
- "SwiftUI project structure large apps"
- "SwiftUI recommended architecture MVVM TCA"
- "Swift Package Manager vs CocoaPods 2025"

## Package Manager
**Swift Package Manager (SPM)** - Always preferred unless library only supports CocoaPods.

## Project Structure

```
{AppName}/
├── App/
│   ├── {AppName}App.swift
│   └── ContentView.swift
├── Features/
│   ├── Home/
│   │   ├── Views/
│   │   ├── ViewModels/
│   │   └── Models/
│   ├── Auth/
│   └── Settings/
├── Core/
│   ├── Network/
│   ├── Storage/
│   └── Services/
├── Design/
│   ├── Theme/
│   └── Components/
├── Resources/
└── Tests/
```

## Architecture: MVVM

```swift
@MainActor
class HomeViewModel: ObservableObject {
    @Published private(set) var state: ViewState = .idle
    @Published private(set) var items: [Item] = []
    
    func loadItems() async {
        state = .loading
        do {
            items = try await repository.fetchItems()
            state = .loaded
        } catch {
            state = .error(error.localizedDescription)
        }
    }
}

struct HomeView: View {
    @StateObject private var viewModel = HomeViewModel()
    var body: some View { /* ... */ }
}
```

## Essential Libraries

| Category | Package | Notes |
|----------|---------|-------|
| Networking | Alamofire or URLSession | Prefer native for simple apps |
| Images | Kingfisher | SwiftUI native |
| DI | Factory | Lightweight |
| Database | SwiftData (iOS 17+) | Apple native |
| Analytics | Firebase | Industry standard |

## Setup Commands

```bash
# Via Xcode:
# File > New > Project > iOS > App > SwiftUI

# Add packages:
# File > Add Package Dependencies > Enter GitHub URL
```

## Conventions

- Views end with `View`: `HomeView`
- ViewModels end with `ViewModel`: `HomeViewModel`
- Use `@StateObject` for owned state, `@ObservedObject` for passed state
- Extract subviews as computed properties
- Use `.task { }` for async loading
