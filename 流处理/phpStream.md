```
<?php

class DBStream {
    private $_pdo;
    private $_ps;
    private $_rowId = 0;

    function stream_open($path, $mode, $options, &$opath)
    {
        $url = parse_url($path);
        $url['path'] = substr($url['path'], 1);
        try{
            $this->_pdo = new PDO("mysql:host={$url['host']};dbname={$url['path']}", $url['user'], isset($url['pass'])? $url['pass'] : '', array());
        } catch(PDOException $e){ return false; }
        switch ($mode){
            case 'w' :
                $this->_ps = $this->_pdo->prepare('INSERT INTO data VALUES(null, ?, NOW())');
                break;
            case 'r' :
                $this->_ps = $this->_pdo->prepare('SELECT id, data FROM data WHERE id > ? LIMIT 1');
                break;
            default  : return false;
        }
        return true;
    }

    function stream_read()
    {
        $this->_ps->execute(array($this->_rowId));
        if($this->_ps->rowCount() == 0) return false;
        $this->_ps->bindcolumn(1, $this->_rowId);
        $this->_ps->bindcolumn(2, $ret);
        $this->_ps->fetch();
        return $ret;
    }

    function stream_write($data)
    {
        $this->_ps->execute(array($data));
        return strlen($data);
    }

    function stream_tell()
    {
        return $this->_rowId;
    }

    function stream_eof()
    {
        $this->_ps->execute(array($this->_rowId));
        return (bool) $this->_ps->rowCount();
    }

    function stream_seek($offset, $step)
    {
        //No need to be implemented
    }
}

stream_register_wrapper('db', 'DBStream');

$fr = fopen('db://root:123456@localhost/swoft', 'r');
$fw = fopen('db://root:123456@localhost/swoft', 'w');
//The two forms above are accepted : for the former, the default password "" will be used

$alg = hash_algos();
$al = $alg[array_rand($alg)];
$data = hash($al, rand(rand(0, 9), rand(10, 999))); // Some random data to be written
fwrite($fw, $data); // Writing the data to the wrapper
while($a = fread($fr, 256)){ //A loop for reading from the wrapper
    echo $a . "\n";
}
?>
```

##参考文档
- [php文档](https://www.php.net/manual/zh/stream.streamwrapper.example-1.php)