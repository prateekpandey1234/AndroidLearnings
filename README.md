# AndroidLearnings

Observers
   1. there are 4 types of observers : Live Data , StateFlow , Flow and SharedFlow
   2. Live Data is basic state holder which obervers any change in the state of data like life cycle changes , also it is lifecycler aware
   3. State Flow is LiveData+fLOW benefits , also stores latest changes in state but can perform multiple operations like mapping or transform
     , also made up with coroutines so no need of suspend function declaration, but it is not life cycle aware so we have to collect latest state when life cycle state changes
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
   7.  flow is a one timer thing doesn't hold the state or any data , if reset it will go to orginal data before changing

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
   1.  to test suspend functions using runTest{} to run coroutine functions , this allows to skip any time delay there is in suspend function
   2.  runTest makes a TestScope which is similar to CoroutineScope for a coroutine , default dispatcher for runTest is StandardTestDispatcher
   3.  use advanceUntilIdle() to make sure all coroutines work and free the test coroutines
       a. if we are starting new coroutines from top level testing coroutines then we need to use TestingDispatchers to schedule the new coroutines
       b. also when switching to different dispatcher using withContext , we need
   4.there are 3 tests - ui(requires user input) , unit (no need of emulator and ui , ex:- db functions , api calls ) and integration ( normal view stuff which doesn't requires
      user input but still needs emulator)
   
    No 1 theory of TDD best practice is
   
   “Write test cases before the function implementation”
   
   These are steps to follow
   
   Write your function signature
   Write test case for what that function is going to do and what you expect from it .
   Write function logic so test case passes.
   This procedure is carefully followed when writing unit tests and recommended for Integration and UI testing as well.
   
   
   Robolectric.setupActivity. You write a test case where you call MyActivity.onCreate to check that some preinitialisation is done when called. This test case will fail at super.onCreate call which is forced by android system.
   
   Mock will not help because you don't use a member variable which could be mocked.
   
   A Stub will not help because of the inheritance you can stub the onCreate method for you activity which makes testing pointless.
   
   You miss Spy but this will also not help because of the inheritance. With Spy can avoid the real onCreate call like stubbing but makes testing also pointless.
   
   The Shadow can help at this situation. This handles more like a proxy. There can be an explicit proxy for each inherited class.
    It can intercept each type of method call also for static methods. For you example we can create a proxy
    for the android.app.Activity which will shadow the onCreate method and instead of throwing exceptions it
    will do nothing... There you can save this event so you can later check that this super.method was called
    with expected arguments if necessary
   
    https://proandroiddev.com/understanding-the-role-of-mocks-and-spies-in-unit-testing-66e23b0e330b
