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

### Tester Functions

Tester functions define what we want an specific stream to test against. They get passed each individual element of the stream as they become available, so you have access to all its details.

Unfortunately, this means that you have to know the element type and structure.

All tests must have a type signature `Any -> Boolean`

```python
def region_is_capital(event):
    return event.place.region in ['Madrid', 'Barcelona']


def has_rating(event):
    return event.rating != None;


def has_taxonomies(event):
    return len(event.taxonomies) > 0
```

### Test Cases

Once you have your streams and tests defined, you can construct your test case.

A test case consists of four parts: the integration it must notify on failure, the test explanation, and an optional test body, followed by an `expect` call:

```python
@failure >> slack
@check 'some test description so you can put whatever you want here':
    expect(some_stream)(some_test)
```

- `@failure >> slack` - This tells pysellus to notify 'slack' whenever an error happens in the test.
- `@check 'description'` - Define a test case, followed by a non-empty explanation of the test.
- `expect(some_stream)(some_test)` - Here, we say that all elements in `some_stream` must pass `some_test`. Whenever an element doesn't pass, this will get notified to whatever integration we defined in the `@failure` decorator.


You can find the implementation for [`@failure`](https://github.com/Pysellus/pysellus/blob/49f1fd529a3ed1dd49d689f7f948fb523ab4f0db/pysellus/integrations.py#L10) and [`expect`](https://github.com/Pysellus/pysellus/blob/49f1fd529a3ed1dd49d689f7f948fb523ab4f0db/pysellus/registrar.py#L14) on the main [repository](https://github.com/Pysellus/pysellus)

### Test Case Execution

As you saw, test cases are not really Python source, but are easily (and automatically) preprocessed.

So this...

```python
@failure >> slack
@check 'some test description so you can put whatever you want here':
    # test case body goes here
```

... becomes this:

```python
@on_failure('slack')
def pscheck_some_test_description_so_you_can_put_whatever_you_want_here:
    # test case body goes here 
```

Here is a practical use case of the previous notation...

```python
events = stream('smartvel')

@failure >> slack
@check 'all events in Madrid have at least a two star rating':

    # First, filter the stream to get the specific case 
    # that we want to test
    all_events_in_madrid = events.filter(event_in_madrid)

    # then, we expect that all elements in the test satisfy the given test
    expect(all_events_in_madrid)(has_at_least_two_stars)
```

...that will then expanded into the following:

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

### Integrations

Pysellus defines an 'integrator protocol' via the [`AbstractIntegration`](https://github.com/Pysellus/pysellus/blob/49f1fd529a3ed1dd49d689f7f948fb523ab4f0db/pysellus/interfaces.py#L6) class, that all built-in and custom integrations must implement.

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
