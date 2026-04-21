# Testing Views

Covers how to test the HTML output of your Rails views. Mirrors the **Testing Views** section of the Rails 8.1.3 testing guide (`guides/source/testing.md` §1848–2128).

Testing the response to your request by asserting the presence of key HTML elements and their content is one way to test the views of your application. Like route tests, view tests are stored in `test/controllers/` or are part of controller tests.

## Querying the HTML

Methods like `assert_dom` and `assert_dom_equal` allow you to query HTML elements of the response by using a simple yet powerful syntax.

`assert_dom` is an assertion that will return true if matching elements are found. For example, you could verify that the page title is "Welcome to the Rails Testing Guide" as follows:

```ruby
assert_dom "title", "Welcome to the Rails Testing Guide"
```

You can also use nested `assert_dom` blocks for deeper investigation.

In the following example, the inner `assert_dom` for `li.menu_item` runs within the collection of elements selected by the outer block:

```ruby
assert_dom "ul.navigation" do
  assert_dom "li.menu_item"
end
```

A collection of selected elements may also be iterated through so that `assert_dom` may be called separately for each element. For example, if the response contains two ordered lists, each with four nested list elements then the following tests will both pass.

```ruby
assert_dom "ol" do |elements|
  elements.each do |element|
    assert_dom element, "li", 4
  end
end

assert_dom "ol" do
  assert_dom "li", 8
end
```

The `assert_dom_equal` method compares two HTML strings to see if they are equal:

```ruby
assert_dom_equal '<a href="http://www.further-reading.com">Read more</a>',
  link_to("Read more", "http://www.further-reading.com")
```

For more advanced usage, refer to the [`rails-dom-testing` documentation](https://github.com/rails/rails-dom-testing).

In order to integrate with [rails-dom-testing][], tests that inherit from `ActionView::TestCase` declare a `document_root_element` method that returns the rendered content as an instance of a [Nokogiri::XML::Node](https://nokogiri.org/rdoc/Nokogiri/XML/Node.html):

```ruby
test "renders a link to itself" do
  article = Article.create! title: "Hello, world"

  render "articles/article", article: article
  anchor = document_root_element.at("a")

  assert_equal article.name, anchor.text
  assert_equal article_url(article), anchor["href"]
end
```

If your application depends on [Nokogiri >= 1.14.0](https://github.com/sparklemotion/nokogiri/releases/tag/v1.14.0) or higher, and [minitest >= 5.18.0](https://github.com/minitest/minitest/blob/v5.18.0/History.rdoc#5180--2023-03-04-), `document_root_element` supports [Ruby's Pattern Matching](https://docs.ruby-lang.org/en/master/syntax/pattern_matching_rdoc.html):

```ruby
test "renders a link to itself" do
  article = Article.create! title: "Hello, world"

  render "articles/article", article: article
  anchor = document_root_element.at("a")
  url = article_url(article)

  assert_pattern do
    anchor => { content: "Hello, world", attributes: [{ name: "href", value: url }] }
  end
end
```

If you'd like to access the same [Capybara-powered Assertions](https://rubydoc.info/github/teamcapybara/capybara/master/Capybara/Minitest/Assertions) that your [System Testing](system-tests.md) tests utilize, you can define a base class that inherits from `ActionView::TestCase` and transforms the `document_root_element` into a `page` method:

```ruby
# test/view_partial_test_case.rb

require "test_helper"
require "capybara/minitest"

class ViewPartialTestCase < ActionView::TestCase
  include Capybara::Minitest::Assertions

  def page
    Capybara.string(rendered)
  end
end

# test/views/article_partial_test.rb

require "view_partial_test_case"

class ArticlePartialTest < ViewPartialTestCase
  test "renders a link to itself" do
    article = Article.create! title: "Hello, world"

    render "articles/article", article: article

    assert_link article.title, href: article_url(article)
  end
end
```

More information about the assertions included by Capybara can be found in the [Capybara Assertions](system-tests.md#capybara-assertions) section.

## Parsing View Content

Starting in Action View version 7.1, the `rendered` helper method returns an object capable of parsing the view partial's rendered content.

To transform the `String` content returned by the `rendered` method into an object, define a parser by calling [`register_parser`](https://api.rubyonrails.org/classes/ActionView/TestCase/Behavior/ClassMethods.html#method-i-register_parser). Calling `register_parser :rss` defines a `rendered.rss` helper method. For example, to parse rendered [RSS content][] into an object with `rendered.rss`, register a call to `RSS::Parser.parse`:

```ruby
register_parser :rss, -> rendered { RSS::Parser.parse(rendered) }

test "renders RSS" do
  article = Article.create!(title: "Hello, world")

  render formats: :rss, partial: article

  assert_equal "Hello, world", rendered.rss.items.last.title
end
```

By default, `ActionView::TestCase` defines a parser for:

* `:html` - returns an instance of [Nokogiri::XML::Node](https://nokogiri.org/rdoc/Nokogiri/XML/Node.html)
* `:json` - returns an instance of [ActiveSupport::HashWithIndifferentAccess](https://api.rubyonrails.org/classes/ActiveSupport/HashWithIndifferentAccess.html)

```ruby
test "renders HTML" do
  article = Article.create!(title: "Hello, world")

  render partial: "articles/article", locals: { article: article }

  assert_pattern { rendered.html.at("main h1") => { content: "Hello, world" } }
end

test "renders JSON" do
  article = Article.create!(title: "Hello, world")

  render formats: :json, partial: "articles/article", locals: { article: article }

  assert_pattern { rendered.json => { title: "Hello, world" } }
end
```

[rails-dom-testing]: https://github.com/rails/rails-dom-testing
[RSS content]: https://www.rssboard.org/rss-specification

## Additional View-Based Assertions

There are more assertions that are primarily used in testing views:

| Assertion                                                 | Purpose |
| --------------------------------------------------------- | ------- |
| `assert_dom_email`                                     | Allows you to make assertions on the body of an e-mail. |
| `assert_dom_encoded`                                   | Allows you to make assertions on encoded HTML. It does this by un-encoding the contents of each element and then calling the block with all the un-encoded elements.|
| `css_select(selector)` or `css_select(element, selector)` | Returns an array of all the elements selected by the _selector_. In the second variant it first matches the base _element_ and tries to match the _selector_ expression on any of its children. If there are no matches both variants return an empty array.|

Here's an example of using `assert_dom_email`:

```ruby
assert_dom_email do
  assert_dom "small", "Please click the 'Unsubscribe' link if you want to opt-out."
end
```

## Testing View Partials

[Partial](layouts_and_rendering.html#using-partials) templates - usually called "partials" - can break the rendering process into more manageable chunks. With partials, you can extract sections of code from your views to separate files and reuse them in multiple places.

View tests provide an opportunity to test that partials render content the way you expect. View partial tests can be stored in `test/views/` and inherit from `ActionView::TestCase`.

To render a partial, call `render` like you would in a template. The content is available through the test-local `rendered` method:

```ruby
# test/views/article_partial_test.rb
class ArticlePartialTest < ActionView::TestCase
  test "renders a link to itself" do
    article = Article.create! title: "Hello, world"

    render "articles/article", article: article

    assert_includes rendered, article.title
  end
end
```

Tests that inherit from `ActionView::TestCase` also have access to [`assert_dom`](#testing-views) and the [other additional view-based assertions](#additional-view-based-assertions) provided by [rails-dom-testing][]:

```ruby
test "renders a link to itself" do
  article = Article.create! title: "Hello, world"

  render "articles/article", article: article

  assert_dom "a[href=?]", article_url(article), text: article.title
end
```

## Testing View Helpers

A helper is a module where you can define methods which are available in your views.

In order to test helpers, all you need to do is check that the output of the helper method matches what you'd expect. Tests related to the helpers are located under the `test/helpers` directory.

Given we have the following helper:

```ruby
# app/helpers/users_helper.rb
module UsersHelper
  def link_to_user(user)
    link_to "#{user.first_name} #{user.last_name}", user
  end
end
```

We can test the output of this method like this:

```ruby
# test/helpers/users_helper_test.rb
class UsersHelperTest < ActionView::TestCase
  test "should return the user's full name" do
    user = users(:david)

    assert_dom_equal %{<a href="/user/#{user.id}">David Heinemeier Hansson</a>}, link_to_user(user)
  end
end
```

Moreover, since the test class extends from `ActionView::TestCase`, you have access to Rails' helper methods such as `link_to` or `pluralize`.

See also: [`assertions.md`](assertions.md) for Rails-specific assertions; [`system-tests.md`](system-tests.md) for the Capybara assertions referenced by `ViewPartialTestCase`.
