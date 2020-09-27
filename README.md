# authenticate-pgbouncer
Shows how to authenticate pgbouncer via the `userlist.txt` file or `auth_query` method.

## Pre-requisite
Make sure the postgres user have complete access to files `/etc/pgbouncer/pgbouncer.ini`, `/var/run/pgbouncer`, `/var/log/pgbouncer`

## Authenticate via userlist.txt 
Follow the steps below to authenticate users or clients connection to postgres database through pgbouncer.   

- On the terminal execute the following commands:  

This command switches to the postgres user.

`sudo -u postgres psql`   

- Execute the query below as a postgres user.

```
COPY (
SELECT '"' || rolname || '" "' ||
coalesce(rolpassword, '') || '"'
FROM pg_authid
)
TO '/etc/pgbouncer/userlist.txt';
```

