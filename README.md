# NAME 

Dancer2::Plugin::Auth::Extensible::Provider::Usergroup - authenticate as a member of a group

# SYNOPSIS

Define that a user must be logged in and have the proper permissions to 
access a route:

```perl
get '/unsubscribe' => require_role forum => sub { ... };
```

# DESCRIPTION

This class is an authentication provider designed to authenticate users against
a DBIC schema, using [Dancer2::Plugin::DBIC](https://metacpan.org/pod/Dancer2%3A%3APlugin%3A%3ADBIC) to access a database.

[Dancer2::Plugin::Passphrase](https://metacpan.org/pod/Dancer2%3A%3APlugin%3A%3APassphrase) is used to handle hashed passwords securely; you wouldn't
want to store plain text passwords now, would you?  (If your answer to that is
yes, please reconsider; you really don't want to do that, when it's so easy to
do things right!)

See [Dancer2::Plugin::DBIC](https://metacpan.org/pod/Dancer2%3A%3APlugin%3A%3ADBIC) for how to configure a database connection
appropriately; see the ["CONFIGURATION"](#configuration) section below for how to configure this
authentication provider with database details.

See [Dancer2::Plugin::Auth::Extensible](https://metacpan.org/pod/Dancer2%3A%3APlugin%3A%3AAuth%3A%3AExtensible) for details on how to use the
authentication framework, including how to use "require\_login" and "require\_role".

# CONFIGURATION

This provider tries to use sensible defaults, so you may not need to provide
much configuration if your database tables look similar to those in the
["SUGGESTED SCHEMA"](#suggested-schema) section below.

The most basic configuration, assuming defaults for all options, and defining a
single authentication realm named 'usergroup':

```perl
plugins:
    Auth::Extensible:
        realms:
            usergroup:
                provider: 'Usergroup'
```

You would still need to have provided suitable database connection details to
[Dancer2::Plugin::DBIC](https://metacpan.org/pod/Dancer2%3A%3APlugin%3A%3ADBIC), of course;  see the docs for that plugin for full
details, but it could be as simple as, e.g.:

```perl
plugins:
    Auth::Extensible:
        realms:
            usergroup:
                provider: 'Usergroup'
                schema_name: 'usergroup'
    DBIC:
        usergroup:
            chema_class: Usergroup::Schema
            dsn: "dbi:SQLite:dbname=/path/to/usergroup.db"
```

A full example showing all options:

```perl
plugins:
    Auth::Extensible:
        realms:
            usergroup:
                provider: 'Usergroup'
                
                # optional schema name for DBIC (default 'default')
                schema_name: 'usergroup'

                # optionally specify names of result sets if they're not the defaults
                # (defaults are 'User' and 'Role')
                user_rset: 'User'
                user_role_rset: 'Role'

                # optionally set the column names (see the SUGGESTED SCHEMA
                # section below for the default names; if you use them, they'll
                # Just Work)
                user_login_name_column: 'login_name'
                user_passphrase_column: 'passphrase'
                user_role_column: 'role'
                
                # optionally set a column name that makes a user useable
                # (not all login names can be used to login)
                user_activated_column: 'activated'
```

See the main [Dancer2::Plugin::Auth::Extensible](https://metacpan.org/pod/Dancer2%3A%3APlugin%3A%3AAuth%3A%3AExtensible) documentation for how to
configure multiple authentication realms.

# ATTRIBUTES

## schema\_name

Defaults to 'default',

## schema

Defaults to a DBIC schema using ["schema\_name"](#schema_name).

## user\_rset

The name of the DBIC result class for the user table.

Defaults to 'User'.

## user\_role\_rset

The name of the DBIC result class for the role view.

Defaults to 'Role'.

## user\_login\_name\_column

The login\_name column in ["user\_rset"](#user_rset).

Defaults to 'login\_name'.

## user\_passphrase\_column

The passphrase column in ["user\_rset"](#user_rset).

Defaults to 'passphrase'.

## user\_role\_column

The role column in ["user\_role\_rset"](#user_role_rset).

Defaults to 'role'.

## user\_activated\_column

The user activated column in ["user\_rset"](#user_rset).

Defaults to 'activated'.

# SUGGESTED SCHEMA

If you use a schema similar to the examples provided here, you should need
minimal configuration to get this authentication provider to work for you.

The examples given here should be SQLite-compatible; minimal changes should be
required to use them with other database engines.

## user table

You'll need a table to store user accounts in, of course.  A suggestion is
something like:

```perl
CREATE TABLE users (
    id INTEGER PRIMARY KEY,
    login_name TEXT UNIQUE NOT NULL,
    passphrase TEXT NOT NULL,
    activated INTEGER
);
```

You will quite likely want other fields to store e.g. the user's name, email
address, etc; all columns from the users table will be returned by the
`logged_in_user` keyword for your convenience.

## group table

You'll need a table to store a list of available groups in.

```
CREATE TABLE groups (
    id INTEGER PRIMARY KEY,
    group_name TEXT UNIQUE NOT NULL
);
```

## membership table

To make users a member you'll need a table to store
user <-> group mappings.

```perl
CREATE TABLE memberships (
    id INTEGER PRIMARY KEY,
    user_id INTEGER NOT NULL REFERENCES users (id),
    group_id INTEGER NOT NULL REFERENCES groups (id)
  );
```

## role view

Map the user role by name.

```perl
CREATE VIEW roles AS
SELECT login_name, group_name AS role
    FROM users
    LEFT JOIN memberships ON users.id = memberships.user_id
    LEFT JOIN groups ON groups.id = memberships.group_id
;
```

## indexes

You want your data quickly.

```perl
CREATE UNIQUE INDEX login_name ON users (login_name);
CREATE UNIQUE INDEX group_name ON groups (group_name);
CREATE UNIQUE INDEX user_group ON memberships (user_id, group_id);
CREATE INDEX member_user ON memberships (user_id);
CREATE INDEX member_group ON memberships (group_id);
```

# INTERNALS

#### get\_user\_details

Used by [Dancer2::Plugin::Auth::Extensible](https://metacpan.org/pod/Dancer2%3A%3APlugin%3A%3AAuth%3A%3AExtensible)

#### match\_password

Used by [Dancer2::Plugin::Auth::Extensible](https://metacpan.org/pod/Dancer2%3A%3APlugin%3A%3AAuth%3A%3AExtensible)

#### authenticate\_user

Used by [Dancer2::Plugin::Auth::Extensible](https://metacpan.org/pod/Dancer2%3A%3APlugin%3A%3AAuth%3A%3AExtensible)

#### get\_user\_roles

Used by [Dancer2::Plugin::Auth::Extensible](https://metacpan.org/pod/Dancer2%3A%3APlugin%3A%3AAuth%3A%3AExtensible)

# COPYRIGHT

Copyright (c) 2014 Henk van Oers

# LICENSE

This library is free software and may be distributed under the same terms
as perl itself.
