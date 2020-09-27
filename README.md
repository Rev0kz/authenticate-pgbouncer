# authenticate-pgbouncer
Shows how to authenticate pgbouncer via the `userlist.txt` file or `auth_query` method.

## Pre-requisite
Make sure the postgres user have complete access to files `/etc/pgbouncer/pgbouncer.ini`, `/var/run/pgbouncer`, `/var/log/pgbouncer`

## Authenticate via userlist.txt file
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

## Authenticate pgbouncer via the auth_query method  
Follow the steps below to create a user to access password on behalf of other users from the postgres database via `auth_query` method who connect to pgpbouncer.

```
CREATE ROLE pgbouncer LOGIN;
-- set a password for the user
\password pgbouncer
 
CREATE FUNCTION public.lookup (
   INOUT p_user     name,
   OUT   p_password text
) RETURNS record
   LANGUAGE sql SECURITY DEFINER SET search_path = pg_catalog AS
$$SELECT usename, passwd FROM pg_shadow WHERE usename = p_user$$;
 
-- make sure only "pgbouncer" can use the function
REVOKE EXECUTE ON FUNCTION public.lookup(name) FROM PUBLIC;
GRANT EXECUTE ON FUNCTION public.lookup(name) TO pgbouncer;
```

Inside the `/etc/pgbouncer/pgbouncer.ini` file under the `pgbouncer` section, edit it as specified below: 

```
[pgbouncer]
auth_type = md5
auth_file = /etc/pgbouncer/userlist.txt
auth_user = pgbouncer
auth_query = SELECT p_user, p_password FROM public.lookup($1)
```



