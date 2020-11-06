---
layout: post
title:  How to calculate business time between two times efficiently
date:   2017-03-05
---

Given a company’s work schedule and two arbitrary times in two arbitrary days, we want to calculate the amount of time between these two times that falls inside the company’s work schedule (accounting for holidays too).

### Example

Given a company based in the United States that opens Monday-Friday 9:00–17:00, how much time is there between Monday November 21st at 14:00 and Thursday December 1st at 11:00? Notice that Thursday November 24th is Thanksgiving and the company is not open:

![Calendar](/assets/img/business-time/calendar.png)

So, in this case, we have:

* Nov 21st: 3 hours from 14:00 to 17:00.
* 6 full working days: 6 * 8 hours = 48 hours.
* Thursday Dec 1st: 2 hours from 9:00 to 11:00.

Total: 53 hours.

![Hours](/assets/img/business-time/hours.png)

### Brute force solution

Iterate through all the dates in the interval:

* If the date is not a weekday count 0.
* If the date is a holiday count 0.
* If the date is a weekday and not a holiday calculate business hours in day. For dates days outside of the edges of the interval it’s just closing time-opening time. For dates on the edges we would need to calculate the intersection.

**Complexity**: As you can see we need to loop through all dates inside of the interval and for each of the dates we need to check if it’s a holiday. Therefore the complexity is NxM, where N is the distance between the dates and M is the length of the array that contains the holidays. We could use a Hash to store the holidays and this would take down the complexity to linear.

### Optimal solution

#### Step 1

Calculate business time inside the interval excluding the edges, this way we can count full working days and weeks.

Let’s call _W_ the set of _weekdays_ between two dates. And let’s call _H_ the set of _holidays_ between two dates. The number of _workdays_ between these two dates will be _W-(W∩H)_.

![Intersection](/assets/img/business-time/intersection.png)

**We now the _schedule_ ahead of time, therefore we can calculate the following before hand:**

* How much business time there is during a natural week. (eg 40 hours)
* How much time there is between any combination of two weekdays, for instance, between Monday and Tuesday there are 16 hours, Monday and Wednesday there are 24 hours, Sunday and Wednesday there are 24 hours, …

This way we can calculate the total time during workdays as _(number of weeks x hours per week) + (hours between weekday of first day and weekday of last day)_. And therefore we can calculate it in constant time.

**Calculate amount of time during holidays on weekdays between two dates:**

We will also know the _holidays_ ahead of time and we will do some calculations ahead of time too. We will iterate through every date between _first holiday_ and _last holiday_ and we will calculate the following:

* Whether that date is a holiday.
* Number of _“business hours”_ that occurred on _holidays_ before that date. These are not actual business hours, but they would’ve been if they hadn’t been on a holiday.

So the pseudo-code for this calculation would look like this:

```
holidays = [...] // we are passed an array with all the holidays
holiday_hours = 0
holiday_info = {}for date in range(holidays.first, holidays.last) do
  holiday = date.inside?(holidays)
  holiday_info[date] = {
    holiday: holiday,
    holiday_hours_before: holiday_hours
  }  if holiday and date.weekday? do
    holiday_hours = holiday_hours + business_hours_in(date)  
  end
end  
```

This way if we want to calculate how many _“business hours”_ fell during _holidays_ between _date1_ and _date2_, we will just need to calculate:

```
holiday_info[date2 + 1] - holiday_info[date1]
```

Which can be calculated in constant time.

#### Step 2

Calculate business time on the edges of the interval:

* If the edge doesn’t fall on a weekday count 0.
* If the edge is a holiday count 0.
* Otherwise, for left edge calculate hours between _left edge start time_ and _end of business day_. For _right edge_ calculate hours between _start of business day_ and _right edge end time_.

All this can be calculated in constant time too.
