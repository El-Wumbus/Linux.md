# Linux File Permissions & Ownership

## File Ownership

There are three kinds of owners,

* User

The single owner of the file.
This type of ownership is attained when creating a file or changing ownership with `chown`.

* Group

A group contains one, or more, users.

* Other
  
Every user on the system is part of other.

### Ownership precedence

Ownership, and permission, is evaluated in the order of `User -> Group -> Other`.
The permissions and ownership of User take precedence over Group, Group takes precedene over Other.

This means even if group has more power than User, only the permissions of user are evaluated (this is if the user is the owner).


