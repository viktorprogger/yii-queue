<p align="center">
    <a href="https://github.com/yiisoft" target="_blank">
        <img src="https://avatars0.githubusercontent.com/u/993323" height="100px">
    </a>
    <h1 align="center">Yii Queue Extension</h1>
    <br>
</p>

An extension for running tasks asynchronously via queues.

Documentation is at [docs/guide/README.md](docs/guide/README.md).

[![Latest Stable Version](https://poser.pugx.org/yiisoft/yii-queue/v/stable.svg)](https://packagist.org/packages/yiisoft/yii-queue)
[![Total Downloads](https://poser.pugx.org/yiisoft/yii-queue/downloads.svg)](https://packagist.org/packages/yiisoft/yii-queue)
[![Build status](https://github.com/yiisoft/yii-queue/workflows/build/badge.svg)](https://github.com/yiisoft/yii-queue/actions)
[![Scrutinizer Code Quality](https://scrutinizer-ci.com/g/yiisoft/yii-queue/badges/quality-score.png?b=master)](https://scrutinizer-ci.com/g/yiisoft/yii-queue/?branch=master)
[![Code Coverage](https://scrutinizer-ci.com/g/yiisoft/yii-queue/badges/coverage.png?b=master)](https://scrutinizer-ci.com/g/yiisoft/yii-queue/?branch=master)

Installation
------------

The preferred way to install this extension is through [composer](http://getcomposer.org/download/).

Either run

```
php composer.phar require --prefer-dist yiisoft/yii-queue
```

or add

```
"yiisoft/yii-queue": "~3.0"
```

to the `require` section of your `composer.json` file.

## Basic Usage

#### Declaring jobs

Each task which is sent to queue should be defined as a separate class.
For example, if you need to download and save a file the class may look like the following:

```php
class DownloadJob implements \Yiisoft\Yii\Queue\JobInterface
{
    public string $url;
    public string $file;
    
    public function __construct(string $url, string $file)
    {
        $this->url = $url;
        $this->file = $file;
    }
    
    public function execute(): void
    {
        file_put_contents($this->file, file_get_contents($this->url));
    }
}
```

Here's how to send a task into the queue:

```php
$queue->push(
    new DownloadJob('http://example.com/image.jpg', '/tmp/image.jpg')
);
```

This package provides 3 different ways to extend queue job execution:
- `Yiisoft\Yii\Queue\Job\DelayableJobInterface` for jobs to execute after some delay.
- `Yiisoft\Yii\Queue\Job\PrioritisedJobInterface` to set priority to your job.
- `Yiisoft\Yii\Queue\Job\RetryableJobInterface` for jobs which should be retried after fail.
These interfaces can be combined as you wish with no limitations. The exact set of interfaces you can implement in a job is described in a concrete driver you are using.

#### Executing jobs

You can run queued jobs either with console commands, or programmatically. Definitely there are two ways to execute jobs:

- To obtain and execute tasks in a loop until the queue is empty either run in console `vendor/bin/yii queue/run` or execute this code: `$queue->run();`
- To launch a daemon which infinitely queries the queue either run in console `vendor/bin/yii queue/listen` or execute this code: `$queue->listen();`


See the documentation for more details about driver specific console commands and their options.

The component has also the ability to track the status of a job which was pushed into queue.

```php
// Push a job into the queue and get a message ID.
$id = $queue->push(new SomeJob());

// Get current status for the job
$status = $queue->status($id);

// Check whether the job is waiting for execution.
$status->isWaiting();

// Check whether a worker got the job from the queue and executes it.
$status->isReserved($id);

// Check whether a worker has executed the job.
$status->isDone($id);
```

To get the queue working you will also need to install one of the supported driver packages:
- Synchronous driver executes all the jobs in the same php session. It is built in this package and is commonly used in development and test modes.
- [yiisoft/yii-queue-amqp](https://github.com/yiisoft/yii-queue-amqp) is based on [php-amqplib/php-amqplib](https://github.com/php-amqplib/php-amqplib) package.
- You may also write your own driver. It should be a class implementing [`DriverInterface`](src/Driver/DriverInterface.php).

For more details see [the guide](docs/guide/README.md).
