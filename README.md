Documentation
=============

Create a basic conection, insert and query
------------------------------------------

```
$GLOBALS['tables']['test'] = array('_id_'=>'INTEGER AUTOINCREMENT','data'=>'INTEGER NOT NULL');

include_once('../PHP/inc.sqlite3.php');
$d = microtime(1);

$db = sqlite3_open('test.db');

sqlite3_exec('BEGIN;',$db);
$i = 1;while($i < 10001){
	$r = sqlite3_insertIntoTable2('test',array('_id_'=>$i,'data'=>$i),array('db'=>$db));
	if(isset($r['errorDescription'])){print_r($r);exit;}
	$i++;
}

sqlite3_close($db,true);
echo round(microtime(1)-$d,2).PHP_EOL;
```
