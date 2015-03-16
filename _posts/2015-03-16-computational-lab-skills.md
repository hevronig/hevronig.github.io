## Computational lab skills


An ever updating-unorganized-list of short snippets of code I find usefull.

**This is list is mainly derived from [Software Carpentry Lessons](http://software-carpentry.org/lessons.html) and Vince Buffalo's excellent book [Bioinformatics Data Skills](http://shop.oreilly.com/product/0636920030157.do)**, together with solutions I found online (e.g. stacksoverflow, biostar, etc.).

### Basic terminal commands

##### What user are you logged into?

```
$ whoami
```

##### What is your current directory (folder)?

```
$ pwd
```

##### What's in the directory?

```
$ ls
```

##### Distinguish between files and subdirectories with -F

```
$ ls -F
```

##### Navigate between directories

```
$ cd git-swc/
$ cd ..
```

##### Create a new directory

```
$ mkdir test_dir
$ ls -F
$ cd test_dir
```

##### Create a file

```
$ touch file.txt
```

or

```
$ > file.txt # Overwrite file if exist
$ >>file.txt # Append to file if exist
```

or with a text editor:  

```
$ vi file.txt # Can be used with text editor of preference (e.g. vim, nano, etc.)
```

##### merge multiple files into one

```
$ cat *.csv>merge.csv
```

##### Delete file or folder

```
$ rm test_file.txt # Delete file
$ rmdir test_dir # Delete an empty folder
$ rm -r test_dir # WARNING!!! Recursive delete folder with its files - NO WAY TO UNDO THIS!
```

##### Copy or move a file to a different directory

```
$ cp another_test_file.txt destination_dir # Copy the file destination directory
$ mv another_test_file.txt destination_dir # Move the file destination directory
```

##### Copy multiple files from one dir to another:

```
$ cp test_dir/{test_file.txt, another_test_file.txt} destination_dir
```

if the file names are continuos

```
$ cp test_dir/test_file{1..4} destination_dir
```

or for a specific file type

```
$ cp -n *.txt destination_dir
```

##### Count lines, words, charecters in a file

```
$ wc some_file.txt # Count all three
$ wc -l some_file.txt # Count lines
$ wc -c some_file.txt # Count charecters
```

### Pipes and short bash scripts

##### Count lines in files -> Sort by ascending line number -> Display first five

```
$ wc -l *.txt | sort -n | head -5 
```

##### Sort and write unique values to a file

```
$ sort list_with_duplicates.txt | uniq > unique_list.txt
```

##### Copy last n command lines from the terminal to a file (to document a workflow for reproducibility)

```
$ history | tail -4 | colrm 1 7 > commandrecord.sh
```

##### Sort (by number of lines) files with specific extention in a directory and return the highest
(Where, "$1" = directory path, "$2" = file extention w/o '.')

```
$ wc -l "$1"/*."$2" | sort -nr | head -2 | tail -1
```

##### Bash script to iterate a list of files; cut, sort and returns unique entries

```
$   for animal in "$@"
... do
...     echo $animal
...     cut -f 2 -d',' $animal | sort | uniq
...     echo "----"
... done
```

### Biopieces 

##### Search and extract subset from a fasta file with an index (for fastq --> change fasta to fastq)

```
$ read_fasta -i path/to/file.fa | grab -p name_of_read_1, name_of_read_2, name_of_read_3, name_of_read_4 path/to/index/file_index.txt | write_fasta -xo path/to/output_file.fa
```

##### Sort fasta file and remove duplicates

```
$ read_fasta -i mock.fa | uniq_seq -c | add_ident -k SEQ_NAME | write_fasta -x > new_mock.fa
```
or better:

```
$ cat file.fa | parallel --block 100M -k "read_fasta -i - | uniq_vals -k SEQ_NAME | write_fasta -x > file_uniq.fa"
```

### Local BLAST
Requires installation of [ncbi blast+](http://blast.ncbi.nlm.nih.gov/Blast.cgi?PAGE_TYPE=BlastDocs&DOC_TYPE=Download)

For blast options (after installation) type `blastn -help` in the terminal.

##### Run local blast against nr
```
$ time blastx -db nr -query /path/to/query/file.fa -out /path/to/results/dir/file.tsv -outfmt '6 qseqid sseqid stitle pident length mismatch gapopen qstart qend sstart send evalue bitscore staxids sskingdoms' -num_threads 8 -evalue 0.01 &
```

##### Iterate over a directory with query files and blast against nr
```
for i in ~/path/to/query/dir/*.fa; do time blastx -db nr -query $i -out ${i/.fa/.tsv} -outfmt '6 qseqid sseqid stitle pident length mismatch gapopen qstart qend sstart send evalue bitscore staxids sskingdoms' -num_threads 8 -evalue 0.01; done
```


### SQLite

##### Show a list of all the column names. 

[stacksoverflow](http://stackoverflow.com/questions/947215/how-to-get-a-list-of-column-names-on-sqlite3-iphone)

```
> PRAGMA table_info(table_name);
```

##### Select specific data and perform arithmatic function

```
> SELECT table_info1 / 100 FROM table_name WHERE table_info2="value1" AND table_info3="value2";
```

##### UNION operator combines the results of two queries
```
> SELECT table_info1, table_info2 / 100 FROM table_name WHERE table_info3="value1" AND table_info4="value2" UNION SELECT table_info1, table_info2 FROM table_name WHERE table_info3!="value1" AND table_info4="value2";
```

##### instr(X, Y) and substr(X, I)

_"The "in string" function instr(X, Y) returns the 1-based index of the first occurrence of string Y in string X, or 0 if Y does not exist in X. The substring function substr(X, I) returns the substring of X starting at index I_". 
[w3resource.com](http://www.w3resource.com/sqlite/core-functions-instr.php)

```
>SELECT DISTINCT substr(table_info, 0,instr(table_info, "-")) FROM table_name;
```

##### GROUPBY groups all the records with the same value for the specified field together so that aggregation can process each batch separately.

```
> SELECT   table_info1, count(table_info2), round(avg(table_info2), 2)
> FROM     table_name
> WHERE    table_info3="value"
> GROUP BY table_info1;
```

##### Useful SQLite3 Dot Commands. Vince Buffalo: “Bioinformatics Data Skills.” Table 13-1

Command	| Description
---|---
.exit, .quit | Exit SQLite
.help | List all SQLite dot commands
.tables | Print a list of all tables
.schema | Print the table schema (the SQL statement used to create a table)
.headers <on,off> | Turn column headers on, off
.import | Import a file into SQLite (see (to come))
.indices | Show column indices (see Indexing)
.mode | Set output mode (e.g. csv, column, tabs, etc., see .help for full list)
.read <filename> | Execute SQL from file

##### Join many tables with ON (primary key, forein key)

```
> SELECT table_name1.table_info1, table_name1.table_info2, table_name2.table_info3, table_name3.table_info4, table_name3.table_info5
> FROM   table_name1 JOIN table_name2 JOIN table_name3
> ON     table_name1.table_info6=table_name2.table_info7
> AND    table_name2.table_info8=table_name3.table_info9
> AND    table_name2.table_info10 IS NOT NULL;
```

##### Create (or delete=DROP) a new table; arguments are the names and types of the table's columns
```
> CREATE TABLE table_name1(table_info1 integer, table_info2 text, table_info3 real, table_info4 real);
> DROP TABLE table_name;
```

or more detailed

```
CREATE TABLE table_name1(
    table_info1   integer not null, -- where reading taken
    table_info2  text,             -- may not know who took it
    table_info3   real not null,    -- the quantity measured
    table_info4 real not null,    -- the actual reading
    primary key(table_info1, table_info3),
    foreign key(table_info1) references table_name2(table_info5),
    foreign key(table_info2) references table_name3(table_info6)
);
```

##### Insert values into a table

```
> INSERT INTO table_name VALUES('value1', 'value2', 'value3');
```

##### Insert values into one table directly from another

```
> CREATE TABLE new_table_name(table_info1 text, table_info2 text);
> INSERT INTO new_table_name SELECT table_info1, table_info2 FROM old_table_name;
```

##### Modify existing records

**(be care to not forget the WHERE clause or the update statement will modify all of the records in the database)**

```
> UPDATE table_name SET table_info1='value1', table_info2='value2' WHERE table_info3='values4' 
```

##### Delete data from a table (Warning - see referential integrity problem)

```
DELETE FROM table_name WHERE table_info = 'value';
```
