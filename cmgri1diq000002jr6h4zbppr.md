---
title: "Ruby on Rails: Loading Locales with Yes, No, On, and Off"
seoTitle: "Localization in Ruby on Rails: Yes/No, On/Off"
seoDescription: "Ruby on Rails uses `psych` for YAML locale processing that will load Yes/No, On/Off as booleans"
datePublished: Wed Oct 15 2025 04:36:51 GMT+0000 (Coordinated Universal Time)
cuid: cmgri1diq000002jr6h4zbppr
slug: ruby-on-rails-loading-locales-with-yes-no-on-and-off
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1760502950488/61a08276-fb05-418b-af46-24907ce52018.png
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1760502958853/225f052e-5b35-4ac4-a299-ec9e7b7465e1.png
tags: ruby, ruby-on-rails, gems

---

Ruby on Rails uses the gem `psych` to load the YAML files for locales.

If you have something like this in your `en.yml` file:

```yaml
en:
  terms:
    yes: Yes
    no: No
    switch_on: On
    switch_off: Off
    accept: true
    reject: false
```

And then you will try to load them via Rails console like this:

```ruby
I18n.locale = :en # make sure you have set a default locale
I18n.backend.send(:translations)[:en][:terms]
# => {true => true, false => false, switch_on: true, switch_off: false, accept: true, reject: false}
```

Notice there that there is no key `yes`, `no` and more so all values `Yes`, `No`, `On` `Off`, `true`, `false` were converted to TrueClass or FalseClass in Ruby.

That for me was quite interesting so I dig deeper to understand why this is happening.

Ruby on Rails loads locales using the gem `i18n` for locales. And that gem is using `psych` or `yaml` (which is aliased from `psych` - see [https://github.com/ruby/yaml](https://github.com/ruby/yaml))

## Processing YES, yes, NO, no, on, ON, off, OFF …

Looking at [`psych`](https://github.com/ruby/psych) here are some tests that we can run:

```ruby
assert_equal true, Psych.safe_load("--- YES")
assert_equal true, Psych.safe_load("--- Yes")
assert_equal true, Psych.safe_load("--- yes")

assert_equal true, Psych.safe_load("--- TRUE")
assert_equal true, Psych.safe_load("--- True")
assert_equal true, Psych.safe_load("--- true")

assert_equal true, Psych.safe_load("--- ON")
assert_equal true, Psych.safe_load("--- on")
assert_equal true, Psych.safe_load("--- On")
```

Notice that it will transform all of those values in `TrueClass` / `true` .

If we then run these tests it will return `FalseClass` / `false` :

```ruby
assert_equal false, Psych.safe_load("--- NO")
assert_equal false, Psych.safe_load("--- No")
assert_equal false, Psych.safe_load("--- no")

assert_equal false, Psych.safe_load("--- FALSE")
assert_equal false, Psych.safe_load("--- False")
assert_equal false, Psych.safe_load("--- false")

assert_equal false, Psych.safe_load("--- OFF")
assert_equal false, Psych.safe_load("--- Off")
assert_equal false, Psych.safe_load("--- off")
```

Could not find the exact documentation about this but it seems like `psych` implements YAML version 1.1 that defines the following:

[![Screenshot from YAML version 1.1](https://cdn.hashnode.com/res/hashnode/image/upload/v1760498156367/ed80e4a1-d4d5-4edc-a8f0-551249a50941.png align="center")](https://yaml.org/type/bool.html)

Notice there that it also defines `Y, y, N, n` as booleans.

```ruby
assert_equal "Y", Psych.safe_load("--- Y")
assert_equal "y", Psych.safe_load("--- y")

assert_equal "N", Psych.safe_load("--- N")
assert_equal "n", Psych.safe_load("--- n")
```

But in our case when `psych` does not do that.

## Why Psych does not convert `Y, y, N, n` to booleans?

The difference [is documented](https://github.com/ruby/psych/blob/master/test/psych/test_boolean.rb) for example in psych gem test suite for example:

```ruby
    # source: https://github.com/ruby/psych/blob/master/test/psych/test_boolean.rb
    # YAML spec says "y" and "Y" may be used as true, but Syck treats them
    # as literal strings
    def test_y
      assert_equal "y", Psych.load("--- y")
      assert_equal "Y", Psych.load("--- Y")
    end

    # YAML spec says "n" and "N" may be used as false, but Syck treats them
    # as literal strings
    def test_n
      assert_equal "n", Psych.load("--- n")
      assert_equal "N", Psych.load("--- N")
    end
```

`syck` was an old parser for YAML written in C that seems to have been implemented since Ruby `1.9` (maybe even earlier) and syck implemented YAML version 1.0. See [https://ruby-doc.org/stdlib-1.9.3/libdoc/syck/rdoc/Syck.html](https://ruby-doc.org/stdlib-1.9.3/libdoc/syck/rdoc/Syck.html)

I could not find the exact specification in [YAML version 1.0](https://yaml.org/spec/1.0/index.html) that defines the booleans so it could be there was a discussion about this and [`syck`](https://github.com/ruby/syck), as the YAML parser implemented in C and used by Ruby, decided to adopt it.

Here is the `test_boolean.rb` from `syck` and notice there `YAML spec says "y" and "Y" may be used as true, but Syck treats them # as literal strings`

```ruby
###
    # YAML spec says "y" and "Y" may be used as true, but Syck treats them
    # as literal strings
    def test_y
      assert_equal "y", Psych.load("--- y")
      assert_equal "Y", Psych.load("--- Y")
    end

    ###
    # YAML spec says "n" and "N" may be used as false, but Syck treats them
    # as literal strings
    def test_n
      assert_equal "n", Psych.load("--- n")
      assert_equal "N", Psych.load("--- N")
    end
```

## How to protect from this using RubySchema.org

[RubySchema](https://github.com/yippee-fun/rubyschema):

> Ruby schema is a collection of JSON schemas for common Ruby gems. With these schemas, we can now enjoy auto-complete, validation and inline documentation right in our YAML files.

If your code editor supports YAML Language Server (and most of them do) make sure you have it installed and then if you add the following lines at the beginning of your YAML locales file:

```yaml
# yaml-language-server: $schema=https://www.rubyschema.org/i18n/locale.json
%YAML 1.1
---
```

Then in case you will write inside the file something like this:

```yaml
# yaml-language-server: $schema=https://www.rubyschema.org/i18n/locale.json
%YAML 1.1
---
en:
  yes: "Yes"
  on: "On"
```

Then your editor will show you a message like this:

![Screenshot of Neovim displaying an error message about using yes, no as locales key](https://cdn.hashnode.com/res/hashnode/image/upload/v1760585180174/2b5457f6-efb1-4f43-8097-bfc111ed5a43.png align="center")

![Screenshot of Zed displaying an error mesaage about using `on` as key in locales](https://cdn.hashnode.com/res/hashnode/image/upload/v1760585231589/23879934-6c61-4b3a-9b7d-1cc95db1d1cf.png align="center")