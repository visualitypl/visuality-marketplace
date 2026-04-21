# Source traceability — view-tests.md

Backs [`../skills/rails-testing/references/view-tests.md`](../skills/rails-testing/references/view-tests.md). Rails version: **8.1.3**.

| Claim | Source |
| :--- | :--- |
| Intro: view tests live in `test/controllers/` or are part of controller tests | `guides/source/testing.md` lines 1851–1854 |
| `assert_dom` / `assert_dom_equal` syntax; `title` example | `guides/source/testing.md` lines 1858–1867 (signatures authoritative in external `rails-dom-testing` gem, not pinned here) |
| Nested `assert_dom` + iterating selected collection (`ul.navigation`, `ol`/`li` examples) | `guides/source/testing.md` lines 1869–1895 |
| `assert_dom_equal` link_to example | `guides/source/testing.md` lines 1897–1903 |
| `ActionView::TestCase` tests declare `document_root_element` returning a `Nokogiri::XML::Node` | `guides/source/testing.md` lines 1908–1923; `actionview/lib/action_view/test_case.rb` lines 15 (`class TestCase < ActiveSupport::TestCase`), 49 (`include Rails::Dom::Testing::Assertions`), 329–331 (`def document_root_element`) |
| Pattern-matching support with Nokogiri ≥ 1.14.0 and minitest ≥ 5.18.0 | `guides/source/testing.md` lines 1925–1944 |
| `ViewPartialTestCase` Capybara base class; `page` from `Capybara.string(rendered)`; `ArticlePartialTest` `assert_link` example | `guides/source/testing.md` lines 1946–1979 |
| `rendered` returns a parseable object as of Action View 7.1; `register_parser`; default `:html` and `:json` parsers | `guides/source/testing.md` lines 1986–2031; `actionview/lib/action_view/test_case.rb` lines 112 (`attr_accessor :rendered`), 197–202 (`def register_parser`) |
| `assert_dom_email` / `assert_dom_encoded` / `css_select` table; `assert_dom_email` block example | `guides/source/testing.md` lines 2040–2052 (signatures authoritative in external `rails-dom-testing` gem, not pinned here) |
| Partial tests live in `test/views/` and inherit from `ActionView::TestCase`; `render` + `rendered`; `ArticlePartialTest` examples | `guides/source/testing.md` lines 2056–2093 |
| `UsersHelper#link_to_user`; `UsersHelperTest < ActionView::TestCase`; access to `link_to` / `pluralize` | `guides/source/testing.md` lines 2097–2127 |
