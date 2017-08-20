---
title: laravel 知识点总结
date: 2017-08-09 21:00:50
tags:
- php
- laravel
category:
- php
---


# 关键概念
## Ioc容器

官方解释

>Laravel 服务容器是一个用于管理类依赖和执行依赖注入的强大工具

个人理解，容器就是一个大箱子，可以往里注册一个类（当然不限于类，也可以是php中任何的变量，后边统称为服务），也可以从里往外拿已经注册过的类，他提供了一个上下文的环境，我们运行的所有类都是来自于这个大箱子。
其最主要的方法是bind和make，bind是往箱子里塞东西即所谓的服务的绑定，make是从箱子里拿东西也就是所谓的服务的解析。

在代码中获取容器可以通过 帮助函数`app()` 也可以通过门面类`App`，但一般不用主动获取，我们需要的服务完全可以通过容器自动注入，例如在构造函数中进行注入。

## Facade模式
门面就是用来简化调用的，省去了我们从容器中获取服务的过程，门面就是一个静态代理，它通过`__callStatic()`方法代理容器中真正的服务。


## 服务提供者
`服务提供者`可以理解为不同类型服务的配置文件，也可以理解为载入服务的工具类，通过这个配置文件可以把服务注册到容器中（通过register方法，而register方法的主要作用就是调用容器的bind进行服务绑定），也可以对服务进行初始化（通过boot方法），容器会维护一个映射关系保存一个服务是由哪个`服务提供者`来进行注册的，服务提供者可以延迟载入容器（注意是服务提供者不是服务，提供者被延迟了它提供的服务当然也会被延迟绑定），是通过设置`$defer = true`并提供一个`provides`方法，这个方法会返回这个`服务提供者`所能提供的服务，方便容器去维护那个映射关系，当我们真正需要去使用一个延迟的服务的时候，容器就会根据映射关系先找到服务提供者，然后通过服务提供者把这个服务注册到容器。

## 中间件
中间件就是处理流程中定义的一个个可插拔的处理环节，是管道模式的一种实现，中间件在`app/Http/Kernel`中配置，其中包括两类中间件 一类是全局中间件，即每个请求都会被应用这套处理，一类是路由中间件，只有匹配特定的路由的时候才会应用这些处理。关于管道模式参见 [这里](http://laravelacademy.org/post/3088.html)

# 请求的生命周期

## 启动准备阶段

```
<?php
require __DIR__.'/../bootstrap/autoload.php';
$app = require_once __DIR__.'/../bootstrap/app.php';
$kernel = $app->make(Illuminate\Contracts\Http\Kernel::class);
$response = $kernel->handle($request = Illuminate\Http\Request::capture());
$response->send();
$kernel->terminate($request, $response);
```

请求到达index.php后首先载入自动加载文件autoload.php实现类的自动加载。
然后通过`bootstrap/app.php` 实例化容器`Application`，得到容器的实例`$app`，容器在初始化的时候会做几件事，会将自身的引用也注册入容器，方便后续程序调取，会注册两个基础服务提供者`EventServiceProvider` 和`RoutingServiceProvider`

```
<?php
$app = new Illuminate\Foundation\Application(
    realpath(__DIR__.'/../')
);
$app->singleton(
    Illuminate\Contracts\Http\Kernel::class,
    App\Http\Kernel::class
);
$app->singleton(
    Illuminate\Contracts\Console\Kernel::class,
    App\Console\Kernel::class
);
$app->singleton(
    Illuminate\Contracts\Debug\ExceptionHandler::class,
    App\Exceptions\Handler::class
);
return $app;
```

接着从容器中取出Http\Kernel类 ，此类主要负责初始化环境以及调度各种服务完成一个http的处理流程 主要括中间件处理-> 路由处理->控制器执行一系列的流程，是所有http处理的核心，类似linux的内核功能。

## 请求实例化

`$request = Illuminate\Http\Request::capture()`，将请求相关信息封装到`Request`类中

## 请求处理

进入到kernel的handle方法

```
protected $bootstrappers = [
        \Illuminate\Foundation\Bootstrap\LoadEnvironmentVariables::class,
        \Illuminate\Foundation\Bootstrap\LoadConfiguration::class,
        \Illuminate\Foundation\Bootstrap\HandleExceptions::class,
        \Illuminate\Foundation\Bootstrap\RegisterFacades::class,
        \Illuminate\Foundation\Bootstrap\RegisterProviders::class,
        \Illuminate\Foundation\Bootstrap\BootProviders::class,
    ];

public function handle($request)
    {
        try {
            $request->enableHttpMethodParameterOverride();

            $response = $this->sendRequestThroughRouter($request);
        } catch (Exception $e) {
            $this->reportException($e);

            $response = $this->renderException($request, $e);
        } catch (Throwable $e) {
            $this->reportException($e = new FatalThrowableError($e));

            $response = $this->renderException($request, $e);
        }

        $this->app['events']->dispatch(
            new Events\RequestHandled($request, $response)
        );

        return $response;
    }

```
handle 中会首先完成环境的检测、配置文件的加载、门面类的注册、服务提供者的注册、服务初始化等一系列初始化工作。其中最重要的是载入服务提供者，即配置在`config/app.php`中的provider。

初始化完成后，通过`sendRequestThroughRouter`调用开始管道中的逐个中间件处理。

```
new Pipeline($this->app))
                    ->send($request)
                    ->through($this->app->shouldSkipMiddleware() ? [] : $this->middleware)
                    ->then($this->dispatchToRouter()
```

send 是发送到管道进行传递的对象，through是传递的过程，then是传递完成后的处理,注意在这里调用的是全局中间件。

## 路由处理

全局中间件处理完毕后就进入到路由处理，路由这里又执行了一个管道处理，就是下面的这段代码，这里执行的中间件就是匹配这个路由的`路由中间件`。

```
protected function runRouteWithinStack(Route $route, Request $request)
    {
        $shouldSkipMiddleware = $this->container->bound('middleware.disable') &&
                                $this->container->make('middleware.disable') === true;

        $middleware = $shouldSkipMiddleware ? [] : $this->gatherRouteMiddleware($route);

        return (new Pipeline($this->container))
                        ->send($request)
                        ->through($middleware)
                        ->then(function ($request) use ($route) {
                            return $this->prepareResponse(
                                $request, $route->run()
                            );
                        });
    }
```

## 执行控制器

管道处理的最后会交给`$route->run()`方法，开始调用我们写的控制器。

```
public function run()
    {
        $this->container = $this->container ?: new Container;

        try {
            if ($this->isControllerAction()) {
                return $this->runController();
            }

            return $this->runCallable();
        } catch (HttpResponseException $e) {
            return $e->getResponse();
        }
    }
```

控制器执行完成后返回`Response`对象，控制权转回给Kernel开始对这个Response执行发送处理，最后执行终止处理调用中间件的`terminate`方法 ，完成整个生命周期。







