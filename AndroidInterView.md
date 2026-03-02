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

    
BroadCast Receivers 

  1. A BroadcastReceiver is a component that allows your app to listen for and respond to system-wide broadcast messages or app-specific broadcasts. These broadcasts are typically triggered by the system or other applications to signal various events, such as battery status changes, network connectivity updates, or custom intents sent within an app. BroadcastReceivers are a useful mechanism for building responsive applications that react to dynamic system or app-level events . 

  2. System Broadcasts: Sent by the Android operating system to inform apps about system events like battery level changes, time zone updates, or network connectivity changes.
  
  3. Custom Broadcasts: Sent by applications to communicate specific information or events within or between apps.
  
  4. Static Registration (via Manifest): Use this for events that need to be handled even when the app is not running. Add an <intent-filter> in the AndroidManifest.xml file.
  
  5. Dynamic Registration (via Code): Use this for events that need to be handled only when the app is active or in a specific state.
  
  6. Monitoring changes in network connectivity.
     Responding to SMS or call events.
     Updating the UI in response to system events like charging status.
     Scheduling tasks or alarms with custom broadcasts.
     
   

        
  
  

  
