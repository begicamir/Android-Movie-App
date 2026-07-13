# Android MovieApp (Jetpack Compose)
The Movie app is a modern Android application built with Jetpack Compose and Kotlin. It follows clean architecture principles with an MVVM + unidirectional data flow (MVI‑like) approach and leverages Hilt for dependency injection. The app integrates Retrofit for networking, Room for local persistence, and 
Coil for efficient image loading. Navigation is powered by Navigation‑Compose, and Accompanist is used to control system UI colors.

This project structure and implementation are inspired by clean, scalable Android patterns.


## About The App
With this app you can browse movies from The Movie Database (TMDB):
- Explore Popular and Upcoming movie lists with infinite pagination.
- View detailed information for a selected movie (title, overview, release date, rating, artwork).
- Enjoy fast loading and smooth UI thanks to Jetpack Compose and image caching.
- **Offline‑first experience:** previously fetched lists are cached locally and served instantly when available.


## Key Features
- Jetpack Compose UI: Declarative UI with Material 3 theming and responsive layouts.
- Kotlin: Entirely written in Kotlin for clarity and conciseness.
- Clean Architecture: Clear separation of concerns across data, domain, and presentation layers.
- MVVM + Unidirectional Data Flow: StateFlows for state, sealed events for user intents, and a Resource wrapper for loading/success/error.
- Hilt DI: Streamlined dependency injection (Retrofit, OkHttp, Room, Repositories, ViewModels).
- Retrofit + OkHttp: Robust network stack with logging interceptor.
- Room: Local caching of movie lists for offline‑first behavior and quick subsequent loads.
- Navigation‑Compose: In‑app navigation between home (tabs) and details screens.
- Coil: Efficient image loading and caching; artwork dominates each card and details.
- Accompanist System UI Controller: Status/navigation bars color customization.


## Tech Stack and Libraries
- Language: Kotlin (JVM target 1.8)
- UI: Jetpack Compose (Material 3, Compose BOM)
- Navigation: androidx.navigation:navigation‑compose (2.7.5)
- DI: Hilt (com.google.dagger:hilt‑android 2.48, hilt‑navigation‑compose 1.1.0)
- Persistence: Room (2.6.0) with KTX and Paging artifact (prepared)
- Networking: Retrofit (2.9.0), Gson converter (2.9.0), OkHttp (4.11.0) + Logging Interceptor (4.10.0)
- Images: Coil‑Compose (2.4.0)
- UI Extras: Material Icons Extended (1.5.4), Accompanist System UI Controller (0.27.0)
- AndroidX: Core‑ktx, Lifecycle‑runtime‑ktx
- Compose Compiler Extension: 1.5.14
- SDK: minSdk 24, target/compileSdk 34

All versions are defined either directly in app/build.gradle.kts or via Gradle version catalogs (libs.versions.toml).


## Architecture Overview
The app is organized around a clean architecture with three main concerns:

- Data Layer
  - Remote: Retrofit interface (MovieApi) to call TMDB endpoints.
  - Local: Room database (MovieDB) with DAO (MovieDAO) to cache movie entities (MovieEntity).
  - Mappers: Safe conversion between network DTOs (MovieDTO) and local entities/domain models.
  - Repository: MovieListRepositoryImpl orchestrates data sources and exposes Flows of Resource<T> to the domain/presentation.

- Domain Layer
  - Models: Domain model Movie represents the data consumed by UI.
  - Repository Interface: MovieListRepository abstracts data operations for ViewModels.

- Presentation Layer
  - ViewModels: MovieListViewModel and DetailsViewModel manage UI state with MutableStateFlow and handle user events.
  - State & Events: MovieListState, MovieListEvent, DetailsState provide typed state and intents.
  - UI: Composables (HomeScreen, PopularMovieScreen, UpcomingMovieScreen, DetailsScreen, MovieItem) render data and dispatch events.
  - Navigation: Screen sealed class defines routes for NavHost destinations.

**Unidirectional data flow:**
User action -> ViewModel.onEvent(...) -> Repository (Flow<Resource<...>>) -> ViewModel updates StateFlow -> Composables observe state -> UI.


## Data and Offline Strategy
- Endpoint used: GET movie/{category}
  - Path param: category = "popular" or "upcoming"
  - Query params: page, api_key
- Base URLs:
  - API: https://api.themoviedb.org/3/
  - Images: https://image.tmdb.org/t/p/w500
- Caching: On successful remote fetch, results are mapped to MovieEntity and upserted into Room. Subsequent loads serve cached data when present unless a forced remote fetch is triggered by pagination.
- Offline behavior: If cached data exists and remote is not forced, lists are immediately served from Room. Details are resolved from local DB by ID.


## Modules and Key Classes
- DI (Hilt)
  - AppModule: Provides Retrofit(MovieApi), OkHttp client with HttpLoggingInterceptor, Room database instance.
  - RepositoryModule: Binds MovieListRepositoryImpl to MovieListRepository.

- Networking
  - MovieApi: Defines getMoviesList(category, page, apiKey) and contains BASE_URL, IMAGE_BASE_URL, API_KEY.

- Database
  - MovieDB: Room database with MovieDAO.
  - MovieDAO: upsertMovieList, getMovieById, getMovieListByCategory.
  - MovieEntity: Room entity representing cached movie rows.

- Data Mappers
  - MovieDTO.toMovieEntity(category)
  - MovieEntity.toMovie(category)

- Domain
  - Movie (domain model)
  - MovieListRepository (interface)

- Repository Implementation
  - MovieListRepositoryImpl: Exposes Flow<Resource<List<Movie>>> for lists and Flow<Resource<Movie>> for details; handles IO exceptions and emits loading/success/error.

- Presentation
  - MovieListViewModel: Manages popular/upcoming pages, pagination, and current tab; exposes MovieListState via StateFlow; handles Navigate and Paginate events.
  - DetailsViewModel: Loads a movie by ID from local DB.
  - State/Events: MovieListState, MovieListEvent; DetailsState (holds loading + movie).

- UI Composables
  - HomeScreen: Scaffold with TopAppBar and BottomNavigation; hosts Popular and Upcoming tabs (via nested NavHost) and routes to Details.
  - PopularMovieScreen / UpcomingMovieScreen: Grid list with lazy pagination trigger; shows CircularProgressIndicator on first load.
  - MovieItem: Card with artwork, dynamic background gradient using average color, rating stars, and click to navigate to Details.
  - DetailsScreen: Displays large poster/backdrop, title, overview, release date, and rating; uses RatingBar and Coil.

- Utilities
  - Resource: Sealed class (Success, Error, Loading) used across flows.
  - Screen: Sealed class for routes (Home, PopularMovieList, UpcomingMovieList, Details).
  - RatingBar: Compose helper to render filled/half/outline stars based on rating.
  - getAverageColor: Extracts and slightly darkens average image color for dynamic theming.


## Project Structure (high level)
- app/src/main/java/com/example/syllablemovieapp/
  - App.kt (HiltAndroidApp)
  - MainActivity.kt (AndroidEntryPoint, Compose NavHost)
  - movieList/
    - core/presentation/ (HomeScreen, PopularMovieScreen, UpcomingMovieScreen, components/MovieItem)
    - data/
      - local/movie/ (MovieDB, MovieDAO, MovieEntity)
      - mappers/ (MovieMapper)
      - remote/ (MovieApi, response DTOs: MovieDTO, MovieListDTO, Dates)
      - repository/ (MovieListRepositoryImpl)
    - details/presentation/ (DetailsScreen, DetailsViewModel, DetailsState)
    - di/ (AppModule, RepositoryModule)
    - domain/ (model/Movie, repository/MovieListRepository)
    - presentation/ (MovieListViewModel, MovieListState, MovieListEvent)
    - util/ (Resource, Screen, Category, RatingBar, AverageColor)
  - ui/theme/ (Material 3 theme, colors, typography)


## Getting Started
1) Prerequisites
- Android Studio Giraffe/Koala+ with Gradle support
- Android SDK 34, minSdk 24
- A TMDB API key (https://www.themoviedb.org/)

2) Clone
- git clone https://example.com/your/syllablemovie.git

3) Configure API key
- For simplicity, this project currently uses a constant in MovieApi.kt:
  - MovieApi.API_KEY = "<your-api-key>"
- Replace the placeholder with your own TMDB API key.
- Note: For production apps, prefer secure configuration (e.g., BuildConfig fields, local.properties, or remote config), not checked into VCS.

4) Build and Run
- Open in Android Studio, let it sync dependencies.
- Select a device/emulator and run.


## How It Works (Data Flow)
- On app start, MovieListViewModel loads Popular and Upcoming pages from the repository.
- Repository checks Room cache; if available and not forced, returns cached data. Otherwise, calls Retrofit -> TMDB, maps DTOs to entities, upserts into Room, and emits domain models.
- Screens observe StateFlow to render UI; when user scrolls to the end, a Paginate event requests the next page.
- Tapping a movie navigates to Details where data is loaded from the local DB by ID.


## Testing
- Project includes basic Android/Unit test skeletons; extend with ViewModel tests using coroutine TestDispatcher and Room in‑memory DB for repository tests.


## Notes & Limitations
- API key is hardcoded for simplicity in MovieApi.kt; move it to secure config for real apps.
- Current feature set includes Popular and Upcoming lists with pagination and a details screen. Search, trailers/playback, and TV series are not implemented in this codebase.
- Pagination triggers in the grid screens are simple; consider adding loading footers and duplicate‑load guards.


## Acknowledgments
- The Movie Database (TMDB) for the public API and images.
- Android Open Source community for libraries and guidance.

# Images from the app 

<img width="1080" height="2400" alt="Screenshot_20260710_193647" src="https://github.com/user-attachments/assets/374482d3-38b8-4538-8d49-a48a2cf42ee7" />
<img width="1080" height="2400" alt="Screenshot_20260710_193542" src="https://github.com/user-attachments/assets/5b1788bc-f23a-4171-b436-45a14297de60" />
<img width="1080" height="2400" alt="Screenshot_20260710_193705" src="https://github.com/user-attachments/assets/a47f2c5d-9605-4e03-aab5-af0e41c4b7b0" />
