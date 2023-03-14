---
layout: posts
title:  "Explicit Time Testing is Essential for Reliable Software: RSpec Best Practices"
tags: ruby, rails, active-record, rspec, testing
---


Testing is an essential part of any software development process, and RSpec is one of Ruby's most popular testing frameworks. If you’re working on a project with a lot of legacy code, like I have been, refactoring the code and improving the test suite can make the process more enjoyable for everyone.

I’ve learned over the years that using Time.now your test files can be a recipe for disaster. It's a code smell that stands out because I used to do it too. But after reading some great books from Uncle Bob, Kent Beck, and the Martin Series, I learned that testing should be explicit and deterministic.

Testing is not “the real world,” and while there are arguments to be made for using implicit specs, these are in the minority. So, I would like to highlight why using `Time.now` should be avoided, how you can ease into using explicit times, and plan your specs around explicit times.

```ruby
RSpec.describe Book, type: :model do

  let(:now) { Time.new(2023,01,01,0,0,0) } # Strict -- never changes. 
  let(:actually_now) { Time.now } # NDT - Changes every test run. 

end
```

Deterministic tests (DTs) are a crucial principle in testing. These tests guarantee that the same output will be produced every time the test is run with the same input, ensuring that the tested system or algorithm is functioning correctly and consistently.

Using Time.now in tests can lead to non-deterministic tests (NDTs). These tests may produce different outputs, even with the same input each time. This can happen due to the nature of Time.now, which changes every time the test is run, leading to inconsistent inputs.
Working with time objects can be tricky, and incorporating time zones into the mix can make it even more complex. Anyone who has ever dealt with a flaky test knows that it can be a can of worms.

To avoid these issues, it’s recommended to set and test against time objects that you control explicitly. This will lead to better DTs and ensure that the tests are more reliable and easier to maintain.

At some point, a test that relies heavily on Time.now will likely fail at some point. This can lead to false test failures or test that pass on one machine but fail on another. The test suite becomes unreliable, and in almost every case where I had a flaky test that revolved around a time mismatch, it was because an NDT used `Time.now` vs. explicitly setting the time. By using exact times, our test becomes more reliable, consistently producing the same results and allowing us the control to change it to test some other aspect.

Ensure that your tests are deterministic. I will say that a few times because it’s worth repeating. Also, remember to keep your tests focused and straightforward. This means that each test should only test one specific aspect of the code and not include any unnecessary elements.

Another best practice when testing times is to avoid making unnecessary time adjustments. This means that you should only adjust the time in your tests if it is vital for the test to pass. For example, if you are testing a background job that runs at a specific time, it may be necessary to adjust the Time to ensure that the job runs at the expected time. However, if you are testing a feature that is not time-sensitive, it is best to avoid adjusting the time in your tests.

RSpec provides several valuable tools to help you test times, including shared contexts and shared examples. Shared contexts allow you to share code between multiple tests, which can be particularly useful when testing times. For example, if you have many tests that need to use the same time value, you can define a shared context that sets the Time and then includes that context in all of your tests. Shared examples are similar to shared contexts but are intended to be used for multiple examples within the same example group.

One of the most useful RSpec time helpers is travel, which allows you to temporarily change the current time in your tests. You can use travel to fast forward, rewind Time, or set Time to a specific point in the past or future. Here’s an example of how you might use travel to test a method that calculates the number of days until a specific date:

```ruby 
describe "#days_until_event" do
  it "returns the number of days until the event" do
    event_date = Time.new(2022, 12, 25)
    travel_to(Time.new(2022, 12, 20))
    expect(days_until_event(event_date)).to eq(5)
  end
end
```
Another practical RSpec time helper is freeze_time, which allows you to freeze time at a specific point. This is useful when testing code that relies on the current time, but you want to ensure that the time remains constant throughout the test. Here's an example of how you might use freeze_time to test a method that calculates the number of seconds since the start of the year:

```ruby
describe "#seconds_since_year_start" do
  it "returns the number of seconds since the start of the year" do
    freeze_time(Time.new(2022, 5, 15))
    expect(seconds_since_year_start).to eq(131_907_200)
  end
end
```
Finally, the helper travel_back is the inverse of travel, it allows you to change the time in your tests to a point in the past. Here's an example of how you might use travel_back to test a method that calculates the number of days since a specific date:

```ruby
describe "#days_since_event" do
  it "returns the number of days since the event" do
    event_date = Time.new(2022, 12, 25)
    travel_back(10)
    expect(days_since_event(event_date)).to eq(10)
  end
end

```
In conclusion, RSpec’s built-in time helpers, travel, travel_to, travel_back and freeze_time make it easy to test time-sensitive code and ensure that your tests are consistent and reliable. These helpers are very simple to use, and you can use them in any RSpec test to control the current time and make your tests more robust. Avoid using Time.now — remember to make your tests DTs and not NDTs. Explicitly test what you need too — and happy coding!