# Using IMDbPY to construct a MySQL database of the IMDb Data

## Motivation & Introduction

The [IMDbPY package](http://imdbpy.sourceforge.net/) is a great tool to get and manage database with the data from [IMDb](www.imdb.com/). 
As great as it is, I struggled (I was totally new to **all** of this ) a little to get started and construct a MySQL database filled with the database dumps of IMDb.

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

## Downloading the IMDb Data

Finally we are ready to download the IMDb data... 

Every week IMDb dumps a copy of the "publicly available" data to an ftp server. We will use [this one](ftp://ftp.fu-berlin.de/pub/misc/movies/database/)

We are going to use *wget* to download all the .gz files.

Here's how I did it:

* Create a directory to save the IMDb data
```bash
cmd> mkdir D:\0Local.Files\Data_Movies\imdb
cmd> cd D:\0Local.Files\Data_Movies\imdb
```

* have wget.exe in the folder where I want the data to be downloaded to
    - alternatively have it in somewhere your machine knows (PATHS)

* Download the data
```bash
cmd> wget -r --accept="*.gz" --no-directories --no-host-directories --level 1 ftp://ftp.fu-berlin.de/pub/misc/movies/database/
```

Now all of the data is saved to your machine, now we use **IMDbPY** to spin this into a MySQL data base.

## Create a new database in MySQL to store the data

I create a database  in MySQL called ‘imdb’, with user ‘imdb’ and password ‘imdb’. I then GRANT the user to the designated database.

* Open MySQL command line

```bash
mysql> CREATE DATABASE imdb;
mysql> GRANT ALL PRIVILEGES ON imdb.* TO 'imdb'@'localhost' IDENTIFIED BY 'imdb';
mysql> FLUSH PRIVILEGES;
```

* Then Exit MySQL command line

```bash
mysql> exit
```

## Use IMDbPY to create the database

We are almost there!

Now we use IMDbPY to convert all the data stored in .gz files to a MySQL database.

Here's how I did it:

* Make sure you still in a python 2 environment
```bash
cmd> python -V
```
it should tell you you are using python 2.7; if not, reactivate the python 2 environment (see above)

* We want to use the script "imdbpy2sql.py"; Navigate to where that script is located
```bash
cmd> cd C:\Users\ldeer\AppData\Local\Continuum\Anaconda3\envs\python2\Scripts
```
* Run the script to import data using the **-d** and **-u** flags
```bash
cmd> python imdbpy2sql.py -d D:\0Local.Files\Data_Movies\imdb -u mysql://imdb:imdb@localhost/imdb
```

 **-d** is the directory of the .gz dump files are located and **-u** is the connection string for our MySQL database server.

For me, the entire process took quite a while:

* About 3 hours until it started "adding foreign keys"
* More than 24 hours from "adding foreign keys" until end of script.

**Now you are done!**

## Acknowledgements

In setting up this guide I was heavily influenced - and shamelessly copied in parts, a shorter blog entre by "SECAGUY" located [here](http://blog.secaserver.com/2013/08/importing-imdb-sample-data-set-mysql/). In constructing this file, I hope to have elaborated a little more on some things that I spent hours googling around for as errors kept appearing when I first followed these steps.

## Appendix: Changing where MySQL stores data at a later date

You may have decided to not change where MySQL stores data by default when you ran through the first time. But later you may want to move it. Here is how, including copying all data across to the new location.

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

* Next, make a copy of the entire data folder in the new location
```bash
cmd> xcopy "C:\ProgramData\MySQL\MySQL Server 5.1\data" D:/MySQLdata /s
```

* Restart MySQL server
```bash
cmd> net start mysql57
```

* Try logging into mysql. Once you can login to mysql successfully, run this command:

```bash
mysql>show variables like 'datadir';
```

If you did everything properly. you should see that the new data directory is "D:/MySQLdata"

Once you have all your apps working and finding the new data directory you can delete everything in C:\ProgramData\MySQL\MySQL Server 5.1\data\
