# ReadMe Draft for getting IMDbPy to build a sql database

```bash
mysql> wc -l en_US.twitter.txt 
```


All on a Windows 8 machine. Getting all this right took quite a while, so we document it here for future versions of ourselves

# Python set up

* python via anaconda
* IMDbPY needs python 2
    - Create a python 2 environment
    - activate and use must be in the cmd prompt
* need to install some things
    - pip install IMDbPY
    - conda install mysql-python

## Creating and activating  a python 2 environment with Anaconda 

* In cmd.exe:
* conda create -n python2 python=2.7 anaconda
* activate python2

# Other programs we need to have installed
* Need to install MySQL
* Microsoft Visual C++ Compiler for Python 2.7
    - https://www.microsoft.com/en-us/download/details.aspx?id=44266
* wget for windows
    - https://eternallybored.org/misc/wget/
* I think this is all...

# Import the IMDb Data

* use wget
    - I have wget.exe in the folder where I want the data to be dumped
    - alternatively have it in somewhere your machine knows (PATHS)
* Commands:
    - mkdir /root/data
    - cd /root/data
    - wget -r --accept="*.gz" --no-directories --no-host-directories --level 1 ftp://ftp.fu-berlin.de/pub/misc/movies/database/
* Unzip all .gz files


# Check where MySQL is storing data

* open mysql command line
* Command
    - show variables like 'datadir';
* If this is not where you want to store data, we need to move the data directory (see below)

## Moving the Data Directory in MySQL

* Is more of a challenge than i hoped it to be
* Remark: I...
    - am running MySQL in Windows 8
    - used the MySQL MSI to install it

So let's move the data directory

* Open command prompt and change directory to
    - cd C:/ProgramData/MySQL/MySQL Server 5.7
    - dir *.ini
* We need to change one line in my.ini, to do so requires opening it with administrator priviledges
    - I opened notepad with admin privs
    - find the line with datadir = ...
    - change to where you want it to go
    - Command
        + datadir=D:/MySQLdata
    - Save the my.ini
* Now we need to stop mysql from the cmd prompt. open cmd.exe with admin privs, then
    - net stop mysql57
* Next, make a copy of the entire data folder in the new location
    - xcopy "C:\ProgramData\MySQL\MySQL Server 5.1\data" D:/MySQLdata /s
* start up MySQL from command prompt
    - net start mysql57
* Try logging into mysql. Once you can login to mysql successfully, run this command:
    - show variables like 'datadir';
If D:/MySQLdata shows up as the datadir, CONGRATULATIONS, YOU HAVE DONE IT RIGHT !!!

Once you are satisfied all your apps hitting MySQL works, you can delete everything in C:\ProgramData\MySQL\MySQL Server 5.1\data\*

# Create a database in MySQL
* open MySQL command line
* Commands:
    - CREATE DATABASE imdb;
    - GRANT ALL PRIVILEGES ON imdb.* TO 'imdb'@'localhost' IDENTIFIED BY 'imdb';
    - FLUSH PRIVILEGES;


# Import to .list files to MySQL using IMDbPY

* use cmd.exe and a python2 environment
* navigated to where imdb2sql.py was stored 
    - C:\Users\ldeer\AppData\Local\Continuum\Anaconda3\envs\python2\Scripts
* Command:
    - python imdbpy2sql.py -d /root/data/ -u mysql://imdb:imdb@localhost/imdb

