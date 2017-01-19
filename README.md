EctoEnum
========

[![Hex.pm version](https://img.shields.io/hexpm/v/ecto_enum.svg?style=flat)](https://hex.pm/packages/ecto_enum) 
[![Hex.pm downloads](https://img.shields.io/hexpm/dt/ecto_enum.svg?style=flat)](https://hex.pm/packages/ecto_enum) 
[![Inline docs](http://inch-ci.org/github/gjaldon/ecto_enum.svg?branch=master)](http://inch-ci.org/github/gjaldon/ecto_enum)
[![Build Status](https://travis-ci.org/gjaldon/ecto_enum.svg?branch=master)](https://travis-ci.org/gjaldon/ecto_enum)

EctoEnum is an Ecto extension to support enums in your Ecto models.

## Usage

First, we add `ecto_enum` to `mix.exs`:

```elixir
def deps do
  [{:ecto_enum, "~> 1.0"}]
end
```

We will then have to define our enum. We can do this in a separate file since defining
an enum is just defining a module. We do it like:

```elixir
# lib/my_app/ecto_enums.ex

import EctoEnum
defenum StatusEnum, registered: 0, active: 1, inactive: 2, archived: 3
```

Once defined, `EctoEnum` can be used like any other `Ecto.Type` by passing it to a field
in your model's schema block. For example:

```elixir
defmodule User do
  use Ecto.Model

  schema "users" do
    field :status, StatusEnum
  end
end
```

In the above example, the `:status` will behave like an enum and will allow you to
pass an `integer`, `atom` or `string` to it. This applies to saving the model,
invoking `Ecto.Changeset.cast/4`, or performing a query on the status field. Let's
do a few examples:

```elixir
iex> user = Repo.insert!(%User{status: 0})
iex> Repo.get(User, user.id).status
:registered

iex> %{changes: changes} = cast(%User{}, %{"status" => "active"}, ~w(status), [])
iex> changes.status
:active

iex> from(u in User, where: u.status == ^:registered) |> Repo.all() |> length
1
```

Passing a value that the custom Enum type does not recognize will result in an error.

### Reflection

The enum type `StatusEnum` will also have a reflection function for inspecting the
enum map in runtime.

```elixir
iex> StatusEnum.__enum_map__()
[registered: 0, active: 1, inactive: 2, archived: 3]
iex> StatusEnum.__valid_values__()
[0, 1, 2, 3, :registered, :active, :inactive, :archived, "active", "archived",
"inactive", "registered"]
```

### Using Postgres's Enum Type

[Enumerated Types in Postgres](https://www.postgresql.org/docs/current/static/datatype-enum.html) are now supported. To use Postgres's Enum Type with EctoEnum, use the `defenum/3` macro
instead of `defenum/2`. We do it like:

```elixir
# lib/my_app/ecto_enums.ex

import EctoEnum
defenum StatusEnum, :status, [:registered, :active, :inactive, :archived]
```

The second argument is the name you want used for the new type you are creating in Postgres.
Note that `defenum/3` expects a list of atoms(could be strings) instead of a keyword
list unlike in `defenum/2`. Another notable difference is that you can no longer
use integers in place of atoms or strings as values in your enum type. Given the
above code, this means that you can only pass the following values:

```elixir
[:registered, :active, :inactive, :archived, "registered", "active", "inactive", "archived"]
```

In your migrations, you can make use of helper functions like:

```elixir
def up do
  StatusEnum.create_type
  create table(:users_pg) do
    add :status, :status
  end
end

def down do
  drop table(:users_pg)
  StatusEnum.drop_type
end
```

`create_type/0` and `drop_type/0` are automatically defined for you in
your custom Enum module.


## Important links

  * [Documentation](http://hexdocs.pm/ecto_enum)
  * [License](https://github.com/gjaldon/ecto_enum/blob/master/LICENSE)
