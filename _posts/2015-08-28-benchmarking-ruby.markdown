---
layout: post
title: Benchmarking Ruby
summary: How to benchmark ruby code and compare different solutions to a problem
date: 2015-08-28
published: true
categories: ruby
---

Benchmarking Ruby code is essential for improving the performance of applications. The Ruby
Standard Library provides a [Benchmark module](http://ruby-doc.org/stdlib-2.5.0/libdoc/benchmark/rdoc/Benchmark.html)
that can be used to measure the running time of any Ruby code block.

First, lets require it:

```rb
require 'benchmark'
# => true
```

#### Benchmarking a single block of code

For benchmarking simple blocks of code, use the `#measure` method:

```rb
puts Benchmark.measure { 10_000_000.times { Object.new } }
#        user     system      total        real
# => 1.280000   0.000000   1.280000 (  1.283235)
```

The result is in seconds, so instantiating 10 million objects takes a little more than a second.

If we need to print a custom message, we can capture the result and process it:

```rb
result = Benchmark.realtime { 10_000_000.times { Object.new } }

puts "Creating ten million objects: #{realtime.round(2)}s"
# => Creating ten million objects: 1.29s
```

#### Comparing several blocks

We often need to compare several approaches to a problem in order to find out the best one. Let's
say we want to compare the following methods for finding the n-th fibonacci number:

```rb
# DP version
def fib_dp(n)
  (2..n).reduce([0, 1]) { |m| m << m.last(2).reduce(:+) }[n]
end

# Recursive version
def fib_rec(n)
  return 0 if n == 0
  return 1 if n == 1

  fib_rec(n - 1) + fib_rec(n - 2)
end
```

The unoptimized recursive version is very slow and will choke on values larger than 40. We can use
the `#bm` method to measure just how much slower it is:

```rb
Benchmark.bm(10) do |x|
  x.report('dp:')        { fib_dp(35) }
  x.report('recursive:') { fib_rec(35) }
end

#                  user     system      total        real
# dp:          0.000000   0.000000   0.000000 (  0.000035)
# recursive:   1.680000   0.000000   1.680000 (  1.671631)
```

The first argument to `#bm` defines the label width and larger values will shift the results
further to the right. Anyway, we got what we wanted - the recursive function takes more than a
second and a half and the DP version takes a negligible amount of time. In fact, it can calculate
very large fibonacci numbers in very little time:

```rb
puts Benchmark.measure { fib_dp(100_000) }
#        user     system      total        real
# => 0.350000   0.000000   0.350000 (  0.350249)
```

There is another method in the Benchmark library called `#bmbm` which runs the tests twice - the
first time to warm up the runtime environment and the second time to measure the results. You will
want to use this method if you worry that the order of execution of the different code blocks will
have an effect on their execution time.

#### Using benchmark-ips

The [benchmark-ips](https://github.com/evanphx/benchmark-ips) gem provides even more features than
the default Benchmark module. Install it and then require it in your program:

```rb
require 'benchmark/ips'
```

Let's use it to test our previous methods:

```rb
Benchmark.ips do |x|
  x.report('dp: ')        { fib_dp(35) }
  x.report('recursive: ') { fib_rec(35) }

  x.compare!
end

# Calculating -------------------------------------
#                 dp:      5.600k i/100ms
#          recursive:      1.000  i/100ms
# -------------------------------------------------
#                 dp:      60.299k (± 2.0%) i/s -    302.400k
#          recursive:       0.517  (± 0.0%) i/s -      3.000  in   5.800686s
#
# Comparison:
#                 dp: :    60299.5 i/s
#          recursive: :        0.5 i/s - 116590.15x slower
```

The only difference is the `x.compare!` call at the end, but we get a lot more valuable information
about the performance of the two methods - iterations per second, standard deviation and finally a
comparison which shows that the unoptimized recursive version is more than 100 000 times slower
than the DP version!

There are a few more options available, like setting the time for the warmup and calculation phases
or creating a custom suite.

#### Conclusion and further reading

If you ever wondered if `Enumerable#each` is faster than a `for` loop or if `[].map.flatten` is
slower than `[].flat_map`, take a look at the
[fast-ruby](https://github.com/JuanitoFatas/fast-ruby) repo. It contains answers to those questions
and to many others.

The Benchmark module and benchmark-ips gem are really easy to use and provide great information
about the performance of your code. They are an essential tool in every developers toolbelt.
