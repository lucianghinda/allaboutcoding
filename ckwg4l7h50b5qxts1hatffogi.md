## Simple guidelines for working with Time and TimeZones for APIs

Working with DateTime or Time and timezones can be sometimes confusing. 

Here are some guidelines that I defined while working with dates/times. They are working well in Rails (meaning Rails is already doing some requires transformations especially when saving/reading from DB via ActiveRecord) but they can be applied easily to any other framework: 


**1) The request body should accept time only in format ISO8601 and the time should be defined in reference to UTC+0 **


Example in JSON: 

```json
{
  "sent_at": "2021-11-26T07:23:16Z"
}
``` 
That "Z" at the end means that time is referenced to UTC timezone. 

Why ISO8601 and not Unixtimestamp? 

First, because it is human readable. Unpopular opinion: I think the API format should be designed to be easy to read and understand by people. Even if it is consumed by computers, the most errors will come from the way other developers will use it in their code. So making the response body as human-friendly as possible is a good way to ensure better and compliant usage. 

**2) The request body should always require a time_zone attribute to be defined**

Example in JSON using time zone as region/country/city/area: 

```json
{
  "sent_at": "2021-11-26T07:23:16Z",
  "time_zone": "Europe/Bucharest"
}
```
or 

Example in JSON with time zone defined as an offset from UTC: 

```json
{
  "sent_at": "2021-11-26T07:23:16Z",
  "time_zone": "+02:00"
}
```

If you are implementing an API with Ruby on Rails I suggest implementing the version with time zone as region/country/city/area. Rails already have very good  [support for timezones](https://api.rubyonrails.org/classes/ActiveSupport/TimeZone.html). 

I anyhow recommend using it like this if you plan to show to a dropdown with time_zones for a user to choose. 

Why?
- it is easier for users to choose a region/country/city/area than to choose a time difference
- if you plan to save and retrieve the time zone and show it to the end-user then saving +02:00 will not help a lot to know what time zone they choose. For example, if they are choosing Europe/Berlin and you save +01:00 then when retrieving it you will not know if they choose Europe/Berlin or Europe/Amsterdam or Europe/Paris. They all are +01:00 UTC. 

**3) When saving to DB always save as UTC**

No matter if you choose to follow the first two guidelines, I recommend adopting this one. Save to the DB the time as UTC no matter how you accept the request body. 
In case you are using Rails with Active Record then it will do this automatically for you.

---

The following two guidelines are only for Ruby and Ruby on Rails developers: 

**4) Use only `Time.current` and do not use `Time.now`.** If you need time in a timezone use `Time.current.in_time_zone`. 

**5) Do not use `DateTime`**. It is deprecated - see  [here](https://ruby-doc.org/stdlib-3.0.2/libdoc/date/rdoc/DateTime.html) . Use `Time` instead. 

For more details and examples about how to work with Time and Timezones I really recommend this article from Thoughtbot:  [It's About Time (Zones)](https://thoughtbot.com/blog/its-about-time-zones)  




