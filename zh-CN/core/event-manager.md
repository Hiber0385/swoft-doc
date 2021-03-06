# 事件管理

## 自定义事件

基本的事件注册与触发管理

- implement the [Psr 14](https://github.com/php-fig/fig-standards/blob/master/proposed/event-dispatcher.md) - Event dispatcher
- 支持设置事件优先级
- 支持快速的事件组注册
- 支持通配符事件的监听

### 事件分组

除了一些特殊的事件外，在一个应用中，大多数事件是有关联的，此时我们就可以对事件进行分组，方便识别和管理使用。

- **事件分组**  推荐将相关的事件，在名称设计上进行分组

例如：

```text
// 模型相关：
model.insert
model.update
model.delete

// DB相关：
db.connect
db.disconnect
db.query

// 应用相关：
app.start
app.run
app.stop
```

### 事件通配符 `*`

支持使用事件通配符 `*` 对一组相关的事件进行监听, 分两种。

1. `*` 全局的事件通配符。直接对 `*` 添加监听器(`$em->attach('*', 'global_listener')`), 此时所有触发的事件都会被此监听器接收到。
2. `{prefix}.*` 指定分组事件的监听。eg `$em->attach('db.*', 'db_listener')`, 此时所有触发的以 `db.` 为前缀的事件(eg `db.query` `db.connect`)都会被此监听器接收到。

> 当然，你在事件到达监听器前停止了本次事件的传播`$event->stopPropagation(true);`，就不会被后面的监听器接收到了。

## 使用自定义事件

> 作为核心服务组件，事件管理会自动启用

```php
'eventManager'    => [
    'class'     => \Swoft\Event\EventManager::class,
],		     
```

### 注册事件监听

- 用注解tag `@Listener("event name")` 来注册用户自定义的事件监听

```php
/**
 * 应用加载事件
 *
 * @Listener(AppEvent::APPLICATION_LOADER)
 */
class ApplicationLoaderListener implements EventHandlerInterface
{
    /**
     * @param EventInterface $event      事件对象
     */
    public function handle(EventInterface $event)
    {
        // do something ....
    }
}
```

> 事件名称管理推荐放置在一个单独类的常量里面，方便管理和维护

- 触发事件

```php
\Swoft::trigger('event name', null, $arg0, $arg1);
// OR use \Swoft\App::trigger();
```

## Swoole 事件

用注解tag `@SwooleListener("event name")` 来注册swoole的回调事件监听, 支持所有swoole官网列出来的事件回调名。 
具体请查看 `SwooleEvent::class`。

## Server 事件

用注解tag `@ServerListener("event name")` 来注册服务器级别的事件监听。
它是对 `@SwooleListener` 的补充扩展，除了支持swoole的事件以外，还增加了一些额外的可用事件监听。

区别是：

 - SwooleListener 中一个事件的监听器只允许一个，并且是直接注册到 swoole server上的(**监听相同事件将会被覆盖**)
 - ServerListener 允许对swoole事件添加多个监听器，会逐个通知
 - ServerListener 不影响基础swoole事件的监听

> swoole和server级别的事件监听器，应当放置在boot阶段。(即通常应放置于 `App\Boot` 空间下)
