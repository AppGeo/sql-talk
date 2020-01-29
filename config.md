# config files

Files are in `/etc/postgresql/<version>/main/`

with version being things like `9.6` or `12`;

Config can be reloaded with zero down time with `sudo pg_ctlcluster <version> main reload`

`sudo pg_ctlcluster <version> main restart` can be used to reboot the server(some downtime, needed for certain things).

## pg_hba.conf

`/etc/postgresql/<version>/main/pg_hba.conf` has where you can white list various ips

' TYPE    ' DATABASE       ' USER   ' ADDRESS         ' METHOD '
'---------'----------------'--------'-----------------'--------'
' hostssl ' all            ' calvin ' 20.256.17.22/32 ' md5    '
' hostssl ' all            ' calvin ' 256.16.0.0/14   ' md5    '

The top row means users named speed with that IP address can access all databases via ssl with a password

This should have the database specified not all, that's bad on me

The 2nd row lets container engine connect, and it uses CIDR notation (they technically both do, but /32 means just a single ip address), 256.16.0.0/14 means 256.16.0.0 - 256.19.255.255

for type
- A lot of places have 'host' for the type, that means it allows non ssl connections, this is bad don't do it.
- Local is a type which is what is usually set up for a local connection on your computer it uses unix sockets
- md5 automatically upgrades to a better password algo if the database is configured correctly.
- If trust is used it just lets anyone log in, this is probably how your local database is set up (and that's fine)
- If peer is in the method it uses the unix user name to log in, this is how it's set up for the postgres superuser and allows you to bootstrap access to a remote server by loging into the postgres account with `sudo su postgres` and then using `psql` to connect to the database, you can then create other users

## postgresql.conf

`/etc/postgresql/<version>/main/postgresql.conf`

has 2 settings you probalby want to change

- Setting `listen_address='*'` is necessary if you want anyone not on the that machine to be able to connect to it (almost all apps should have this set)
- `password_encryption=scram-sha-256` should be set to make sure passwords use the better encryption method
