# AndroidLearnings

State Hoisting in compose
   1. there are two types of compose, statefull and stateless .
   2. statefull compose are those who use remember tyype of delegation to observe and recompose in case of any event occurence like text change , but this makes the composable           hard to test and can cause unneccessary recompositions sometimes
   3. stateless is the scenario when the compose is being controlled by it's caller making the compose stateless , this is also called state hoisting
   4. most of the time the collecting and emitting works on same dispatcher of current context in which flow is called , we can switch that like using flowOn which allows
      emitting on Io thread but for collection you have to use withContext to switch dispcatcher

Observers
   1. there are 4 types of observers : Live Data , StateFlow , Flow and SharedFlow
   2. Live Data is basic state holder which obervers any change in the state of data like life cycle changes , also it is lifecycler aware
   3. State Flow is LiveData+fLOW benefits , also stores latest changes in state but can perform multiple operations like mapping or transform
     , also made up with coroutines so no need of suspend function declaration, but it is not life cycle aware so we have to collect latest state when life cycle state changes
      issue with this is that it will only emit value once inside current lifecycle if previous and current value are same
   5. flow is a oberver but it will only fetch data when collector is called like when button clicked you have to trigger flow.collect to get latest data , it utilises coroutines
      , allows transformation and mapping .
   6. Shared flow : It emits all the values and does not care about the distinct from the previous item. It emits consecutive repeated values also.

Sequence(https://medium.com/android-news/kotlin-sequences-ac6dc7c883d3)
   1. Used in place of collections when data is in millions of order .
   2. sequence are lazy evaluters , that means they process each item one by one and avoid creation of temproray collection(list,map etc) for any operations.
   3. these have two functions : intermediate and terminal
   4. intermediate will process one object and give to terminal function to add it to result , intermediate functions don't create or use temp collection
   5. this saves whole memory in process but hard to implement and complexes code
   6. val list = (1..1_000_000).toList()
      // Creates 3 intermediate lists!
      val result = list
          .filter { it % 2 == 0 }   // Intermediate list 1
          .map { it * 2 }           // Intermediate list 2
          .take(10)                 // Intermediate list 3
          .toList()
   
      val sequence = (1..1_000_000).asSequence()
      // Processes elements one by one, no intermediate lists!
      val result = sequence
          .filter { it % 2 == 0 }  // Intermediate
          .map { it * 2 }          // Intermediate
          .take(10)                // Intermediate
          .toList()                // Terminal

Side Effects
   1. LaunchedEffects is triggered evey time when value of it's is changed and utilises coroutinescope to run any IO operatons
   2. Disposable effect is like launchedeffect except for coroutinescope and is it moslty used to handle handle lifecyle events , onDispose is must
      as it is called whenever the key leave recompostion

Flows
   1. // Flow is like a  stream of data flowing through a pipe.
      // it can be of 2 ways (hot which is active everytime or cold which works only when collected)
      // flow is very best when working with complex async coroutines calls
      // can use functions such as filter , map or many other functions to play with data
      // cold flows also help in saving memory as they only work when collected
      // flow is not lifecycle aware
   2. flow has producer(emits the data using emit()) , intermediaries(optional , vcan modify the emitted data) and consumer (collect() to get flow output)
   3. flow work only when you want to receive data  On the other hand, Kotlin Flow is suitable for handling asynchronous operations that produce multiple values over time. This makes it ideal for scenarios where you need to observe changes in data, such as streaming data from a network or observing changes in a database. Kotlin Flow allows you to handle streams of data asynchronously and apply operators like map, filter, and transform
   4. https://medium.com/androiddevelopers/consuming-flows-safely-in-jetpack-compose-cde014d0d5a3
   5. https://medium.com/@devarshbhalara3072/building-efficient-api-calls-in-android-with-retrofit-coroutines-flow-and-dependency-injection-60e022b16987
   6.  always remember to add flow as an lifecycleaware component
   8.  flow is a one timer thing doesn't hold the state or any data , if reset it will go to orginal data before changing

Coroutines
   1. https://medium.com/androiddevelopers/coroutines-on-android-part-i-getting-the-background-3e0e54d20bb


ViewModel
   1. use job{ ViewmodelScope.launch()} , to cancel any type of long running jobs running in the background like fetching and processing data
      https://medium.com/@paritasampa95/coroutinescope-coroutinecontext-discussed-in-depth-to-understand-better-part-2-550dbc7af3d2
   2. https://x.com/nagataro_san475/status/1875593012648276202
   3.  private val _isLoading  = MutableLiveData<Boolean>()
       val isLoading  :LiveData<Boolean> = _isLoading
      this allows immutablitiy of isLoading by outer access , only edited inside the viewModel even when _isLoading is mutable



DataBase 

   /**
    * * {https://developer.android.com/static/images/training/data-storage/room_nested_relationships.png}
    * * https://medium.com/androiddevelopers/database-relations-with-room-544ab95e4542
    *
    * * Unitl now i have used , map for storing and mapping infraObjects to it's ids . the map is singleton object which not life
    *   cycle aware and is mutable also .
    * * the data of infra connect specially for spaces is special as one space has multiple childspaces and nested data like that ,
    *   here we can easily say that this one to many relationship .
    * * Refer to post for mor info , normally for sqlite we have to run two queries . first to get all infraobjects and then get all rows from
    *   db to get infraobjects related to that id .
    * * In Room we can use @Relation to tell room that one parameter in parent is corresponding to one parameter in child to
    *   get list of children at once
    * * Here we don't require that because same data class is used , we can just run queries simply for both way mapping
    */


Testing

   https://testing.googleblog.com/2010/12/test-sizes.html

   1. Remote DataSource Testing [Retrofit] - Mockserver | Fakes | Mocks
   2. Local DataSource database [Room] - Android X Test | Robolectric 
   3. Local DataSource Prefs [SharedPrefernces] - Android X Test | Robolectric 
   4. Local DataSource Store [DataSource] - Android X Test | Robolectric 
   5. Repository Layer | UseCases | business-logic  - JUnit 4/5 | Fakes | Mocks
   6. ViewModel & LiveData - Android X Test | Robolectric 
   7. Navigation [Navhost|Intents] - Android X Test | Robolectric 
   8. View & Animations [Activity|Fragments|CustomViews] - Espresso|Barista Instrumentation
   9. Notifications Test - Instrumentation and ADB
   10. Broadcast Test - Instrumentation and ADB
   11. Content provider - Instrumentation and ADB
   12. Firebase Testing(depends on things you are testing) - Espresso|Barista Instrumentation | Android X Test | Robolectric | ADB

remember to make a interface repo which is then used by 2 different repositories :-
   a. first repo is used by viewmodel to make api , db calls
   b. second repo is fale repo or kind of mock one which allows to emit or set any value we  want for testing purposes .
   1.  to test suspend functions using runTest{} to run coroutine functions , this allows to skip any time delay there is in suspend function
   2.  runTest makes a TestScope which is similar to CoroutineScope for a coroutine , default dispatcher for runTest is StandardTestDispatcher
   3.  use advanceUntilIdle() to make sure all coroutines work and free the test coroutines
       a. if we are starting new coroutines from top level testing coroutines then we need to use TestingDispatchers to schedule the new coroutines
       b. also when switching to different dispatcher using withContext , we need
   4.there are 3 tests - ui(requires user input) , unit (no need of emulator and ui , ex:- db functions , api calls ) and integration ( normal view stuff which doesn't requires
      user input but still needs emulator)
   
    No 1 theory of TDD best practice is
   
   “Write test cases before the function implementation”
   
   

1. Unit Testing

   a. These are steps to follow
         Write your function signature
         Write test case for what that function is going to do and what you expect from it .
         Write function logic so test case passes.
         This procedure is carefully followed when writing unit tests and recommended for Integration and UI testing as well.
   
   c. Robolectric.setupActivity. You write a test case where you call MyActivity.onCreate to check that some preinitialisation is done when called. This test case will fail at    
   super.onCreate call which is forced by android system.
   
   d. Mock will not help because you don't use a member variable which could be mocked.
   
   e. A Stub will not help because of the inheritance you can stub the onCreate method for you activity which makes testing pointless.
   
   f. You miss Spy but this will also not help because of the inheritance. With Spy can avoid the real onCreate call like stubbing but makes testing also pointless.

   g. https://proandroiddev.com/understanding-the-role-of-mocks-and-spies-in-unit-testing-66e23b0e330b
   
   h. The Shadow can help at this situation. This handles more like a proxy. There can be an explicit proxy for each inherited class.
    It can intercept each type of method call also for static methods. For you example we can create a proxy
    for the android.app.Activity which will shadow the onCreate method and instead of throwing exceptions it
    will do nothing... There you can save this event so you can later check that this super.method was called
    with expected arguments if necessary

3. Instrumented Testing

   a. that instrumentation testing is integration testing with the ability to control the life cycle and the events (onStart, onCreate etc) of the app.
   b. To test UI that relies on the Android framework or system, such as Activities, Services, or UI elements. Tools used: AndroidJUnitRunner, Espresso, UI Automator.

4. UI Testing
   a. createcomposeRule() a function that creates a test rule, which is used to initialize the Compose UI framework and provides utility functions for interacting with UI elements in your tests.
   b. 

   

    
