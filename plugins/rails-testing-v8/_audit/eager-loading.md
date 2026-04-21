# Source traceability — eager-loading.md

Backs [`../skills/rails-testing/references/eager-loading.md`](../skills/rails-testing/references/eager-loading.md). Rails version: **8.1.3**.

| Claim | Source |
| :--- | :--- |
| Dev/test do not eager load; production does; eager-loading in CI catches errors pre-deploy | `guides/source/testing.md` lines 2762–2766 |
| `config.eager_load = ENV["CI"].present?` in `config/environments/test.rb`; default since Rails 7 | `guides/source/testing.md` lines 2770–2782 |
| `Rails.application.eager_load!` in a compliance test for projects without CI | `guides/source/testing.md` lines 2784–2796 |
