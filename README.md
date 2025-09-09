# Android Learnings

## Android Interview Resources

1. [PayPal Interview Experience - SDE 3](https://kanchan-pal.medium.com/paypal-interview-experience-sde-3-a015695d0eba)
2. [Mobile System Design](https://github.com/weeeBox/mobile-system-design)
3. [Additional Resources](https://gist.github.com/atierian/610538f39a4844881e20b673f4c8e8dc)

---

## Kotlin Fundamentals

### Companion Objects & Static Functions

1. **Companion objects** store static functions or classes
2. **Static functions** don't need initialization and are globally accessible
   - Normal variable: `class().var1`
   - Static variable: `class.var2` (globally scoped, not instance-based)

### Extension Functions

3. **`let{}` extension function** serves two purposes:
   - Null checking
   - Thread-safe operations (creates copy of current instance for specific thread)

---

## Pagination

### Types of Pagination

1. **Offset-based pagination**
   - Used for small data or low model complexity
   - Easy implementation with current offset/page tracking
   - Process: determine current offset → API call → add to DB → display on UI
   - Uses mediator for data exchange and caching for UI loading

2. **Cursor-based pagination**
   - Uses ID or key in requests to reduce load
   - ID types: timestamp-based, sequential-based, or other indicators
   - Timestamp: `newerThan` parameter returns data from last request
   - Sequential: request data after previous ID

---

## Memory Leaks

> 📚 Reference: [Everything about Memory Leaks in Android](https://proandroiddev.com/everything-you-need-to-know-about-memory-leaks-in-android-d7a59faaf46a)

### Memory Management Components

1. **Garbage Collector (JVM process)**
   - Frees up space and allocates memory to other processes

2. **Stack**
   - Data structure keeping static memory (part of RAM)
   - Tracks all app processes with LIFO policy
   - Stores references to objects allocated in heap

3. **Heap**
   - Dynamic memory system used by JVM
   - Allocates object classes referenced by stack

### Memory Leak Causes

Memory leaks occur when stack still refers to unused objects in heap.

**Common causes:**
- **Improper lifecycle management** of background workers
- **Wrong use of activity context** - use `getApplicationContext()` instead

### Memory Leak Consequences

- **App lags** - Increased GC load affects 60fps rendering
- **ANR (App Not Responsive)** - UI thread blocked
- **Out of Memory** - JVM can't allocate more objects to heap

---

## Notifications

### Why Device Tokens Change

- **Security Enhancement** - Periodic token rotation prevents unauthorized access
- **Privacy Protection** - Regular changes prevent long-term device tracking  
- **System Updates** - OS updates trigger token regeneration
- **Anti-Fraud Measures** - Prevents malicious reuse of old tokens
- **Platform Requirements** - Apple and Google mandate refresh cycles

### When Device Tokens Change

- App reinstallation
- OS updates
- App updates
- Device restore
- Long periods of inactivity
- Cloud account changes
- Push service reconnection
- Security events

### Best Practices

- Always monitor token changes
- Store tokens server-side
- Handle token refresh gracefully
- Regular token validation

---

## Dependency Injection

### Resources
- [Hilt Android Scoping](https://x.com/nagataro_san475/status/1928560079738917241)
- [Hilt Custom Components](https://medium.com/androiddevelopers/hilt-adding-components-to-the-hierarchy-96f207d6d92d)
- [More on Hilt Custom Components](https://medium.com/mindful-engineering/more-on-hilt-custom-components-custom-scope-f66c441c40c9)

### Component Hierarchy

```
🏠 Application (Lives for entire app lifetime)
    ↓
📱 Activity (Lives while screen is open)
    ↓  
🧩 Fragment (Lives while that piece of screen exists)
    ↓
👁️ View (Lives while that button/text exists)
```

### Key Concepts

- **Components** - Generated objects that act as containers with dependencies ready for injection
- **Entry points** - Gateways for non-Hilt code to access Dagger code
- **@Module** - Tells Dagger this class coordinates dependency injections when installed within component lifecycle scope

---

## State Hoisting in Compose

### Compose Types

1. **Stateful Compose** - Uses `remember` delegation, harder to test, may cause unnecessary recompositions
2. **Stateless Compose** - Controlled by caller (state hoisting), easier to test

### State Management

- **`remember`** - Inline function storing variable value during initial composition
- **`rememberSavable`** - Saves states across configuration changes
- **Compose is declarative UI** - Update UI based on different states instead of adding/changing

### State Hoisting Rules

1. State should be hoisted to at least the **lowest common parent** of all composables that use it (read)
2. State should be hoisted to at least the **highest level** it may be changed (write)  
3. If two states change in response to the same events, they should be **hoisted to the same level**

### Additional Concepts

- **`derivedStateOf`** - Prevents excessive recompositions for frequently changing states
- **Flow collection** - Most operations work on same dispatcher, use `flowOn` to switch for emission

---

## Observers

### Types of Observers

1. **LiveData** - Basic lifecycle-aware state holder
2. **StateFlow** - LiveData + Flow benefits, not lifecycle-aware
3. **Flow** - Observer that fetches data only when collected
4. **SharedFlow** - Emits all values, including consecutive repeated values

### SharedFlow Configuration

- **Replay** - Manages last events given to new subscribers
- **Buffer** - Temporary storage for data/model references
- **Overflow handling** - Remove oldest data when buffer is full

---

## Sequences

> 📚 Reference: [Kotlin Sequences](https://medium.com/android-news/kotlin-sequences-ac6dc7c883d3)

### When to Use
- Used for data in millions of order
- Lazy evaluators processing items one by one
- Avoid creation of temporary collections

### Functions
- **Intermediate** - Process one object, pass to terminal
- **Terminal** - Add to result

### Example Comparison

**Collections (creates intermediate lists):**
```kotlin
val list = (1..1_000_000).toList()
val result = list
    .filter { it % 2 == 0 }   // Intermediate list 1
    .map { it * 2 }           // Intermediate list 2
    .take(10)                 // Intermediate list 3
    .toList()
```

**Sequences (no intermediate lists):**
```kotlin
val sequence = (1..1_000_000).asSequence()
val result = sequence
    .filter { it % 2 == 0 }  // Intermediate
    .map { it * 2 }          // Intermediate
    .take(10)                // Intermediate
    .toList()                // Terminal
```

---

## Side Effects

### LaunchedEffect
- Triggered when key value changes
- Utilizes CoroutineScope for IO operations

### DisposableEffect
- Similar to LaunchedEffect but without CoroutineScope
- Used for lifecycle events
- `onDispose` is mandatory

---

## Flows

### Flow Characteristics
- Stream of data flowing through a pipe
- **Hot flows** - Active everytime
- **Cold flows** - Work only when collected
- Not lifecycle-aware
- Best for complex async coroutine calls

### Flow Components
- **Producer** - Emits data using `emit()`
- **Intermediaries** - Optional, can modify emitted data
- **Consumer** - Uses `collect()` to get flow output

### Resources
- [Consuming Flows Safely in Jetpack Compose](https://medium.com/androiddevelopers/consuming-flows-safely-in-jetpack-compose-cde014d0d5a3)
- [Building Efficient API Calls](https://medium.com/@devarshbhalara3072/building-efficient-api-calls-in-android-with-retrofit-coroutines-flow-and-dependency-injection-60e022b16987)

---

## Coroutines

> 📚 Reference: [Coroutines on Android Part I](https://medium.com/androiddevelopers/coroutines-on-android-part-i-getting-the-background-3e0e54d20bb)

---

## ViewModel

### Best Practices
- Use `job { ViewModelScope.launch() }` to cancel long-running background jobs
- Implement immutability pattern:

```kotlin
private val _isLoading = MutableLiveData<Boolean>()
val isLoading: LiveData<Boolean> = _isLoading
```

### Resources
- [CoroutineScope & CoroutineContext Deep Dive](https://medium.com/@paritasampa95/coroutinescope-coroutinecontext-discussed-in-depth-to-understand-better-part-2-550dbc7af3d2)
- [Additional ViewModel Resources](https://x.com/nagataro_san475/status/1875593012648276202)

---

## Workers Android

> 📚 Reference: [Work Manager Android](https://medium.com/@appdevinsights/work-manager-android-6ea8daad56ee)

### Use Cases
- Tasks that run even when app is killed
- Scheduled tasks without user interaction
- Examples: database sync, daily APIs

### Benefits
- Periodic workers
- Add constraints (low battery, network)
- Chain workers together
- Backoff policy for failed tasks
- Automatic threading and coroutines

---

## Suspend Coroutines

### Types
1. **suspendCoroutine & suspendCancellableCoroutine**
   - Callback-based coroutines
   - Return results based on success/failure callbacks
   - Pause → Wait for callback → Resume with result/exception

2. **CompletableDeferred**
   - Waits for function completion
   - Different from suspend coroutines
   - Used with legacy callback-based APIs

---

## BroadCasts

> 📚 Reference: [Understanding Broadcast Receivers](https://medium.com/@khush.panchal123/understanding-broadcast-receivers-in-android-044fbfaa1330)

### Types of Broadcasts

1. **Static/Manifest Broadcasts**
   - Defined in manifest
   - Can trigger `onCreate()` on application context

2. **Context/Dynamic Broadcasts**
   - Registered based on activity context
   - Must be unregistered when not needed

### Key Concepts
- Broadcasts sent/received as intents
- Can be made locally within app
- `RECEIVER_EXPORTED` flag required for custom broadcasts

### Pending Intents
- [Pending Intent in Android](https://medium.com/@appdevinsights/pending-intent-in-android-fc93a8b6c2ac)
- Given to Android system APIs to work upon and send data back

**Types:**
- `PendingIntent.getActivity()` - Start an Activity
- `PendingIntent.getBroadcast()` - Perform a Broadcast
- `PendingIntent.getService()` - Start a Service

### Example Implementation

```kotlin
// Receiver for SMS response
context.registerReceiver(object : BroadcastReceiver(){
    override fun onReceive(p0: Context?, intent: Intent?) {
        if (intent?.action == Telephony.Sms.Intents.SMS_RECEIVED_ACTION) {
            val messages = Telephony.Sms.Intents.getMessagesFromIntent(intent)
            for (sms in messages) {
                val sender = sms.originatingAddress
                if (sender?.replace("+", "")?.contains(phoneNumber.replace("+", "")) == true) {
                    onSuccess()
                }
            }
            context.unregisterReceiver(this)
        }
    }
}, IntentFilter(Telephony.Sms.Intents.SMS_RECEIVED_ACTION), Context.RECEIVER_NOT_EXPORTED)
```

---

## Database

### Room Database Relationships

> 📚 References:
> - [Room Nested Relationships](https://developer.android.com/static/images/training/data-storage/room_nested_relationships.png)
> - [Database Relations with Room](https://medium.com/androiddevelopers/database-relations-with-room-544ab95e4542)

### Database Migration

> 📚 Reference: [Understanding Migrations with Room](https://medium.com/androiddevelopers/understanding-migrations-with-room-f01e04b07929)

#### Why Migration is Necessary
- **Schema Evolution** - Update database structure as features change
- **Data Preservation** - Migrate existing user data to new schema
- **Performance** - Restructure database for better query performance

#### When onUpgrade() is Called
- When database version number in SQLiteOpenHelper changes
- When you increase version number in constructor
- When modifying database schema

#### Version Sources
- **Old Version** - From existing database file on user's device
- **New Version** - Defined in current code's database helper class

### Best Practices
- Keep single database per app with multiple tables
- For multi-user apps: create tables with `TABLE_NAME + user_id`
- Use `withTransaction` for atomic operations:

```kotlin
db.withTransaction {
    db.clear()
    db.insert() // If this fails, clear() is rolled back
}
```

### Transaction Rollback
SQLite maintains rollback journal with:
- **Before-images** - Original data before each change  
- **Undo operations** - Instructions to reverse each change

---

## Caching

### Types of Caching

1. **SharedPreferences** - Key-value pair storage
2. **Room/SQL Database** - Local database storage
3. **LRU Cache** - Least Recently Used cache

### Making Cache-like Behavior
Add conditions for data expiration:
- Time limits
- Size limits  
- Automatic cleanup
- Clear separation from persistent data

---

## Testing

> 📚 Reference: [Test Sizes](https://testing.googleblog.com/2010/12/test-sizes.html)

### Testing Categories

#### Remote DataSource Testing [Retrofit]
- Mockserver | Fakes | Mocks

#### Local DataSource Testing
- **Database [Room]** - Android X Test | Robolectric
- **Preferences [SharedPreferences]** - Android X Test | Robolectric  
- **DataStore** - Android X Test | Robolectric

#### Business Logic Testing
- **Repository Layer | UseCases** - JUnit 4/5 | Fakes | Mocks

#### UI & Integration Testing
- **ViewModel & LiveData** - Android X Test | Robolectric
- **Navigation** - Android X Test | Robolectric
- **Views & Animations** - Espresso | Barista Instrumentation
- **Notifications** - Instrumentation and ADB
- **Broadcasts** - Instrumentation and ADB
- **Content Provider** - Instrumentation and ADB
- **Firebase** - Depends on component being tested

### Testing Best Practices

#### Repository Pattern for Testing
Create interface repository used by:
1. **Production repository** - Used by ViewModel for API/DB calls
2. **Fake/Mock repository** - Emits/sets values for testing

#### Testing Suspend Functions
- Use `runTest {}` to run coroutine functions
- Skips time delays in suspend functions
- Creates TestScope similar to CoroutineScope
- Use `advanceUntilIdle()` to ensure all coroutines complete

#### Test-Driven Development (TDD)
**Golden Rule:** "Write test cases before function implementation"

**Process:**
1. Write function signature
2. Write test case for expected behavior
3. Write function logic to pass tests

### Test Types

1. **Unit Tests** - No emulator/UI needed (DB functions, API calls)
2. **Integration Tests** - Normal view testing without user input (requires emulator)
3. **UI Tests** - Requires user input testing

#### Unit Testing Challenges

**Problem:** Testing Android components like `Activity.onCreate()`
- Mock/Stub won't help due to inheritance
- Spy won't help due to inheritance

**Solution:** Use **Shadow** (proxy pattern)
- Intercepts method calls including static methods
- Can shadow `onCreate()` method to do nothing instead of throwing exceptions
- Saves events for later verification

#### Instrumented Testing
- Integration testing with lifecycle and event control
- Tests UI relying on Android framework
- Tools: AndroidJUnitRunner, Espresso, UI Automator

#### Worker Testing
Test worker for:
- Periodic execution
- Network connection constraints
- Battery/charging constraints
- Re-doable on failure

#### UI Testing with Compose
- Use `createComposeRule()` for Compose UI framework initialization
- Use `createAndroidComposeRule()` for activity context/lifecycle
- Add `testTag` to composables in modifier for testing
- Place composable in activity for testing

---

This README provides comprehensive coverage of Android development concepts, from basic Kotlin fundamentals to advanced testing strategies. Each section includes practical examples and references for deeper learning.
