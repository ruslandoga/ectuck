Eh?

```sh
brew install duckdb

duckdb --version
# v1.3.0 71c5c07cdd

brew --prefix duckdb
# /opt/homebrew/opt/duckdb
```

```elixir
Mix.install([{:ectuck, github: "ruslandoga/ectuck"}],
  force: true,
  system_env: [
    DUCKDB_NIFS_CFLAGS: "-I/opt/homebrew/opt/duckdb/include",
    DUCKDB_NIFS_LDFLAGS: "-L/opt/homebrew/opt/duckdb/lib"
  ]
)

defmodule Repo do
  use Ecto.Repo, adapter: Ectuck, otp_app: :demo
end

Application.put_env(:demo, Repo, database: ":memory:")
Repo.start_link()

Repo.query!("select version()")

Repo.query!("""
copy (
  select * from range(1, 10000000)
) to 'numbers.parquet' (
  format parquet, compression zstd, parquet_version v2
)
""")

import Ecto.Query

file = "numbers.parquet"
numbers_q = from e in fragment("read_parquet(?)", ^file)

Repo.aggregate(:count, numbers_q)
File.rm!("numbers.parquet")
```
