---
title: "Avoid Microsecond Pitfalls When Comparing Times in Tests"
seoTitle: "Avoid Errors in Time Comparison Tests"
seoDescription: "Learn how to avoid pitfalls in time comparisons in tests using the iso8601 method for consistency and reliability"
datePublished: Fri Oct 10 2025 03:58:21 GMT+0000 (Coordinated Universal Time)
cuid: cmgkbgm0y000002l5aj0770rq
slug: avoid-microsecond-pitfalls-when-comparing-times-in-tests
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1760068666546/27bdcfb2-ff3c-4b44-a005-78bdb38dc7d0.png
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1760068685345/508e9877-1d02-49b2-82f6-df48c5520b2e.png
tags: ruby, rails, ruby-on-rails, testing

---

If microsecond precision is not required when testing Time, DateTime, or ActiveSupport::TimeWithZone, use iso8601 to assert equality between different times.   There are two ways to avoid test failures caused by execution delays:

1. Use time.iso8601
    
2. Use time.to\_fs(:iso8601)
    

## Comparing two DateTime values

For example, to compare two DateTime values:

```ruby
def test_question_answered_at_the_same_as_survey
   assert_equal question.answered_at, survey.answered_at
end

describe 'question#answered_at'
   it 'is the same as survey' do 
       expect(question.answered_at).to eq(survey.answered_at)
   end
end
```

This approach can cause issues if there are microsecond delays when saving values to the database.

There are several ways to address this issue.

One option is to use .change to set microseconds to zero.

```ruby
def test_question_answered_at_the_same_as_survey
   assert_equal question.answered_at.change(usec: 0), survey.answered_at.change(usec: 0)
end

describe 'question#answered_at'
   it 'is the same as survey' do 
       expect(question.answered_at.change(usec: 0)).to eq(survey.answered_at.change(usec: 0))
   end
end
```

Alternatively, you can use the [`Time#to_i`](https://docs.ruby-lang.org/en/master/Time.html#method-i-to_i) method, which truncates subseconds:

```ruby
def test_question_answered_at_the_same_as_survey
   assert_equal question.answered_at.to_i, survey.answered_at.to_i
end

describe 'question#answered_at'
   it 'is the same as survey' do 
       expect(question.answered_at.to_i).to eq(survey.answered_at.to_i)
   end
end
```

I recommend using iso8601 for this comparison:

```ruby
def test_question_answered_at_the_same_as_survey
   assert_equal question.answered_at.iso8601, survey.answered_at.iso8601
end

describe 'question#answered_at'
   it 'is the same as survey' do 
       expect(question.answered_at.iso8601).to eq(survey.answered_at.iso8601)
   end
end
```

In Rails, you can also use to\_fs(:iso8601):

```ruby
def test_question_answered_at_the_same_as_survey
   assert_equal question.answered_at.to_fs(:iso8601), survey.answered_at.to_fs(:iso8601)
end

describe 'question#answered_at'
   it 'is the same as survey' do 
       expect(question.answered_at.to_fs(:iso8601)).to eq(survey.answered_at.to_fs(:iso8601))
   end
end
```

## Why use .iso8601 or .to\_fs(:iso8601)

First, ISO8601 is a widely adopted standard, so its behavior is consistent across Ruby and Rails implementations.

Second, using time.iso8601 is more descriptive than time.change(usec: 0), making the comparison method clearer and supporting consistent conventions in the codebase.

Third, this approach allows you to compare with fixed time values as well:

```ruby

assert_equal question.answered_at.iso8601, "2025-10-07T13:35:25+03:00"
expect(question.answered_at.iso8601).to eq "2025-10-07T13:35:25+03:00"
```

I know that someone might say well to understand what iso8601 does tyou have to know about it. But here I think it is a normal expectation for an web engineer to know about ISO8601 standard when working with time.

Of course you should choose the one that works best for you.