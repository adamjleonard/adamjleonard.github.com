---
layout: post
title: Refactoring - Making use of database functions
category: posts
---

We've been doing some heavy performance optimization on the MoviePass
application. As I was working on improving the performance of a users
check-in, I noticed a method that was pulling in all the check-ins
for the day and calculating the sum of the ticket prices.

The code looked like this

{% highlight ruby %}
  def self.cumulative_price_for_checkins_today
    updated_today.inject(0) { |sum, res| sum + (res.ticket_price || 0) }
  end
{% endhighlight %}

There would be nothing wrong with this method if we were not pulling in
a bunch of **ActiveRecord** models. We are doing a lot of unnecessary work that 
our database could be doing for us. Let's be honest any thing that we can move 
away from Ruby is usually a speed advantage for us.

MySQL provides us with a bunch of neat functions, but obviously the one
we want to use is **SUM**. The **SUM** function takes the column name and
optionally the **DISTINCT** keyword as arguments. The **DISTINCT** keyword will 
cause only unique values to be sumed.

Rails provides us with an easy way to do this. Simply call the **sum**
method on an **ActiveRecord::Relation** class, while passing the column as a
parameter and it will handle the rest.

So the performance refactoring of this method ended up looking like

{% highlight ruby %}
  def self.cumulative_price_for_checkins_today
    updated_today.sum(:ticket_price)
  end
{% endhighlight %}

This is a much better solution than what we had before. We could obviously do better 
by incrementing the cumulative price in a store such as redis after each
movie ticket was redeemed. Then querying for the value within this method. That would remove the need for going to MySQL for the value, unless it was a failover.

**Quick Fact**: If you were to execute the **SQL** this method generates,
MySQL would return **NULL** and not **0**. Rails returns **0**, if the result
happens to be **NULL**. You can see this happening on line 5.

**activerecord/lib/active_record/calculations.rb**, line **296**
{% highlight ruby linenos %}
  def type_cast_calculated_value(value, column, operation = nil)
    if value.is_a?(String) || value.nil?
      case operation.to_s.downcase
      when 'count' then value.to_i
      when 'sum'   then type_cast_using_column(value || '0',
        column)
      when 'avg' then value.try(:to_d)
      else type_cast_using_column(value, column)
      end
    else
      value
    end
  end
{% endhighlight %}

I will update this post after doing some benchmarks on the performance
difference.
