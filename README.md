# ed-when-unless
By [article](https://blog.devgenius.io/php-using-when-and-unless-in-your-own-classes-97035c81a0da)

## About

Sometimes the logic around an object depends on a condition. Let’s say, we need to change the state of something if a variable is true, and do nothing if is otherwise false.

```
$transport->setPackage($package);if ($express) {
    $transport->setExpressDelivery();
}$transport->send();
```

There is a neat way to compress these lines in something less verbose, by using a when() and unless() method helpers on the object instance. The behaviour would look like this:

```
$transport->setPackage($package)
    ->when($express, [$transport, 'setExpressDelivery'])
    ->send();
```

And it should allow more complex logic using a Closure:

```
$transport->setPackage($package)
    ->unless($isSmallPackage, function ($transport) {
        $transport->setNormalDelivery();
        $transport->setBigPackageHandling();
    })
    ->send();
```

For some folks, breaking the flow with an if may be standard and more approachable, but I personally like understandable and expressive fluent methods so another developer can just read the thing and understand it.

## When and Unless

For this to work, we can create a simple trait so we are able to use the helpers on anything we want. These methods will take the condition, and call a callable if it evaluates to true. For unless, we can just call when() but with the condition flipped:

```
<?php

trait Conditionable
{
    public function when(bool $condition, callable $callable): self
    {
        if ($condition) {
            $callable($this, $condition);
        }

        return $this;
    }

    public function unless(bool $condition, callable $callable): self
    {
       return $this->when(!$condition, $callable);
    }
}
```

And walá, we can fully do things like the above. Since it receives a callable, we can even call public static methods from elsewhere just because we can.

```
$transport->when($express, [Courier::class, 'setExpress'])
```
