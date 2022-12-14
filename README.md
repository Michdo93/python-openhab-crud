# Python openHAB CRUD
A simple `CRUD` for accessing the openHAB REST API. Whether via the cloud or locally. The `CRUD` can be used only for openHAB `Items`. `CRUD` means that you can run `Create`, `Read`, `Update` and `Delete` on `Items`. The openHAB Cloud does not allow to use `Create` and `Delete`. This is the case for security reasons. In this case you have to create or delete an item locally.

## Installation

You can install it by using `pip`:

```
python3 -m pip install python-openhab-crud
```

## Usage

`CRUD` means that you can run `Create`, `Read`, `Update` and `Delete` on `Items`. `Reading` an `Item` can have two different meanings: You can `Read` the whole `Item` or only the `State` of an `Item`. The latter can be done with the `getState` function. Also can `Updating` an `Item` have two different meanings. In openHAB you can change an `State` by using `sendCommand` or `postUpdate`. The function `update` is an alias for `sendCommand` or `postUpdate`. You can specify `sendCommand` or `postUpdate` as parameters for `update`. If no parameter is specified, `postUpdate` is used automatically.

At first you have to import the `CRUD` module:

```
from openhab import CRUD
```

### Create connection to the openHAB Cloud (as example myopenhab.org)

If you want to create a connection to the `openHAB Cloud` you have to run:

```
crud = CRUD("https://myopenhab.org", "<your_email>@<your_provider>", "<email_password>")
```

Please make sure to replace `<your_email>@<your_provider>` with your email address and `<email_password>` with your password that you used for your `openHAB Cloud account`.

### Create connection to your local openHAB instance

If you want to create a connection to your local `openHAB` instance you have to replace `<username>` and `<password>` with the username and password of your local `openHAB account`:

```
crud = CRUD("http://<your_ip>:8080", "<username>", "<password>")
```

### Get all Items

If you want to retrieve all Items via the `REST API` of `openHAB` you can run:

```
items = crud.getAllItems()
```

The return value is a `list` of `dictionaries` (`dict`). Each item is represented as `dict`. You can check it by using `type`:

```
print(type(items))
```

You can print each item separately by running:

```
for item in items:
    print(item)
```

And of course you can access each property by using the `get` method:

```
for item in items:
    item_type = item.get("type")
    item_name = item.get("name")
    item_state = item.get("state")

    print("Received item type " + item_type + " with name " + item_name + " and state " + item_state)
```

### Creating new items

You can only create a new `Item` if you have access to your local `openHAB` instance. For security reasons, this is prohibited via the cloud.

An `Item` must contain minimum its [Name](https://www.openhab.org/docs/configuration/items.html#name) and its [Type](https://www.openhab.org/docs/configuration/items.html#type). A [Label](https://www.openhab.org/docs/configuration/items.html#label), a given [State](https://www.openhab.org/docs/configuration/items.html#state) or [Groups](https://www.openhab.org/docs/configuration/items.html#groups) will be optional.

So at least you can run as example following:

```
crud.create("testItem", "String", "my test string", ["testGroup"], "ON")
crud.create("testItem2", "String", "my test string", ["testGroup"])
crud.create("testItem3", "Switch", "my test switch")
crud.create("testItem4", "Switch")
```

Of course you can use a different order:

```
crud.create(state="ON", type="String", name="testItem")
```

`Groups` must be passed as a `list`, since an `Item` can be assigned to several `Groups`:

```
crud.create("testItem", "String", "my test string", ["group1", "group2", "group3"])
```

The function automatically checks if the `Type` of the `Item` exists in openHAB and thus can be created. If you want to set a `State` for this `Item`, it is also automatically checked whether the `State` is consistently valid for this `Type`.

There will be no response if it is correct!

### Reading an item

`Reading` an `Item` means that all information about this `Item` is queried and not only the `State`. You can do this as follows:

```
testItem = crud.read("testItem")
```

You will receive an `Item` as a `dictionary` (`dict`):

```
print(type(testItem))
```

And of course you can access each property by using the `get` method:

```
item_type = testItem.get("type")
item_name = testItem.get("name")
item_state = testItem.get("state")

print("Received item type " + item_type + " with name " + item_name + " and state " + item_state)
```

### Updating an item

`Updating` an `Item` have two different meanings. In openHAB you can change an `State` by using `sendCommand` or `postUpdate`. The function `update` is an alias for `sendCommand` or `postUpdate`. You can specify `sendCommand` or `postUpdate` as parameters for `update`. If no parameter is specified, `postUpdate` is used automatically.

So you can run as example

```
crud.update("testItem", "Hello World")
```

or

```
crud.update("testItem", "Hello World", "postUpdate")
```

instead of

```
crud.postUpdate("testItem", "Hello World")
```

Or if you want to use `sendCommand` in `openHAB` you can run

```
crud.update("testItem", "Lorem ipsum", "sendCommand")
```

instead of

```
crud.sendCommand("testItem", "Lorem ipsum")
```

Note: `postUpdate` and `sendCommand` have validation capabilities. With `type` you can set the `type` of the `Item`. This will call a function which checks that the `value` which should change the `State` of the `Item` is `persistent`. However, this does not mean that you are checking the correct `Type`, because `openHAB` may have a different `Type` than the one you want to update. If you will set the `validate` variable to `True` the `Item` is queried via the `REST API` and the `Type` is checked.

For checking if you put a right value for the `State` of the assumed `Type` you can run as example

```
crud.sendCommand("testItem", "Lorem ipsum", "Switch")
```

or

```
crud.postUpdate("testItem", "Hello World", "Switch")
```

If you want to check if the actual `Type` on openHAB is correct and you can set the `State` to this value you can run:


```
crud.sendCommand("testItem", "Lorem ipsum", None, True)
```

or

```
crud.postUpdate("testItem", "Hello World", None, True)
```

Another possibility is: name:str, value, type:str = None, validate:bool = None):

```
crud.sendCommand("testItem", "Lorem ipsum", validate=True)
```

or

```
crud.postUpdate("testItem", "Hello World", validate=True)
```

Of course you can change the order as example to:

```
crud.postUpdate(validate=True, type="Switch", name="testItem")
```

There will be no response if it is correct!

### Deleting new items

You can only delete an `Item` if you have access to your local `openHAB` instance. For security reasons, this is prohibited via the cloud.

In this case you can run following:

```
crud.delete("testItem")
```

There will be no response if it is correct!

### Getting the state of an item

In most cases you only want to query the current `State` of an `Item` and not get all informations of an `Item`. The `getState` function can be used for this purpose:

```
state = crud.getState("testItem")
```

You get the state as a string:

```
print(type(state))
```

This is due to the data transmission via `http`. Numerical values must be parsed. `openHAB` specific values like `UP`, `DOWN`, `PLAY`, `PAUSE`, `ON`, `OFF` etc. have no data type in Python, i.e. here a string (`str`) still makes sense. Here the capitalization must be considered!

At least you can print the `State` with:

```
print(state)
```

### Close session

Since we may want to make several requests in a row, a `session` is opened when the CRUD `object` is created. This can of course also be terminated at the end:

```
crud.close()
```
