Fragment 
<img width="291" height="400" alt="Screenshot 2026-03-02 at 10 02 55 AM" src="https://github.com/user-attachments/assets/2b11634f-5a36-47b8-bf45-bc40152a34c7" />

  
  1. A view    which has a very similar lifecycle to a activity and can act as a container which holds multiple other views . A fragment lifecycle is directly associated and 
  controlled by a activity which uses the fragmentmanager to manage multiple fragment . 
  
  2. A fragment can also hold fragment in itself by using childfragment manager to manage those . 
  
  3. viewLifecycleOwner is a LifecycleOwner associated with the Fragment’s View hierarchy. It represents the
  lifecycle of the Fragment’s View, which starts when the Fragment’s onCreateView is called and ends when
  onDestroyView is invoked. This allows you to bind UI-related data or resources specifically to the lifecycle of the
  Fragment’s View, preventing issues like memory leaks.
