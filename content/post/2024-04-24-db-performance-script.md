---
date: "2024-04-24T00:00:00Z"
title: PostgreSQL performance test script
---


In case you want to run a quick performance test (in simple terms of access to DB) on a PostgreSQL server, this can be done quickly through a python module called psycopg2.

This is the python script I've used:

```python
#!/usr/bin/python3
import psycopg2
from psycopg2 import Error
import time

def create_connection():
    """ create a database connection to a PostgreSQL database """
    try:
        conn = psycopg2.connect(
            database="postgres",
            user="postgres",
            password="postgres",
            host="1.2.3.4",
            port="5432"
        )
        print('Successfully Connected to PostgreSQL')
        return conn
    except Error as e:
        print(e)

def check_table_exists(conn):
    """ check if a table exists in the PostgreSQL database """
    cur = conn.cursor()
    cur.execute(f"SELECT 1;")

def main():
#    table_name = 'pg_auth_members'

    # create a database connection
    conn = create_connection()

    # check if table exists
    with conn:
        start_time = time.time()
        checks = 0
        while True:
            check_table_exists(conn)
            checks += 1
            elapsed_time = time.time() - start_time
            if elapsed_time > 0:
                tps = checks / elapsed_time
                print(f'Current TPS: {tps}')
                print (f'Current checks: {checks}')

if __name__ == '__main__':
    main()

```

In heere you should update the database, user, password, host and port to something relevant to your environment.
Everything else can stay the same

But in order to run it, you will need to have python installed, pip3 package manager and psycopg2_binary python library.

In my case I already had python installed, I had to install python3-pip from a remote yum repository. I also had the pyscopg2 locally downloaded, but in most cases this could be installed [as per the official documentation](https://www.psycopg.org/install/).

```bash
[user@hostname ~]$ sudo yum install -y python3-pip
...
[user@hostname ~]$ sudo python3 -m pip install psycopg2_binary-2.8.6-cp36-cp36m-manylinux1_x86_64.whl
WARNING: Running pip install with root privileges is generally not a good idea. Try `__main__.py install --user` instead.
Processing ./psycopg2_binary-2.8.6-cp36-cp36m-manylinux1_x86_64.whl
Installing collected packages: psycopg2-binary
Successfully installed psycopg2-binary-2.8.6
[user@hostname ~]$

```

Then all you have to do is to start the test script:
```python
[user@hostname ~]$ python3 db-test.py
...
Current checks: 2356
Current TPS: 4067.3281247042887
Current checks: 2357
Current TPS: 4067.341871726699
Current checks: 2358
Current TPS: 4067.4275044859687
Current checks: 2359
...
```

As you can see, here is only a snippet, but the script is pretty powerfull, it runs continuously until stopped, and it gets to +4000 TPS. Don't forget to stop the script otherwise you may end up DOS-ing your own DB server.
