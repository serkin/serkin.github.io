---
layout: post
title:  "5 libraries for validating your MongoDB data in PHP"
description: "We'll go through several libraries for validating data."
date:   2015-12-14
---

NoSQL solutions like MongoDB becomes popular and popular day by day and there are many reasons why this growth will continue. 
I started to use <code class="highlighter-rouge">MongoDB</code> about 3 years ago and I never sorry about my decision. 
Almost a half of my last projects needed non-relational table structures. And without waiting long time I fired up my first 
instance of <code class="highlighter-rouge">MongoDB</code> bind <code class="highlighter-rouge">php-mongo</code> extension 
and started to enjoy how fast easy and efficient it was.

<p>But after 3 months I found unusual data in my database. Where I expected to see 
<code class="highlighter-rouge">string</code> I could find empty <code class="highlighter-rouge">array</code> or even associated array. 
It was that time when I started to search validation libraries and today I want to share my experience with you and maybe you’ll find 
some of these libraries useful for your own project. All packages below you can install via <code class="highlighter-rouge">composer</code></p>

<h3><a href="https://github.com/romaricdrigon/MetaYaml">MetaYaml</a></h3>
<p>A schema validator library written by <a href="https://github.com/romaricdrigon">romaricdrigon</a> using PHP arrays, Yaml, Json or XML as schema definition. 
I like this library because it is very strait-forward and easy to get started. There are numerous ways to define schema, 
you can choose the most appropriate way that fits your project. As for me I prefer <code class="highlighter-rouge">YAML</code>. 
Lets create file <code class="highlighter-rouge">schema.yml</code>
root: # root is always required (note no prefix here)</p>

<figure class="highlight"><pre><code class="language-yml" data-lang="yml">    <span class="s">_type</span><span class="pi">:</span> 
<span class="s">array</span> <span class="c1"># each element must always have a '_type'</span>
    <span class="s">_children</span><span class="pi">:</span> <span class="c1">#  defining their children</span>
        <span class="s">flowers</span><span class="pi">:</span>
            <span class="s">_type</span><span class="pi">:</span> <span class="s">array</span>
            <span class="s">_required</span><span class="pi">:</span> <span class="s">true</span> <span class="c1"># optional, default false</span>
            <span class="s">_children</span><span class="pi">:</span>
                <span class="s">rose</span><span class="pi">:</span>
                    <span class="s">_required</span><span class="pi">:</span> <span class="s">true</span>
                    <span class="s">_type</span><span class="pi">:</span> <span class="s">text</span></code></pre></figure>

<p>Now lets load schema and convert it to array. Assuming that data for validation comes in $_POST[‘form’]</p>

<figure class="highlight"><pre><code class="language-php" data-lang="php">use RomaricDrigon\MetaYaml\Loader\YamlLoader;

$yaml_loader = new YamlLoader()
$schema = $yaml_loader->loadFromFile('path/to/schema.yml'))
$data = $_POST['form'];</code></pre></figure>

<p>And now we can run our validation process</p>

<figure class="highlight"><pre><code class="language-php" data-lang="php">use RomaricDrigon\MetaYaml\MetaYaml;

$schema = new MetaYaml($schema);
if($schema->validateSchema()) {
    $schema->validate($data); // return true or throw an exception
} else {
 echo 'Schema is not valid';
}</code></pre></figure>

<h3><a href="https://github.com/gwkunze/Structr">Structr</a></h3>
<p>Currently unmaintained library written by <a href="https://github.com/gwkunze">Gijs Kunze</a>. I decided to include it to this list because it was
 the first validation library I was working with. The schema definition resembles <a href="http://symfony.com/doc/current/components/config/definition.html">TreeBulder</a>
  in Symfony Config component. Library is useful when you have single-nested array but if your data has lots of nested arrays the schema can become a little fuzzy</p>

<figure class="highlight"><pre><code class="language-php" data-lang="php">use Structr\Structr;

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
    ->run();</code></pre></figure>

<h3><a href="https://github.com/Respect/Validation">Respect/Validation</a></h3>
<p>Community driven library has lots of stars on github and according to <code class="highlighter-rouge">versioneye</code> 
has <a href="https://www.versioneye.com/php/respect:validation/references">140</a> other packages built on top of it. Features:
 Lots of pre-built validators for almost any case, many features. If you want <code class="highlighter-rouge">pipe</code> like schema definition this library
  is what you need if no take a look at <code class="highlighter-rouge">Volan</code> library below. Let’s see how validating works in this library.</p>

<figure class="highlight"><pre><code class="language-php" data-lang="php">use Respect\Validation\Validator as v;
user = new stdClass;
$user->name = 'Alexandre';
$user->birthdate = '1987-07-01';
Is possible to validate its attributes in a single chain:

$userValidator = v::attribute('name', v::string()->length(1,32))
                  ->attribute('birthdate', v::date()->age(18));

$userValidator->validate($user); //true</code></pre></figure>

<h3><a href="https://github.com/ircmaxell/filterus">Filterus</a></h3>
<p>Written by <a href="https://github.com/ircmaxell">Anthony Ferrara</a> very handy library for validating single-nested arrays.</p>

<figure class="highlight"><pre><code class="language-php" data-lang="php">$array = array(
    'foo' => 2,
    'bar' => 'test',
);

$filter = Filter::map(array(
    'foo' => 'int',
    'bar' => 'string,min:4',
));

var_dump($filter->validate($array)); // true</code></pre></figure>

<h3><a href="https://github.com/serkin/volan">Volan</a></h3>
<p>Lightweight and simple library written by <a href="https://github.com/serkin">serkin</a> build especially for NoSQL like MongoDB and has some similarities 
with <code class="highlighter-rouge">MetaYaml</code> but much simpler. The library does only two things: It validates given array and gives you extended description
 about occurred validation errors. If you want to load schema in <code class="highlighter-rouge">yaml</code> format you can implement your own loader 
 with <a href="https://github.com/symfony/Yaml">symfony/yaml</a> component. Features: Numerous pre-built validators, logging, easy extendable validators,
  customizable validation process. What I like about this library is that schema definition couldn’t be easier, let’s look at the example:</p>

<figure class="highlight"><pre><code class="language-php" data-lang="php">$schema = [
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
];</code></pre></figure>

<p>Each element has to have <code class="highlighter-rouge">_type</code> element pointing to validator class. In case of 
<code class="highlighter-rouge">'price' => ['_type' => 'number']</code> validator will be <code class="highlighter-rouge">NumberValidator</code> class.
 You can created your own or extend any pre-built validator. You can end up with validator name like <code class="highlighter-rouge">NumberBetween10And20OrMongoidValidator</code> 
 which at the beginning seems difficult to grasp but in mean time you’ll see how handy it is to define your validator in one single string.
  Read more in <a href="https://github.com/serkin/volan#custom-validators">Volan custom validators</a> Validation process</p>

<figure class="highlight"><pre><code class="language-php" data-lang="php">$validator = new \Volan\Volan($schema);
$result = $validator->validate($book);

// if $result === false you can get full information
// about invalid node
var_dump($validator->getErrorInfo());</code></pre></figure>

<h2>Bonus</h2>
<h3><a href="">Valitron</a></h3>
<p>Documentation tells that library focus on readable and concise syntax. I didn’t use this library for validation but maybe you will find it interesting.
 This library is very useful if you dynamically create rules for your schema. This is an example taken for official documentation.</p>

<figure class="highlight"><pre><code class="language-php" data-lang="php">$v = new Valitron\Validator($_POST);
$v->rule('required', ['name', 'email']);
$v->rule('email', 'email');
if($v->validate()) {
    echo "Yay! We're all good!";
} else {
    // Errors
    print_r($v->errors());
}</code></pre></figure>

<p>Thank you for reading my first article. I hope you find it useful. If you have any other validation libraries please add them in comments.</p>
