# Source traceability — mailer-tests.md

Backs [`../skills/rails-testing/references/mailer-tests.md`](../skills/rails-testing/references/mailer-tests.md). Rails version: **8.1.3**.

| Claim | Source |
| :--- | :--- |
| Intro: goals of mailer testing; unit vs. functional split; `[fixture](#fixtures)` link | `guides/source/testing.md` lines 2129–2147 |
| Unit Testing opener | `guides/source/testing.md` lines 2149–2153 |
| Mailer fixtures live in `test/fixtures/<mailer_name>/`; generator does not create stub fixtures | `guides/source/testing.md` lines 2155–2167 |
| `UserMailerTest < ActionMailer::TestCase`; `assert_emails 1 do … end`; `read_fixture("invite")`; `email.from` / `email.to` / `email.subject` / `email.body.to_s` | `guides/source/testing.md` lines 2169–2195; `actionmailer/lib/action_mailer/test_case.rb` line 15 (`class TestCase < ActiveSupport::TestCase`), lines 82–84 (`read_fixture`); `actionmailer/lib/action_mailer/test_helper.rb` lines 35–42 (`assert_emails`) |
| Multi-part note: `email.text_part.body.to_s` / `email.html_part.body.to_s` | `guides/source/testing.md` lines 2202–2204 |
| `invite` fixture contents | `guides/source/testing.md` lines 2206–2214 |
| `ActionMailer::Base.delivery_method = :test` in `config/environments/test.rb`; appends to `ActionMailer::Base.deliveries` | `guides/source/testing.md` lines 2216–2222; `actionmailer/lib/action_mailer/base.rb` line 460 (doc comment on `delivery_method` default `:smtp`), line 469 (`deliveries` array) |
| `deliveries` reset in `ActionMailer::TestCase` / `ActionDispatch::IntegrationTest`; manual `ActionMailer::Base.deliveries.clear` note | `guides/source/testing.md` lines 2224–2227; `actionmailer/lib/action_mailer/test_case.rb` lines 87–92 (`initialize_test_deliveries` clears deliveries) |
| `assert_enqueued_email_with` matches `deliver_later` with `args:` and/or `params:` | `guides/source/testing.md` lines 2229–2238; `actionmailer/lib/action_mailer/test_helper.rb` lines 157–174 |
| `assert_enqueued_email_with UserMailer, :create_invite, args: […]` positional-args example | `guides/source/testing.md` lines 2240–2257 |
| `args: [{ from:, to: }]` named-args example | `guides/source/testing.md` lines 2259–2277 |
| `params:` + `args:` parameterized-mailer example using `UserMailer.with(all: "good")` | `guides/source/testing.md` lines 2279–2298 |
| Alternative parameterized form: `assert_enqueued_email_with UserMailer.with(to: …), :create_invite` | `guides/source/testing.md` lines 2300–2317 |
| Functional/system intro | `guides/source/testing.md` lines 2319–2324 |
| Integration test `UsersControllerTest < ActionDispatch::IntegrationTest` with `assert_emails 1 do post …` | `guides/source/testing.md` lines 2326–2338 |
| System test `UsersTest < ActionDispatch::SystemTestCase`, `driven_by :selenium, using: :headless_chrome`, `assert_emails 1 do click_on "Invite"` | `guides/source/testing.md` lines 2340–2355 |
| `assert_emails` works with `deliver_now` or `deliver_later`; `assert_enqueued_email_with` / `assert_enqueued_emails` for explicit enqueue checks; link to `ActionMailer/TestHelper` API docs | `guides/source/testing.md` lines 2357–2363; `actionmailer/lib/action_mailer/test_helper.rb` lines 91–93 (`assert_enqueued_emails`), lines 191–193 (`assert_no_enqueued_emails`) |
