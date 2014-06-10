config_masticator
=================

a library for mashing inheritable config up into final specs


#what is this for?

say you have a bunch of different things that are being configured. Maybe you are using a configurable dependency injection library like [the-works](https://github.com/SQUARE-WAVES/theWorks).

In a lot of cases those things will have a bunch of parts which are all the same, stuff like api endpoint urls, or logging specs or something like that. Wouldn't it be nice to be able to specify a base set, and then just write out the differences?

say you were writing an api integration, with some api that had a few methods, and you have a generic library that needs to be configured with a url to each different method. You could specify each out individually:

```

var getUsersConfig = {
	'protocol':'http:',
	'method':'GET'
	'host':'stupidapi.dumb',
	'path':'/users'
};

var getDocsConfig = {
	'protocol':'http:',
	'method':'GET'
	'host':'stupidapi.dumb',
	'path':'/horse_specs'
};

var getHorseBettingForm = {
	'protocol':'http:',
	'method':'GET'
	'host':'stupidapi.dumb',
	'path':'/betting_form'
};

```

this is ok, and pretty easy thanks to text editors and copy-pasting. However when it's time to change things it gets quite cumbersome, say you wanted to change the host, you would have to remember to change it in all 3 places. Using the config masticator you would specify a base config, and then overlay the changes onto it. Then you only have to change the base config to change everything else.

```
var apiBase = {
	'protocol':'http:',
	'method':'GET'
	'host':'stupidapi.dumb',
};

var getUsersConfig = configMasticator.overlay([apiBase],{
	'path':'/users'
}]);

var getDocsConfig = configMasticator.overlay([apiBase],{
	'path':'/horse_specs'
}]);

var getHorseBettingForm = configMasticator.overlay([apiBase],{
	'path':'/betting_form'
}]);


```
#why do you pass an array in?

in case you have more than one base set! Consider the above example with a bit of modification, now our api funcitons need a url, an databaseConnection, and a logger spec

```
var urlBase = {
	'api_url':{
		'protocol':'http:',
		'method':'GET'
		'host':'stupidapi.dumb',
	}
};

var dbConfig = {
	'dbConnection':{
		'host':localhost,
		'port':69420
	}
}

var loggerBase = {
	'logger':{
		'transport':'syslog',
		'facility':17
	}
}

var getUsersConfig = configMasticator.overlay([urlBase,dbConfig,loggerBase,{
 'api_url':{
 		'path':'/users'
	},
	'dbConnection':{
		'table':'horse_gamblers'
	},
	'logger':{
		'level':'debug',
		'prefix':'api_get_users'
	}
}]);

var getDocsConfig = configMasticator.overlay([urlBase,dbConfig,loggerBase,{
 'api_url':{
 		'path':'/horse_specs'
	},
	'dbConnection':{
		'table':'horse_stats'
	},
	'logger':{
		'level':'debug',
		'prefix':'api_get_docs'
	}
}])

var getHorseBettingForm = configMasticator.overlay([urlBase,dbConfig,loggerBase,{
 'api_url':{
 		'path':'/betting_form'
	},
	'dbConnection':{
		'table':'bets'
	},
	'logger':{
		'level':'debug',
		'prefix':'api_get_bets'
	}
}])

```

once again, in each new piece, you only had to specify what was different.

#what are the rules by which configs get merged.

For some mysterious reason the rules closely match _.merge from lodash, with a couple of exceptions however.

first: Arrays are treated as single values, and are not merged. eg:
```
	var base = {'a':[1,2,3]};
	var derived = configMasticator.overlay([base,{
		'a':['a','b','c']
	}]);

	derived.a -> ['a','b','c']
```

second: null values are stripped out, and never show up in a final merge (meaning you can use null to get rid of thigns you don't want) eg:
```
	var base = {
		'a':[1,2,3],
		'b':{
			'ducks':true,
			'dogs':false
		}
	};

	var derived = configMasticator.overlay([base,{
		'b':null
	}]);

	derived.a -> [1,2,3]
	derived.hasOwnProperty('b') -> false
```

#what if I want to get rid of a lot of stuff, and only use a few parts from the base set?

we export a different function for that! It's called trimout, and it takes 2 arguments, the first is a list of configs exactly like overlay, and the last is the final bit of config you care about. Say in our previous "api" example you wanted to add a new thing which didn't use the logger or the database

```

var weirdRouteConfig = configMasticator.trimout([urlBase,dbConfig,loggerBase],{
	'api_url':{
		'path':'/what/the/heck/man'
	}
});

weirdRouteConfig.dbConnection -> undefined;
weirdRouteConfig.logger -> undefined;
weirdRouteConfig.api_url -> {
		'protocol':'http:',
		'method':'GET'
		'host':'stupidapi.dumb',
		'path': '/what/the/heck/man'
	}
};

```

