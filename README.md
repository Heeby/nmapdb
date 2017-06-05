# NmapDB

nmapdb parses nmap's XML output files and inserts them into an SQLite database.

## Example usage:

```
$ sudo nmap -A -oX scanme.xml scanme.nmap.org

Starting Nmap ...

$ ls scanme.xml
scanme.xml
```
### Running:

```
$ ./nmapdb.py 
usage: nmapdb.py [-h] [--debug] [-d SCANDB] nmap_xml
nmapdb.py: error: too few arguments

$ ./nmapdb.py -h 
usage: nmapdb.py [-h] [--debug] [--force-update] [-d SCANDB] nmap_xml

Nmap XML file to SQLite database. Default file nmap.db will be used if none is
supplied.

positional arguments:
  nmap_xml              Nmap XML file you wish to parse

optional arguments:
  -h, --help            show this help message and exit
  --debug               Print verbose information
  --force-update        Overwrite previous information.
  -d SCANDB, --database SCANDB
                        Filename to use for database. If file doesn't exist it
                        will be created. Default is 'nmap.db.'

```
**nmapdb** will create a database from internal schema if the tables hosts and ports doesn't exist in your sqlite database. If no database file is supplied it will use the default file `nmap.db`. 

So long as you are ok with the defaults the program can be used with: `./nmapdb.py output.xml`. 

```
$ ./nmapdb.py scanme.xml   
[+] Start  >> Using DB file: nmap.db and XML file scanme.xml

$ file nmap.db 
nmap.db: SQLite 3.x database, last written using SQLite version 3014001

$ sqlite3 nmap.db 
SQLite version 3.14.1 2016-08-11 18:53:32
Enter ".help" for usage hints.
sqlite> select * from hosts;
45.33.32.156||scanme.nmap.org|ipv4|DD-WRT v24-sp2 (Linux 2.4.37)|Linux|100|2.4.X|1496637122|up||
sqlite> select * from ports;
45.33.32.156|22|tcp|ssh|open|product: OpenSSH version: 6.6.1p1 Ubuntu 2ubuntu2.8 extrainfo: Ubuntu Linux; protocol 2.0 ostype: Linux
45.33.32.156|80|tcp|http|open|product: Apache httpd version: 2.4.7 extrainfo: (Ubuntu)|[45.33.32.156:80] http-favicon:
[45.33.32.156:80] 
[45.33.32.156:80] http-title:
[45.33.32.156:80] Go ahead and ScanMe!
[45.33.32.156:80]   Elements:
[45.33.32.156:80]     title: Go ahead and ScanMe!
[45.33.32.156:80] 
45.33.32.156|9929|tcp|nping-echo|open|product: Nping echo|
45.33.32.156|31337|tcp|tcpwrapped|open||
sqlite> 
```

### Debug

Turning on debug with `--debug` will show you a lot more information:

```
$ ./nmapdb.py --debug scanme.xml
[+] Start  >> Debug Enabled
[+] Start  >> Using DB file: nmap.db and XML file scanme.xml
[+] Start  >> Successfully connected to SQLite DB "nmap.db"
[+] Start  >> Database already exists. Continuing.
[+] Parser >> Nmap Results: Nmap done at Mon Jun  5 00:32:02 2017; 1 IP address (1 host up) scanned in 16.90 seconds
[+] [host] >> ip:          45.33.32.156
[+] [host] >> mac:         
[+] [host] >> hostname:    scanme.nmap.org
[+] [host] >> protocol:    ipv4
[+] [host] >> os_name:     DD-WRT v24-sp2 (Linux 2.4.37)
[+] [host] >> os_family:   Linux
[+] [host] >> os_accuracy: 100
[+] [host] >> os_gen:      2.4.X
[+] [host] >> last_update: 1496637122
[+] [host] >> state:       up
[+] [host] >> mac_vendor:  
[+] [host] >> info:        
[+] [host] >> Inserting 45.33.32.156 in to database
[+] [host] >> Entry alread exists for 45.33.32.156 in database attempting update
[+] [port] >> Found script output for ssh-hostkey
[+] [port] >> ---------------------- Start  22    ----------------------
[+] [port] >> ip:       45.33.32.156
[+] [port] >> port:     22
[+] [port] >> protocol: tcp
[+] [port] >> name:     ssh
[+] [port] >> state:    open
[+] [port] >> service:  product: OpenSSH version: 6.6.1p1 Ubuntu 2ubuntu2.8 extrainfo: Ubuntu Linux; protocol 2.0 ostype: Linux
```

### Updates

Subsequent scans can be entered into the same database. New script output will be appended to existing script output. If there are conflicts between what is in the database and the XML file you will be prompted to update:

```
$ ./nmapdb.py --debug newscan.xml
[+] [port] >> ---------------------- Start  445    ----------------------
[+] [port] >> ip:       10.11.1.128
[+] [port] >> port:     445
[+] [port] >> protocol: tcp
[+] [port] >> name:     microsoft-ds
[+] [port] >> state:    open
[+] [port] >> service:  product: Windows 2000 microsoft-ds
[+] [port] >> info:     
[+] [host] >> Attempting to inserting 10.11.1.128:445 in to database
[+] [port] >> Entry alread exists for 10.11.1.128:445 in database attempting update
[+] [host] >> Could not update automatically - Manual update required
[!] [!!!!] >> ------------------- MANUAL UPDATE REQUIRED! -----------------------
[+] [port] >> 10.11.1.128:445 tcp exists
[+] [port] >> Name:     'Old' --> 'New'
[+] [port] >> ip:       '10.11.1.128' --> '10.11.1.128'
[+] [port] >> port:     '445' --> '445'
[+] [port] >> protocol: 'tcp' --> 'tcp'
[+] [port] >> name:     'microsoft-ds' --> 'microsoft-ds'
[+] [port] >> state:    'open' --> 'open'
[+] [port] >> service:  'product: Microsoft Windows 2000 microsoft-ds ostype: Windows 2000' --> 'product: Windows 2000 microsoft-ds'
[+] [port] >> Update entry? y/n
```


Feel free to fork, submit patches, whatever.

Huge thanks to Patroklos Argyroudis <argp at domain census-labs.com> for the original nmapdb.py which was the catalyst for this script. 

