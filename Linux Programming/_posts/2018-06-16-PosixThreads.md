A method for writing extensible and maintainable multithreaded Linux applications while keeping ones sanity.

--- 
---
(c) Wm Parker MacKenzie, 2018

# Introduction
It is all too easy to just start writing an application and have it get away from you. Without proper planning, each thread added to the application will create runtime efficiency at the cost of maintainability. This is especially true as an application is created over a period of time and doubly so if there is more than a single developer. In the end many of the threads, mutexes, and conditions will be built in slightly different ways for no apparent reason. Eventually what started as a simple application, with the best of intentions becomes an unruly and hard to maintain mess which has grown large enough to affect the companies bottom line.

What is needed is a way to commit to a set of rules describing how the threading resources should be used in the context of benefitting the application. Further, the rules should then be codified in a way which makes the use of the threading library consistent throughout the application. As the application matures, the rules as to how threads interact may need to change; however, this will be done with knowledge and intent. A simple set of codified rules in a single place will make the application more maintainable.

Libraries, such as the POCO may be leveraged to accomplish these goals; however no external library can capture how you want your application to function and how the threads interact. It is you, the application developer, which needs to define the use of the functionality. 

It need not be complex, nor should it be. What follows are some simple ideas, which if done up front, will pay off in spades later. For those existing products, the following can be used as an improvement and integrated later in the life cycle of a product and migrated to over time. 

The following strategies have been and continue to be refined on carrier grade products as well as my own DIY projects. As an example these strategies have been applied successfully when building carrier grade LoRaWAN gateways at Senet. Many of these gateways are expected to run for decades hundreds of feet above the ground, beyond reach of human interaction. 

## Embrace the tools provided
"I suppose it is tempting, if the only tool you have is a hammer, to treat everything as if it were a nail." - Abraham Maslow, 1966, "The Psychology of Science"

For much of our professional engineering lives, we are tempted to treat every problem with the tools we know. This is a great way to approach understanding of a problem. However, to be successful, we need to embrace another great quote from Saint Ambrose "When in Rome, do as the Romans do". 

This is true for Posix Threads as well. If one approaches Posix Threads as if they were tasks in VxWorks, FreeRTOS, mbedOS, or Visual C++, there would be a constant stress on the application and future maintainers. This means one should move towards acceptance quickly and build the application from the ground up with the use of cancellation points, pushing and popping thread functions, condition signals, and more. Many of which are similar at a high level to the features of other operating systems; however, beware of petting the tiger as if it is your neighbors kitty cat. 

At stake is the successful cleanup of resources and successful termination of your application. This is especially true in an embedded system where it is required to set the hardware being controlled to a well known state before exiting. As an example, if your application's process needs to be restarted as part of an upgrade it will need to provide the new process with the current state of the hardware and leave the controlling bus in a working state. From experience, I2C and SPI busses can be finnicky, especially when the master goes away in the middle of a transaction.  

## Ingredients
The ingredients for a successful multithreaded application are as follows:
- Thread cleanup handlers
- Application thread cancellation points
- Cancellation state and type enabled and deferred (default)
- Correct use of thread synchronization mechanisms mutex and rwlock
- Planned and controlled thread startup and shutdown sequence
- A common API for defining a thread as well as the synchronization mechanisms. 
- Follow the advice of Michael Kerrisk in "The Linux Programming Interface" Chapter 33 "Dealing with Asynchronous Signals Sanely"
	

### Anatomy of a thread
The first stop in the adventure is to ask yourself, what is a thread and what does it mean to be a thread in my application. For most, a thread should have at least the following attributes:
- A way of being initialized
- A way of being started
- A way of being monitored to determine if it has run amok
- A way of being stopped 
- A way of being canceled 
- Mechanisms which ease debugging
- A Name
- A Priority

Abstracting these methods and attributes of the complex Posix Threading API using the Facade Design Pattern simplifies the application, restricts the functionality to a subset defined by the application, and provides a clear set of locations where changes can be made later. 

### Thread Management
For medium to large applications it is not enough to simply define threading behavior, one also needs to take into account how threads coexist within the application. This includes defining the order in which threads start, the order in which threads stop, handling threads which do not start or do not stop in a timely manner, signal handling, and thread monitoring with a watchdog. 

A simple approach would be to provide all of this functionality in a single file. Thread instantiation, thread start, and thread stop can simply be how the associated functions are ordered in the file. The file provides a function to start and stop the threads as well as the other functionality. Unfortunately, with more threads the functions get bigger, look cluttered, and maintainability is reduced as each addition to the file could inadvertently impact other threads requiring closer inspection for regressions during quality analysis.

A slightly more maintainable approach is to build a thread management object in which each of the threads get registered. This is a slightly scaled down Mediator design pattern as it only controls how threads run and not how they communicate. The manager is provided a list of threads with the necessary control instructions; for convenience the threads are tightly coupled with the manager and keep many of these attributes internally. 

# Implementation
In the class diagram below, the intricate details of the Posix Threading functions are integrated into the abstract Thread class. For each thread the application requires, a concrete object of the Thread class is created. Control of these objects is handed over to the singleton ThreadManager object.

![]({{"/assets/images/PosixThreadBlogPost-ClassDiagram.png"}})

## ApplicationThread
An ApplicationThread provides, at a minimum, the necessary implementation required by the Thread class. Once the Thread class and ThreadManager are developed, an application creates all of its long running threads by instantiating objects based on the Thread class and then passing control of them to the ThreadManager. 

### ApplicationThread::prologue
The prologue function is called by the ThreadManager, allowing the thread to instantiate and initialize any variables it may need before it starts. This is the place where objects owned by this thread and modified by other threads must be initialized. Waiting to initialize these shared objects during the run function will be too late as it is a race, another thread may use the shared object before it is fully initialized. By putting the initialization of the shared objects in the prologue one is guaranteed that they will be initialized before any other thread is started; eliminating timely and hard to reproduce errors caused by race conditions. 
```cpp
void ThreadStats::prologue()
{
  mutex_            = mutexCreate();
  wakeUpCondition_  = conditionCreate();
}
```

### ApplicationThread::run
For each thread the application provides the implementation of the run function; this is where all of the interesting work associated with the application gets done. In general this function contains a loop which does not exit. Within the main loop of the run function, the application should call the alive function which performs two essential actions; first it kicks the thread's watch dog and second it acts as a cancellation point. 
```cpp
void ThreadStats::run()
{
  const unsigned int TIMEOUT_S = 10;

  timespec timeout; timeout.tv_sec = TIMEOUT_S; timeout.tv_nsec = 0;

  while( 1 )
  {
    unsigned int data = 0;

    MUTEX_LOCK( mutex_ );
    while( dataQueue_.empty() )
    {
      pthread_cond_timedwait( wakeUpCondition_,
                              mutex_,
                              &timeout );
      alive();
    }
    data = dataQueue_.front(); dataQueue_.pop();

    MUTEX_UNLOCK();

    alive();

    printf ("ThreadStats[%3u] data:%u\n", __LINE__, data );
  }
}
```

### ApplicationThread::epilogue
Upon thread termination the epilogue function is called. In the epilogue function, the application must return any resources allocated; this includes any resources allocated in the prologue function. This is where it is important to remember that you are embracing the tools provided by the Posix threading API; due to cancellation points, any function called may cause the thread to terminate before control reaches back to your run function. For embedded systems it is important to shutdown any interfaces from the epilogue function to leave the hardware and associated busses in a well known working state. 
```cpp
void ThreadStats::epilogue()
{
  if( 0 != mutex_ )
  {
    mutexDestroy( mutex_ );
    mutex_ = 0;
  }

  if( 0 != wakeUpCondition_ )
  {
    conditionDestroy( wakeUpCondition_ );
    wakeUpCondition_ = 0;
  }
}
```

By providing the Thread class facade, each implementor of a thread is spared the complexities of handling these intricacies; increasing maintainability while reducing risk and time to market. 

## Thread
At the heart of this toolbox is the Thread class, it acts as a facade, hiding much of the implementation details of the Posix Threading library and providing a single location where the application's threading rules can be implemented.

The constructor at a minimum should take the name to be associated with the thread in its argument list; this adds significant clarity when displaying debug information associated with the ApplicationThread.

### Thread::alive
The alive function provides the application thread with a mechanism of kicking the watchdog as well as a cancellation point. Additionally this could be a place in which a thread is paused; if another thread was to be restarted, this would require all other threads to become temporarily stopped before the restart occurs. 
```cpp
void Thread::alive()
{
  pthread_testcancel();
  watchDogKick();
}
```

### Thread::stateGet
The stateGet function provides the application with the current state of the thread for debugging purposes. The Thread class works in conjunction with the ThreadManager class, storing the current state among other things. When we dig into the ThreadManager class we will see that threads transition from uninitialized to starting and eventually to stopped. 

### Thread::watchDogSet
The watchDogSet function is used by the thread manager to set and change the watchdog parameters. It could be called by the application thread. However, it is generally better to give up this control to the ThreadManager. For example, the thread manager can automatically determine if it is being run through gdb by looking at the processes status for a TracerPid, then setting the watch dog action to stop instead of exit or abort; giving the program a single obvious place to place a breakpoint. 

### Thread::init
The init function is private to all but the ThreadManager. As part of the process of starting all of the threads, the ThreadManager calls each of the threads' init function in order. This function creates a Posix Thread which waits on a Posix Condition until the start method is called, in this way all of the Posix threads can be created, initialized, and ready to start at a later time. The init function also initializes the thread's watch dog and calls the prologue function described earlier. At the successful end of the init function the Thread Manager has successfully allocated and initialized the thread's resources before moving on to the next thread. 

### Thread::start
The start function is another function private to the package; it is called by the Thread Manager to allow the thread to run, which it does, by signaling the start condition created in the init function. Upon receiving the start signal the Thread object pushes the threadCleanup function onto the thread stack, more on this later, and calls the run function provided by the child ApplicationThread. If a cancellation point is encountered within the run function, the threadCleanup function will be run when the operating system pops it off of the thread stack. Otherwise, if the run function were to exit naturally, the start function explicitly pops and executes the cleanup function.
```cpp
void Thread::start()
{
  {
    MUTEX_LOCK( mutex_ );
    state_ = STATE_STARTING;
    MUTEX_UNLOCK();
  }

  pthread_cond_signal( &threadStartCond_ );
}
```

### Thread::stop
With the work to handle an unexpected exit of a thread complete, all the stop function needs to do is to cancel the thread with a call to pthread_cancel.
```cpp
void Thread::stop()
{
  if( STATE_STOPPED != state_ )
  {
    state_ = STATE_STOPPING;
    int ret = pthread_cancel( thread_ );
  }
}
```

### Thread::threadFunction
Although the Posix Thread is created in the init function, at that point, the system is not ready for the thread's run function to start yet. This is where the threadFunction comes in; it is run by the Posix Thread upon creation. This function waits for a condition variable to be signaled before pushing the epilogue onto the thread cleanup stack then calling run. This is the heart of creating a deterministic environment for starting threads; guaranteeing all objects associated with all of the application's threads are instantiated before any of the threads gets a chance to run.
```cpp
void Thread::threadFunction()
{
  {
    MUTEX_LOCK( mutex_ );
    while( state_ != STATE_STARTING )
    {
      int ret = pthread_cond_wait( &threadStartCond_, mutex_ );
      if( 0 != ret )
      {
        printf( "Thread wait start failed (%d): %s\n", 
                ret, name_.c_str() );
        state_ = STATE_STOPPED_ERROR;
        abort();
      }
    }
    MUTEX_UNLOCK();
  }

  state_ = STATE_RUNNING;

  pthread_cleanup_push( Thread::threadCleanupPosixCallback, this );

  run();

  state_ = STATE_STOPPING;

  pthread_cleanup_pop( 1 );
}
```

### Thread::watchDogKick
The watchDogKick function arms the timer associated with the watchdog to trigger at a predetermined time relative to the current time and is configured to expire once by setting the it_value and it_interval to 0.

### Posix callback functions
The last 3 functions (threadRunPosixCallback, threadCleanupPosixCallback, and watchDogTimeoutPosixCallback) provide an interface to the Posix Thread's C API into our C++ Thread object; in order to provide this interface, each of these functions must be static and the pointer to the instance of our C++ thread must be stored in the underlying Posix C data structure. When registering with pthread_create and pthread_cleanup_push the Thread instance is associated with the callback through the argument arg; for the watch dog the Thread's instance is stored as part of the signal event associated with the timer. 
```cpp
void* Thread::threadRunPosixCallback( void* instance )
{
  ((Thread*)instance)->threadFunc();
  return( 0 );
}
```

## ThreadManager
The ThreadManager object is the conductor of this symphony of threads. It dictates when each of the threads starts, when each of the threads stop, how each of the threads responds to a watchdog event, and can provide the handling of signals into the application's process. 

The ThreadManager is implemented as a singleton, that is to say, there is only one ThreadManager per application. The life of the application is generally to setup shared resources (e.g. logging, hardware), instantiate and add threads to the ThreadManager, and then start the threads.

### ThreadManager::add
Once an ApplicationThread is instantiated, the add function is called by the application with a pointer to the new object, the order in which the Thread should be started, the default watchdog timeout, as well as the default watchdog action. Once called, the threads are put into an ordered list, the default watchdog settings are configured, and the state of the thread is set to a default value. 

### ThreadManager::start
Once all of the application's threads have been added to the ThreadManager, the application calls the start function. The start function first iterates through all of the threads calling the init function of each. At this point a mapping of name, Posix Thread ID (Light Weight Process), and ApplicationThread can be made for future debug and control. As mentioned previously, on each successful call to init by the start function, each of the Thread object's prologue functions will be called, setting up the required resource for the thread to operate with the other threads in the system. Once all threads have been initialized the list of threads is iterated through once again, this time calling the Thread object's start function. The ThreadManager start function then waits for each thread to complete, never returning back to the application until all the threads are stopped.

![]({{"/assets/images/PosixThreadBlogPost-StateDiagram.png"}})

## Threading Operations
For consistency, common mechanisms for working with condition variables, mutex locks, and read-write locks should be created for use throughout the application. 
 
```cpp
pthread_cond_t* conditionCreate()
{
  pthread_cond_t*     condition;
  pthread_condattr_t  conditionAttributes;

  condition = new pthread_cond_t;

  pthread_condattr_init( &conditionAttributes );
  pthread_condattr_setclock( &conditionAttributes, CLOCK_MONOTONIC );
  pthread_cond_init( condition, &conditionAttributes );
  pthread_condattr_destroy( &conditionAttributes );

  return( condition );
}

int conditionDestroy( pthread_cond_t* condition )
{
  int ret = pthread_cond_destroy( condition );
  return( ret );
}
```

# Conclusion
Below you will find a simple example application which was written against the mechanisms described in this post. For large applications, it is highly recommended to break the threads into their own files. As with all examples posted, error handling has been removed to maintain brevity in an effort to increase readability. 

Whether you are building a carrier grade product or a weekend DIY home automation device, with some advance planning and embracing of the tools provided, long lived threaded applications can be both efficient and maintainable. Codified rules used provide the consistency required to avoid hair pulling - hard to find bugs.

If you have thoughts, questions, or suggestions feel free to contact me. 

-W Parker MacKenzie, 2018

# Simple Example Application
```cpp
#include <queue>
#include <stdio.h>
#include <string>
#include <threadManager.hxx>
#include <unistd.h>

class ThreadStats;

class ThreadGenerate : public Thread
{
public:
  ThreadGenerate( const std::string& name,
                  int                relativePriority,
                  unsigned int       watchDogTimeout_ms,
                  WDAction           watchDogAction,
                  ThreadStats&       threadStats );
  virtual ~ThreadGenerate();

  void epilogue();
  void prologue();
  void run();

private:
  unsigned int    count_;
  ThreadStats&    threadStats_;
};

class ThreadStats : public Thread
{
public:
  ThreadStats( const std::string& name,
               int                relativePriority,
               unsigned int       watchDogTimeout_ms,
               WDAction           watchDogAction );
  virtual ~ThreadStats();

  void epilogue();
  void prologue();
  void run();
  void wakeUp( unsigned int data );

private:
  std::queue< unsigned int >  dataQueue_;
  pthread_mutex_t*            mutex_;
  pthread_cond_t*             wakeUpCondition_;
};

ThreadGenerate::ThreadGenerate( const std::string& name,
                                int                relativePriority,
                                unsigned int       watchDogTimeout_ms,
                                WDAction           watchDogAction,
                                ThreadStats&       threadStats ):
  Thread( name, relativePriority, watchDogTimeout_ms, watchDogAction ),
  count_( 0 ),
  threadStats_( threadStats )
{
}

ThreadGenerate::~ThreadGenerate()
{}

void ThreadGenerate::epilogue()
{}

void ThreadGenerate::prologue()
{}

void ThreadGenerate::run()
{
  timespec sleepTime; sleepTime.tv_sec = 0; sleepTime.tv_nsec = 500000000;
  bool doSleep = true;
  timespec sleepTimeRemain;

  while( 1 )
  {
    alive();

    printf ("ThreadGenerate[%3u] count:%u\n", __LINE__, count_ );
    threadStats_.wakeUp( count_++ );

    sleepTimeRemain = sleepTime; doSleep = true;
    while( doSleep )
    {
      int ret = nanosleep( &sleepTimeRemain, &sleepTimeRemain );
      if( 0 == ret ) doSleep = false;
      else if( errno != EINTR ) printf( "Danger will robinson...%d\n", errno );
    }
  }
}

ThreadStats::ThreadStats( const std::string&  name,
                          int                 relativePriority,
                          unsigned int        watchDogTimeout_ms,
                          WDAction            watchDogAction ) :
Thread( name, relativePriority, watchDogTimeout_ms, watchDogAction )
{
}

ThreadStats::~ThreadStats()
{
}

void ThreadStats::epilogue()
{
  if( 0 != mutex_ )
  {
    mutexDestroy( mutex_ );
    mutex_ = 0;
  }

  if( 0 != wakeUpCondition_ )
  {
    conditionDestroy( wakeUpCondition_ );
    wakeUpCondition_ = 0;
  }
}

void ThreadStats::prologue()
{
  mutex_            = mutexCreate();
  wakeUpCondition_  = conditionCreate();
}

void ThreadStats::run()
{
  const unsigned int TIMEOUT_S = 10;

  timespec timeout; timeout.tv_sec = TIMEOUT_S; timeout.tv_nsec = 0;

  while( 1 )
  {
    unsigned int data = 0;

    MUTEX_LOCK( mutex_ );
    while( dataQueue_.empty() )
    {
      pthread_cond_timedwait( wakeUpCondition_,
                              mutex_,
                              &timeout );
      alive();
    }
    data = dataQueue_.front(); dataQueue_.pop();

    MUTEX_UNLOCK();

    alive();

    printf ("ThreadStats[%3u] data:%u\n", __LINE__, data );
  }
}

void ThreadStats::wakeUp( unsigned int data )
{
  MUTEX_LOCK( mutex_ );
  dataQueue_.push( data );
  MUTEX_UNLOCK();
  pthread_cond_signal( wakeUpCondition_ );
}


enum 
{
  THREAD_FIRST       = 0,
  THREAD_RECEIVE     = THREAD_FIRST,
  THREAD_GENERATE,
  THREAD_STATS,
  THREAD_RESERVED_LAST
};

struct ThreadAttributes
{
  std::string       name_;
  unsigned int      startOrder_;
  int               relativePriority_;
  unsigned int      watchDogTimeout_ms_;
  Thread::WDAction  watchDogAction_;
};

ThreadAttributes threadAttributes[ THREAD_RESERVED_LAST ] = {
// name         startOrder  relativePriority  watchDogTImeout_ms    watchDogAction   
{ "tReceive",           0,                0,                 600,   Thread::WDACTION_ABORT },
{ "tGenerate",          1,                0,                 500,   Thread::WDACTION_ABORT },
{ "tStats",             2,                0,                 500,   Thread::WDACTION_ABORT } };


int main()
{
  ThreadGenerate* threadGenerate      = 0;
  ThreadStats*    threadStats         = 0;

  threadStats = 
    new ThreadStats( threadAttributes[THREAD_STATS].name_,
                     threadAttributes[THREAD_STATS].relativePriority_,
                     threadAttributes[THREAD_STATS].watchDogTimeout_ms_,
                     threadAttributes[THREAD_STATS].watchDogAction_ );
  
  ThreadManager::add( threadStats, 
                      threadAttributes[THREAD_STATS].startOrder_ );

  threadGenerate = 
    new ThreadGenerate( threadAttributes[THREAD_GENERATE].name_,
                        threadAttributes[THREAD_GENERATE].relativePriority_,
                        threadAttributes[THREAD_GENERATE].watchDogTimeout_ms_,
                        threadAttributes[THREAD_GENERATE].watchDogAction_,
                        *threadStats );
  
  ThreadManager::add( threadGenerate, 
                      threadAttributes[THREAD_GENERATE].startOrder_ );


  ThreadManager::start();
  return( 0 );
}
```