# Livia [![Build Status](https://scrutinizer-ci.com/g/CharlotteDunois/Livia/badges/build.png?b=master)](https://scrutinizer-ci.com/g/CharlotteDunois/Livia/build-status/master)
Livia is a Discord Bot framework, which utilizes [Yasmin](https://github.com/CharlotteDunois/Yasmin), the Discord API library.

Commando was used as template for the design of Livia.

# Built-in Argument Types
Livia ships with a few argument types you can use for your command arguments. Here's a list:

* boolean
* channel
* command
* command-or-group
* float
* group
* integer
* member
* role
* string
* user

# Built-in Commands
The following commands are coming with Livia:

* disable (disable commands/groups)
* enable (enable commands/groups)
* load (load new commands)
* reload (reload command(s))
* unload (unload a command)
* eval (evaluates PHP code)
  * All PHP errors get thrown.
  * Defines randomized eval namespace for eval execution.
  * It has an anonymous function called `$doCallback`, which takes whatever value you give, inspects it and sends it as reply to Discord.
  * The client is exposed through `$this->client` and `$client`.
  * You have access to the specific `CommandMessage` instance through `$message`.
  * The last eval result is accessible through `$this->lastResult`.
* help (displays a help message in DM)
* ping (calculates the bot's latency)
* prefix (gets or sets the bot's prefix in the guild/globally)

# Setting Provider
A Setting Provider sits as middle-man between Livia and the DBMS. The job of the Setting Provider is to permanently store the settings, such as guild prefixes.<br>
Included in Livia is a provider for MySQL/MariaDB, which utilizes [`react/mysql`](https://github.com/friends-of-reactphp/mysql).

Usage:
```php
$factory = new \React\MySQL\Factory($your_livia_client->getLoop());
$factory->createConnection('user:password@localhost/database')->done(function (\React\MySQL\ConnectionInterface $db) use ($your_livia_client) {
    $provider = new \CharlotteDunois\Livia\Providers\MySQLProvider($db);
    $your_livia_client->setProvider($provider);
});
```

# Making Commands
Livia features Commands Reloading, which requires you to return an anonymous function in your command file, which returns a new anonymous class.

Do not declare functions outside of the class. This will turn into a **Fatal Error** as the function can not be re-declared.<br>
Other restrictions apply as well. If you need any (helper) functions or constants, declare them inside the class as method or class constant.

Example:
```php
// /rootBot/commands/moderation/ban.php

// Livia forces you to use lowercase command name and group ID.
// (moderation = group ID, ban = command name)

// Livia will automatically call the anonymous function and pass the LiviaClient instance.
return function ($client) {
    // Extending is required
    return (new class($client) extends \CharlotteDunois\Livia\Commands\Command {
        function __construct(\CharlotteDunois\Livia\LiviaClient $client) {
            parent::__construct($client, array(
                'name' => 'ban',
                'aliases' => array(),
                'group' => 'moderation',
                'description' => 'Bans an user.',
                'guildOnly' => true,
                'throttling' => array( // Throttling is per-user
                    'usages' => 2,
                    'duration' => 3
                ),
                'args' => array(
                    array(
                        'key' => 'user',
                        'prompt' => 'Which user do you wanna ban?',
                        'type' => 'member'
                    )
                )
            ));
        }
        
        // Checks if the command is allowed to run - the default method from Command class also checks userPermissions.
        // Even if you don't use all arguments, you are forced to match that method signature.
        function hasPermission($message, bool $ownerOverride = true) {
            return $message->member->roles->has('SERVER_STAFF_ROLE_ID');
        }
        
        // Even if you don't use all arguments, you are forced to match that method signature.
        function run(\CharlotteDunois\Livia\CommandMessage $message, \ArrayObject $args,
                      bool $fromPattern) {
            // Do what the command has to do.
            // You are free to return a Promise, or do all-synchronous tasks synchronously.
            
            // If you send any messages (doesn't matter how many),
            // return (resolve) the Message instance, or an array of Message instances.
            // Promises are getting automatically resolved.
            
            return $args->user->ban()->then(function () use ($message) {
                return $message->reply('The user got banned!');
            });
        }
    });
};
```

# Example

```php
require_once(__DIR__.'/vendor/autoload.php');

$loop = \React\EventLoop\Factory::create();
$client = new \CharlotteDunois\Livia\LiviaClient(array(
    'owners' => array('YOUR_USER_ID'),
    'unknownCommandResponse' => false
), $loop);

// Registers default commands, command groups and argument types
$client->registry->registerDefaults();

// Register the command group for our example command
$client->registry->registerGroup(array('id' => 'moderation', 'name' => 'Moderation'));

// Register our commands (this is an example path)
$client->registry->registerCommandsIn(__DIR__.'/commands/');

// If you have created a command, like the example above, you now have registered the command.

$client->on('ready', function () use ($client) {
    echo 'Logged in as '.$client->user->tag.' created on '.
           $client->user->createdAt->format('d.m.Y H:i:s').PHP_EOL;
});

$client->login('YOUR_TOKEN')->done();
$loop->run();
```

# Documentation
https://livia.neko.run

# Issues
If you think something is wrong, or not working as expected, then try to listen on the `error` event. This event gets emitted when an error inside the library (or event listener) gets caught. Make sure you also have a rejection handler for all promises, as unhandled promise rejections get swallowed. Feel free to open an issue with as much information as you can get.

# Need help?
[![](https://discordapp.com/api/guilds/389502182065700876/embed.png?style=banner1&v=1)](https://discord.gg/hUpnqam)
