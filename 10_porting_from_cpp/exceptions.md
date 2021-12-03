# Exception Handling / Safety

There are no hard and fast rules for when a function in C++ should throw an exception and when it should return a code. So one codebase may have a tendency to throw lots of exceptions while another might throw none at all.

Therefore, the question of when it is right to throw an exception (or not) is often debated. The general rule of thumb is to throw exceptions for truly exceptional
circumstances outside of the program's ability to control, e.g. out of disk space.

As we'll see below, there are caveats about where and how exceptions should thrown.

Aside from that, code may or may not be exception safe. That is, it may or may not free up its resources if it suffers an exception. Articles have been
written to describe the levels of guarantees that code can aim for with [exception safety](http://www.boost.org/community/exception_safety.html).

## Constructors

You may also be advised to throw exceptions in constructors because there is no easy way to signal the object is an error otherwise
except to set the new object into some kind of zombie / dead state via a flag that has to be tested.

```c++
DatabaseConn::DatabaseConn() {
  db_ = connect();
  if (db_ == NULL) {
    throw string("The database connection is null");
  }
}

// These both recover their memory
DatabaseConn db1;
DatabaseConn *db2 = new DatabaseConn();
```

But if DatabaseConn() had allocated some memory before throwing an exception, this would NOT be recovered and so ~DatabaseConn would have to clean it up.

```c++
DatabaseConn::DatabaseConn() {
  buffer_ = new char[100];
  // ... exception throwing code
}

DatabaseConn::~DatabaseConn() {
  if (buffer_) {
    delete[] buffer_;
  }
}
```

But if we waited until after the exception throwing to allocate memory then maybe buffer_ is not set to NULL, so we'd have to ensure we initialised it to NULL.

```c++
DatabaseConn::DatabaseConn() : buffer_(NULL) {
  // ... exception throwing code
  buffer_ = new char[100];
}
```

## Destructors

But you will be advised NOT to throw exceptions in destructors because throwing an exception during a stack unwind from handling another exception is fatal.

```c++
BadNews::~BadNews() {
    if (ptr == NULL) {
      throw string("This is a bad idea");
   }
}
```

## How Rust helps

Rust doesn't have exceptions so issues surrounding exceptions cannot happen. The recommended way of dealing with errors is to use the `Option` and `Result` types to formally pass errors up through your call chain.

The nuclear option for truly exceptional situations is the `panic`. This is a little like an uncaught exception in that it will cause the entire thread to unwind. If the main thread panics then the process terminates.

A `panic!()` can be caught in some situations and recovered from in some scenarios but it should still be thought of as the nuclear option.

Lacking exceptions might seem a bad idea but C++ demonstrates that they come with a whole raft of considerations of their own.
