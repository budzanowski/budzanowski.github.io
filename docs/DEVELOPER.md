# WooRelease development documentation.

All things related to modification or enhancement of the WooRelease script.

## Menu:

* [Logging](#logging)
* [Structure](#structure)
* [Standards](#standards)

## Logging

WooRelease logs all relevant information using [Monolog]( https://github.com/Seldaek/monolog ) php library. Logs are displayed in terminal during execution of the script and are also logged to the logs files inside the `logs` folder located in WooRelease main folder. The `Monolog` library implements a lot of different features that can easily enhance logging capabilities of the application.

### Logging subsystem

Logging subsystem consists of a single class named `WR_Logger`. This class is responsible for the creation and configuration of the logger. Using `Monolog\Registry` mechanics, it exposes the logger globally - this means that the logger is available everywhere directly. A static interface was added to allow working with multiple loggers. This may be useful, for example, for implementing a dedicated DB logger that should not write to the output stream simultaneously.

The main logger is called `default`. Accessing it using the static interface looks as follows:

```php
$logger = WR_Logger::default();
$logger->info( "This message is logged." );
```

### Logging levels

`Monolog` library supports PSR-3 style log levels. The levels in order of criticality:
 - DEBUG
 - INFO
 - NOTICE
 - WARNING
 - ERROR
 - CRITICAL
 - ALERT
 - EMERGENCY

Accessing different log levels requires calling log level functions from the logger object:
```php
$logger->info(...)    // INFO
$logger->notice(...)  // NOTICE
$logger->warning(...) // WARNING
...
```

The `default` logger will create logs with the INFO log level. This means that DEBUG type information is not displayed.

To make the presentation more readable WooRelease uses `bramus/monolog-colored-line-formatter` that enhances the terminal output with different colors for different log levels. For more information please take a look at: [Default Color Scheme](https://github.com/bramus/monolog-colored-line-formatter/blob/master/readme.md#color-scheme-defaultscheme).


### Logging message formatting

WR_Logger class uses `PsrLogMessageProcessor` message processor. This enables PSR-3 style text placeholders that can be used in logger messages. For example:

```php
$logger->info(
    "Processing {extension} from {repository}.",
    array(
        'extension'  => $extension_name,
        'repository' => $repository_url
    )
);
```
In this example `{extension}` will be substituted with the content of `$extension_name`.

### Log files.

Log files are created using `RotatingFileHandler`. The main feature of `RotatingFileHandler` is that it automatically creates a log file for each day and is able to clean up old log files. Default setup stores logs insides the the `logs` folder in the WooRelease main folder. Location can be controlled using the `WOORELEASE_LOGS_FOLDER` env variable.

Log files format is `woorelease-{date}.log` where `{date}` is the date when the script was executed. For example: `woorelease-2020-04-03.log`.
Up to 10 log files are kept. Older ones are deleted.

## Structure

You will find all of the WooRelease files in the /includes/ directory. Each component is located in a single file.

Global options for the tool can be retrieved using the `get_options()` function, like so:

`$opts = get_options();`

Options retrieved in this way should be treated as read-only and should not be mutated. If a change must be made to an option, the standard practice is to pass the option through to your functions as arguments.

## Standards

- Code should follow the [Extendables](https://github.com/woocommerce/woocommerce-extension-library/tree/add-phpcs-rule) PHP code standards.
- Unhandled errors should not return error values; throw an **Exception** instead.
- Function names should be prefixed based on the filename. Example: `ext_process_woorelease()` in extension.php.