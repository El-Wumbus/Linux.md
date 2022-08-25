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

### Changing ownership

`chown` is a command that allows for changing the ownership of a file or directory. The syntax for using is as follows:

``` bash
chown User:Group <file or directory>
```

So to set the ownership of `file.example.txt` to the user `admin` and the group `wheel` you could do the following:

```bash
chown admin:wheel ./file.example.txt
```

You can omit either the user or the group.

```bash
chown :wheel ./file.example.txt
```

```bash
chown admin ./file.example.txt
```

## File Permissions

There are three permissions a file or directory can have.

* Read - Can view & copy file contents -- Can list and copy files from within a directory
* Write - Can modify file contents -- Can add or delete files from within a directory
* Execute - Can execute a file (if it's executable) -- Can enter a directory

File permissions can be represented as letters or numbers.

* Letter representation
  
  * Read - `r`
  * Write - `w`
  * Execute - `x`
  * Nothing - `-`
  
  We can, for example, combine these to give a user read, write, and execute permissions.

  > Permissions are always in the order of read, write and execute, i.e., rwx. And then these permissions are set for all three kind of owners in the order of User, Group and Other.
  
  | Permission | Meaning               |
  |------------|-----------------------|
  | `- - -`    | Nothing               |
  | `- - x`    | Execute               |
  | `- w -`    | Write                 |
  | `- w x`    | Write & Execute       |
  | `r - -`    | Read                  |
  | `r - x`    | Read & Execute        |
  | `r w -`    | Read & Write          |
  | `r w x`    | Read, Write & Execute |

* Number representation
  
  * Read - `4`
  * Write - `2`
  * execute - `1`
  * Nothing - `0`

  These can be added to eachother in several ways.
  
  | Permission | Number |
  |------------|--------|
  | `- - -`    | 0      |
  | `- - x`    | 1      |
  | `- w -`    | 2      |
  | `- w x`    | 3      |
  | `r - -`    | 4      |
  | `r - x`    | 5      |
  | `r w -`    | 6      |
  | `r w x`    | 7      |

### Changing permissions

`chmod` is a command that allows for changing the permissions of a file or directory. The syntax for using is as follows:

``` bash
chmod <user><group><other> <file or directory>
```

So to set the permissions of `file.example.txt` to let user `r w x` and the group `r - x`  and other `r - -`you could do the following:

```bash
chmod 754 ./file.example.txt
```
