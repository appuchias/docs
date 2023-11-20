---
created: 2023-10-25T15:45:30 (UTC +02:00)
tags: []
source: https://stackoverflow.com/questions/6076984/sqlite-how-do-i-save-the-result-of-a-query-as-a-csv-file/17938847#17938847
author: RayLovelessRayLoveless
---

# SQLite: How do I save the result of a query as a CSV file? - Stack Overflow

> ## Excerpt
> Is there a way I can export the results of a query into a CSV file?

---

To include column names to your csv file you can do the following:

```
sqlite> .headers on
sqlite> .mode csv
sqlite> .output test.csv
sqlite> select * from tbl1;
sqlite> .output stdout
```

To verify the changes that you have made you can run this command:

```
sqlite> .show
```

Output:

```
echo: off   
explain: off   
headers: on   
mode: csv   
nullvalue: ""  
output: stdout  
separator: "|"   
stats: off   
width: 22 18 
```
