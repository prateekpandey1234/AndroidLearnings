#Android Interview Posts
1. https://kanchan-pal.medium.com/paypal-interview-experience-sde-3-a015695d0eba
2.https://github.com/weeeBox/mobile-system-design
3. https://gist.github.com/atierian/610538f39a4844881e20b673f4c8e8dc





# AndroidLearnings
Normal kotlin stuff 
   1. a companion object is where we store static functions or classes  , static functions are those which don't need intialisation or anything to be accessible and run 
      , these functions or variables are globally scoped and can be changed anywhere . 
   2. like normal variable -> class().var1
      static variable -> class.var2  as we can see these are globally scoped not based on single instance of app.
   3. let{} extesnsion function is used 2 ways : 1-> is used to null check and 2-> it also thread safe operation as it creates copy of current instance and do modification for that
      specific thread.


Pagination
   1. there are two types of paginations :-
         a. Offset based when you handle with small data or low model complexity , the implementation is fairly easy with determining the current offset/page user is then make api call                and then add it to db and then show it to the ui screen .
         b. We a mediator to handle this data exchange , we also use caching (small db) handling for ui loading . 

Notifications

   Why Device Tokens Change:
   
      Security Enhancement - Tokens are rotated periodically to prevent unauthorized access and maintain security integrity
      Privacy Protection - Regular token changes help protect user privacy by making it harder to track devices over long periods
      System Updates - OS updates often trigger token regeneration as part of security improvements
      Anti-Fraud Measures - Token rotation helps prevent malicious actors from reusing old tokens for spam or unauthorized notifications
      Platform Requirements - Both Apple and Google mandate token refresh cycles as part of their push notification service protocols
      
   When Device Tokens Change:
   
      App Reinstallation - New installation always generates a fresh token
      OS Updates - Major or minor system updates frequently trigger token regeneration
      App Updates - Updating the app version can cause token refresh, especially with significant changes
      Device Restore - Restoring from backup or factory reset creates new tokens
      Long Periods of Inactivity - Tokens may expire and regenerate after extended periods without app usage
      Cloud Account Changes - Signing out/in to iCloud (iOS) or Google account (Android) can trigger token refresh
      Push Service Reconnection - When the device reconnects to APNs (iOS) or FCM (Android) after network issues
      Security Events - Detected security threats or suspicious activity may force token regeneration
      
   Best Practices:
   
      Always Monitor Token Changes - Implement callbacks to detect and update tokens immediately
      Store Tokens Server-Side - Keep updated tokens in your backend database
      Handle Token Refresh Gracefully - Don't assume tokens remain constant
      Regular Token Validation - Periodically verify tokens are still valid before sending notifications
      RetryClaude can make mistakes. Please double-check responses. 

Dependency Injection
   1. scoping and cutom components hilt android https://x.com/nagataro_san475/status/1928560079738917241 ,https://medium.com/androiddevelopers/hilt-adding-components-to-the-hierarchy-96f207d6d92d , https://medium.com/mindful-engineering/more-on-hilt-custom-components-custom-scope-f66c441c40c9
   2. The Component Hierarchy (Like a Family Tree)
      Hilt organizes your app like a family tree. Each level lives for a different amount of time:
      
      🏠 Application (Lives for entire app lifetime)
      
      ↓
      
      📱 Activity (Lives while screen is open)
      
      ↓  

      🧩 Fragment (Lives while that piece of screen exists)
      
      ↓

      👁️ View (Lives while that button/text exists)
      Components are those generated objects which help in generate abstract class which acts as a container full with dependcies and code ready to inject in classes with the            logic 
   4. Entry points are like gateways to non hilt code to access the dragger code . just like we use @AndroidEntryPoint for activities,fragments and view . we have to provide a           entry point for . @AndroidEntryPoint also handle injection of depencies and how and when inside the android componenet . 
   5. @Module or module is used to tell dragger that this is class or object which when installed within a component life cycle scope will help in coordinate depedency injections .
      It acts as a configuration point, declaring which objects will be provided for injection and their scope. Essentially, a module tells the dependency injection framework how          to create and provide instances of classes, particularly those that might not be directly owned by your application

State Hoisting in compose
   1. there are two types of compose, statefull and stateless .
   2. statefull compose are those who use remember tyype of delegation to observe and recompose in case of any event occurence like text change , but this makes the composable           hard to test and can cause unneccessary recompositions sometimes
   3. stateless is the scenario when the compose is being controlled by it's caller making the compose stateless , this is also called state hoisting
   4. most of the time the collecting and emitting works on same dispatcher of current context in which flow is called , we can switch that like using flowOn which allows
      emitting on Io thread but for collection you have to use withContext to switch dispcatcher
   5. remember is inline compose function which stores value of variable when it was intially composed , then on recompositions it will store value , using remeberSavable we can         save states across any configuration changes , this makes compose statefull.
   6. Compose is decalartive UI , instead of adding, chaging ui . we update ui depending on different ui states .
   7. When hoisting state, there are three rules to help you figure out where state should go:
      
            a. State should be hoisted to at least the lowest common parent of all composables that use the state (read).
            b. State should be hoisted to at least the highest level it may be changed (write).
            c. If two states change in response to the same events they should be hoisted to the same level.
      You can hoist the state higher than these rules require, but if you don’t hoist the state high enough, it might be difficult or impossible to follow unidirectional data flow.
   8. derived state of is also an inline function which helps to update ui from any state changes but these state changes are very frequent like scrolling state changes , what it        does is that it helps in stopping execssive recompositions and only recomposes when certain condition or threashold is reached for that state which changing many times .
      using normal remember will cause mulitple recomposition so we use derived state which creates a new state itself to stop those extra recompositions .
      (https://medium.com/androiddevelopers/jetpack-compose-when-should-i-use-derivedstateof-63ce7954c11b)
   
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

Workers Android(https://medium.com/@appdevinsights/work-manager-android-6ea8daad56ee)
   1. workers are android based api used to run tasks even when app is killed or tasks which needs to scheduled , user doesn't has any interaction during this process and 
      ex:- data base cync , daily regular apis
   2. there are several benefits of a workmanager :-
         a. can make a worker periodic , add constraints like(low battery , netowrk ) , can chain workers together .
         b. add backoff policy(time after which worker runs agiain when failing) , use automatic threading and coroutines

Suspend Coroutine , Complete defferable and suspend cancel coroutine
   1. suspend coroutine and suspend cancel coroutine are both call based coroutine which can be return result based on the call back received whether it is success or failure            either exception.
   2. what they is they Pause a coroutine , Wait for some callback-based code to finish and Resume the coroutine with either a result or an exception .
   3. Complete defferable is also other coroutine based api which waits for function to complete or function returns some result or complete itself with a result .
   4. these mostly are used with legacy call back based api callings ... complete deffered where else is different .
     

BroadCasts (https://medium.com/@khush.panchal123/understanding-broadcast-receivers-in-android-044fbfaa1330)
   1. BroadCasts are kind of android system wise events which can be received by app if it has registered that event type , these events are generally network connection ,               airplane mod or sms 
   2. there are 2 types of broadcasts we can use:-
         a. static / manifest broadcast are those which can cause triggereing of onCreate() on application context , these are defined within the mainfest .
         b. context/dynamic broadcast are those which can be registered based on context of activity and they have to be unregistered when in no need .
   3. broadcast are sent / recievedin terms of intent ... these broadcasts can also be made locally for any other receiver within the app .
   4. when receiving a broadcast to register remeber :- RECEIVER_EXPORTED flag needs to be add for custom broadcast, it indicates that other apps can send the broadcast to our             app.
   5. Pending intents are used whenever you want try to recive any data or info from broadcast.(https://medium.com/@appdevinsights/pending-intent-in-android-fc93a8b6c2ac)
   6. this a type of intent which is given to android system apis to work upon and send data back to our app , we can set whether these intent work ,
               PendingIntent.getActivity() : Retrieve a PendingIntent to start an Activity
               PendingIntent.getBroadcast() : Retrieve a PendingIntent to perform a Broadcast
               PendingIntent.getService() : Retrieve a PendingIntent to start a Service
   7. also when registering the broadcast , mention it clearly which type of intent broadcast you need to see like IntentFilter("SMS_deliver") to detect if the intent which is           using this filter is completed or not .
   8.                                 // receiver used for receiving sms response
            context.registerReceiver(object : BroadcastReceiver(){
                override fun onReceive(p0: Context?, intent: Intent?) {
                    if (intent?.action == Telephony.Sms.Intents.SMS_RECEIVED_ACTION) {
                        val messages = Telephony.Sms.Intents.getMessagesFromIntent(intent)
                        for (sms in messages) {
                            val sender = sms.originatingAddress
                            // Check if this is from the number we sent to
                            if (sender?.replace("+", "")?.contains(phoneNumber.replace("+", "")) == true) {

                                onSuccess()
                                // You can call another callback here to handle the received message
                                // For example: onSmsReceived(sender, body)
                            }
                        }
                        context.unregisterReceiver(this)
                    }
                }

            },IntentFilter(Telephony.Sms.Intents.SMS_RECEIVED_ACTION),Context.RECEIVER_NOT_EXPORTED)



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

   2. remember that whenever you make changes into your schema like adding/deleting column, creating new table .. you have to migrate to newer version of db and whenever that            happens make sure to tell your db class all changes on onUpgrade() callback for sql lite . also what happens for ex user has db version 1 and both version 2, version 3 and         version 4 have modified the schema by anymeans so when user updates app , first inupdate will go for version 2 then version3 and version 4 ... executing all the updates and        changes said there and does not skip any migration changes , also check if you are not adding same column again before upfrading.
   3. for db migration on room :-https://medium.com/androiddevelopers/understanding-migrations-with-room-f01e04b07929
   4. Why is Database Migration Necessary?
         Schema Evolution
         Apps need to update database structure as features change
         Allow adding new tables or columns without losing existing data
         Data Preservation
         Migrate existing user data to new schema
         Prevent data loss during app updates
         Restructure database for better query performance
         Remove deprecated columns or tables
   5. When is onUpgrade() Called?
         onUpgrade() is called when the database version number in your SQLiteOpenHelper changes
         Specifically, when you increase the version number in the SQLiteOpenHelper constructor
         This happens when you want to modify your database schema (add/remove tables, change column types)
         
      Where are Old and New Versions Fetched?
         
         Old Version: Comes from the existing database file on the user's device
         New Version: Defined in your current code's database helper class
         The version comparison happens automatically by the Android SQLite framework
   6. always keep database single in app don't create different databases , you can create different table within them .
   7. for Room db with sql injection , to create db for multiple users is to create different tables with names like TABLE_NAME+user_id and as userId changes , db changes and gets       created .
   8. withTransaction is an extensiuon function inside the room-ktx library which allows to operate multiple db queries sequentially  , like you want to refresh db . so
      
            db.withTransaction{
               db.claer()
               db.insert() // if this fails then the clear() query is also called off and db is restored 
            }
      this allows to make sure our db data is safe even if something goes wrong.
      Yes, clear() does remove everything from the database immediately. But here's the key part:
      Where does the rollback data come from?
      SQLite maintains a rollback journal (or WAL - Write-Ahead Log in WAL mode) that contains:
      
      Before-images: The original data before each change
      Undo operations: Instructions on how to reverse each change
      
      So when clear() executes:
      
      SQLite first writes all the existing data to the rollback journal
      Then it deletes the data from the main database
      If insert() later fails, SQLite reads from the rollback journal to restore the original data
   10. so what does sql in room does is that it stores a journal logs of the queries it ran like if it deleted some data , it stores that in disk . this also is done in paging-cache way to 
      avoid lockdowns and heavy db time taking .
      


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
   c. testing a worker manager is kind of similar to ui testing , the main points would be that you have to test for a worker:-
      i.   I'd like this to be done periodically
      ii.  I'd want to be sure the device has an available network connection
      iii. The device of user should have sufficient battery or is probably charging.
      iv.  This task should be _re-doable_, i.e if the network fails, the task will run again.

4. UI Testing
   a. createcomposeRule() a function that creates a test rule, which is used to initialize the Compose UI framework and provides utility functions for interacting with UI elements in your tests.
   b. normally when tetsing a compose we can use createAndroidComposeRule() to help get acitvity context or lifecycles , it mocks the behaviour of activity itself
   c. to test a compose you need to add testTag to the compose itself in the modifier where you are calling it , having composable as in activity .
   d. 

   

    
