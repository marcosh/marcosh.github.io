# PHP ❤️ types

---

## 👋, I'm Marco

---

## PHP 💔 types

---

```php
in_array('7.10', ['7.1']); // true
```

---

```php
$z = 'z';
++$z; // 'aa'
```

---

```php
'066' < '07x' // true
'07x' < '28'  // true
'28'  < '066' // true
```

---

## PHP <span style="font-family:'noto color emoji'">❤️‍🩹</span> types

---

### PHP 7.0

[scalar type declarations](https://www.php.net/manual/en/language.types.declarations.php) <!-- .element: class="fragment" -->

[return type declarations](https://wiki.php.net/rfc/return_types) <!-- .element: class="fragment" -->

---

### PHP 7.1

[nullable types](https://www.php.net/manual/en/migration71.new-features.php#migration71.new-features.nullable-types) <!-- .element: class="fragment" -->

[void functions](https://www.php.net/manual/en/migration71.new-features.php#migration71.new-features.void-functions) <!-- .element: class="fragment" -->

[iterable pseudo-type](https://www.php.net/manual/en/language.types.iterable.php) <!-- .element: class="fragment" -->

---

### PHP 7.2

[object type](https://www.php.net/manual/en/migration72.new-features.php#migration72.new-features.object-type) <!-- .element: class="fragment" -->

---

### PHP 7.4

[typed properties](https://www.php.net/manual/en/migration74.new-features.php#migration74.new-features.core.typed-properties) <!-- .element: class="fragment" -->

[return type covariance and argument type contravariance](https://www.php.net/manual/en/migration74.new-features.php#migration74.new-features.core.type-variance) <!-- .element: class="fragment" -->

---

### PHP 8.0

[union types](https://www.php.net/manual/en/language.types.declarations.php#language.types.declarations.union) <!-- .element: class="fragment" -->

[`static` as return type](https://www.php.net/manual/en/migration80.new-features.php#migration80.new-features.core.others) <!-- .element: class="fragment" -->

[`Stringable` interface](https://www.php.net/manual/en/class.stringable.php) <!-- .element: class="fragment" -->

[support for `mixed` type](https://www.php.net/manual/en/language.types.declarations.php#language.types.declarations.mixed) <!-- .element: class="fragment" -->

---

### PHP 8.1

[enums](https://php.watch/versions/8.1/enums) <!-- .element: class="fragment" -->

[`never` return type](https://php.watch/versions/8.1/never-return-type) <!-- .element: class="fragment" -->

---

## 🤔 WHAT ARE TYPES?

---

Types are a way to specify the **meaning** of values

---

### Meaning as operations

A type describes which **operations** I can do on a value

---

> I can add two integers to get their sum

<div class="fragment">

> I can get the first name of a customer

</div>

---

### Meaning as properties

Types help us encode and enforce **invariants**

---

> An item must have a positive price

<div class="fragment">

> The password should be at least 8 characters long

</div>

Note: Depending on the type system we are working with, there are more or less properties we can encode in the type system.

---

Force properties **by construction**, not by runtime validation

[Make illegal states unrepresentable](https://blog.janestreet.com/effective-ml-revisited/) <!-- .element: class="fragment" -->

---

## 🧐 WHY TYPES?

---

### Catching **bugs**

Just a side effect of good domain modelling

---

### Fast **feedback**

On whether the invariants are respected

---

### Safe **Refactoring**

Types guide you towards the next step

---

### Safer than tests

As a **proof** is more convincing than an example

[Curry-Howard correspondence](https://en.wikipedia.org/wiki/Curry%E2%80%93Howard_correspondence) <!-- .element: class="fragment" -->

<!--
### Types are your friends

- you can ask them things about your program
- they know what you are doing
- they help you when you're doing complicated things
-->

---

## 🐘 TYPES IN PHP

---

### [Native types](https://www.php.net/manual/en/language.types.intro.php)

---

#### Scalar types

- `bool`
- `int`
- `float`
- `string`

---

#### Compound types

- `array`
- `object`
- `callable`
- `iterable`

---

#### Special types

- `resource`
- `NULL`

---

### User defined types

**Classes** and **interfaces**

---

#### **Domain driven** types

Convey more meaning using types

<div class="fragment">

```php
final class Age
{
  private int $age;
}
```

</div><div class="fragment">

```php
final class JobDescription
{
  private string $jobDescription;
}
```

</div>

Note: defining new types this way is quite verbose, creates friction. But it's an investment for the future

---

## 📜 [PSALM](https://psalm.dev/)

Note: there are other tools which do similar things. I'm talking about Psalm because that's what I use and know

---

## TYPE CHECKER

```php [1-2|4-8|10-14]
/** @psalm-immutable */
final class Foo
{
  /** @var positive-int */
  public int $bar;

  /** @var callable(int): string */
  public callable $baz;

  /**
   * @param class-string $s
   * @return array{a: int, b: string}
   */
  public function caf(string $s): array {}
}
```

---

### Static analysis tool

```bash
./vendor/bin/psalm
```

<div class="fragment">

<pre class="code-wrapper"><code class="hljs" style="white-space:pre-line">ERROR: InvalidReturnType - Not all code paths of Foo::caf end in a return statement, return type array{a: int, b: string} expected

ERROR: MissingConstructor - Foo has an uninitialized property Foo::$bar, but no constructor

ERROR: MissingConstructor - Foo has an uninitialized property Foo::$baz, but no constructor</code></pre>

</div>

---

### Improves scalar types

- [`positive-int`](https://psalm.dev/docs/annotating_code/type_syntax/scalar_types/#positive-int)
- [`class-string`](https://psalm.dev/docs/annotating_code/type_syntax/scalar_types/#class-string-interface-string)
- [`trait-string`](https://psalm.dev/docs/annotating_code/type_syntax/scalar_types/#trait-string)
- [`callable-string`](https://psalm.dev/docs/annotating_code/type_syntax/scalar_types/#callable-string)
- [`numeric-string`](https://psalm.dev/docs/annotating_code/type_syntax/scalar_types/#numeric-string)
- [`array-key`](https://psalm.dev/docs/annotating_code/type_syntax/scalar_types/#array-key)
- [`numeric`](https://psalm.dev/docs/annotating_code/type_syntax/scalar_types/#numeric)
- [`scalar`](https://psalm.dev/docs/annotating_code/type_syntax/scalar_types/#scalar)

---

### Improves array type

- [`non-empty-array`](https://psalm.dev/docs/annotating_code/type_syntax/array_types/)
- [`string[]`](https://psalm.dev/docs/annotating_code/type_syntax/array_types/#phpdoc-syntax)
- [`array<int, string>`](https://psalm.dev/docs/annotating_code/type_syntax/array_types/#generic-arrays)
- [`list`](https://psalm.dev/docs/annotating_code/type_syntax/array_types/#lists)
- [`array{foo: int, bar: string}`](https://psalm.dev/docs/annotating_code/type_syntax/array_types/#object-like-arrays)
- [`callable-array`](https://psalm.dev/docs/annotating_code/type_syntax/array_types/#callable-array)

---

### [Improves callable type](https://psalm.dev/docs/annotating_code/type_syntax/callable_types/)

Allows specification of parameters and return value types

```php
/** @param callable(int): string $bar */
function foo(callable $bar) {...}
```

---

### [Union types](https://psalm.dev/docs/annotating_code/type_syntax/union_types/)

```php
/** @param int|string|null $bar */
function foo($bar) {...}
```

Note: good for typing legacy code, not for domain modelling; leads to sloppy API

---

### [Templates](https://psalm.dev/docs/annotating_code/templated_annotations/)

Adds type variables to the type system

```php [2|1|4-5|7-8|11-12]
/** @template A */
final class Repository
{
  /** @return A|null */
  public function find(Id $id) {...}

  /** @param A $item */
  public function add($item) {...}
}

/** @var Repository<User> */
$userRepository = new Repository();
```

Note: allows to abstract code to more generic and reusable piaces

---

## 🤹 LET'S JUGGLE

---

### ✅ Validation

It's a work for types! <!-- .element: class="fragment" -->

---

#### Let's model validation

<svg id="mermaid-1622217192608" width="100%" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" height="144" style="height: 300px;" viewBox="0 0 291.3166809082031 144">
  <style>#mermaid-1622217192608{font-family:"trebuchet ms",verdana,arial,sans-serif;font-size:16px;fill:#333;}#mermaid-1622217192608 .error-icon{fill:#552222;}#mermaid-1622217192608 .error-text{fill:#552222;stroke:#552222;}#mermaid-1622217192608 .edge-thickness-normal{stroke-width:2px;}#mermaid-1622217192608 .edge-thickness-thick{stroke-width:3.5px;}#mermaid-1622217192608 .edge-pattern-solid{stroke-dasharray:0;}#mermaid-1622217192608 .edge-pattern-dashed{stroke-dasharray:3;}#mermaid-1622217192608 .edge-pattern-dotted{stroke-dasharray:2;}#mermaid-1622217192608 .marker{fill:#333333;stroke:#333333;}#mermaid-1622217192608 .marker.cross{stroke:#333333;}#mermaid-1622217192608 svg{font-family:"trebuchet ms",verdana,arial,sans-serif;font-size:16px;}#mermaid-1622217192608 .label{font-family:"trebuchet ms",verdana,arial,sans-serif;color:#333;}#mermaid-1622217192608 .cluster-label text{fill:#333;}#mermaid-1622217192608 .cluster-label span{color:#333;}#mermaid-1622217192608 .label text,#mermaid-1622217192608 span{fill:#333;color:#333;}#mermaid-1622217192608 .node rect,#mermaid-1622217192608 .node circle,#mermaid-1622217192608 .node ellipse,#mermaid-1622217192608 .node polygon,#mermaid-1622217192608 .node path{fill:#ECECFF;stroke:#9370DB;stroke-width:1px;}#mermaid-1622217192608 .node .label{text-align:center;}#mermaid-1622217192608 .node.clickable{cursor:pointer;}#mermaid-1622217192608 .arrowheadPath{fill:#333333;}#mermaid-1622217192608 .edgePath .path{stroke:#333333;stroke-width:1.5px;}#mermaid-1622217192608 .flowchart-link{stroke:#333333;fill:none;}#mermaid-1622217192608 .edgeLabel{background-color:#e8e8e8;text-align:center;}#mermaid-1622217192608 .edgeLabel rect{opacity:0.5;background-color:#e8e8e8;fill:#e8e8e8;}#mermaid-1622217192608 .cluster rect{fill:#ffffde;stroke:#aaaa33;stroke-width:1px;}#mermaid-1622217192608 .cluster text{fill:#333;}#mermaid-1622217192608 .cluster span{color:#333;}#mermaid-1622217192608 div.mermaidTooltip{position:absolute;text-align:center;max-width:200px;padding:2px;font-family:"trebuchet ms",verdana,arial,sans-serif;font-size:12px;background:hsl(80,100%,96.2745098039%);border:1px solid #aaaa33;border-radius:2px;pointer-events:none;z-index:100;}#mermaid-1622217192608:root{--mermaid-font-family:"trebuchet ms",verdana,arial,sans-serif;}</style>
  <g>
    <g class="output">
      <g class="edgePaths">
        <g class="edgePath LS-A LE-Validation fragment" data-fragment-index="2" style="opacity: 1;" id="L-A-Validation">
          <path class="path" d="M38.66667175292969,72L42.83333841959635,72C47.00000508626302,72,55.33333841959635,72,63.66667175292969,72C72.00000508626302,72,80.33333841959636,72,84.50000508626302,72L88.66667175292969,72" marker-end="url(#arrowhead100)" style="fill:none"></path>
          <defs>
            <marker id="arrowhead100" viewBox="0 0 10 10" refX="9" refY="5" markerUnits="strokeWidth" markerWidth="8" markerHeight="6" orient="auto">
              <path d="M 0 0 L 10 5 L 0 10 z" class="arrowheadPath" style="stroke-width: 1px; stroke-dasharray: 1px, 0px;"></path>
            </marker>
          </defs>
        </g>
        <g class="edgePath LS-Validation LE-B fragment" data-fragment-index="3" style="opacity: 1;" id="L-Validation-B">
          <path class="path" d="M163.68166034141282,52.5L170.1930528276422,48.333333333333336C176.70444531387156,44.166666666666664,189.7272302863303,35.833333333333336,202.48028893060004,31.666666666666668C215.23334757486978,27.5,227.71667989095053,27.5,233.95834604899088,27.5L240.20001220703125,27.5" marker-end="url(#arrowhead101)" style="fill:none"></path>
          <defs>
            <marker id="arrowhead101" viewBox="0 0 10 10" refX="9" refY="5" markerUnits="strokeWidth" markerWidth="8" markerHeight="6" orient="auto">
              <path d="M 0 0 L 10 5 L 0 10 z" class="arrowheadPath" style="stroke-width: 1px; stroke-dasharray: 1px, 0px;"></path>
            </marker>
          </defs>
        </g>
        <g class="edgePath LS-Validation LE-Error fragment" data-fragment-index="4" style="opacity: 1;" id="L-Validation-Error">
          <path class="path" d="M163.68166034141282,91.5L170.1930528276422,95.66666666666667C176.70444531387156,99.83333333333333,189.7272302863303,108.16666666666667,200.40528943922632,112.33333333333333C211.0833485921224,116.5,219.41668192545572,116.5,223.5833485921224,116.5L227.75001525878906,116.5" marker-end="url(#arrowhead102)" style="fill:none"></path>
          <defs>
            <marker id="arrowhead102" viewBox="0 0 10 10" refX="9" refY="5" markerUnits="strokeWidth" markerWidth="8" markerHeight="6" orient="auto">
              <path d="M 0 0 L 10 5 L 0 10 z" class="arrowheadPath" style="stroke-width: 1px; stroke-dasharray: 1px, 0px;"></path>
            </marker>
          </defs>
        </g>
      </g>
      <g class="nodes">
        <g class="node default fragment" data-fragment-index="1" style="opacity: 1;" id="flowchart-A-96" transform="translate(23.333335876464844,72)">
          <rect rx="0" ry="0" x="-15.333335876464844" y="-19.5" width="30.666671752929688" height="39" class="label-container"></rect>
          <g class="label" transform="translate(0,0)">
            <g transform="translate(-5.333335876464844,-9.5)">
              <foreignObject width="10.666671752929688" height="19">
                <div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;">A</div>
              </foreignObject>
            </g>
          </g>
        </g>
        <g class="node default fragment" data-fragment-index="2" style="opacity: 1;" id="flowchart-Validation-97" transform="translate(133.20834350585938,72)">
          <rect rx="0" ry="0" x="-44.54167175292969" y="-19.5" width="89.08334350585938" height="39" class="label-container" style="fill: aquamarine"></rect>
          <g class="label" transform="translate(0,0)">
            <g transform="translate(-34.54167175292969,-9.5)">
              <foreignObject width="69.08334350585938" height="19">
                <div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;">Validation</div>
              </foreignObject>
            </g>
          </g>
        </g>
        <g class="node default fragment" data-fragment-index="3" style="opacity: 1;" id="flowchart-B-99" transform="translate(255.5333480834961,27.5)">
          <rect rx="0" ry="0" x="-15.333335876464844" y="-19.5" width="30.666671752929688" height="39" class="label-container"></rect>
          <g class="label" transform="translate(0,0)">
            <g transform="translate(-5.333335876464844,-9.5)">
              <foreignObject width="10.666671752929688" height="19">
                <div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;">A</div>
              </foreignObject>
            </g>
          </g>
        </g>
        <g class="node default fragment" data-fragment-index="4" style="opacity: 1;" id="flowchart-Error-101" transform="translate(255.5333480834961,116.5)">
          <rect rx="0" ry="0" x="-27.78333282470703" y="-19.5" width="55.56666564941406" height="39" class="label-container"></rect>
          <g class="label" transform="translate(0,0)">
            <g transform="translate(-17.78333282470703,-9.5)">
              <foreignObject width="35.56666564941406" height="19">
                <div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;">Error</div>
              </foreignObject>
            </g>
          </g>
        </g>
      </g>
    </g>
  </g>
</svg>

---

#### Actually

<svg id="mermaid-1622217192608" width="100%" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" height="144" style="height: 300px;" viewBox="0 0 291.3166809082031 144">
  <style>#mermaid-1622217192608{font-family:"trebuchet ms",verdana,arial,sans-serif;font-size:16px;fill:#333;}#mermaid-1622217192608 .error-icon{fill:#552222;}#mermaid-1622217192608 .error-text{fill:#552222;stroke:#552222;}#mermaid-1622217192608 .edge-thickness-normal{stroke-width:2px;}#mermaid-1622217192608 .edge-thickness-thick{stroke-width:3.5px;}#mermaid-1622217192608 .edge-pattern-solid{stroke-dasharray:0;}#mermaid-1622217192608 .edge-pattern-dashed{stroke-dasharray:3;}#mermaid-1622217192608 .edge-pattern-dotted{stroke-dasharray:2;}#mermaid-1622217192608 .marker{fill:#333333;stroke:#333333;}#mermaid-1622217192608 .marker.cross{stroke:#333333;}#mermaid-1622217192608 svg{font-family:"trebuchet ms",verdana,arial,sans-serif;font-size:16px;}#mermaid-1622217192608 .label{font-family:"trebuchet ms",verdana,arial,sans-serif;color:#333;}#mermaid-1622217192608 .cluster-label text{fill:#333;}#mermaid-1622217192608 .cluster-label span{color:#333;}#mermaid-1622217192608 .label text,#mermaid-1622217192608 span{fill:#333;color:#333;}#mermaid-1622217192608 .node rect,#mermaid-1622217192608 .node circle,#mermaid-1622217192608 .node ellipse,#mermaid-1622217192608 .node polygon,#mermaid-1622217192608 .node path{fill:#ECECFF;stroke:#9370DB;stroke-width:1px;}#mermaid-1622217192608 .node .label{text-align:center;}#mermaid-1622217192608 .node.clickable{cursor:pointer;}#mermaid-1622217192608 .arrowheadPath{fill:#333333;}#mermaid-1622217192608 .edgePath .path{stroke:#333333;stroke-width:1.5px;}#mermaid-1622217192608 .flowchart-link{stroke:#333333;fill:none;}#mermaid-1622217192608 .edgeLabel{background-color:#e8e8e8;text-align:center;}#mermaid-1622217192608 .edgeLabel rect{opacity:0.5;background-color:#e8e8e8;fill:#e8e8e8;}#mermaid-1622217192608 .cluster rect{fill:#ffffde;stroke:#aaaa33;stroke-width:1px;}#mermaid-1622217192608 .cluster text{fill:#333;}#mermaid-1622217192608 .cluster span{color:#333;}#mermaid-1622217192608 div.mermaidTooltip{position:absolute;text-align:center;max-width:200px;padding:2px;font-family:"trebuchet ms",verdana,arial,sans-serif;font-size:12px;background:hsl(80,100%,96.2745098039%);border:1px solid #aaaa33;border-radius:2px;pointer-events:none;z-index:100;}#mermaid-1622217192608:root{--mermaid-font-family:"trebuchet ms",verdana,arial,sans-serif;}</style>
  <g>
    <g class="output">
      <g class="edgePaths">
        <g class="edgePath LS-A LE-Validation" style="opacity: 1;" id="L-A-Validation">
          <path class="path" d="M38.66667175292969,72L42.83333841959635,72C47.00000508626302,72,55.33333841959635,72,63.66667175292969,72C72.00000508626302,72,80.33333841959636,72,84.50000508626302,72L88.66667175292969,72" marker-end="url(#arrowhead100)" style="fill:none"></path>
          <defs>
            <marker id="arrowhead100" viewBox="0 0 10 10" refX="9" refY="5" markerUnits="strokeWidth" markerWidth="8" markerHeight="6" orient="auto">
              <path d="M 0 0 L 10 5 L 0 10 z" class="arrowheadPath" style="stroke-width: 1px; stroke-dasharray: 1px, 0px;"></path>
            </marker>
          </defs>
        </g>
        <g class="edgePath LS-Validation LE-B" style="opacity: 1;" id="L-Validation-B">
          <path class="path" d="M163.68166034141282,52.5L170.1930528276422,48.333333333333336C176.70444531387156,44.166666666666664,189.7272302863303,35.833333333333336,202.48028893060004,31.666666666666668C215.23334757486978,27.5,227.71667989095053,27.5,233.95834604899088,27.5L240.20001220703125,27.5" marker-end="url(#arrowhead101)" style="fill:none"></path>
          <defs>
            <marker id="arrowhead101" viewBox="0 0 10 10" refX="9" refY="5" markerUnits="strokeWidth" markerWidth="8" markerHeight="6" orient="auto">
              <path d="M 0 0 L 10 5 L 0 10 z" class="arrowheadPath" style="stroke-width: 1px; stroke-dasharray: 1px, 0px;"></path>
            </marker>
          </defs>
        </g>
        <g class="edgePath LS-Validation LE-Error" style="opacity: 1;" id="L-Validation-Error">
          <path class="path" d="M163.68166034141282,91.5L170.1930528276422,95.66666666666667C176.70444531387156,99.83333333333333,189.7272302863303,108.16666666666667,200.40528943922632,112.33333333333333C211.0833485921224,116.5,219.41668192545572,116.5,223.5833485921224,116.5L227.75001525878906,116.5" marker-end="url(#arrowhead102)" style="fill:none"></path>
          <defs>
            <marker id="arrowhead102" viewBox="0 0 10 10" refX="9" refY="5" markerUnits="strokeWidth" markerWidth="8" markerHeight="6" orient="auto">
              <path d="M 0 0 L 10 5 L 0 10 z" class="arrowheadPath" style="stroke-width: 1px; stroke-dasharray: 1px, 0px;"></path>
            </marker>
          </defs>
        </g>
      </g>
      <g class="nodes">
        <g class="node default" style="opacity: 1;" id="flowchart-A-96" transform="translate(23.333335876464844,72)">
          <rect rx="0" ry="0" x="-15.333335876464844" y="-19.5" width="30.666671752929688" height="39" class="label-container"></rect>
          <g class="label" transform="translate(0,0)">
            <g transform="translate(-5.333335876464844,-9.5)">
              <foreignObject width="10.666671752929688" height="19">
                <div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;">A</div>
              </foreignObject>
            </g>
          </g>
        </g>
        <g class="node default" style="opacity: 1;" id="flowchart-Validation-97" transform="translate(133.20834350585938,72)">
          <rect rx="0" ry="0" x="-44.54167175292969" y="-19.5" width="89.08334350585938" height="39" class="label-container" style="fill: aquamarine"></rect>
          <g class="label" transform="translate(0,0)">
            <g transform="translate(-34.54167175292969,-9.5)">
              <foreignObject width="69.08334350585938" height="19">
                <div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;">Validation</div>
              </foreignObject>
            </g>
          </g>
        </g>
        <g class="node default" style="opacity: 1;" id="flowchart-B-99" transform="translate(255.5333480834961,27.5)">
          <rect rx="0" ry="0" x="-15.333335876464844" y="-19.5" width="30.666671752929688" height="39" class="label-container" style="fill: orange"></rect>
          <g class="label" transform="translate(0,0)">
            <g transform="translate(-5.333335876464844,-9.5)">
              <foreignObject width="10.666671752929688" height="19">
                <div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;">B</div>
              </foreignObject>
            </g>
          </g>
        </g>
        <g class="node default" style="opacity: 1;" id="flowchart-Error-101" transform="translate(255.5333480834961,116.5)">
          <rect rx="0" ry="0" x="-27.78333282470703" y="-19.5" width="55.56666564941406" height="39" class="label-container"></rect>
          <g class="label" transform="translate(0,0)">
            <g transform="translate(-17.78333282470703,-9.5)">
              <foreignObject width="35.56666564941406" height="19">
                <div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;">Error</div>
              </foreignObject>
            </g>
          </g>
        </g>
      </g>
    </g>
  </g>
</svg>

[Parse, don't validate](https://lexi-lambda.github.io/blog/2019/11/05/parse-don-t-validate) <!-- .element: class="fragment" -->

---

```php
/** @template A */
interface Validator
{
  /** @param A $data */
  public function validate($data);
}
```

what should the return type be? <!-- .element: class="fragment" -->

---

#### The return type

<div class="fragment">

should be **either** `valid` **or** `errors`

</div><div class="fragment">

api **forces** to process **both** `valid` and `errors` cases

</div><div class="fragment">

access information only in the **relevant** case

</div>

---

#### Ideas

<div class="fragment">

Exceptions <span style="visibility:hidden">❌</span>

</div><div style="visibility:hidden"><div>

`isValid` <span>❌</span>

</div><div>

Inheritance <span>❌</span>

</div><div>

Union types <span>❌</span>

</div></div>

---

#### Exceptions

```php [1-5|7-12]
/**
 * @template A
 * @template B
 */
interface Validator
{
  /**
   * @param A $data
   * @return B
   * @throws ValidationFailure
   */
  public function validate($data);
}
```

---

#### Ideas

<div>

Exceptions <span class="fragment">❌</span>

</div><div class="fragment">

`isValid` <span style="visibility:hidden">❌</span>

</div><div style="visibility:hidden"><div>

Inheritance <span>❌</span>

</div><div>

Union types <span>❌</span>

</div></div>

---

#### IsValid

```php [1-4|6-7|9-10]
/** @template A */
final class ValidationResult
{
  public function isValid(): bool;

  /** @return A */
  public function getResult();

  /** @return list<string> */
  public function errorMessages();
}
```

---

#### Ideas

<div>

Exceptions <span>❌</span>

</div><div>

`isValid` <span class="fragment">❌</span>

</div><div class="fragment">

Inheritance <span style="visibility:hidden">❌</span>

</div><div style="visibility:hidden"><div>

Union types <span>❌</span>

</div></div>

---

#### Inheritance

```php [1|4-5,8-9]
interface ValidationResult
{...}

final class ValidationSuccess
  implements ValidationResult
{...}

final class ValidationFailure
  implements ValidationResult
{...}
```

---

#### Ideas

<div>

Exceptions <span>❌</span>

</div><div>

`isValid` <span>❌</span>

</div><div>

Inheritance <span class="fragment">❌</span>

</div><div class="fragment">

Union types <span style="visibility:hidden">❌</span>

</div>

---

#### Union types


```php [1-5|7-12]
/**
 * @template A
 * @template B
 */
interface Validator
{
  /**
   * @param A $data
   * @return ValidationSuccess<B>
   *       | ValidationFailure
   */
  public function validate($data);
}
```

---

#### Ideas

<div>

Exceptions <span>❌</span>

</div><div>

`isValid` <span>❌</span>

</div><div>

Inheritance <span>❌</span>

</div><div>

Union types <span class="fragment">❌</span>

</div>

---

#### [`ValidationResult`](https://psalm.dev/r/c7e2fc1690)

<div class="fragment">

```php [2|1|4-11|13-16|18-27|30-33]
/** @template A */
final class ValidationResult
{
  /**
   * @template B
   * @param B $result
   * @return self<B>
   */
  public static function valid(
    $result
  ): self {...}

  /** @param list<string> $messages */
  public static function errors(
    array $messages
  ): self {...}

  /**
   * @template B
   * @param callable(A): B $onSuccess
   * @param callable(list<string>): B $onFailure
   * @return B
   */
  public function process(
    callable $onSuccess,
    callable $onFailure
  ) {...}
}

$validationResult->process(
  onSuccess: fn ($result) {...},
  onFailure: fn (array $messages) {...}
)
```

</div>

---

### 👻 [Phantom types](https://psalm.dev/r/7663e6d418)

---

#### Back to repositories

```php
/** @template A */
final class Repository
{
  /** @return A|null */
  public function find(Id $id) {...}
}
```

---

#### To be avoided

```php
/** @var Id */
$id = new Id()

$user = $userRepository->find($id);

$order = $orderRepository->find($id);
```

---

#### Could use inheritance

```php
final class UserId extends Id {...}

final class OrderId extends Id {...}
```

Note: we lose the generic repository

---

#### Use type-level information

```php [1-2|4-12]
/** @template A */
final class Id {...}

/** @template A */
final class Repository
{
  /**
   * @param Id<A> $id
   * @return A|null
   */
  public function find(Id $id) {...}
}
```

---

```php [1-2|4-5|7-8]
/** @var Id<User> */
$id = new Id()

// works
$user = $userRepository->find($id);

// Psalm complains
$order = $orderRepository->find($id);
```

---

> One week of debugging saved me from an hour of type modelling

---

## 🙏 THANK YOU! 🙏

---

## 🙋 QUESTIONS?

---

## FURTHER MATERIAL

- [Psalm](https://psalm.dev/docs/) and [PHPStan](https://phpstan.org/)
- [My blog](http://marcosh.github.io/)
  - Maybe in PHP [1](http://marcosh.github.io/post/2017/06/16/maybe-in-php.html) and [2](http://marcosh.github.io/post/2017/10/27/maybe-in-php-2.html)
  - [Functional data validation in PHP](http://marcosh.github.io/post/2018/12/03/php-validation-dsl.html)
  - Higher kinded types in PHP [1](http://marcosh.github.io/post/2020/04/15/higher-kinded-types-php-issue.html) and [2](http://marcosh.github.io/post/2020/04/15/higher-kinded-types-php-solution.html)
  - [Who's scared of phantom types](http://marcosh.github.io/post/2020/05/26/phantom-types-in-php.html)
- Learn [Haskell](https://www.haskell.org/)
