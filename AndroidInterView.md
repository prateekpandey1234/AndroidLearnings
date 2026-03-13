# Fragment 


  <img width="291" height="400" alt="Screenshot 2026-03-02 at 10 02 55 AM" src="https://github.com/user-attachments/assets/2b11634f-5a36-47b8-bf45-bc40152a34c7" />

  
  1. A view    which has a very similar lifecycle to a activity and can act as a container which holds multiple other views . A fragment lifecycle is directly associated and 
  controlled by a activity which uses the fragmentmanager to manage multiple fragment . 
  
  2. A fragment can also hold fragment in itself by using childfragment manager to manage those . 
  
  3. viewLifecycleOwner is a LifecycleOwner associated with the Fragment’s View hierarchy. It represents the
  lifecycle of the Fragment’s View, which starts when the Fragment’s onCreateView is called and ends when
  onDestroyView is invoked. This allows you to bind UI-related data or resources specifically to the lifecycle of the
  Fragment’s View, preventing issues like memory leaks.

  

# Service 

  <img width="379" height="419" alt="Screenshot 2026-03-02 at 10 21 40 AM" src="https://github.com/user-attachments/assets/70b2f634-85f6-48d2-b86d-1fe0a17d9cfd" />


  1. A Service is a background component that allows an app to perform long-running operations independently of user interactions. Unlike activities, services do not have a user          interface and can continue running even when the app is not in the foreground. They are commonly used for background tasks such as downloading files, playing music, syncing      data, or handling network operations.
  2. Started Service : A service is started when an application component calls startService(). It runs in the background indefinitely until it stops itself using stopSelf() or is explicitly stopped using stopService().Example Usage:Playing background music ,Uploading or downloading files . 
  3.  Bound Service : A bound service allows components to bind to it using bindService(). The service remains active as long as there are bound clients and automatically stops when all clients disconnect. using this line specifically ,  override fun onBind(intent: Intent?): IBinder = binder
  4. Foreground Service : A foreground service is a special type of service that remains active while displaying a persistent notification. It is used for tasks that require ongoing user awareness, such as music playback, navigation, or location tracking.

    class ForegroundService : Service() {

      override fun onCreate() {
          super.onCreate()
          // Initialize resources
      }
  
      override fun onStartCommand(intent: Intent?, flags: Int, startId: Int): Int {
          val notification = createNotification()
          startForeground(1, notification) // Start service as foreground
          // Perform task
          return START_STICKY
      }
  
      private fun createNotification(): Notification {
          val notificationChannelId = "ForegroundServiceChannel"
          val channel = NotificationChannel(
              notificationChannelId,
              "Foreground Service",
              NotificationManager.IMPORTANCE_DEFAULT
          )
          getSystemService(NotificationManager::class.java).createNotificationChannel(channel)
          return NotificationCompat.Builder(this, notificationChannelId)
              .setContentTitle("Foreground Service")
              .setContentText("Running in the foreground")
              .setSmallIcon(R.drawable.ic_notification)
              .build()
      }
  
      override fun onDestroy() {
          super.onDestroy()
          // Clean up resources
      }
  
      override fun onBind(intent: Intent?): IBinder? = null
    }

    
# BroadCast Receivers 

  1. A BroadcastReceiver is a component that allows your app to listen for and respond to system-wide broadcast messages or app-specific broadcasts. These broadcasts are typically triggered by the system or other applications to signal various events, such as battery status changes, network connectivity updates, or custom intents sent within an app. BroadcastReceivers are a useful mechanism for building responsive applications that react to dynamic system or app-level events . 

  2. System Broadcasts: Sent by the Android operating system to inform apps about system events like battery level changes, time zone updates, or network connectivity changes.
  
  3. Custom Broadcasts: Sent by applications to communicate specific information or events within or between apps.
  
  4. Static Registration (via Manifest): Use this for events that need to be handled even when the app is not running. Add an <intent-filter> in the AndroidManifest.xml file.
  
  5. Dynamic Registration (via Code): Use this for events that need to be handled only when the app is active or in a specific state.
  
  6. Monitoring changes in network connectivity.
     Responding to SMS or call events.
     Updating the UI in response to system events like charging status.
     Scheduling tasks or alarms with custom broadcasts.
     
   

# Content Providers (https://medium.com/@YodgorbekKomilo/understanding-and-implementing-content-providers-in-android-with-kotlin-and-jetpack-compose-b59f8bc32570)

  1. A ContentProvider is a component that manages access to a structured set of data and provides a standardized interface for sharing data between applications. It serves as a central repository that other apps or components can use to query, insert, update, or delete data, ensuring secure and consistent data sharing across apps.
  
  2. ContentProviders are especially useful when multiple apps need access to the same data or when you want to provide data to other apps without exposing your database or internal storage structure.
  
  3. Purpose of ContentProvider: The primary purpose of a ContentProvider is to encapsulate data access logic, making it easier and more secure to share data across apps. It abstracts the underlying data source, which can be an SQLite database, file system, or even network-based data, and provides a unified interface for interacting with the data.
  
  4. A ContentProvider uses a URI (Uniform Resource Identifier) as its address for data access. The URI consists of:
        1. Authority: Identifies the ContentProvider (e.g., com.example.myapp.provider).
        2. Path: Specifies the type of data (e.g., /users or /products).
        3. ID (optional): Refers to a specific item within the dataset.
        
  5. onCreate(): Initializes the ContentProvider.
    query(): Retrieves data.
    insert(): Adds new data.
    update(): Modifies existing data.
    delete(): Removes data.
    getType(): Returns the MIME type for the data.

  6. To make your ContentProvider accessible to other apps, you must declare it in your AndroidManifest.xml file. The authority attribute uniquely identifies your ContentProvider.

  7. You can use the ContentResolver class to interact with a ContentProvider from another app. TheContentResolver provides methods to query, insert, update, or delete data.    
          val contentResolver = context.contentResolver

          // Query data
          val cursor = contentResolver.query(
              Uri.parse("content://com.example.myapp.provider/users"),
              null,
              null,
              null,
              null
          )
          
          // Insert data
          val values = ContentValues().apply {
              put("name", "John Doe")
              put("email", "johndoe@example.com")
          }
          contentResolver.insert(Uri.parse("content://com.example.myapp.provider/users"), values)

  8.  UseCases : 
      Sharing data between different applications.
      Initializing components or resources during the app startup process.
      Providing access to structured data, such as contacts, media files, or app-specific data.
      Enabling integration with Android’s system features, such as the Contacts app or File Picker.
      Allowing data access with fine-grained security controls.          


# Deep Links 
  1. Deep links allow users to navigate directly to a specific screen or feature within your app from an external source, such as a URL or notification. Handling deep links involves defining them in the AndroidManifest.xml and processing the incoming intents in the corresponding activities or fragments.
  2. To enable deep linking, declare an intent filter in the `AndroidManifest.xml` for the activity that should handle the deep link. The intent filter specifies the URL structure or scheme your app responds to.
  
    ```
    <activity android:name=".MyDeepLinkActivity">
        <intent-filter>
            <action android:name="android.intent.action.VIEW" />
            <category android:name="android.intent.category.DEFAULT" />
            <category android:name="android.intent.category.BROWSABLE" />
            <data
                android:scheme="https"
                android:host="example.com"
                android:pathPrefix="/deepLink" />
        </intent-filter>
    </activity>'''

  3. Inside the activity, retrieve and process the incoming intent data to navigate to the appropriate screen or perform an action.
             
              val intentData: Uri? = intent?.data
              if (intentData != null) {
                  val id = intentData.getQueryParameter("id") // Example: Retrieve query parameters
                   navigateToFeature(id)
              }
  4. To test the deep link, you can use the adb command below , This command simulates a deep link and opens the app to handle it.:
           adb shell am start -a android.intent.action.VIEW \
           -d "https://example.com/deepLink?id=123" \
          com.example.myapp


# Shared ViewModels 

  1.  A shared ViewModel refers to a ViewModel instance that is shared between multiple fragments within the same activity. This is achieved using the activityViewModels() method, 
  2. // Shared ViewModel
           
           class SharedViewModel : ViewModel() {
            
                private val _userData = MutableStateFlow<User?>(null)
                val userData: StateFlow<User?> = _userData
            
                fun setUserData(user: User) {
                    _userData.value = user
                }
            }
            
            // Fragment A (Sending data)
            class FirstFragment : Fragment() {
                private val sharedViewModel: SharedViewModel by activityViewModels()
            
                fun updateUser(user: User) {
                    sharedViewModel.setUserData(user)
                }
            }
            
            // Fragment B (Receiving data on another Fragment)
            class SecondFragment : Fragment() {
                private val sharedViewModel: SharedViewModel by activityViewModels()
            
                override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
                    super.onViewCreated(view, savedInstanceState)
                    
                    lifecycleScope.launch {
                        viewLifecycleOwner.repeatOnLifecycle(Lifecycle.State.RESUMED) {
                            sharedViewModel.userData.collectLatest { user ->
                                // ..
                            }
                        }
                    }
                }
            }
            
            // Activity (Receiving data on the Activity)
            class MainActivity : ComponentActivity() {
                private val sharedViewModel: SharedViewModel by viewModels()
            
                override fun onCreate(savedInstanceState: Bundle?) {
                    super.onCreate(savedInstanceState)
                    
                    lifecycleScope.launch {
                        lifecycle.repeatOnLifecycle(Lifecycle.State.RESUMED) {
                            sharedViewModel.userData.collectLatest { user ->
                                // ..
                            }
                        }
                    }
                }
            }        
                        
# Configuration Changes 
1.  When a configuration change occurs in Android (e.g., screen rotation, theme changes, font size adjustments, or language updates), the system may destroy and recreate the current Activity to apply the new configuration. This behavior ensures that the app's resources are reloaded to reflect the updated configuration.

    ## Default Behavior During Configuration Changes
    
    ### 1. Activity Destruction and Recreation
    
    When a configuration change happens, the Activity is destroyed and then recreated. This process involves the following steps:
    
    - The system calls the `onPause()`, `onStop()`, and `onDestroy()` methods of the current Activity.
    - The Activity is recreated by calling its `onCreate()` method with the new configuration.
    
    ### 2. Resource Reloading
    
    The system reloads resources (such as layouts, drawables, or strings) based on the new configuration, allowing the app to adapt to changes like screen orientation, theme, or locale.
    
    ### 3. Data Loss Prevention
    
    To prevent data loss during recreation, developers can save and restore instance state using the `onSaveInstanceState()` and `onRestoreInstanceState()` methods or by leveraging a `ViewModel`.
    
    ```kotlin
    override fun onSaveInstanceState(outState: Bundle) {
        super.onSaveInstanceState(outState)
        outState.putString("user_input", editText.text.toString())
    }
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        
        val restoredInput = savedInstanceState?.getString("user_input")
        editText.setText(restoredInput)
    }
    ```
    
    ## Key Configuration Changes Triggering Recreation
    
    1. **Screen Rotation**: Changes the screen's orientation between portrait and landscape, causing layouts to be reloaded to fit the new dimensions.
    
    2. **Dark/Light Theme Changes**: When a user switches between dark and light modes, the app reloads theme-specific resources (e.g., colors and styles).
    
    3. **Font Size Changes**: Adjustments to the device's font size setting reload text resources to reflect the new scale.
    
    4. **Language Changes**: Updates to the system language trigger the loading of localized resources (e.g., strings in a different language).
    
    ## Avoiding Activity Recreation
    
    To handle configuration changes without restarting the Activity, you can use the `android:configChanges` attribute in the manifest. This approach delegates responsibility to the developer to handle changes programmatically.
    
    ```xml
    <activity
        android:name=".MainActivity"
        android:configChanges="orientation|screenSize|keyboardHidden" />
    ```
    
    In this scenario, the system does not destroy and recreate the Activity. Instead, the `onConfigurationChanged()` method is invoked, allowing developers to handle the change manually.
    
    ```kotlin
    override fun onConfigurationChanged(newConfig: Configuration) {
        super.onConfigurationChanged(newConfig)
        
        if (newConfig.orientation == Configuration.ORIENTATION_LANDSCAPE) {
            // Handle landscape-specific changes
        } else if (newConfig.orientation == Configuration.ORIENTATION_PORTRAIT) {
            // Handle portrait-specific changes
        }
    }
    ```
    
    ## Summary
    
    When configuration changes occur, the default behavior is to destroy and recreate the Activity, reloading resources to adapt to the new configuration. Developers can use `onSaveInstanceState()` to preserve transient UI state or a `ViewModel` for non-UI state. To avoid recreation, the `android:configChanges` attribute can be used in the manifest, delegating the responsibility to handle changes to the developer.


  
# Android Accessibility Guide

  1. Accessibility ensures that your application is usable by everyone, including people with disabilities, such as visual, auditory, or physical impairments. Implementing accessibility features enhances the user experience and ensures compliance with global accessibility standards, such as WCAG (Web Content Accessibility Guidelines).
  
  ## Leveraging Content Descriptions
  
  Content descriptions provide textual labels for UI elements, enabling screen readers like TalkBack to announce them to users with visual impairments. Use the `android:contentDescription` attribute for interactive or informational elements like buttons, images, and icons. If an element is decorative and should be ignored by screen readers, set `android:contentDescription` to null or use `View.IMPORTANT_FOR_ACCESSIBILITY_NO`.
  
  ```xml
  <ImageView
      android:contentDescription="Profile Picture"
      android:src="@drawable/profile_image" />
  ```
  
  ## Supporting Dynamic Font Sizes
  
  Ensure that your app respects the user's font size preferences set in the device settings. Use `sp` units for text sizes to automatically scale based on accessibility settings.
  
  ```xml
  <TextView
      android:textSize="16sp"
      android:text="Sample Text" />
  ```
  
  ## Focus Management and Navigation
  
  Properly manage focus behavior, especially for custom views, dialogs, and forms. Use `android:nextFocusDown`, `android:nextFocusUp`, and related attributes to define logical navigation paths for keyboard and D-pad users. Additionally, test your app with a screen reader to ensure that focus moves naturally between elements.
  
  ## Color Contrast and Visual Accessibility
  
  Provide sufficient contrast between text and background colors to improve readability for users with low vision or color blindness. Tools like Android Studio's Accessibility Scanner can help evaluate and optimize color contrast in your app.
  
  ## Custom Views and Accessibility
  
  When creating custom views, implement the `AccessibilityDelegate` to define how screen readers interact with your custom UI components. Override the `onInitializeAccessibilityNodeInfo()` method to provide meaningful descriptions and states for your custom elements.
  
  ```kotlin
  class CustomView(context: Context) : View(context) {
      init {
          importantForAccessibility = IMPORTANT_FOR_ACCESSIBILITY_YES
          setAccessibilityDelegate(object : AccessibilityDelegate() {
              override fun onInitializeAccessibilityNodeInfo(host: View, info: AccessibilityNodeInfo) {
                  super.onInitializeAccessibilityNodeInfo(host, info)
                  info.text = "Custom component description"
              }
          })
      }
  }
  ```
  
  ## Testing for Accessibility
  
  Use tools such as **Accessibility Scanner** and the **Layout Inspector** in Android Studio to identify and fix accessibility issues. These tools help ensure that your app is accessible to users relying on assistive technologies.
  
  ## Summary
  
  Ensuring accessibility in Android applications involves:
  
  - Providing content descriptions
  - Supporting dynamic font sizes with `sp` units
  - Managing focus for navigation
  - Ensuring adequate color contrast
  - Adding accessibility support for custom views
  
  By leveraging Android tools and testing thoroughly, you can build applications that are inclusive and accessible to all users. For more information about accessibility, check out the official documentation.


  
# RecyclerView Internal Mechanism

  RecyclerView is a useful and flexible Android component designed for efficiently displaying large datasets by recycling item views instead of inflating new ones repeatedly. It achieves this efficiency using an object pool-like mechanism for managing views, known as the ViewHolder pattern.
  
  ## Core Concepts of RecyclerView's Internal Mechanism
  
  ### 1. Recycling Views
  
  RecyclerView reuses existing views rather than creating new ones for every item in the dataset. When a view scrolls out of the visible area, it is added to a view pool (known as `RecyclerView.RecycledViewPool`) instead of being destroyed. When a new item comes into view, RecyclerView retrieves an existing view from this pool if available, avoiding the overhead of inflation.
  
  ### 2. ViewHolder Pattern
  
  RecyclerView uses a `ViewHolder` to store references to the views within an item layout. This prevents repeated calls to `findViewById()` during binding, which improves performance by reducing layout traversal and view lookup.
  
  ### 3. Adapter's Role
  
  The `RecyclerView.Adapter` bridges the data source and the RecyclerView. The adapter's `onBindViewHolder()` method binds data to the views when they are reused, ensuring that only the visible items are updated.
  
  ### 4. RecycledViewPool
  
  The `RecycledViewPool` serves as the object pool where unused views are stored. It allows RecyclerView to reuse views across multiple lists or sections with similar view types, further optimizing memory usage.
  
  ## How the Recycling Mechanism Works
  
  ### 1. Scrolling and Item Visibility
  
  As the user scrolls, items that go out of view are detached from the RecyclerView but are not destroyed. Instead, they are added to the `RecycledViewPool`.
  
  ### 2. Rebinding Data to Recycled Views
  
  When new items come into view, RecyclerView first checks the `RecycledViewPool` for an available view of the required type. If a match is found, it reuses the view by rebinding it with new data using `onBindViewHolder()`.
  
  ### 3. Inflation if No View is Available
  
  If no suitable view is available in the pool, RecyclerView inflates a new view using `onCreateViewHolder()`.
  
  ### 4. Efficient Memory Usage
  
  By recycling views, RecyclerView minimizes memory allocation and garbage collection, which can cause performance issues in scenarios involving large datasets or frequent scrolling.



# ViewModel Lifecycle in Android

  ## Overview
  
  The lifecycle of a ViewModel is tied to its associated `ViewModelStoreOwner`, which could be an Activity, Fragment, or other lifecycle-aware components. A ViewModel exists as long as the `ViewModelStoreOwner` remains in scope, ensuring that data and state survive configuration changes like screen rotations.
  
  This lifecycle design makes ViewModel an essential component for managing UI-related data efficiently and preserving it across such changes. For instance, in the case of an Activity, the ViewModel persists until the Activity is fully finished and removed from memory. This lifecycle design ensures that data and state are preserved across configuration changes, such as screen rotations.
  
  ## ViewModel Lifetime
  
  The illustration below outlines the lifetime of a ViewModel in relation to an Activity's lifecycle. The diagram highlights how a ViewModel instance is created and retained through Activity lifecycle events, surviving even when the Activity is temporarily destroyed and recreated. While this example uses an Activity, the same principles apply to Fragments and other lifecycle-aware components that act as `ViewModelStoreOwner`.
  <img width="295" height="413" alt="Screenshot 2026-03-04 at 5 36 33 PM" src="https://github.com/user-attachments/assets/130a3f49-04f9-455f-b83c-bec6709ae201" />

  ## Surviving Configuration Changes
  
  In Android, the Jetpack ViewModel is designed to survive configuration changes such as screen rotations or device language updates. When a ViewModel is created for a UI component (like an Activity or Fragment), it is tied to the component's lifecycle owner. For activities, this lifecycle is usually the `ComponentActivity`, while for fragments, it is the `Fragment` itself. The key factory is `ViewModelStore`, which is retained across configuration changes, enabling the ViewModel to persist its data without being re-created.
  
  ## Internal Implementation
  
  If you explore the internal implementation of Jetpack libraries, you'll find that both `androidx.activity.ComponentActivity` and `androidx.fragment.app.Fragment` implement the `ViewModelStoreOwner` interface. This interface enables activities and fragments to have their own `ViewModelStore`, ensuring that ViewModel instances persist across configuration changes. 
  
  The `ViewModelStore` itself manages ViewModel instances by maintaining a `Map` where each ViewModel is associated with a unique `String` key.
  
  ## Key Points
  
  - **Lifecycle Scope**: ViewModel exists as long as its `ViewModelStoreOwner` is in scope
  - **Configuration Changes**: Data survives screen rotations, language changes, and other configuration updates
  - **ViewModelStore**: Acts as the container that persists across configuration changes
  - **ViewModelStoreOwner**: Interface implemented by `ComponentActivity` and `Fragment`
  - **Storage Mechanism**: ViewModels are stored in a `Map` with unique `String` keys  



# Test Driven Development 

1. TDD comprises of 3 stages : Red -> Green -> Blue 
2. Red -> write a piece of code very basic so that it fails the test , not getting the desired output .
3. Green -> complete the code which passes the test and works .
4. Blue (Refractor) -> refactor and optimize the code further while it passes tests without changing the test themselves 
5. main concern is that we want to drive our code from the tests written . 


# SOLID Principles
1. Single Responsibility Principle (SRP) : A class should have only one reason to change, meaning it should only have one job or responsibility.In Android, it’s common to see large     classes that handle UI, data processing, and more. SRP encourages breaking these tasks into smaller, focused classes. Example: Imagine we have an activity that both loads data from a server and displays it on the UI.Instead of doing everything in one activity, we can separate concerns:
    
    A DataRepository class handles data fetching.
    The MainActivity class focuses on displaying data.

2. Open/Closed Principle (OCP) : Software entities should be open for extension but closed for modification. This means we should be able to add new functionality without changing      existing code. This is particularly useful for Android projects that grow over time and require new features. this means you should be able to add new features or behaviors to your app without touching existing, already-tested code. You do this by using interfaces, abstract classes, and polymorphism.   

    ```kotlin
    // GOOD: Adheres to OCP

    // 1. Define the abstraction (Closed for modification)
    interface EventTracker {
        fun track(eventName: String)
    }
    
    // 2. Implement specific behaviors (Open for extension)
    class FirebaseTracker : EventTracker {
        override fun track(eventName: String) { /* Send to Firebase */ }
    }
    
    class MixpanelTracker : EventTracker {
        override fun track(eventName: String) { /* Send to Mixpanel */ }
    }
    
    // 3. Add a new tracker without touching the existing code!
    class AmplitudeTracker : EventTracker {
        override fun track(eventName: String) { /* Send to Amplitude */ }
    }
    
    // 4. The Usage
    class AnalyticsManager(private val trackers: List<EventTracker>) {
        fun trackAll(eventName: String) {
            // This class never needs to change, even if you add 100 new trackers.
            trackers.forEach { it.track(eventName) } 
        }
    }```

3. Liskov Subsitution principle : objects of a superclass should be replaceable with objects of its subclasses without breaking the application. , In simpler terms: A child class must fully honor the contract established by its parent class. If you pass a subclass into a function that expects the parent class, the program should not crash, throw unexpected exceptions, or behave bizarrely.     

  ```kotlin
  class ProfileFragment : Fragment() {
    
    override fun onCreate(savedInstanceState: Bundle?) {
        // If you delete this next line, the app crashes.
        super.onCreate(savedInstanceState) 
        
        // Your custom logic...
    }
}   
```
4. Interface Segregation Principle (ISP) : Clients should not be forced to implement interfaces they don’t use. Instead of one large interface, create smaller, more specific ones.
In Android, you often work with interfaces in callback patterns and listeners. Following ISP keeps the interfaces focused and manageable.Example: Imagine a single interface for user interactions on a music player that includes methods for play, pause, stop, and record. Instead of forcing classes to implement all these methods, split the interfaces.

```kotlin
  interface Playable {
    fun play()
    fun pause()
}
interface Recordable {
    fun record()
    fun stop()
}
```
5. Dependency Inversion Principle (DIP)
High-level modules should not depend on low-level modules; both should depend on abstractions. This principle encourages dependency injection, improving modularity and testing.
In Android, dependency inversion is commonly achieved through dependency injection frameworks like Hilt or Dagger.
Example: Suppose an activity directly depends on DataRepository, creating a tight coupling. To follow DIP, we define an interface, and the activity depends on this interface instead.
```kotlin
interface Repository {
    fun fetchData(): List<String>
}
class DataRepository : Repository {
    override fun fetchData(): List<String> {
        // Fetch data from the server or database
        return listOf("Data from Repository")
    }
}
class MainActivity : AppCompatActivity() {
    private lateinit var repository: Repository
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        repository = DataRepository() // Or injected by DI framework like Hilt
        val data = repository.fetchData()
        displayData(data)
    }
    private fun displayData(data: List<String>) {
        // Code to update UI
    }
}
```
6. benefits of SOLID principles:- testability , maintainence , readable and scalable .


