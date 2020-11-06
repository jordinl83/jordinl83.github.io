---
layout: post
title:  How to run periodic jobs with Sidekiq for free
date:   2018-08-06
---

One of the features not included in the free version of **Sidekiq** is running periodic jobs (or cron jobs or recurring jobs). There are gems that offer this functionality as an add-on, but it turns out that it can be implemented in less that 40 lines of code, so there is no need to pull an additional bloated dependency.

All we need is a _**Scheduler**_ worker that when you run it:

* It knows when to run other workers.
* It re-schedules itself to be executed after some time (1 minute, 10 minutes, 1hour, … whatever is necessary).

Also, we need to kick off the **_Scheduler_** worker for the first time in the **Sidekiq** initializer.

Below you can find the implementation for the **_SchedulerWorker_** class and the changes to the **Sidekiq** initializer. With the current implementation of the **_SchedulerWorker_** it will run every minute and check if there are workers that must be run or not. The workers are configured in the **_SCHEDULE_** hash, containing the periodic workers as keys and the values are lambdas that return true when the worker must be run. For instance, in the code below, **_FirstWorker_** will run 5 minutes after the hour every hour. **_SecondWorker_** will run every day at 9AM. **_ThirdWorker_** will run every 10 minutes.

Enjoy.

```ruby
# app/workers/scheduler_worker.rb

class SchedulerWorker
  include Sidekiq::Worker
  sidekiq_options queue: 'critical'

  SCHEDULE = {
    FirstWorker  => -> (time) { time.min == 5 },
    SecondWorker => -> (time) { time.min == 0 && time.hour == 9 },
    ThirdWorker  => -> (time) { time.min % 10 == 0 },
  }

  def perform
    execution_time = Time.zone.now
    execution_time -= execution_time.sec

    self.class.perform_at(execution_time + 60) unless scheduled?

    SCHEDULE.each do |(worker_class, schedule_lambda)|
      worker_class.perform_async if !scheduled?(worker_class) && schedule_lambda.call(execution_time)
    end
  end

  def scheduled?(worker_class = self.class)
    scheduled_workers[worker_class.name]
  end

  private

  def scheduled_workers
    @scheduled_workers ||= Sidekiq::ScheduledSet.new.entries.each_with_object({}) do |item, hash|
      hash[item['class']] = true
    end
  end
end
```

```ruby
# config/initializers/sidekiq.rb

Sidekiq.configure_server do |config|
  config.on(:startup) do
    SchedulerWorker.perform_async unless SchedulerWorker.new.scheduled?
  end
end
```
