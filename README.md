## Sparkify Redshift Data Model

The purpose of this project is to create a suitable Redshift data model on AWS which can be used to analyse user behaviour based on song play logs and additional song data. 

### Description of Data Model:
We create various Redshift tables and insert data into them from the log and song files which are stored in Amazon S3. During this process, we copy data from the files in S3 to staging tables on Redshift, and then insert the data into the final tables from these staging tables. A short description of the tables follows.

#### Staging tables
1. staging_events: the data from all the log files in S3 are inserted as is into this staging table. Columns: artist, auth, firstName, gender, itemInSession, lastName, length, level, location, method,  page, registration, sessionId, song, status, ts, userAgent, userId.
2. staging_songs: the data from all the song files in S3 are inserted as is into this staging table. Columns: num_songs, artist_id, artist_latitude, artist_longitude, artist_location, artist_name, song_id, title, duration, year.

#### Dimension tables
1. songs: contains information about songs. This is fetched from the song data. Columns: song_id, title, artist_id, year, duration.
2. artists: contains information about the songs' artists. This is fetched from the song data.
3. time: contains information about when songs were played. This information is present in the log files. Columns: start_time, hour, day, week, month, year, weekday.
4. users: a table containing user information. The user information is obtained from the log files. Columns: user_id, first_name, last_name, gender, level.

#### Fact table
1. songplays: contains information pertaining to the event that a song is played. Contains user, time, artist and song info. Columns: songplay_id, start_time, user_id, level, song_id, artist_id, location, session_id, user_agent.

#### Choice of distkey and sortkeys

I chose the song_id as the distkey in the fact table songplays because it might create more equal slices of data than the other tables (except maybe, artists), and because songs is the largest dimension table (by no. of rows). A corresponding distkey is created on the primary key song_id in the songs table. (https://docs.aws.amazon.com/redshift/latest/dg/c_best-practices-best-dist-key.html)

The primary key of each dimension table is used as the sortkey in each slice. The sortkey of the fact table songplays is set to be the same as its distkey (i.e., song_id) so that Redshift can use the sort-merge join instead of the slower hash join (https://docs.aws.amazon.com/redshift/latest/dg/c_best-practices-sort-key.html). 

### Description of files.
The log and song data files are present in S3. They are all Json (Jsonline) files. An overview of the different files in the repository is given below. 

#### Python programs (.py files)
1. create_clusters.py: contains Infrastructure as code (IAC) code to create an IAM role on AWS, create a Redshift cluster and open incoming TCP ports on the corresponding EC2 machine.
2. sql_queries.py: contains Redshift/PostgreSQL DDL queries to create and drop tables, as well as DML queries to copy data into the staging tables and insert data into the final tables. 
3. create_tables.py: creates all the dimension and fact tables using the queries from sql_queries.py
4. etl.py: The ETL program which extracts data from the log files as well as the song files, stages them into temporary tables where required, and loads them into the Postgres tables. Temporary tables are used while inserting into the users, time and songplays tables.
5. get_table_counts.py: a program which executes select(*) queries on all the tables and stores the results in table_frequencies.txt.

### Execution
1. Run create_clusters.py to create a Redshift cluster.
1. Run create_tables.py to create all the Redshift tables (both staging and final analytics tables).
2. Run etl.py to copy data from the log and song files on S3 into staging tables and then insert this data into the analytics tables 
3. Run get_table_counts.py to get the table frequencies of all the tables.