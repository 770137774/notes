###1，种子函数 srand()

相同的随机数种子会产生相同的随机数！
```
   for ($i=0 ;$i < 10 ;$i++){
        srand(time());
        echo rand(1,200);
        echo "<br/>";
   }
          
```
https://www.php.net/mt_rand

###2，register_tick_function()打点函数
```
$n = 1000; // Size of your input

declare(ticks=1);

class Counter {
    private $counter = 0;

    public function increase()
    {
        $this->counter++;
    }
   
    public function print()
    {
        return $this->counter;   
    }
}

$obj = new Counter;

register_tick_function([&$obj, 'increase'], true);

for ($i = 0; $i < 100; $i++)
{
    $a = 3;
}

// unregister_tick_function([&$obj, 'increase']);
// Not sure how to de-register, you can use static methods and members in the Counter instead.

var_dump("Number of basic low level operations: " . $obj->print());

```

###3，register_shutdown_function()注册一个会在php中止时执行的函数
```
function shutdown()
{
    // This is our shutdown function, in 
    // here we can do any last operations
    // before the script is complete.

    echo 'Script executed with success', PHP_EOL;
}

register_shutdown_function('shutdown');
```