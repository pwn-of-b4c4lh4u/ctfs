## Numberizer

We are given the following php code.
Essentially it gets a maximum of 5 numbers and add them up. 
It checks if each number has more then 4 characters and if its a number with is_numeric.
If so, then it does two intval which get the integer from a variable and check if the number is higher then 0.
To get the sum needs to be bellow then 0.

```php
<?php
ini_set("error_reporting", 0);

if(isset($_GET['source'])) {
    highlight_file(__FILE__);
}

include "flag.php";

$MAX_NUMS = 5;

if(isset($_POST['numbers']) && is_array($_POST['numbers'])) {

    $numbers = array();
    $sum = 0;
    for($i = 0; $i < $MAX_NUMS; $i++) {
        if(!isset($_POST['numbers'][$i]) || strlen($_POST['numbers'][$i])>4 || !is_numeric($_POST['numbers'][$i])) {
            continue;
        }
        $the_number = intval($_POST['numbers'][$i]);
        if($the_number < 0) {
            continue;
        }
        $numbers[] = $the_number;
    }
    $sum = intval(array_sum($numbers));


    if($sum < 0) {
        echo "You win a flag: $FLAG";
    } else {
        echo "You win nothing with number $sum ! :-(";
    }
}
?>

<html>
    <head>
        <title>Numberizer</title>
    </head>
    <body>
        <h1>Numberizer</h1>
        <form action="/" method="post">
            <label for="numbers">Give me at most 10 numbers to sum!</label><br>
            <?php
            for($i = 0; $i < $MAX_NUMS; $i++) {
                echo '<input type="text" name="numbers[]"><br>';
            }
            ?>
            <button type="submit">Submit</button>
        </form>
        <p>To view the source code, <a href="/?source">click here.</a>
    </body>
</html>

```

The first thought was int overflow. Looking at the intval docs https://www.php.net/manual/en/function.intval.php we have an example: `echo intval(420000000000000000000);   // -4275113695319687168`. However since we can only sum numbers with 4 digits we cant reach that number.

Well, it was what we thought but is_numeric validates the following as a number: `'1337e0' is numeric` and intval has an example with that too: `echo intval('1e10');   // 10000000000`. Put everything together we get a int overflow with the following payload: `numbers%5B%5D=8e90&numbers%5B%5D=8e90&numbers%5B%5D=8e90&numbers%5B%5D=8e10&numbers%5B%5D=8e10` which is: `numbers[]=8e90&numbers[]=8e90&numbers[]=8e90&numbers[]=8e10&numbers[]=8e10`.
