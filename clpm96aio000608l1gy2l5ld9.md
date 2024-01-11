---
title: "Review Ruby on Rails Code using Rubymine AI and ChatGPT"
seoTitle: "Review Rails Code: Rubymine AI & ChatGPT"
seoDescription: "Optimize Rails code parsing gem lists using ChatGPT & RubyMine AI. Compare AI suggestions to improve readability, efficiency, and maintainability."
datePublished: Fri Dec 01 2023 06:37:50 GMT+0000 (Coordinated Universal Time)
cuid: clpm96aio000608l1gy2l5ld9
slug: review-ruby-on-rails-code-using-rubymine-ai-and-chatgpt
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1701408626639/5ff1e5a8-bc19-4dad-a4f2-3bb8161a6450.png
tags: ai, code-review, ruby, ruby-on-rails, chatgpt

---

I wrote a piece of code that will take as input a string that could be in one of the following two formats:

* a list of gem names, one per line
    
* Gemfile format
    

## Initial code

This is for my Rails app that I use to curate the content for the [Short Ruby Newsletter](https://newsletter.shortruby.com) and so in general I have little time to write code for it.

So I wrote a very quick draft of a working code along with some simple tests.

Here is the initial code:

```ruby
class Parser
  def initialize(list, rubygems_client: RubyGems::Client.new)
    @list = list
    @rubygems_client = rubygems_client
  end

  def parse
    return parse_gemfile_format if gemfile_format?

    parse_single_line_format
  end

  private

    attr_reader :list, :rubygems_client

    def gemfile_format? = lines.any? { gem_method_call?(_1) }

    def parse_gemfile_format
      ast = Prism.parse(list)
      root_node = ast.value

      parsed_gems = []

      root_node.statements.body.each do |call_node|
        case call_node.name
        when :gem
          name = gem_name_from_call_node(call_node)
          parsed_gems << name
        when :group
          names = gem_names_from_group(call_node)
          parsed_gems.concat(names) if names.present?
        end
      end

      parsed_gems.compact_blank.map { rubygems_client.info(_1) }
    end

    def parse_single_line_format
      lines.filter_map do |line|
        gem_name = line.strip
        gem_info = rubygems_client.info(gem_name) unless gem_name.empty?
        gem_info
      end.compact_blank
    end

    def gem_names_from_group(call_node)
      statements_node = call_node&.block&.body
      statements_node.body.map { gem_name_from_call_node(_1) }
    end

    def gem_name_from_call_node(call_node) = call_node&.arguments&.arguments&.first&.unescaped

    def gem_method_call?(line) = line.strip.include?("gem ")

    def lines = list.lines
end
```

You can notice that the code is not very OOP and it also has some issues with naming. Add to that the fact that `parse_gemfile_format` is quite a long method and it can get a bit complex. It does not handle exceptions at all and it abuses a bit the Ruby safe operator (`&.`) when calling methods. Also, there are some cases that this parser will fail to handle like a more complex syntax of Gemfile.

So it seemed to me like a good fit to use it to assess the usefulness of using ChatGPT and RubyMine AI to review this code and propose improvements.

## The prompt

I wrote a simple prompt that explains what I wanted to achieve with this code and describes some areas of concern:

```markdown
I have a Ruby on Rails object designed for parsing a list of gems in 
various formats, including the Gemfile format, and for each gem, 
I will call a RubyGems client to get information about it.  

The current implementation differentiates between the format of 
specifying one gem name per line and the Gemfile format and, 
based on this, will branch out into separate logic for each case. 
It uses Prism to parse the Gemfile format and identify the name 
of the gems. 

I'm concerned about the efficiency and readability of object and 
code design. 

I aim to refine the code to be more idiomatic and maintainable, 
ensuring it aligns with Ruby and Rails best practices. 
I also want to make use of the latest language features and syntax. 
One concern I have is how I use `prism` gem 
(repository is at https://github.com/ruby/prism and 
documentation at https://ruby.github.io/prism/rb/index.html).

The application runs on Ruby 3.2 and Rails 7. 
Any suggestions should be compatible with these versions.

Extra guidelines:
I want the private methods ordered in the order the public 
methods require.
When guidelines are conflicting, choose readability over 
anything else.
```

I know that the prompt can be made better but I wanted to make it as close as possible (while still making it a prompt) to maybe a discussion that anyone could have with a friend or colleague when asking for feedback.

## ChatGPT

Here is the response from ChatGPT after I provided the prompt and the code:

> Your Ruby on Rails code for parsing a list of gems is well-structured, but there are some improvements and refinements that can be made to enhance its efficiency, readability, and maintainability. Here are my suggestions:
> 
> 1. **Refactor Conditional Logic**:
>     
>     * The `parse` method uses a conditional to differentiate between formats. This is clear, but you might want to encapsulate the format-checking logic in more descriptive methods to improve readability.
>         
> 2. **Use Memoization**:
>     
>     * For methods that are called multiple times and return the same result, like `lines`, consider memoizing their results to improve efficiency.
>         
> 3. **Refactor** `parse_gemfile_format`:
>     
>     * This method can be refactored to reduce its complexity and increase readability. Extract parts of its logic into smaller, more focused methods.
>         
> 4. **Refine** `gem_names_from_group` **and** `gem_name_from_call_node` Methods:
>     
>     * These methods can be made more robust. Ensure that they handle potential `nil` cases gracefully.
>         
> 5. **Update Method Names for Clarity**:
>     
>     * Renaming methods like `gem_method_call?` to something more descriptive can improve readability.
>         
> 6. **Error Handling**:
>     
>     * Consider adding error handling for cases where the gem list might be in an unexpected format or when external calls fail.
>         
> 7. **Documentation and Comments**:
>     
>     * Adding comments to explain complex parts of the code or the reasoning behind certain decisions can be very helpful for maintainability.
>         
> 8. **Leverage Ruby 3.2 Features**:
>     
>     * Review the new features in Ruby 3.2 to see if any can be applied to improve the code. For example, pattern matching might be useful in parsing different formats.
>         
> 9. **Private Method Ordering**:
>     
>     * Reorder the private methods to match the sequence in which they are called by the public methods.
>         

It provided the following code:

```ruby

class Parser
  def initialize(list, rubygems_client: RubyGems::Client.new)
    @list = list
    @rubygems_client = rubygems_client
  end

  def parse
    return parse_gemfile_format if gemfile_format?

    parse_single_line_format
  end

  private

    attr_reader :list, :rubygems_client

    def gemfile_format?
      lines.any? { |line| line.include?('gem ') }
    end

    def parse_gemfile_format
      ast = Prism.parse(list).value
      parse_ast_nodes(ast.statements.body)
    end

    def parse_single_line_format
      lines.filter_map do |line|
        next if line.strip.empty?
        rubygems_client.info(line.strip)
      end.compact
    end

    def parse_ast_nodes(nodes)
      nodes.each_with_object([]) do |node, gems|
        case node.name
        when :gem
          gems << gem_name_from_call_node(node)
        when :group
          gems.concat(gem_names_from_group(node))
        end
      end.compact.map { |name| rubygems_client.info(name) }
    end

    def gem_names_from_group(group_node)
      block_node = group_node&.block&.body
      block_node&.body&.map { |node| gem_name_from_call_node(node) } || []
    end

    def gem_name_from_call_node(node)
      node&.arguments&.first&.unescaped
    end

    def lines
      @lines ||= list.lines
    end
end
```

### Review of the provided code

It is a bit more readable, the methods are smaller and I like the split into smaller methods.

It added memoization for lines. I am not sure how long the execution of `String#lines` take but could be an improvement to have them memoized. But it does not consider [Object Shapes](https://island94.org/2023/10/writing-object-shape-friendly-code-in-ruby) when introducing this change even if it was asked to use Ruby 3.2 features

It adds a bug that makes tests fail in the method `gem_name_from_call_node(node)` where it did the following change:

```diff
- node&.arguments&.arguments&.first&.unescaped
+ node&.arguments&.first&.unescaped
```

This bug is a bit subtle and to catch it before running the tests one has to know that Prism returns the arguments from a CallNode (`Prism:CallNode`) as [`Prism::ArgumentsNode`](https://ruby.github.io/prism/rb/Prism/ArgumentsNode.html) that has an attribute called `arguments` that will return the arguments as Arary.

Looking at this method:

```ruby
def parse_ast_nodes(nodes)
  nodes.each_with_object([]) do |node, gems|
    case node.name
    when :gem
      gems << gem_name_from_call_node(node)
    when :group
      gems.concat(gem_names_from_group(node))
    end
  end.compact.map { |name| rubygems_client.info(name) }
end
```

It is more idiomatic to use `each_with_object` to collect the gem names over the previous implementation where I manually created an accumulator variable but I find the `end.compact.map` syntax as inclining more to idiomatic than readability. At the same time, this is very subjective so I cannot say it is wrong.

I would have preferred:

```ruby
def parse_ast_nodes(nodes)
  gemlist = nodes.each_with_object([]) do |node, gems|
    case node.name
    when :gem
      gems << gem_name_from_call_node(node)
    when :group
      gems.concat(gem_names_from_group(node))
    end
  end

  gemlist.compact.map { |name| rubygems_client.info(name) }
end
```

It also removed all numbered block params and all endless methods, even if they are valid syntax for Ruby 3.2 and I dare say they are more modern.

## RubyMine AI

Here is the response from RubyMine AI:

> From looking at your current implementation, you've done a great job at keeping your code well-structured and easy to reason about. The usage of syntax and language features from Ruby 3.2 shows a good understanding of modern Ruby idioms.
> 
> Here are some suggestions for refinements:
> 
> 1. `include?` is not the most accurate method to use in `gem_method_call?`. Prefixing a gem name with 'gem' might not be on purpose and can lead to false positives. Using a regex that checks specifically for a 'gem' followed by a space at the start of a line would work better here; This avoids matching lines where 'gem' is somewhere in the gem's name rather than at the start.
>     
> 2. Prism's parsing will fail for Gemfiles that include logic or DSL methods beyond simple `gem` or `group` calls, For robustness, consider adding error handling, or even falling back to `parse_single_line_format` method, This way you can always handle gem lists which are not neatly formatted.
>     
> 3. You could consolidate the private methods `gem_name_from_call_node` and `gem_names_from_group` together into a single method that extracts gem names from a node, as they are both doing something quite similar.
>     
> 4. To make the code a bit cleaner and more maintainable, you could refactor the `parse_gemfile_format` method further by extracting a few more smaller private methods out. This should make the code easier to read and understand.
>     

Here is the code that is suggested:

```ruby
class Parser
  def initialize(list, rubygems_client: RubyGems::Client.new)
    @list = list
    @rubygems_client = rubygems_client
  end

  def parse
    return parse_gemfile_format if gemfile_format?

    parse_single_line_format
  rescue StandardError => e
    Rails.logger.error("Parsing gemfile format failed with exception: #{e.message}. Falling back to single line format.")
    parse_single_line_format
  end

  private

  attr_reader :list, :rubygems_client

  def gemfile_format? = lines.any? { gem_method_call?(_1) }

  def parse_gemfile_format
    ast = Prism.parse(list)
    root_node = ast.value

    parsed_gems = nodes_to_gem_names(root_node.statements.body)

    parsed_gems.compact_blank.map { rubygems_client.info(_1) }
  end

  def nodes_to_gem_names(nodes)
    nodes.map do |node|
      case node.name
      when :gem then gem_name_from_node(node)
      when :group then gem_names_from_node(node.block.body)
      end
    end.flatten.compact
  end

  def parse_single_line_format
    lines.filter_map do |line|
      gem_name = line.strip
      gem_info = rubygems_client.info(gem_name) unless gem_name.empty?
      gem_info
    end.compact_blank
  end

  def gem_name_from_node(node) = node&.arguments&.arguments&.first&.unescaped

  def gem_names_from_node(node) = nodes_to_gem_names(node.body)

  def gem_method_call?(line) = line.lstrip.start_with?("gem ")

  def lines = list.lines
end
```

### Review of the response

First, I think it identified two issues: (1) That a more complex Gemfile syntax will fail to be parsed and (2) Having a logic based on `include?("gem")` will fail if the name of a gem has the word `gem` inside.

Regarding the second case, I think it failed to see that the actual check is using `gem<space>` so it could fail for gem names that end with `gem` . But will work even in those cases because there is a `strip` call just before that (the actual code is `strip.include?("gem ")` ) so it will work well even for gems that finish in `<name>gem` because of the strip.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1701409137791/ab6c9820-6288-421b-8e9f-296dec6612bb.png align="center")

I appreciate that it did not refactor my endless methods and left them alone. But it did a subtle change and aligned all private methods to the `private` keyword, while the original code had the private method aligned and then nested under `private` keyword. This is not a bug but more of a style choice.

I like that it added a Rails logger when the parse fails in any way and in that case, will try to parse them as simple gem names.

I think extraction of the logic to `nodes_to_gem_names` makes it a bit better in readability.

And it passes all tests.

## Conclusion

Both failed to identify some things that I think could make the code better:

* There are quite a few methods named like this `gem_` and that is not a good name. I provided this example on purpose but, indeed, the prompt did not instruct specifically to give good names. Still, as the prompt says "readability" I think having good names is a pre-requisite of readability.
    

ChatGPT provided code that failed tests while RubyMine AI provided code that passed the tests.

ChatGPT removed endless methods and refactored them to normal methods. RubyMine AI respected my choice to use endless methods.

I would use an AI tool to kick start a code review and see their answers as a conversaation starter, but will not refactor based on their own review.

---

Enjoyed this article?

👉 Join my [**Short Ruby News**](https://shortruby.com/) **newsletter** for weekly Ruby updates from the community and visit [**rubyandrails.info**](http://rubyandrails.info)**,** a directory with learning content about Ruby.

👐 Subscribe to my Ruby and Ruby on rails courses over email at [learn.shortruby.com](https://learn.shortruby.com) - effortless learning anytime, anywhere

🤝 Let's connect on [**Ruby.social**](https://ruby.social/@lucian) or [**Linkedin**](https://linkedin.com/in/lucianghinda) or [**Twitter**](https://x.com/lucianghinda) where I post mainly about Ruby and Rails.

🎥 Follow me on [**my YouTube channel**](https://www.youtube.com/@shortruby) for short videos about Ruby