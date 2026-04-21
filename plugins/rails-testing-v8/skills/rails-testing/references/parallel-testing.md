# Parallel Testing

Covers how to parallelize the Rails test suite to reduce total run time. Mirrors the **Parallel Testing** section of the Rails 8.1.3 testing guide (`guides/source/testing.md` §2629–2758).

Running tests in parallel reduces the time it takes your entire test suite to run. While forking processes is the default method, threading is supported as well.

## Parallel Testing with Processes

The default parallelization method is to fork processes using Ruby's DRb system. The processes are forked based on the number of workers provided. The default number is the actual core count on the machine, but can be changed by the number passed to the `parallelize` method.

To enable parallelization add the following to your `test_helper.rb`:

```ruby
# test/test_helper.rb
class ActiveSupport::TestCase
  parallelize(workers: 2)
end
```

The number of workers passed is the number of times the process will be forked. You may want to parallelize your local test suite differently from your CI, so an environment variable is provided to be able to easily change the number of workers a test run should use:

```bash
$ PARALLEL_WORKERS=15 bin/rails test
```

When parallelizing tests, Active Record automatically handles creating a database and loading the schema into the database for each process. The databases will be suffixed with the number corresponding to the worker. For example, if you have 2 workers the tests will create `test-database-0` and `test-database-1` respectively.

If the number of workers passed is 1 or fewer the processes will not be forked and the tests will not be parallelized and they will use the original `test-database` database.

Two hooks are provided, one runs when the process is forked, and one runs before the forked process is closed. These can be useful if your app uses multiple databases or performs other tasks that depend on the number of workers.

The `parallelize_setup` method is called right after the processes are forked. The `parallelize_teardown` method is called right before the processes are closed.

```ruby
# test/test_helper.rb
class ActiveSupport::TestCase
  parallelize_setup do |worker|
    # setup databases
  end

  parallelize_teardown do |worker|
    # cleanup databases
  end

  parallelize(workers: :number_of_processors)
end
```

These methods are not needed or available when using parallel testing with threads.

## Parallel Testing with Threads

If you prefer using threads or are using JRuby, a threaded parallelization option is provided. The threaded parallelizer is backed by minitest's `Parallel::Executor`.

To change the parallelization method to use threads over forks put the following in your `test_helper.rb`:

```ruby
# test/test_helper.rb
class ActiveSupport::TestCase
  parallelize(workers: :number_of_processors, with: :threads)
end
```

Rails applications generated from JRuby or TruffleRuby will automatically include the `with: :threads` option.

> **NOTE:** As in the section above, you can also use the environment variable `PARALLEL_WORKERS` in this context, to change the number of workers your test run should use.

## Testing Parallel Transactions

When you want to test code that runs parallel database transactions in threads, those can block each other because they are already nested under the implicit test transaction.

To workaround this, you can disable transactions in a test case class by setting `self.use_transactional_tests = false`:

```ruby
# test/models/worker_test.rb
class WorkerTest < ActiveSupport::TestCase
  self.use_transactional_tests = false

  test "parallel transactions" do
    # start some threads that create transactions
  end
end
```

> **NOTE:** With disabled transactional tests, you have to clean up any data tests create as changes are not automatically rolled back after the test completes.

## Threshold to Parallelize tests

Running tests in parallel adds an overhead in terms of database setup and fixture loading. Because of this, Rails won't parallelize executions that involve fewer than 50 tests.

You can configure this threshold in your `test.rb`:

```ruby
# config/environments/test.rb
config.active_support.test_parallelization_threshold = 100
```

And also when setting up parallelization at the test case level:

```ruby
# test/test_helper.rb
class ActiveSupport::TestCase
  parallelize threshold: 100
end
```

See also: [`test-database.md`](test-database.md) for `use_transactional_tests` and the implicit per-test transaction.
