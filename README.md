# Using IMDbPY to construct a MySQL database of the IMDb Data

## Motivation & Introduction

The [IMDbPY package](http://imdbpy.sourceforge.net/) is a great tool to get and manage database with the data from [IMDb](www.imdb.com/). 
As great as it is, I struggled a little to get started and construct a MySQL database filled with the database dumps of IMDb.

In this file, I **try**  to document as best as possible how I got up and running.

Getting this all to work to me quite a while, so I document it here for future versions of myself and anyone else who is interested.

The instructions are build for a machine running Windows 8 with the following programs already installed:

* [Python](https://www.python.org/) via [Anaconda](https://www.continuum.io/downloads)
* [MySQL](https://www.mysql.com/downloads/) installed with the MySQL MSI installer.

Mileage will likely vary for other setups.

In what follows commands entered in the windows command prompt will appear as

```bash
cmd> /commands/entered
```

And command entered into a MySQL command line interface as 

```bash
mysql> /commands/entered
```

*Remark:* Using GitBash or Windows Powershell will not (unfortunately) work for us here, because we need to set and maintain a Python 2 environment.

## Other Things You Need to Have Installed

In addition to Python and MySQL I have installed the following programs which are necessary for what I do below to work:

* [Microsoft Visual C++ Compiler for Python 2.7](https://www.microsoft.com/en-us/download/details.aspx?id=44266)
* [wget for Windows](https://eternallybored.org/misc/wget/)

## Creating a Python 2 Environment

If your like me, you run a Python 3.5 setup. However IMDbPY only runs on Python 2. 

Let's create a separate Python 2 environment here.

```bash
cmd> conda create -n python2 python=2.7 anaconda
```

And then activate the environment

```bash
cmd> activate python2
```

### Installing additional packages

We need two python packagesto run everything
    1. IMDbPY to build the database
    2. MySQL-python to interface between Python and MySQL

We install these as follows:

```bash
cmd> pip install IMDbPY
cmd> conda install mysql-python
```

Note that "conda install" installs from the Anaconda package manager and was the easiest way for me to get the Python - MySQL interface to work quickly.

## Checking Where MySQL stores data

This didnt occur to me at first, but we may want to change the default path where MySQL stores the data. For me I wanted to move it of my (small) SSD and onto my larger HDD drive.

Here is how to do that:

* Check where MySQL currently stores the data

```bash
mysql> show variables like 'datadir';
```

for me it was "C:/ProgramData/MySQL/MySQL Server 5.7/data"

Then exit

```bash
mysql> exit
```

* Change the default directory
    * First stop MySQL running in the background
```bash
cmd> net stop mysql57
```
    * Find the .ini file that sets the default directory
```bash
cmd> cd C:/ProgramData/MySQL/MySQL Server 5.7
cmd> dir *.ini
```

    * We need to change one line in my.ini, to do so requires opening it with administrator priviledges
        - I opened notepad with admin privs
        - find the line with datadir = ...
        - change to where you want it to go
            + datadir=D:/MySQLdata
        - Save the my.ini
* Restart MySQL server
```bash
cmd> net start mysql57
```

* Check that everything worked
```bash
mysql> show variables like 'datadir';
```

If you did everything properly. you should see that the new data directory is "D:/MySQLdata"

**Remark:** if you skip this step and decide later that you want to change where you store the data, I provide a small how to at the bottom of this document.

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

