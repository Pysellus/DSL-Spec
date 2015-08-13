# Pysellus DSL

You can some complete examples at the [dsl example file](dsl-examples.md)

### Stream

`stream :: String -> Stream[Any]`

```python
# Usage:
events = stream('smartvel')
```

### Filter

`Filter :: (Any -> Boolean) -> Stream[Any]`

```python
# Filter with a simple lambda
football = events.filter(lambda x: x.attribute == value)

# Or use any function with signature
# filter_rugby :: Any -> Boolean
def filter_rugby(element):
    return element.taxonomy__name == "rugby"

# we can chain filter such as
rugby = events.filter(filter_rugby).filter(foo).filter(...)

# You can merge different streams
sports = football.merge(rugby)
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

# We can pass more integration names two bind the test to all of them
@on_failure('slack/issues', 'gmail')
def has_rating(event):
    return event.rating != None;

@on_failure('slack/issues')
def has_taxonomies(event):
    return len(event.taxonomies) > 0
```

You can see the implementation for `@on_failure` on the [main repo](https://github.com/Pysellus/pysellus/blob/49f1fd529a3ed1dd49d689f7f948fb523ab4f0db/pysellus/integrations.py#L10)

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
```

Here is a practical use case of the previous notation:

```python
events = stream('smartvel')

@check 'all events in Madrid have at least a two star rating':

    # First, filter the stream to get the specific case 
    # that we want to test
    all_events_in_madrid = events.filter(event_in_madrid)

    # then, we expect that all elements in the test satisfy the given test
    expect(all_events_in_madrid)(has_at_least_two_stars)
```

This example would get expanded into the following:

```python
events = stream('smartvel')

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
```

You can see the implementation for `expect` on the [main repo](https://github.com/Pysellus/pysellus/blob/49f1fd529a3ed1dd49d689f7f948fb523ab4f0db/pysellus/registrar.py#L14)


### Integrations

Pysellus defines an 'integrator protocol' via the [`AbstractIntegration` class](https://github.com/Pysellus/pysellus/blob/49f1fd529a3ed1dd49d689f7f948fb523ab4f0db/pysellus/interfaces.py#L6), that all built-in and custom integrations must implement.

Some integrations will require some sort of parameters, or API keys that you won't want to put under version control. To this end, when created, this integrations will try to read this parameters from an specific configuration file, `.ps_integrations.yml`. This file should be placed on the same repository of the test files that pysellus will load.

The configuration file has the following format:

```yaml
# integration declarations
notify:
    # all integrations foll
    'user alias':
        integration_type:
            some_parameter: 'some value',
            another_parameter: 'another value'

# custom integrations definitions
custom_integrations:
    custom_integration_type: 'integration class path'
```

A specific example of a configuration file could be:

```yaml
notify:
    'developers_channel':
        slack:
            url: 'https://slack.post.url'
            channel: '#devops'
```
