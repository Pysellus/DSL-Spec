# Pysellus DSL

### Stream

`Stream :: String -> Maybe[Dictionary] -> Stream[Any]`

```python
# Basic Usage
events = Stream("http://api.smartvel.net/v1/events")

# Query parameters or headers (w/ dictionary)
options = {
    "headers": {
        "X-Header": "value"
    },
    "user_agent": "User-Agent-String",
    "basic_auth": ["user", "pass"],
    "query_params": {
        "regions": "Madrid",
        "languages": ["es", "en"]
    }
}

events = Stream("http://api.smartvel.net/v1/events", options)
```

### Filter

`Filter :: (Any -> Boolean) -> Stream[Any]`

Or maybe,

`Filter :: (Any -> Boolean) -> Stream[Any] -> Stream[Any]`

```python
# Filter w/ a simple lambda
football = events.filter(lambda x: x.attribute == value)

# Or use any function with signature
# filter_rugby :: Any -> Boolean
def filter_rugby(element):
    return element.taxonomy__name == "rugby"

# filter can be either a method or a static function

# as a method, we can chain filter such as 
# rugby = events.filter(foo).filter(bar).filter(...)
rugby = events.filter(filter_rugby)

# as a static function, we can apply the same filter to multiple streams such as:
# rugby = Stream.filter(filter_rugby, events, another_stream, ...)
# rugby = Stream.filter(filter_rugby, events)

# You can merge different streams
sports = Stream.merge(football, rugby)
sports = football.merge(rugby)

# You can create a filter by combining two or more filters
# TODO: Obviously, come up with a better name
sports_filter = MagicHelperClass.and(filter_football, filter_rugby, ...)
# this works too
sports_filter = filter_football
                .and(filter_rugby)
                .and(...)
```

### Test Declaration

All tests must have a type signature `Any -> Boolean`


```python

# We can define either filter functions used for 
# limiting the scope of the data you want to test on...
def region_is_capital(event):
    return event.place.region in ['Madrid', 'Barcelona']

# ... or test functions, same as any other filter function
# plus an @on_failure decorator

# You can pass the integration endpoint to the test as a docstring

# TODO: Maybe we can pass more integration names two bind the test to all of them
@on_failure('slack/issues', 'gmail')
def has_rating(event):
    return event.rating != None;

@on_failure('slack/issues')
def has_taxonomies(event):
    return len(event.taxonomies) > 0
```

The `@on_failure` decorator could be implemented as follows:

```python
# this...
@on_failure('integration1', 'integration2', ...)
def test_function(element):
    # test body

# ... becomes this
def on_failure(*integrations):
    def inner_decorator(tester):
        # TODO: Is the functools line really necessary?
        
        @functools.wrap(tester)
        def tester_decorator(item):
            try:
                result = tester(item)
                if not result:
                    for integration in integrations:
                        integrations[integration].notify_failure(item)
                return result
            except:
                # notify integration of exception
                # plus some metadata to make that explicit
                return False
            
        return tester_decorator
    return inner_decorator
    
# notice that the ‘integrations’ hash comes from 
# an outer closure, defined in the core of the service

# Pleaso note that the given test body is enclosed in a try-except block
# any exception inside your test code will be handled
# as a test error to be notified to the assigned integration
# some extra metadata is provided to note that the error came from an exception
# and not from some test failure
```

### Test Execution

Input files are not really Python source, but are easily (and automatically) preprocessed.

So this...

```python
@check 'test description':
    # the test setup body must be valid Python code
```

... becomes this:

```python
# the test description gets expanded into 
# a snake_case’d function definition 

def test_description():
    # valid Python test code

# some alternative notations for the test header could be:
# 'test description':
# > 'test description':
# @ 'test description':
```

Here is a practical use case of the previous notation:

```python
events = streamify('http://api.smartvel.net/v1/events')

@check 'all events in Madrid have at least a two star rating':

    # First, filter the stream to get the specific case 
    # that we want to test
    all_events_in_madrid = events.filter(event_in_madrid)

    # `expect` might be changed to a more descriptive name in the future
    expect(all_events_in_madrid)(has_at_least_two_stars)
```

Where `expect` could be implemented as:

```python
# We should cache all streams passed to expect
# or else they might be garbage-collected when the setup function ends
def expect(a_stream):
    def tester_registrator(*testers):
        observed_streams.append(a_stream)
        
        for tester in testers:
            a_stream.be_verified_by(tester)

    return tester_registrator
```

This would get expanded into the following:

```python
events = streamify('http://api.smartvel.net/v1/events')

# we could also decorate this function (the 'setup' function)
# if it’s useful for something (registering the test name)
# Fix: change this if you change the syntax
def all_events_in_Madrid_have_at_least_a_two_star_rating():
    # First, filter the stream to get the specific data
    # that we want to test
    all_events_in_madrid = events.filter(event_in_madrid)

    # Watches the stream, running the test for each element
    # Then it will notify the integration
    # associated with that test
    expect(all_events_in_madrid)(has_at_least_two_stars)


# automatically append the function call?
# all these unit functions get executed ONCE 
# at the beginning of the system
# all_events_in_Madrid_have_at_least_a_two_star_rating()
```

### Integrations

There are two main viewpoints here:

1. Have a 'global integrator’ service, controlled by the user of `pysellus`, who exposes endpoints for each one of the desired integrations, and merely receives the payload (object which failed the test, test name, etc.) and is in charge of formatting and notifying the actual service integration.

	We put our integration list on `integrations.yml`

	```yaml
	slack:
	    # Developer documentation purposes only
	    name: h4ckademy
	    # Url should have a web hook configured
	    url: global-integrator.com/slackWebHook
	email:
	    name: gmail
	    url: global-integrator.com/emailWebHook
	...
	```

2. `pysellus` assumes the responsibility of performing the integrations by defining an ‘integrator protocol’ which must be implemented by all classes, each representing a service.


