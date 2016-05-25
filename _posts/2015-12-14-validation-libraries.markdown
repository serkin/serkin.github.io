---
layout: post
title:  "5 libraries for validating your MongoDB data in PHP"
date:   2015-12-14
---

NoSQL solutions like MongoDB becomes popular and popular day by day and there are many reasons why this growth will continue. I started to use `MongoDB` about 3 years ago and I never sorry about my decision. Almost a half of my last projects needed non-relational table structures. And without waiting long time I fired up my first instance of `MongoDB` bind `php-mongo` extension and started to enjoy how fast easy and efficient it was.

But after 3 months I found unusual data in my database. Where I expected to see `string` I could find empty `array` or even associated array. It was that time when I started to search validation libraries and today I want to share my experience with you and maybe you’ll find some of these libraries useful for your own project. All packages below you can install via `composer`

<h3><a href="https://github.com/romaricdrigon/MetaYaml">MetaYaml</a></h3>
A schema validator library written by <a href="https://github.com/romaricdrigon">romaricdrigon</a> using PHP arrays, Yaml, Json or XML as schema definition. I like this library because it is very strait-forward and easy to get started. There are numerous ways to define schema, you can choose the most appropriate way that fits your project. As for me I prefer `YAML`. Lets create file `schema.yml`
root: # root is always required (note no prefix here)

{% highlight yml %}
    _type: array # each element must always have a '_type'
    _children: #  defining their children
        flowers:
            _type: array
            _required: true # optional, default false
            _children:
                rose:
                    _required: true
                    _type: text
{% endhighlight %}

Now lets load schema and convert it to array. Assuming that data for validation comes in $_POST['form']

{% highlight php %}

use RomaricDrigon\MetaYaml\Loader\YamlLoader;

$yaml_loader = new YamlLoader()
$schema = $yaml_loader->loadFromFile('path/to/schema.yml'))
$data = $_POST['form'];
{% endhighlight %}
And now we can run our validation process
{% highlight php %}
use RomaricDrigon\MetaYaml\MetaYaml;

$schema = new MetaYaml($schema);
if($schema->validateSchema()) {
    $schema->validate($data); // return true or throw an exception
} else {
 echo 'Schema is not valid';
}
{% endhighlight %}

<h3><a href="https://github.com/gwkunze/Structr">Structr</a></h3>
Currently unmaintained library written by <a href="https://github.com/gwkunze">Gijs Kunze</a>. I decided to include it to this list because it was the first validation library I was working with. The schema definition resembles <a href="http://symfony.com/doc/current/components/config/definition.html">TreeBulder</a> in Symfony Config component. Library is useful when you have single-nested array but if your data has lots of nested arrays the schema can become a little fuzzy

{% highlight php %}

use Structr\Structr;

$value = array(
    "id" => 23,
    "author" => "John",
    "title" => "Foo",
    "text" => "The quick brown fox jumps over the lazy dog's back",
    "tags" => array("foo", "bar", "baz"),
    "comments" => array(
        array(
            "author" => "Mike",
            "text" => "Lorem ipsum dolor..."
        )
    )
);

Structr::define("comment")  // Define the 'comment' sub-document
    ->isMap() // A map is an associative array
        ->strict()
        ->key("author")
            ->defaultValue("Anonymous")
            ->isString()->end()
        ->endKey()
        ->key("text")
            ->isString()->end()
        ->endKey()
    ->end()
    ;

$document = Structr::ize($value)
    ->isMap()
        ->key("id")
            ->isInteger()->coerce()->end()
        ->endKey()
        ->key("author")
            ->defaultValue("No Author")
            ->isString()->end()
        ->endKey()
        ->key("title")
            ->isString()->end()
        ->endKey()
        ->key("text")
            ->isString()->end()
        ->endKey()
        ->key("tags")
            ->isList()
                ->item()
                    ->isString()->end()
                ->endItem()
            ->end()
            ->post(function($v) { sort($v); return $v; })
        ->endKey()
        ->key("comments")
            ->isList()
                ->item()
                    ->is("comment")->end()
                ->endItem()
            ->end()
        ->endKey()
    ->end()
    ->run();
{% endhighlight %}

<h3><a href="https://github.com/Respect/Validation">Respect/Validation</a></h3>
Community driven library has lots of stars on github and according to `versioneye` has <a href="https://www.versioneye.com/php/respect:validation/references">140</a> other packages built on top of it. Features: Lots of pre-built validators for almost any case, many features. If you want `pipe` like schema definition this library is what you need if no take a look at `Volan` library below. Let’s see how validating works in this library.

{% highlight php %}

use Respect\Validation\Validator as v;
user = new stdClass;
$user->name = 'Alexandre';
$user->birthdate = '1987-07-01';
Is possible to validate its attributes in a single chain:

$userValidator = v::attribute('name', v::string()->length(1,32))
                  ->attribute('birthdate', v::date()->age(18));

$userValidator->validate($user); //true
{% endhighlight %}

<h3><a href="https://github.com/ircmaxell/filterus">Filterus</a></h3>
Written by <a href="https://github.com/ircmaxell">Anthony Ferrara</a> very handy library for validating single-nested arrays.
{% highlight php %}
$array = array(
    'foo' => 2,
    'bar' => 'test',
);

$filter = Filter::map(array(
    'foo' => 'int',
    'bar' => 'string,min:4',
));

var_dump($filter->validate($array)); // true

{% endhighlight %}

<h3><a href="https://github.com/serkin/volan">Volan</a></h3>
Lightweight and simple library written by <a href="https://github.com/serkin">serkin</a> build especially for NoSQL like MongoDB and has some similarities with `MetaYaml` but much simpler. The library does only two things: It validates given array and gives you extended description about occurred validation errors. If you want to load schema in `yaml` format you can implement your own loader with <a href="https://github.com/symfony/Yaml">symfony/yaml</a> component. Features: Numerous pre-built validators, logging, easy extendable validators, customizable validation process. What I like about this library is that schema definition couldn’t be easier, let’s look at the example:

{% highlight php %}
$schema = [
    'root' => [ // Schema must begins with 'root'
        'title' => [
            '_type' => 'required_string'
        ],
        'price' => [
            '_type' => 'number'
        ],
        'author' => [
            '_type' => 'string'
        ],
        'instock' => [
            '_type' => 'required_boolean'
        ],
        'comments' => [
            '_type' => 'nested_array',
            'user_id' => [
                '_type' => 'required_number'
            ],
            'comment' => [
                '_type' => 'required_string'
            ]
        ]
    ]
];

{% endhighlight %}

Each element has to have `_type` element pointing to validator class. In case of `'price' => ['_type' => 'number']` validator will be `NumberValidator` class. You can created your own or extend any pre-built validator. You can end up with validator name like `NumberBetween10And20OrMongoidValidator` which at the beginning seems difficult to grasp but in mean time you’ll see how handy it is to define your validator in one single string. Read more in <a href="https://github.com/serkin/volan#custom-validators">Volan custom validators</a> Validation process

{% highlight php %}
$validator = new \Volan\Volan($schema);
$result = $validator->validate($book);

// if $result === false you can get full information
// about invalid node
var_dump($validator->getErrorInfo());

{% endhighlight %}

<h2>Bonus</h2>
<h3><a href="">Valitron</a></h3>
Documentation tells that library focus on readable and concise syntax. I didn’t use this library for validation but maybe you will find it interesting. This library is very useful if you dynamically create rules for your schema. This is an example taken for official documentation.

{% highlight php %}

$v = new Valitron\Validator($_POST);
$v->rule('required', ['name', 'email']);
$v->rule('email', 'email');
if($v->validate()) {
    echo "Yay! We're all good!";
} else {
    // Errors
    print_r($v->errors());
}
{% endhighlight %}
Thank you for reading my first article. I hope you find it useful. If you have any other validation libraries please add them in comments.
