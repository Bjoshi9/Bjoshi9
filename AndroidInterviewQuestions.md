# 📱 Senior Android Developer — 50 Mock Interview Questions & Answers

### Skills Covered: Kotlin · Coroutines · Flow · MVVM · Clean Architecture · Jetpack Compose · Navigation · Room · DataStore · DSA

-----

## 🟦 Section 1: Kotlin (Q1–Q10)

-----

### Q1. What are the key differences between `val`, `var`, and `const val` in Kotlin?

**Answer:**

- `val` — immutable reference, assigned once, can be set at runtime
- `var` — mutable reference, can be reassigned
- `const val` — compile-time constant, must be a primitive or String, evaluated at compile time

```kotlin
val name = "Brijesh"          // immutable, runtime
var age = 25                  // mutable, runtime
const val MAX_RETRY = 3       // compile-time constant

// const val must be top-level or inside object
object Config {
    const val BASE_URL = "https://api.example.com"
}
```

**Rule:** Always prefer `val`. Use `var` only when mutation is truly needed.

-----

### Q2. Explain data classes in Kotlin. What do they auto-generate?

**Answer:**

A `data class` is a class whose primary purpose is to hold data. The compiler automatically generates:

- `equals()` — structural equality
- `hashCode()` — consistent with equals
- `toString()` — readable string representation
- `copy()` — create a modified copy
- `componentN()` — for destructuring

```kotlin
data class Movie(val id: Int, val title: String, val rating: Double)

val movie1 = Movie(1, "Batman", 8.5)
val movie2 = movie1.copy(title = "Batman Begins") // new object, id and rating copied

// Destructuring
val (id, title, rating) = movie1

// Equality
val movie3 = Movie(1, "Batman", 8.5)
println(movie1 == movie3) // true — structural equality
```

-----

### Q3. What are sealed classes and when would you use them over enums?

**Answer:**

A `sealed class` restricts class hierarchies — all subclasses must be in the same file. Unlike enums, each subclass can hold different data.

```kotlin
// Enum — can't carry different data per state
enum class UiState { LOADING, SUCCESS, ERROR }

// Sealed class — each state carries its own data
sealed class UiState {
    object Loading : UiState()
    data class Success(val movies: List<Movie>) : UiState()
    data class Error(val message: String) : UiState()
}

// Exhaustive when — compiler forces all cases
when (state) {
    is UiState.Loading        -> showLoader()
    is UiState.Success        -> showMovies(state.movies)
    is UiState.Error          -> showError(state.message)
    // Miss any → compile error ✅
}
```

**Use sealed class when:** states need to carry different types/amounts of data.
**Use enum when:** states are simple constants with no associated data.

-----

### Q4. Explain extension functions. What are their limitations?

**Answer:**

Extension functions let you add functions to existing classes without inheriting or modifying them.

```kotlin
fun String.isValidEmail(): Boolean = contains("@") && contains(".")
fun Int.toFormattedDuration(): String = "${this / 60}h ${this % 60}m"

// Usage
"user@gmail.com".isValidEmail()  // true
125.toFormattedDuration()         // "2h 5m"
```

**Limitations:**

- Cannot access private members of the class
- Do not actually modify the class — compiled as static functions
- Can be shadowed by member functions (member always wins)
- No polymorphic dispatch — resolved at compile time based on declared type

-----

### Q5. What is the difference between `==` and `===` in Kotlin?

**Answer:**

- `==` — structural equality, calls `equals()` under the hood
- `===` — referential equality, checks if both point to the same object in memory

```kotlin
val movie1 = Movie(1, "Batman", 8.5)
val movie2 = Movie(1, "Batman", 8.5)
val movie3 = movie1

println(movie1 == movie2)  // true  — same content
println(movie1 === movie2) // false — different objects in memory
println(movie1 === movie3) // true  — same reference
```

-----

### Q6. Explain higher-order functions and lambdas with a real Android example.

**Answer:**

A higher-order function takes a function as a parameter or returns a function. Lambdas are anonymous functions passed as arguments.

```kotlin
// Higher-order function
fun performWithLoading(action: suspend () -> Unit) {
    _isLoading.value = true
    viewModelScope.launch {
        action()
        _isLoading.value = false
    }
}

// Usage with lambda
fun loadMovies() {
    performWithLoading {
        val movies = repository.getMovies()
        _uiState.value = UiState.Success(movies)
    }
}

// Real Android examples
button.setOnClickListener { viewModel.onClicked() }  // lambda
movies.filter { it.isReleased }.map { it.toUiModel() } // chained lambdas
```

-----

### Q7. What is the `object` keyword in Kotlin? Explain all its use cases.

**Answer:**

```kotlin
// 1. Singleton — object declaration
object DatabaseManager {
    fun connect() { }
}
DatabaseManager.connect()

// 2. Companion object — static-like members
class Movie {
    companion object {
        fun create(id: Int) = Movie()
        const val MAX_RATING = 10.0
    }
}
Movie.create(1)

// 3. Anonymous object — one-time interface implementation
val listener = object : OnClickListener {
    override fun onClick(v: View) { }
}
```

-----

### Q8. What is null safety in Kotlin and how do you handle nulls idiomatically?

**Answer:**

Kotlin’s type system distinguishes nullable (`String?`) from non-nullable (`String`) types at compile time, eliminating most NPEs.

```kotlin
var name: String = "Brijesh"  // cannot be null
var name: String? = null       // nullable

// Safe call — returns null instead of crashing
val length = name?.length

// Elvis operator — fallback value
val length = name?.length ?: 0

// Let — execute block only if non-null
name?.let {
    println(it.uppercase())
    sendAnalytics(it)
}

// Safe cast — null if cast fails
val movie = item as? Movie

// Not-null assertion — crash if null, use sparingly
val length = name!!.length  // ⚠️ avoid unless certain
```

-----

### Q9. Explain `inline` functions. Why are they useful with higher-order functions?

**Answer:**

`inline` functions copy their body at the call site at compile time, avoiding the overhead of creating a lambda object for each call.

```kotlin
// Without inline — creates lambda object every call (heap allocation)
fun execute(action: () -> Unit) = action()

// With inline — lambda body copied at call site, no object creation
inline fun execute(action: () -> Unit) = action()

// Real benefit — reified type parameters (only possible with inline)
inline fun <reified T> parseJson(json: String): T {
    return Gson().fromJson(json, T::class.java)
}

// Usage
val movie = parseJson<Movie>(jsonString)  // T is known at compile time
```

**Use inline when:** function is called frequently with lambda params, or when you need `reified` generics.

-----

### Q10. What is the difference between `List`, `MutableList`, `ArrayList`, and `Array` in Kotlin?

**Answer:**

```kotlin
// List — immutable interface, read-only
val list: List<Movie> = listOf(movie1, movie2)

// MutableList — mutable interface
val mutableList: MutableList<Movie> = mutableListOf(movie1)
mutableList.add(movie2)

// ArrayList — concrete implementation of MutableList
val arrayList = ArrayList<Movie>()
arrayList.add(movie1)

// Array — fixed size, primitive-friendly
val array = arrayOf(movie1, movie2)
val intArray = IntArray(5) { it * 2 } // [0, 2, 4, 6, 8]
```

**Rule:**

- Default to `List` (immutable) in function signatures
- Use `MutableList` when you need to modify
- `Array` for fixed-size, performance-critical collections

-----

## 🟩 Section 2: Coroutines (Q11–Q18)

-----

### Q11. What is structured concurrency and why does it matter?

**Answer:**

Structured concurrency means a coroutine’s lifetime is tied to the scope it was launched in. Parent coroutines wait for all children to complete, and cancelling a parent cancels all children.

```kotlin
viewModelScope.launch {                    // Parent
    val moviesJob = launch { fetchMovies() }   // Child 1
    val profileJob = launch { fetchProfile() } // Child 2

    // If viewModelScope is cancelled (e.g. ViewModel cleared):
    // → moviesJob cancelled ✅
    // → profileJob cancelled ✅
    // No orphan coroutines, no memory leaks ✅
}
```

**Why it matters:**

- Prevents memory leaks from orphan coroutines
- Automatic cancellation propagates down the hierarchy
- Exceptions propagate up to parent scope

-----

### Q12. Explain coroutine scopes. What is the difference between `viewModelScope`, `lifecycleScope`, and `GlobalScope`?

**Answer:**

```kotlin
// viewModelScope — tied to ViewModel lifetime
// Auto-cancelled when ViewModel is cleared (user navigates away)
class MovieViewModel : ViewModel() {
    init {
        viewModelScope.launch { fetchMovies() }
    }
}

// lifecycleScope — tied to Activity/Fragment lifecycle
// Auto-cancelled when Activity/Fragment is destroyed
class MovieFragment : Fragment() {
    override fun onViewCreated(...) {
        lifecycleScope.launch {
            repeatOnLifecycle(Lifecycle.State.STARTED) {
                viewModel.uiState.collect { render(it) }
            }
        }
    }
}

// GlobalScope — lives for entire app lifetime ⚠️ AVOID
// No automatic cancellation, can cause memory leaks
GlobalScope.launch { } // ❌ never use in production
```

-----

### Q13. What is the difference between `launch` and `async`?

**Answer:**

- `launch` — fire and forget, returns `Job`, no result
- `async` — returns `Deferred<T>`, get result with `.await()`

```kotlin
viewModelScope.launch {

    // launch — don't need result
    launch { repository.syncWatchHistory() }

    // async — need result, run in parallel
    val moviesDeferred  = async { repository.getMovies() }
    val profileDeferred = async { repository.getUserProfile() }

    // Wait for both — total time = max(movies, profile), not sum
    val movies  = moviesDeferred.await()
    val profile = profileDeferred.await()

    _uiState.value = UiState.Success(movies, profile)
}
```

-----

### Q14. How does exception handling work in coroutines?

**Answer:**

```kotlin
// 1. try-catch inside coroutine
viewModelScope.launch {
    try {
        val movies = repository.getMovies()
        _uiState.value = UiState.Success(movies)
    } catch (e: IOException) {
        _uiState.value = UiState.Error(e.message ?: "Network error")
    }
}

// 2. CoroutineExceptionHandler — for launch (not async)
val handler = CoroutineExceptionHandler { _, exception ->
    _uiState.value = UiState.Error(exception.message ?: "Error")
}
viewModelScope.launch(handler) {
    repository.getMovies()
}

// 3. async exceptions — thrown on .await()
val deferred = async { repository.getMovies() }
try {
    val movies = deferred.await()
} catch (e: Exception) {
    // handle here
}

// Note: CancellationException is NOT an error — never catch it broadly
try { } catch (e: Exception) {
    if (e is CancellationException) throw e  // always rethrow ✅
}
```

-----

### Q15. What are coroutine dispatchers and when do you use each?

**Answer:**

Dispatchers determine which thread(s) a coroutine runs on.

```kotlin
viewModelScope.launch {
    // Dispatchers.Main (default for viewModelScope) — UI thread
    _isLoading.value = true

    // Dispatchers.IO — network, database, file I/O
    val movies = withContext(Dispatchers.IO) {
        api.fetchMovies()        // network call
        dao.getMovies()          // database query
        file.readText()          // file read
    }

    // Dispatchers.Default — CPU-intensive work
    val sorted = withContext(Dispatchers.Default) {
        movies.sortedByDescending { it.rating }  // heavy computation
    }

    // Back on Main automatically
    _uiState.value = UiState.Success(sorted)
}
```

|Dispatcher  |Thread Pool     |Use For                     |
|------------|----------------|----------------------------|
|`Main`      |Single UI thread|UI updates, StateFlow writes|
|`IO`        |Up to 64 threads|Network, DB, File           |
|`Default`   |CPU core count  |Sorting, parsing, crypto    |
|`Unconfined`|Caller’s thread |Testing only                |

-----

### Q16. What is cooperative cancellation in coroutines?

**Answer:**

Coroutines don’t get killed forcefully — they check for cancellation at suspension points cooperatively.

```kotlin
// Suspension points automatically check cancellation
viewModelScope.launch {
    val movies = repository.getMovies()  // ← checks cancellation here
    processMovies(movies)                // ← not reached if cancelled
}

// CPU-heavy work has no suspension points — must check manually
viewModelScope.launch {
    for (i in 1..1_000_000) {
        ensureActive()           // throws CancellationException if cancelled ✅
        // or
        if (!isActive) return@launch  // softer check
        heavyComputation(i)
    }
}
```

-----

### Q17. Explain `withContext` vs `launch` vs `async`.

**Answer:**

```kotlin
// withContext — switch context, SEQUENTIAL, returns result
// Use when: you need a result and want to switch threads
val movies = withContext(Dispatchers.IO) {
    api.fetchMovies()  // runs on IO, suspends until done, returns result
}
// continues after movies is ready

// launch — CONCURRENT, no result, fire and forget
// Use when: you don't need the result
launch { repository.syncData() }
// continues immediately, sync runs in background

// async — CONCURRENT, returns Deferred result
// Use when: you need the result AND want parallel execution
val deferred = async { api.fetchMovies() }
val movies = deferred.await()  // suspends until ready
```

-----

### Q18. What is `SupervisorJob` and when would you use it?

**Answer:**

With a regular `Job`, if one child fails, all siblings are cancelled. `SupervisorJob` isolates failures — one child’s failure doesn’t affect others.

```kotlin
// Regular Job — one failure cancels all siblings
val scope = CoroutineScope(Job() + Dispatchers.IO)
scope.launch { fetchMovies() }   // if this throws...
scope.launch { fetchProfile() }  // ...this gets cancelled too ❌

// SupervisorJob — failures are isolated
val supervisorScope = CoroutineScope(SupervisorJob() + Dispatchers.IO)
supervisorScope.launch { fetchMovies() }   // if this throws...
supervisorScope.launch { fetchProfile() }  // ...this continues ✅

// viewModelScope uses SupervisorJob internally — that's why
// one failing coroutine doesn't crash your entire ViewModel
```

**Use case:** Loading multiple independent sections of a screen — one failing shouldn’t prevent others from loading.

-----

## 🟪 Section 3: Flow (Q19–Q26)

-----

### Q19. What is the difference between cold and hot Flow?

**Answer:**

**Cold Flow** — lazy, starts only when collected, each collector gets its own independent stream.

```kotlin
val coldFlow = flow {
    println("Starting...")  // only runs when collected
    emit(1); emit(2); emit(3)
}
coldFlow.collect { }  // starts now
coldFlow.collect { }  // starts again independently
```

**Hot Flow** — always running, collectors tune in to the current stream.

```kotlin
val hotFlow = MutableStateFlow(0)  // running immediately
hotFlow.value = 1
hotFlow.value = 2
hotFlow.collect { }  // gets 2 and future values only
```

|                   |Cold Flow              |Hot Flow                 |
|-------------------|-----------------------|-------------------------|
|Starts             |When collected         |Immediately              |
|Multiple collectors|Independent streams    |Share same stream        |
|Examples           |`flow {}`, Room queries|`StateFlow`, `SharedFlow`|

-----

### Q20. Explain `StateFlow` vs `SharedFlow` vs `LiveData`.

**Answer:**

```kotlin
// StateFlow — current UI state
// Always has value, replays latest to new collectors
private val _uiState = MutableStateFlow<UiState>(UiState.Loading)
val uiState: StateFlow<UiState> = _uiState.asStateFlow()

// SharedFlow — one-time events
// No initial value, configurable replay
private val _events = MutableSharedFlow<UiEvent>()
val events: SharedFlow<UiEvent> = _events.asSharedFlow()

// LiveData — legacy, Android-specific
// Auto lifecycle-aware but less powerful than Flow
val movies: LiveData<List<Movie>> = repository.getMovies().asLiveData()
```

|               |StateFlow   |SharedFlow  |LiveData    |
|---------------|------------|------------|------------|
|Initial value  |Required    |Not required|Not required|
|Replay         |Latest value|Configurable|Latest value|
|Lifecycle aware|❌ Manual    |❌ Manual    |✅ Auto      |
|Kotlin native  |✅           |✅           |❌           |
|Best for       |UI state    |Events      |Legacy code |

-----

### Q21. What are the most important Flow operators and when do you use them?

**Answer:**

```kotlin
// Transform
.map { it.toUiModel() }                    // transform each emission
.flatMapLatest { query -> search(query) }  // cancel prev, start new (search)
.flatMapMerge { id -> getDetails(id) }     // parallel execution
.flatMapConcat { id -> getDetails(id) }    // sequential execution

// Filter
.filter { it.isReleased }                  // only matching items
.distinctUntilChanged()                    // skip consecutive duplicates
.filterNotNull()                           // skip nulls
.take(5)                                   // first 5 only
.drop(1)                                   // skip first emission

// Timing
.debounce(300)                             // wait for silence (search bars)
.sample(1000)                              // emit every 1s (location)

// Combine
.combine(otherFlow) { a, b -> ... }        // react to either updating
.zip(otherFlow) { a, b -> ... }            // pair by pair
merge(flow1, flow2)                        // merge into one stream

// Error handling
.catch { e -> emit(fallback) }             // catch upstream errors
.retry(3)                                  // retry on failure
.retryWhen { cause, attempt -> ... }       // retry with custom logic

// Side effects
.onEach { log(it) }                        // side effect without transforming
.onStart { emit(cached) }                  // emit before first value
.onCompletion { }                          // runs when flow completes
```

-----

### Q22. How do you safely collect Flow in a Fragment?

**Answer:**

```kotlin
// ❌ Wrong — collects even in background, wastes resources
lifecycleScope.launch {
    viewModel.uiState.collect { render(it) }
}

// ❌ Deprecated
lifecycleScope.launchWhenStarted {
    viewModel.uiState.collect { render(it) }
}

// ✅ Correct — pauses in background, resumes in foreground
lifecycleScope.launch {
    repeatOnLifecycle(Lifecycle.State.STARTED) {
        viewModel.uiState.collect { state ->
            render(state)
        }
    }
}

// ✅ Multiple flows — collect simultaneously
lifecycleScope.launch {
    repeatOnLifecycle(Lifecycle.State.STARTED) {
        launch { viewModel.uiState.collect { render(it) } }
        launch { viewModel.events.collect { handle(it) } }
    }
}
```

-----

### Q23. What is `stateIn` and `shareIn`? When do you use each?

**Answer:**

Both convert a cold Flow into a hot Flow in the ViewModel.

```kotlin
// stateIn → converts to StateFlow (requires initial value)
val movies: StateFlow<List<Movie>> = repository.getMovies()
    .stateIn(
        scope = viewModelScope,
        started = SharingStarted.WhileSubscribed(5000), // stays alive 5s after last collector
        initialValue = emptyList()
    )

// shareIn → converts to SharedFlow (no initial value required)
val events: SharedFlow<Event> = repository.getEvents()
    .shareIn(
        scope = viewModelScope,
        started = SharingStarted.WhileSubscribed(5000),
        replay = 1
    )
```

**SharingStarted options:**

- `Eagerly` — starts immediately, never stops
- `Lazily` — starts on first collector, never stops
- `WhileSubscribed(5000)` — starts on first collector, stops 5s after last leaves ✅ (handles rotation)

-----

### Q24. How would you implement a real-time search with Flow?

**Answer:**

```kotlin
class SearchViewModel(
    private val searchUseCase: SearchMoviesUseCase
) : ViewModel() {

    private val _query = MutableStateFlow("")
    val query: StateFlow<String> = _query.asStateFlow()

    val searchResults: StateFlow<UiState> = _query
        .debounce(300)
        .map { it.trim() }
        .distinctUntilChanged()
        .filter { it.isNotEmpty() }
        .flatMapLatest { query ->
            flow {
                emit(UiState.Loading)
                emit(UiState.Success(searchUseCase(query)))
            }.catch {
                emit(UiState.Error("Search failed"))
            }
        }
        .stateIn(viewModelScope, SharingStarted.WhileSubscribed(5000), UiState.Idle)

    fun onQueryChanged(query: String) {
        _query.value = query
    }
}
```

-----

### Q25. What is backpressure in Flow and how do you handle it?

**Answer:**

Backpressure occurs when the producer emits faster than the consumer can process.

```kotlin
val fastProducer = flow {
    repeat(1000) { emit(it) }  // emits very fast
}

// buffer() — queue up emissions
fastProducer
    .buffer(capacity = 64)      // buffer up to 64 items
    .collect { process(it) }    // consumer can be slow

// conflate() — drop intermediate values, keep latest
fastProducer
    .conflate()                 // if consumer is busy, skip intermediate
    .collect { process(it) }

// collectLatest — cancel current processing when new value arrives
fastProducer
    .collectLatest { value ->
        delay(100)              // slow processing
        process(value)          // cancelled if new value arrives during delay
    }
```

**Real use case:** `collectLatest` for search — cancel rendering previous results if new search comes in.

-----

### Q26. How do you combine multiple Flows and what is the difference between `zip` and `combine`?

**Answer:**

```kotlin
val moviesFlow = repository.getMovies()       // emits: [M1,M2] → [M1,M2,M3]
val ratingsFlow = repository.getRatings()     // emits: [R1,R2] → [R1,R2,R3]

// zip — pairs emissions one-by-one, waits for BOTH
moviesFlow.zip(ratingsFlow) { movies, ratings ->
    movies.mapIndexed { i, movie -> movie.copy(rating = ratings[i]) }
}
// Only emits when BOTH emit a new value

// combine — uses latest from each, fires when EITHER updates
combine(moviesFlow, ratingsFlow) { movies, ratings ->
    movies.map { movie -> movie.copy(rating = ratings.find { it.id == movie.id }?.value ?: 0.0) }
}
// Fires when moviesFlow OR ratingsFlow emits, using latest of the other

// merge — combines streams, no pairing
merge(moviesFlow, seriesFlow)  // any emission from either comes out
```

-----

## 🟨 Section 4: MVVM + Clean Architecture (Q27–Q34)

-----

### Q27. Explain Clean Architecture and its layers in Android.

**Answer:**

Clean Architecture separates code into 3 layers with dependencies pointing inward:

```
Presentation → Domain ← Data
```

**Domain Layer (innermost — pure Kotlin, no Android)**

- Entities — business models
- Repository interfaces — contracts
- UseCases — business logic

**Data Layer**

- Repository implementations
- API (Retrofit DTOs)
- Local DB (Room entities)
- Mappers

**Presentation Layer**

- ViewModel
- UI State
- Composables / Fragments

```kotlin
// Domain — pure Kotlin
data class Movie(val id: Int, val title: String, val rating: Double)

interface MovieRepository {
    fun getMovies(): Flow<List<Movie>>
}

class GetTopRatedMoviesUseCase(private val repo: MovieRepository) {
    operator fun invoke(): Flow<List<Movie>> =
        repo.getMovies().map { it.sortedByDescending { m -> m.rating }.take(10) }
}

// Data — implements domain contract
class MovieRepositoryImpl(private val api: MovieApi, private val dao: MovieDao) : MovieRepository {
    override fun getMovies(): Flow<List<Movie>> = flow {
        emit(dao.getMovies().map { it.toDomain() })
        val fresh = api.getMovies().map { it.toDomain() }
        dao.save(fresh.map { it.toEntity() })
        emit(fresh)
    }
}

// Presentation — uses domain
class MovieViewModel(private val getTopRated: GetTopRatedMoviesUseCase) : ViewModel() {
    val movies = getTopRated()
        .stateIn(viewModelScope, SharingStarted.WhileSubscribed(5000), emptyList())
}
```

-----

### Q28. What is the Single Responsibility Principle and how does it apply to Android?

**Answer:**

Every class should have one reason to change — one job.

```kotlin
// ❌ Violates SRP — Fragment does everything
class MovieFragment : Fragment() {
    fun onCreate() {
        val movies = api.getMovies()   // networking
        db.save(movies)                // persistence
        adapter.submit(movies)         // UI
        analytics.log("viewed")       // analytics
    }
}

// ✅ SRP applied — each class has one job
class MovieFragment : Fragment() {         // renders UI
    fun render(state: UiState) { }
}
class MovieViewModel : ViewModel() {       // manages UI state
    val uiState: StateFlow<UiState>
}
class GetMoviesUseCase {                   // business logic
    operator fun invoke(): Flow<List<Movie>>
}
class MovieRepositoryImpl : MovieRepository {  // data access
    fun getMovies(): Flow<List<Movie>>
}
```

-----

### Q29. How do you handle one-time UI events (navigation, toast) in MVVM?

**Answer:**

```kotlin
// ❌ Using StateFlow — replays on rotation, shows toast twice
private val _message = MutableStateFlow<String?>(null)

// ✅ Using SharedFlow — one-time, no replay
sealed class UiEvent {
    data class ShowToast(val message: String) : UiEvent()
    data class Navigate(val destination: String) : UiEvent()
}

class MovieViewModel : ViewModel() {
    private val _events = MutableSharedFlow<UiEvent>()
    val events: SharedFlow<UiEvent> = _events.asSharedFlow()

    fun onPurchaseComplete() {
        viewModelScope.launch {
            _events.emit(UiEvent.ShowToast("Purchase successful!"))
            _events.emit(UiEvent.Navigate("home"))
        }
    }
}

// In Fragment/Composable
lifecycleScope.launch {
    repeatOnLifecycle(Lifecycle.State.STARTED) {
        viewModel.events.collect { event ->
            when (event) {
                is UiEvent.ShowToast -> toast(event.message)
                is UiEvent.Navigate  -> navigate(event.destination)
            }
        }
    }
}
```

-----

### Q30. What is Dependency Injection and how does Hilt implement it?

**Answer:**

DI is a pattern where dependencies are provided to a class rather than created inside it. Hilt is Google’s DI framework built on Dagger.

```kotlin
// Without DI — tightly coupled, hard to test
class MovieViewModel {
    private val repo = MovieRepositoryImpl(  // creates its own dependency ❌
        RetrofitClient.create(),
        Room.databaseBuilder(...).build().movieDao()
    )
}

// With Hilt — dependencies injected ✅
@HiltViewModel
class MovieViewModel @Inject constructor(
    private val getMoviesUseCase: GetMoviesUseCase  // injected
) : ViewModel()

// Hilt Module — tells Hilt how to provide dependencies
@Module
@InstallIn(SingletonComponent::class)
object AppModule {

    @Provides @Singleton
    fun provideDatabase(@ApplicationContext context: Context): AppDatabase =
        Room.databaseBuilder(context, AppDatabase::class.java, "app_db").build()

    @Provides
    fun provideMovieDao(db: AppDatabase): MovieDao = db.movieDao()

    @Provides @Singleton
    fun provideMovieRepository(api: MovieApi, dao: MovieDao): MovieRepository =
        MovieRepositoryImpl(api, dao)
}
```

-----

### Q31. How do you unit test a ViewModel?

**Answer:**

```kotlin
// Dependencies
testImplementation "org.jetbrains.kotlinx:kotlinx-coroutines-test"
testImplementation "app.cash.turbine:turbine"  // Flow testing

@OptIn(ExperimentalCoroutinesApi::class)
class MovieViewModelTest {

    // Replace Main dispatcher for testing
    @get:Rule
    val mainDispatcherRule = MainDispatcherRule()

    private val mockUseCase = mockk<GetMoviesUseCase>()
    private lateinit var viewModel: MovieViewModel

    @Before
    fun setUp() {
        every { mockUseCase() } returns flowOf(listOf(Movie(1, "Batman", 8.5)))
        viewModel = MovieViewModel(mockUseCase)
    }

    @Test
    fun `movies load successfully`() = runTest {
        viewModel.uiState.test {
            val state = awaitItem()
            assert(state is UiState.Success)
            assert((state as UiState.Success).movies.size == 1)
        }
    }
}
```

-----

### Q32. What is the Repository pattern and why is it important?

**Answer:**

The Repository pattern abstracts data sources behind a single interface, giving the rest of the app a clean API regardless of where data comes from.

```kotlin
// Repository interface — Domain layer
interface MovieRepository {
    fun getMovies(): Flow<List<Movie>>
    suspend fun refreshMovies()
    suspend fun toggleFavourite(movieId: Int)
}

// Implementation — Data layer
class MovieRepositoryImpl(
    private val api: MovieApi,
    private val dao: MovieDao,
    private val mapper: MovieMapper
) : MovieRepository {

    // Single Source of Truth — Room drives the UI
    override fun getMovies(): Flow<List<Movie>> =
        dao.getAllMovies().map { it.map(mapper::toDomain) }

    override suspend fun refreshMovies() {
        val fresh = api.getMovies()
        dao.insertAll(fresh.map(mapper::toEntity))
    }
}
```

**Benefits:**

- Swap data sources without changing business logic
- Easy to mock in tests
- Centralises caching strategy
- Enforces Single Source of Truth

-----

### Q33. How do you handle loading, success, and error states in MVVM?

**Answer:**

```kotlin
// UiState sealed class
sealed class UiState<out T> {
    object Loading : UiState<Nothing>()
    data class Success<T>(val data: T) : UiState<T>()
    data class Error(val message: String) : UiState<Nothing>()
}

// ViewModel
class MovieViewModel(private val useCase: GetMoviesUseCase) : ViewModel() {

    private val _uiState = MutableStateFlow<UiState<List<Movie>>>(UiState.Loading)
    val uiState: StateFlow<UiState<List<Movie>>> = _uiState.asStateFlow()

    init {
        viewModelScope.launch {
            useCase()
                .onStart { _uiState.value = UiState.Loading }
                .catch { _uiState.value = UiState.Error(it.message ?: "Error") }
                .collect { _uiState.value = UiState.Success(it) }
        }
    }
}

// Compose UI
@Composable
fun MovieScreen(viewModel: MovieViewModel = hiltViewModel()) {
    val state by viewModel.uiState.collectAsStateWithLifecycle()

    when (state) {
        is UiState.Loading        -> CircularProgressIndicator()
        is UiState.Success        -> MovieList(state.data)
        is UiState.Error          -> ErrorMessage(state.message)
    }
}
```

-----

### Q34. What is the difference between a UseCase and a Repository?

**Answer:**

|              |UseCase                   |Repository                |
|--------------|--------------------------|--------------------------|
|Layer         |Domain                    |Data (interface in Domain)|
|Responsibility|Business logic            |Data access               |
|Knows about   |Repository interface      |API, Database             |
|Example       |`GetTopRatedMoviesUseCase`|`MovieRepositoryImpl`     |

```kotlin
// Repository — HOW to get data
interface MovieRepository {
    fun getMovies(): Flow<List<Movie>>
}

// UseCase — WHAT to do with data (business logic)
class GetTopRatedMoviesUseCase(private val repository: MovieRepository) {
    operator fun invoke(): Flow<List<Movie>> =
        repository.getMovies()
            .map { movies ->
                movies
                    .filter { it.isReleased && it.rating >= 7.0 }  // business rule
                    .sortedByDescending { it.rating }               // business rule
                    .take(10)                                        // business rule
            }
}
```

-----

## 🟧 Section 5: Jetpack Compose (Q35–Q42)

-----

### Q35. What is declarative UI and how does Compose implement it?

**Answer:**

**Imperative (XML + View system):** You manually update every UI element when state changes.

**Declarative (Compose):** You describe what the UI should look like for a given state. Compose automatically updates the UI when state changes.

```kotlin
// Imperative — you manage every change manually
if (isLoading) {
    progressBar.visibility = View.VISIBLE
    recyclerView.visibility = View.GONE
}

// Declarative — describe the UI for each state
@Composable
fun MovieScreen(isLoading: Boolean, movies: List<Movie>) {
    if (isLoading) {
        CircularProgressIndicator()  // Compose handles showing/hiding
    } else {
        LazyColumn {
            items(movies) { MovieCard(it) }
        }
    }
}
// When isLoading changes → Compose automatically recomposes ✅
```

-----

### Q36. What is recomposition and how does Compose minimise it?

**Answer:**

Recomposition is the process of re-running composable functions when their state changes. Compose is smart — it only recomposes the parts that read the changed state.

```kotlin
@Composable
fun MovieScreen() {
    var isLoading by remember { mutableStateOf(true) }
    var movies by remember { mutableStateOf(emptyList<Movie>()) }

    // Only THIS recomposes when isLoading changes
    if (isLoading) LoadingSpinner()

    // Only THIS recomposes when movies changes
    MovieList(movies)

    // Header never recomposes — it doesn't read any changing state
    Header(title = "Movies")
}
```

**How to minimise unnecessary recomposition:**

- Use `remember` to preserve state across recompositions
- Use `derivedStateOf` for computed values
- Pass only the data a composable needs (not entire objects)
- Use `key()` in lists for stable identity

-----

### Q37. Explain state hoisting with a practical example.

**Answer:**

State hoisting moves state up to the caller, making composables stateless, reusable, and testable.

```kotlin
// ❌ Stateful — state trapped inside, not reusable
@Composable
fun SearchBar() {
    var query by remember { mutableStateOf("") }
    TextField(value = query, onValueChange = { query = it })
}

// ✅ Stateless (hoisted) — reusable, testable, previewable
@Composable
fun SearchBar(
    query: String,                      // state flows down
    onQueryChanged: (String) -> Unit    // events flow up
) {
    TextField(value = query, onValueChange = onQueryChanged)
}

// Parent owns the state
@Composable
fun MovieScreen(viewModel: MovieViewModel = hiltViewModel()) {
    val query by viewModel.query.collectAsStateWithLifecycle()

    SearchBar(
        query = query,
        onQueryChanged = viewModel::onQueryChanged  // event goes to VM
    )
}
```

**Rule:** State flows **down**, events flow **up**.

-----

### Q38. What are the side effect APIs in Compose? Explain each.

**Answer:**

```kotlin
// LaunchedEffect — run a coroutine when key changes, cancelled on recomposition
@Composable
fun MovieDetailScreen(movieId: Int, viewModel: MovieViewModel = hiltViewModel()) {
    LaunchedEffect(movieId) {
        viewModel.loadMovie(movieId)  // re-runs if movieId changes
    }
}

// DisposableEffect — setup + cleanup, runs when key changes
@Composable
fun VideoPlayer(viewModel: PlayerViewModel = hiltViewModel()) {
    val lifecycleOwner = LocalLifecycleOwner.current
    DisposableEffect(lifecycleOwner) {
        val observer = LifecycleEventObserver { _, event ->
            when (event) {
                Lifecycle.Event.ON_PAUSE  -> viewModel.exoPlayer.pause()
                Lifecycle.Event.ON_RESUME -> viewModel.exoPlayer.play()
                else -> Unit
            }
        }
        lifecycleOwner.lifecycle.addObserver(observer)
        onDispose {
            lifecycleOwner.lifecycle.removeObserver(observer)  // cleanup ✅
        }
    }
}

// SideEffect — runs after every successful recomposition
@Composable
fun AnalyticsScreen(screenName: String) {
    SideEffect {
        analytics.logScreenView(screenName)  // sync with non-Compose code
    }
}

// rememberCoroutineScope — get scope tied to composable lifetime
@Composable
fun MovieScreen() {
    val scope = rememberCoroutineScope()
    Button(onClick = {
        scope.launch { viewModel.refresh() }  // launch from non-composable callback
    }) { Text("Refresh") }
}
```

-----

### Q39. How do you connect a ViewModel to a Compose screen?

**Answer:**

```kotlin
@Composable
fun MovieScreen(
    viewModel: MovieViewModel = hiltViewModel()  // Hilt provides the VM
) {
    // Collect StateFlow as Compose State, lifecycle-aware
    val uiState by viewModel.uiState.collectAsStateWithLifecycle()
    val query by viewModel.query.collectAsStateWithLifecycle()

    // Handle one-time events
    val context = LocalContext.current
    LaunchedEffect(Unit) {
        viewModel.events.collect { event ->
            when (event) {
                is UiEvent.ShowToast -> Toast.makeText(context, event.message, Toast.LENGTH_SHORT).show()
                is UiEvent.Navigate  -> { /* handle navigation */ }
            }
        }
    }

    // Render based on state
    Column {
        SearchBar(query = query, onQueryChanged = viewModel::onQueryChanged)
        when (uiState) {
            is UiState.Loading -> CircularProgressIndicator()
            is UiState.Success -> MovieList(movies = uiState.movies, onMovieClick = viewModel::onMovieClicked)
            is UiState.Error   -> ErrorView(message = uiState.message, onRetry = viewModel::retry)
        }
    }
}
```

-----

### Q40. What is the difference between `remember` and `rememberSaveable`?

**Answer:**

```kotlin
// remember — survives recomposition, lost on configuration change
var count by remember { mutableStateOf(0) }
// User rotates screen → count resets to 0 ❌

// rememberSaveable — survives recomposition AND configuration changes
var count by rememberSaveable { mutableStateOf(0) }
// User rotates screen → count preserved ✅

// rememberSaveable with custom saver (for complex types)
var movie by rememberSaveable(stateSaver = MovieSaver) { mutableStateOf(defaultMovie) }
```

**Rule:** Use `rememberSaveable` for any UI state that should survive rotation (scroll position, form input, selected tab).

-----

### Q41. How does Compose handle lists efficiently?

**Answer:**

```kotlin
// ❌ Column + forEach — all items composed at once, no recycling
Column {
    movies.forEach { movie ->
        MovieCard(movie)  // ALL 10,000 items composed immediately 😱
    }
}

// ✅ LazyColumn — only visible items composed (like RecyclerView)
LazyColumn {
    items(
        items = movies,
        key = { it.id }  // stable key prevents unnecessary recompositions
    ) { movie ->
        MovieCard(movie)  // only visible items ✅
    }
}

// LazyVerticalGrid — 2-column grid
LazyVerticalGrid(columns = GridCells.Fixed(2)) {
    items(movies, key = { it.id }) { MovieCard(it) }
}

// With headers and sections
LazyColumn {
    stickyHeader { GenreHeader("Action") }
    items(actionMovies, key = { it.id }) { MovieCard(it) }
    stickyHeader { GenreHeader("Drama") }
    items(dramaMovies, key = { it.id }) { MovieCard(it) }
}
```

-----

### Q42. What are Modifiers in Compose and why does order matter?

**Answer:**

Modifiers decorate composables with layout, drawing, and interaction behaviour.

```kotlin
@Composable
fun MovieCard(movie: Movie, onClick: () -> Unit) {
    Card(
        modifier = Modifier
            .fillMaxWidth()
            .height(120.dp)
            .padding(horizontal = 16.dp, vertical = 8.dp)
            .clip(RoundedCornerShape(12.dp))
            .clickable { onClick() }
    ) { }
}

// Order matters — modifiers apply sequentially
// padding then background — padding area has no background
Modifier.padding(16.dp).background(Color.Red)

// background then padding — padding is inside the background
Modifier.background(Color.Red).padding(16.dp)

// clickable before padding — tap target is small
Modifier.clickable { }.padding(16.dp)

// padding before clickable — tap target includes padding ✅
Modifier.padding(16.dp).clickable { }
```

-----

## 🟥 Section 6: Room + DataStore + Navigation (Q43–Q47)

-----

### Q43. How does Room work with Flow for reactive UI updates?

**Answer:**

```kotlin
// DAO returns Flow — Room automatically notifies on data change
@Dao
interface MovieDao {
    @Query("SELECT * FROM movies ORDER BY rating DESC")
    fun getAllMovies(): Flow<List<MovieEntity>>  // reactive ✅

    @Insert(onConflict = OnConflictStrategy.REPLACE)
    suspend fun insertMovies(movies: List<MovieEntity>)
}

// Repository
class MovieRepository(private val dao: MovieDao, private val api: MovieApi) {

    // Single Source of Truth — Room drives the UI
    fun getMovies(): Flow<List<Movie>> =
        dao.getAllMovies().map { it.map(MovieMapper::toDomain) }

    suspend fun refresh() {
        val fresh = api.getMovies()
        dao.insertMovies(fresh.map(MovieMapper::toEntity))
        // Room Flow auto-emits updated list → ViewModel updates → UI recomposes ✅
    }
}
```

-----

### Q44. How do you handle Room database migrations?

**Answer:**

```kotlin
// 1. Increment version
@Database(entities = [MovieEntity::class], version = 2)  // was 1
abstract class AppDatabase : RoomDatabase()

// 2. Define migration
val MIGRATION_1_2 = object : Migration(1, 2) {
    override fun migrate(database: SupportSQLiteDatabase) {
        // Add new column with default value
        database.execSQL("ALTER TABLE movies ADD COLUMN release_year INTEGER NOT NULL DEFAULT 0")
        // Or create new table
        database.execSQL("CREATE TABLE IF NOT EXISTS genres (id INTEGER PRIMARY KEY, name TEXT NOT NULL)")
    }
}

// 3. Register migration
Room.databaseBuilder(context, AppDatabase::class.java, "movies_db")
    .addMigrations(MIGRATION_1_2)
    .build()

// ⚠️ Never use in production — wipes all user data
.fallbackToDestructiveMigration()
```

-----

### Q45. When would you use DataStore over Room?

**Answer:**

```kotlin
// DataStore — for simple key-value preferences
// Auth token, theme, language, feature flags, user settings
object PreferencesKeys {
    val AUTH_TOKEN   = stringPreferencesKey("auth_token")
    val DARK_MODE    = booleanPreferencesKey("dark_mode")
    val LANGUAGE     = stringPreferencesKey("language")
}

suspend fun saveAuthToken(token: String) {
    dataStore.edit { it[PreferencesKeys.AUTH_TOKEN] = token }
}

val authToken: Flow<String?> = dataStore.data.map { it[PreferencesKeys.AUTH_TOKEN] }

// Room — for structured, relational, queryable data
// Movies, episodes, watch history, user profiles, search history
@Entity(tableName = "watch_history")
data class WatchHistoryEntity(
    @PrimaryKey val episodeId: Int,
    val showId: Int,
    val watchedAt: Long,
    val progressSeconds: Long
)
```

**Rule:** Simple fixed set of preferences → DataStore. Growing, structured, queryable data → Room.

-----

### Q46. How do you pass data between screens in Compose Navigation?

**Answer:**

```kotlin
// Define routes with arguments
sealed class Screen(val route: String) {
    object MovieList : Screen("movie_list")
    object MovieDetail : Screen("movie_detail/{movieId}") {
        fun createRoute(movieId: Int) = "movie_detail/$movieId"
    }
}

// NavHost setup
NavHost(navController, startDestination = Screen.MovieList.route) {

    composable(Screen.MovieList.route) {
        MovieListScreen(
            onMovieClick = { movieId ->
                navController.navigate(Screen.MovieDetail.createRoute(movieId))
            }
        )
    }

    composable(
        route = Screen.MovieDetail.route,
        arguments = listOf(navArgument("movieId") { type = NavType.IntType })
    ) { backStackEntry ->
        val movieId = backStackEntry.arguments?.getInt("movieId") ?: return@composable
        MovieDetailScreen(movieId = movieId)
    }
}

// Best practice: pass only IDs, fetch full data in destination ViewModel
@HiltViewModel
class MovieDetailViewModel @Inject constructor(
    savedStateHandle: SavedStateHandle,  // access nav args safely
    private val getMovieUseCase: GetMovieUseCase
) : ViewModel() {
    private val movieId: Int = checkNotNull(savedStateHandle["movieId"])
    val movie = getMovieUseCase(movieId).stateIn(viewModelScope, SharingStarted.WhileSubscribed(5000), null)
}
```

-----

### Q47. How do you manage the back stack with bottom navigation in Compose?

**Answer:**

```kotlin
@Composable
fun MainScreen() {
    val navController = rememberNavController()
    val currentRoute by navController.currentBackStackEntryAsState()

    Scaffold(
        bottomBar = {
            NavigationBar {
                bottomNavItems.forEach { item ->
                    NavigationBarItem(
                        selected = currentRoute?.destination?.route == item.route,
                        onClick = {
                            navController.navigate(item.route) {
                                // Pop to start destination — avoid building up large back stack
                                popUpTo(navController.graph.startDestinationId) {
                                    saveState = true      // save tab state
                                }
                                launchSingleTop = true    // avoid duplicate destinations
                                restoreState = true       // restore tab state when switching back
                            }
                        },
                        icon = { Icon(item.icon, contentDescription = item.label) },
                        label = { Text(item.label) }
                    )
                }
            }
        }
    ) { padding ->
        NavHost(navController, startDestination = "home", modifier = Modifier.padding(padding)) {
            composable("home") { HomeScreen() }
            composable("search") { SearchScreen() }
            composable("profile") { ProfileScreen() }
        }
    }
}
```

-----

## 🔴 Section 7: DSA (Q48–Q50)

-----

### Q48. Given a list of movies, find the two movies whose combined rating is closest to a target value.

**Answer:**

Use the **two-pointer technique** after sorting.

```kotlin
fun findMoviePair(movies: List<Movie>, target: Double): Pair<Movie, Movie>? {
    if (movies.size < 2) return null

    val sorted = movies.sortedBy { it.rating }
    var left = 0
    var right = sorted.size - 1
    var bestPair: Pair<Movie, Movie>? = null
    var bestDiff = Double.MAX_VALUE

    while (left < right) {
        val sum = sorted[left].rating + sorted[right].rating
        val diff = Math.abs(sum - target)

        if (diff < bestDiff) {
            bestDiff = diff
            bestPair = Pair(sorted[left], sorted[right])
        }

        when {
            sum < target -> left++   // need higher sum
            sum > target -> right--  // need lower sum
            else -> return bestPair  // exact match
        }
    }
    return bestPair
}

// Time: O(n log n) for sort + O(n) for two pointers = O(n log n)
// Space: O(n) for sorted copy
```

-----

### Q49. Implement a function to find duplicate movie IDs in a list efficiently.

**Answer:**

Use a `HashSet` for O(n) time complexity.

```kotlin
// Find all duplicate IDs
fun findDuplicateMovieIds(movies: List<Movie>): Set<Int> {
    val seen = HashSet<Int>()
    val duplicates = HashSet<Int>()

    for (movie in movies) {
        if (!seen.add(movie.id)) {  // add returns false if already present
            duplicates.add(movie.id)
        }
    }
    return duplicates
}

// More idiomatic Kotlin
fun findDuplicates(movies: List<Movie>): List<Int> {
    return movies
        .groupBy { it.id }
        .filter { (_, group) -> group.size > 1 }
        .keys
        .toList()
}

// Usage
val movies = listOf(Movie(1, "Batman"), Movie(2, "Superman"), Movie(1, "Batman Returns"))
findDuplicateMovieIds(movies)  // [1]

// Time: O(n) — single pass through list
// Space: O(n) — HashSet storage
```

-----

### Q50. Given a list of episodes grouped by season, flatten and sort them by episode number. What data structures and algorithms would you use?

**Answer:**

```kotlin
data class Episode(val seasonNumber: Int, val episodeNumber: Int, val title: String)
data class Season(val number: Int, val episodes: List<Episode>)

// Flatten seasons → episodes, sort by season then episode
fun flattenAndSort(seasons: List<Season>): List<Episode> {
    return seasons
        .flatMap { it.episodes }                                    // flatten O(n)
        .sortedWith(compareBy({ it.seasonNumber }, { it.episodeNumber }))  // sort O(n log n)
}

// More memory-efficient using sequences (lazy evaluation)
fun flattenAndSortLazy(seasons: List<Season>): List<Episode> {
    return seasons
        .asSequence()
        .flatMap { it.episodes.asSequence() }
        .sortedWith(compareBy({ it.seasonNumber }, { it.episodeNumber }))
        .toList()
}

// If already sorted within each season — merge sort approach O(n log k)
// where k = number of seasons, n = total episodes
fun mergeSortedSeasons(seasons: List<Season>): List<Episode> {
    val result = mutableListOf<Episode>()
    val queue = PriorityQueue<Pair<Episode, Int>>(
        compareBy({ it.first.seasonNumber }, { it.first.episodeNumber })
    )

    // Add first episode from each season
    seasons.forEachIndexed { idx, season ->
        season.episodes.firstOrNull()?.let { queue.add(Pair(it, idx)) }
    }

    val iterators = seasons.map { it.episodes.iterator() }

    while (queue.isNotEmpty()) {
        val (episode, seasonIdx) = queue.poll()
        result.add(episode)
        if (iterators[seasonIdx].hasNext()) {
            queue.add(Pair(iterators[seasonIdx].next(), seasonIdx))
        }
    }
    return result
}

// Time: O(n log n) for sort, O(n log k) for merge approach
// Space: O(n) for result list
```

-----

## ⚡ Quick Reference Cheat Sheet

```
Kotlin
─────────────────────────────────────────────
val/var/const val        → immutable/mutable/compile-time
data class               → equals, hashCode, toString, copy, componentN
sealed class             → restricted hierarchy, exhaustive when
extension function       → add to existing class, no inheritance
?.  ?:  !!  let          → null safety operators
inline + reified         → avoid lambda overhead + runtime generics
== vs ===                → structural vs referential equality

Coroutines
─────────────────────────────────────────────
suspend fun              → single async result, non-blocking
launch                   → fire and forget, Job
async/await              → parallel work, Deferred<T>
withContext              → switch dispatcher, sequential
viewModelScope           → tied to ViewModel, auto-cancelled
lifecycleScope           → tied to Activity/Fragment
SupervisorJob            → isolated child failures
ensureActive()           → manual cancellation check

Dispatchers
─────────────────────────────────────────────
Main                     → UI thread
IO                       → network, database, files
Default                  → CPU-heavy computation

Flow
─────────────────────────────────────────────
flow {}                  → cold, starts when collected
StateFlow                → hot, current value, UI state
SharedFlow               → hot, one-time events
debounce                 → wait for silence
distinctUntilChanged     → skip consecutive duplicates
flatMapLatest            → cancel previous, start new
combine                  → react to either flow updating
zip                      → pair by pair
catch                    → handle upstream errors
stateIn                  → cold → StateFlow in ViewModel
repeatOnLifecycle        → safe collection in Fragment

Architecture
─────────────────────────────────────────────
Presentation → Domain ← Data
UseCase      → business logic (Domain)
Repository   → data access (interface in Domain, impl in Data)
ViewModel    → state management, no Android context
Fragment     → render only, no logic

Compose
─────────────────────────────────────────────
@Composable              → UI function
mutableStateOf           → observable state
remember                 → survives recomposition
rememberSaveable         → survives rotation
State hoisting           → state down, events up
LazyColumn               → efficient large lists
LaunchedEffect           → one-time side effects
DisposableEffect         → setup + cleanup
collectAsStateWithLifecycle → safe Flow collection
```

-----

*Good luck with your Senior Android Developer interview! 💪🚀*