# AndroidLearnings

Flows
1. // Flow is like a  stream of data flowing through a pipe.
   // it can be of 2 ways (hot which is active everytime or cold which works only when collected)
   // flow is very best when working with complex async coroutines calls
   // can use functions such as filter , map or many other functions to play with data
   // cold flows also help in saving memory as they only work when collected
   // flow is not lifecycle aware
2. flow has producer(emits the data using emit()) , intermediaries(optional , vcan modify the emitted data) and consumer (collect() to get flow output)
3. 

Coroutines
1. https://medium.com/androiddevelopers/coroutines-on-android-part-i-getting-the-background-3e0e54d20bb

ViewModel
1. use job{ ViewmodelScope.launch()} , to cancel any type of long running jobs running in the background like fetching and processing data
   https://medium.com/@paritasampa95/coroutinescope-coroutinecontext-discussed-in-depth-to-understand-better-part-2-550dbc7af3d2
2. https://x.com/nagataro_san475/status/1875593012648276202



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
