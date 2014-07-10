Documentation
=============

Create a basic conection, insert and query
------------------------------------------

```
/* Table declaration, if the table doesnt exists when inserting and a position
 * inside $GLOBALS['tables'] matchs the table name, the lib will create the table
 * based on this declaration and will retry the inserction query */
$GLOBALS['tables']['test'] = array('_id_'=>'INTEGER AUTOINCREMENT','data'=>'INTEGER NOT NULL');

include_once('inc.sqlite3.php');
$d = microtime(1);

$db = sqlite3_open('test.db');
/* Begin a transaction */
sqlite3_exec('BEGIN;',$db);

$i = 1;while($i < 10001){
	/* Underscores around the array index means this is the table key, when 
	 * a key collide, the lib will update the row. The struct of a row
	 * should have the same fields declared in table declaration */
	$row = array('_id_'=>$i,'data'=>$i);

	/* First param is table name, second param is an associative array 
	 * mapping the row fields: 'fieldName'=>'fieldValue'. Third is a
	 * mixed array, for example, position 'db' means 'reuse this database 
	 * connection' */
	$r = sqlite3_insertIntoTable2('test',$row,array('db'=>$db));
	if(isset($r['errorDescription'])){print_r($r);exit;}

	$i++;
}

/* Close the database conection, if second param is true then 
 * will COMMIT the current transaction */
$r = sqlite3_close($db,true);
echo 'Time spent: 'round(microtime(1)-$d,2).PHP_EOL;
```

If you are going to make a solely insert, you can point via params the path
of the database and all the other commands will be done in an automatic way

```
$GLOBALS['tables']['test'] = array('_id_'=>'INTEGER AUTOINCREMENT','data'=>'INTEGER NOT NULL');

$params = array('db.file'=>'test.db');
$row = array('_id_'=>1,'data'=>5);
$r = sqlite3_insertIntoTable2('test',$row,$params);
```

Note: When doing multiple secuential inserts is highly recomendable begin a 
transaction first


Query data from the database
----------------------------

```
/* Open database, if we are not going to write it better choose SQLITE3_OPEN_READONLY mode,
 * default mode if not supplied is (SQLITE3_OPEN_READWRITE | SQLITE3_OPEN_CREATE) */
$db = sqlite3_open('test.db',SQLITE3_OPEN_READONLY);

/* Query de data, first param is the table name. Second param is the whereClause, 
 * in other words the clause you write following WHERE in a query sentence. Third 
 * param is a mixed array:
 * 'db' means 'reuse this database connection'
 * 'order' param will be appended in the query, just like 'ORDER BY '.
 * 'limit' param will be appended in the query, just like 'LIMIT'.
 * 'group' param will be appended in the query, just like 'GROUP BY '.
 * 'indexBy' will be the field that acts as index of $rows array, it can be 'false' for
 * no indexing the array. */
$params = array('db'=>$db,'order'=>'id DESC','limit'=>10);
$rows = sqlite3_getWhere('test','(id > 10)',$params);
$r = sqlite3_close($db);
print_r($rows);
```

Alternatively you can supply just the path for the database to query, and
all the other commands will be done in an automatic way
```
$params = array('db.file'=>'test.db','order'=>'id DESC','limit'=>10);
$rows = sqlite3_getWhere('test','(id > 10)',$params);
print_r($rows);
```

Working with encryption
-----------------------

Inserting data

```
/* Open database with third param, this time the password */
$db = sqlite3_open('test.db',SQLITE3_OPEN_READWRITE | SQLITE3_OPEN_CREATE,'password');

/* Begin transaction */
sqlite3_exec('BEGIN;',$db);

/* When making inserctions provide an array with a list of fields to encrypt (as indexes)
 * in the position 'db.encrypt' */
$params = array('db'=>$db,'db.encrypt'=>array('data'=>0));

$i = 1;while($i < 10001){
	$r = sqlite3_insertIntoTable2('test',array('_id_'=>$i,'data'=>$i),$params);
	if(isset($r['errorDescription'])){print_r($r);exit;}
	$i++;
}

/* Close conection with true as second param to commit */
sqlite3_close($db,true);
```

Quering data

```
/* Open database with third param, this time the password */
$db = sqlite3_open('test.db',SQLITE3_OPEN_READONLY,'password');

/* Pass a list of fields to decrypt (as indexes) in the position 'db.encrypt' */
$rows = sqlite3_getWhere('test',1,array('db'=>$db,'db.encrypt'=>array('data'=>0),'limit'=>10));

/* Close conection */
sqlite3_close($db);
```

```
/* Open database with third param, this time the password */
$db = sqlite3_open('test.db',SQLITE3_OPEN_READONLY,'password');

$params = array('db'=>$db,'db.encrypt'=>array('data'=>0));
$rows = sqlite3_getWhere('test','decrypt(data) = 10',$params);

/* Close conection */
sqlite3_close($db);
```

Debug
-----

After every process the next globals will be feed:

$GLOBALS['DB_LAST_QUERY'] // last query/exec

$GLOBALS['DB_LAST_QUERY_ERRNO'] // last error number

$GLOBALS['DB_LAST_QUERY_ERROR'] // last error description

$GLOBALS['DB_LAST_QUERY_CHANG'] // last changes (when suitable)


