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
	 * mapping the row fields: 'fieldName'=>'fieldValue'. Third is an
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
