# Slack Bot Manager

[![Gem Version](https://badge.fury.io/rb/slack-bot-manager.svg)](http://badge.fury.io/rb/slack-bot-manager)
[![Build Status](https://travis-ci.org/betaworks/slack-bot-manager.svg?branch=master)](https://travis-ci.org/betaworks/slack-bot-manager)
[![Code Climate](https://codeclimate.com/github/betaworks/slack-bot-manager/badges/gpa.svg)](https://codeclimate.com/github/betaworks/slack-bot-manager)

Slack Bot Manager is a Ruby gem that allows for the management of multiple Slack RTM connections based on tokens. With only a few configuration changes, you can run a system for handling hundreds of simulatenous RTM connections for your Slack app. 

__**At this time, slack-bot-manager requires `redis` for tracking the status of tokens.**__

_This is in pre-release and may change before release of version 0.1.0._


__How to tell if you need this:__

* You are making a Slack app requiring Real-time Messaging
* You want to be able to handle multiple RTM connections
* You don't want to make your own RTM (websocket) connection manager


## Installation

`gem 'slack-bot-manager'`

__**At this time, slack-bot-manager requires `redis` for tracking the status of tokens.**__

_You will need to have `redis` running for this gem to work. This dependency will be removed soon but will remain the default._



## Getting Started

To get started, get a token (or few) and start your script.

```
botmanager = SlackBotManager::Manager.new
botmanager.add_token('token1', 'token2', 'token3')
botmanager.start
botmanager.monitor
```



## Running the Slack Bot Manager

Once you initialize a new `SlackBotManager::Manager`, you can use the following connection and token methods to run your cool new Slack bot app.


### Manager Connection Methods

You can run a manager supporting multiple RTM connections with just __**three**__ lines!

```
botmanager = SlackBotManager::Manager.new
botmanager.start
botmanager.monitor
```

These are the available connecton methods:

methods     | description
------------|----------------------------------------------------------------------------------------------
`start`     | Start connections by fetching known tokens and creating each connection
`stop`      | Stop connections
`restart`   | Restart connections
`status`    | Get the status of the current manager (number of connections).
`monitor`   | Run the manager in a continuous loop, checking for changes in connections and token statuses.


### Token Management Methods

Tokens are managed using key storage, currently only supporting Redis. SlackBotManager will manage and monitor these  keys for additions, updates, and removals. New connections will be added into the key `teams_key`, like so:

```
botmanager = SlackBotManager::Manager.new
botmanager.add_token('token1', 'token2', 'token3') # takes array
```

These are the available token methods:

methods                 | description
------------------------|----------------------------------------------------------------------------
`add_token(*tokens)`    | Add new token(s), will connect within `monitor` loop. [array]
`remove_token(*tokens)` | Remove token(s), will disconnect within `monitor` loop. [array]
`update_token(*tokens)` | Update token(s), will trigger update methods within `monitor` loop. [array]
`check_token(*tokens)`  | Check the status of token(s), output status(es). [array]



## Client Connections

Each RTM connection handled by `SlackBotManager::Manager` is generated by `SlackBotManager::Client`. This client class assists in checking RTM (websocket) connection status, storing various attributes, and includes event listener support. 

The following instance variables are accessible by Client and the included Commands module:

variable      | description
--------------|----------------------------------------------------------------------------------------
`connection`  | `Slack::RealTime::Client`  connection
`id`          | Team's Slack ID (ex. `T123ABC`) _(set after successful connection)_
`token`       | Team's Slack access token (ex. `xoxb-123abc456def`)
`status`      | Known connection status. (`connected`, `disconnected`, `rate_limited`, `token_revoked`)


### Adding Event Listeners

You will want to handle your own RTM event listeners to perform specific functions. This is achieved by extending the `SlackBotManager:Commands` module, which is included within the `SlackBotManager::Client` class (and access to subsequent instance variables specific to that connection).

Each event must be prefixed with `on_`, e.g. `on_messsage` will handing incoming messages.

```
module SlackBotManager
  module Commands
    def on_hello(data)
      puts "Connected to %s" % self.id
    end

    def on_team_join(data)
      puts "New team member joined: %s" % data['user']['username']
    end
  end
end
```

(A full list of events is available from the [Slack API docs](https://api.slack.com/rtm#events).)



## Configuration

### Manager configuration options

setting           | description
------------------|-----------------------------------------------------------------------------------
`tokens_key`      | Redis key name for where tokens' status are stored. _(default: tokens:statuses)_
`teams_key`       | Redis key name for where teams' tokens are stored. _(default: tokens:teams)_
`check_interval`  | Interval (in seconds) for checking connections and tokens status. _(default: 5)_
`storage`         | Define your connection (Redis). _(default: Redis.new)_
`logger`          | Define the logger to use. _(default: Rails.logger or ::Logger.new(STDOUT))_
`log_level`       | Explicity define the logger level. _(default: ::Logger::WARN)_
`verbose`         | When true, set `log_level` to ::Logger::DEBUG. _(default: false)_

You can define these configuration options as:

```
SlackBotManager::Manager.configure do |config|
  config.storage = Redis.new(host: '0.0.0.0', port: 6379)
  config.check_interval = 10 # in seconds
end
```

### Client configuration options

setting           | description
------------------|-----------------------------------------------------------------------------------
`logger`          | Define the logger to use. _(default: Rails.logger or ::Logger.new(STDOUT))_
`log_level`       | Explicity define the logger level. _(default: ::Logger::WARN)_
`verbose`         | When true, set `log_level` to ::Logger::DEBUG. _(default: false)_

You can define these configuration options as:

```
SlackBotManager::Client.configure do |config|
  config.check_interval = 10 # in seconds
  config.log_level = ::Logger::INFO
end
```


### Additional configuration options

For customization of Slack connections, including proxy, websocket ping, endpoint, user-agent, and more, check out the [slack-ruby-client README](https://github.com/dblock/slack-ruby-client/blob/master/README.md).



## Examples

You can check a few creative examples in the [examples](examples/) folder.



## History

This gem will be released soon, and is based on earlier work created by [betaworks](https://betaworks.com) for [PlusPlus++](https://plusplus.chat) Slack app.

Also thanks to [slack-ruby-client](https://github.com/dblock/slack-ruby-client).



## Contributing

See [CONTRIBUTING](CONTRIBUTING.md).

Thanks to our contributors [Greg Leuch](https://gleu.ch) and [Alex Baldwin](http://goose.im).


## Copyright and License

Copyright (c) 2016 [Greg Leuch](https://gleu.ch) & [betaworks](https://betaworks.com).

Licensed under [MIT License](LICENSE.md).
