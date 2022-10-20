# Install Drive Unix ODBC

## 1. References

- [Unix ODBC](http://www.unixodbc.org/)
- [FreeTDS - Stable](https://www.freetds.org/files/stable/)

-------

## 2. Install Unix ODBC

```bash
mkdir -p ~/lib-external/
pushd ~/lib-external/

#wget http://www.unixodbc.org/unixODBC-2.3.9.tar.gz
#tar -xvf unixODBC-2.3.9.tar.gz
wget --no-check-certificate  http://www.unixodbc.org/unixODBC-2.3.11.tar.gz
tar -xvf unixODBC-2.3.11.tar.gz

#cd unixODBC-2.3.9
cd unixODBC-2.3.11
./configure
make -j6
sudo make install
popd
```

## 3. Install FreeTDS

```sh
mkdir -p ~/lib-external/
pushd ~/lib-external/

wget --no-check-certificate https://www.freetds.org/files/stable/freetds-1.3.12.tar.gz
tar -xvf freetds-1.3.12.tar.gz
cd freetds-1.3.12
./configure
make -j6
sudo make install

wget https://www.freetds.org/files/stable/freetds-1.3.6.tar.gz
tar -xvf freetds-1.3.6.tar.gz
cd freetds-1.3.6
./configure
make -j6
sudo make install
popd

```


## 4. Configure FreeTDS

* Edit the `/usr/local/etc/freetds.conf` file to add servers.
* Test the connection with `/usr/local/bin/tsql`
* Build and test the package that requires FreeTDS.

> sudo vim /usr/local/etc/freetds.conf

```ini
[DevServerSQLServer]
        host = xx.xx.xx.xx
        port = 1433
        tds version = 7.3
        database myDatabase
```

```bash
/usr/local/bin/tsql  -S DevServerSQLServer -U myuser  -P myPassword
```


> sudo vi /usr/local/etc/odbcinst.ini

```ini
[FreeTDS]
Description     = v0.63 with protocol v8.0
Driver          = /usr/local/lib/libtdsodbc.so
# /usr/local/freetds/lib/libtdsodbc.so
```


> sudo vim /usr/local/etc/odbc.ini

```ini
[MSSQLDev]
# Driver      = FreeTDS
Driver      = /usr/local/lib/libtdsodbc.so
Description = My database SQL Server
Trace       = No
TDS_Version = 8.0
Server      = xx.xx.xx.xx
Port        = 1433
Database    = myDatabase
```

```bash
export LD_LIBRARY_PATH=${LD_LIBRARY_PATH}:/usr/local/lib/
isql -v MSSQLDev myuser myPassword
```

