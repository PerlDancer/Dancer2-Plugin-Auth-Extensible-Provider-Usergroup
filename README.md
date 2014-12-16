Dancer2-Plugin-Auth-Extensible-Provider-Usergroup
=================================================

Authenticate as a member of a group

This class is an authentication provider designed to authenticate users
against a DBIC schema, using Dancer2::Plugin::DBIC to access a database.

Dancer2::Plugin::Passphrase is used to handle hashed passwords securely;
you wouldn't want to store plain text passwords now, would you?  (If
your answer to that is yes, please reconsider; you really don't want to
do that, when it's so easy to do things right!)

See Dancer2::Plugin::Auth::Extensible for details on how to use the
authentication framework, including how to use "require_login" and
"require_role".
