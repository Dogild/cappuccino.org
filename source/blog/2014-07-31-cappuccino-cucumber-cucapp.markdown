---
title: Cappuccino + Cucumber = Cucapp
author: Alexandre Wilhelm
author_email: alexandre.wilhelmfr@gmail.com
date: '2014-07-31'
tags:
- Cappuccino
- Cucumber
- Cucapp
- test
- Automation test
- Functional test
- Nuage Networks
---

At [Nuage Networks](http://www.nuagenetworks.net) we faced the problem about the testing of our `Cappuccino` application.

For unit-testing, it was pretty easy, we are eaxctly doing as `Cappuccino`. [OJTest](https://github.com/cappuccino/OJTest) allows you to create unit-tests easily and it works pretty well.

For the functional tests, we ended up to believe that [Cucumber](http://cukes.info) would be a great solution for our needs, so we took a look to [Cucapp](https://github.com/cappuccino/cucapp). And this is awesome ! We have now a suite of automation tests who tests our application after each commit.

### Philosophy of Cucapp

`Cucapp` is an interface between `Cucumber` and `Cappuccino`. The `Cappuccino` application is served by a `Thin` server and a small piece of codes is injected in the `Cappuccino` application. This piece of codes connects the Cucumber scripts and the application via AJAX requests.

Once `Cucumber` was able to send actions to our application and have , we wanted to simulate what an user could do with our application. The first thought was "Wow, an user can do whatever he wants, there is an unlimited possibility". In reality, an user can do a small amount of things :

* He can hit a key(s) of his keyboard
* He can move his mouse
* He can click (left or right)
* He can drag
* He can scroll

And that's it ! Our first job was to add these new features to `Cucapp` and allows a tester to easily simulate an event on a `CPResponder`. Then we added some features to help us to easily launch a suite of tests.

### New features

##### Environment variables

`Cucapp` provides a set of environement variable :

* `CUCAPP_PORT` allows you to specify the post used by the server `Thin`.
* `CUCAPP_APPDIRECTORY` allows you to specify where is located your `Cappuccino` application.
* `CUCAPP_BUNDLE` allows you to specify if you want to use or not the compiled version of `Cucapp`.
* `CUCAPP_APPLOADINGMODE` allows tou to specify which version (build or debug) of your app you want to test.

##### Categories

- `HelperCategories.j` is a category used to make a xml dump of the UI. This dump contains the reference of each `CPResponder` currently displayed. This is used to make a relation between the element you want to access in your test and the Cappuccino application.

- `CPResponder+CuCapp.j` is a category who adds the method `- (void)setCucappIdentifier` to the `CPResponder`. You need to include this category in your Cappuccino application to use it. The variable `cucappIdentifier` can be massivelly used in the `Cucumber` features to get an easy access to a targeted `CPResponder`.

- `CucumberCategories.j` will be loaded (not required) by `Cucapp` when launching `Cucumber`. This file has to be located in `features/support/CucumberCategories.j`. This category allows you to add new Cappuccino methods needed for your tests (for instance a method to check the color of a CPView).

##### Simulate events

`Cucapp` provides a set of methods to simulate user events, this is what you can simulate from `Cucumber`:

###### Ruby methods

```ruby
def simulate_keyboard_event charac, flags
def simulate_keyboard_events string, flags
def simulate_left_click xpath, flags
def simulate_left_click_on_point x, y, flags
def simulate_double_click xpath, flags
def simulate_double_click_on_point x, y, flags
def simulate_dragged_click_view_to_view xpath1, xpath2, flags
def simulate_dragged_click_view_to_point xpath1, x, y, flags
def simulate_dragged_click_point_to_point x, y, x2, y2, flags
def simulate_right_click xpath, flags
def simulate_right_click_on_point x, y, flags
def simulate_scroll_wheel xpath, deltaX, deltaY, flags
```

- The `xpath` param is a path to access to a `CPResponder`. For example the xpath CPTextField[cucappIdentifier='field-name'] will make an access to the `CPTextField` who has a cucappIdentifier set to field-name.
- The `flags` param is an array of the key masks you want to simulate (for example `$CPCommandKeyMask` or/and `$CPShiftKeyMask`). All of this masks are available in the framework `Cucapp` in `lib/encumber.rb`.

###### Cucumber features

This is an example of a step :

```ruby
I want to fill a form and send the informations do
  #Wait that the element CPTextField[cucappIdentifier='field-name'] is displayed
  app.gui.wait_for                    "//CPTextField[cucappIdentifier='field-name']"

  #Simulate a left click on the CPTextField[cucappIdentifier='field-name']
  app.gui.simulate_left_click         "//CPTextField[cucappIdentifier='field-name']", []

  #Simulate the keyboard event cmd+a
  app.gui.simulate_keyboard_event     "a", [$CPCommandKeyMask]

  #Simulate the keyboard event delete
  app.gui.simulate_keyboard_event     $CPDeleteCharacter, []

  #Simulate the keyboard events "m", "y", "_" etc etc...
  app.gui.simulate_keyboard_events    "my_new_name", []

  #Simulate the keyboard event tab
  app.gui.simulate_keyboard_event     $CPTabCharacter , []

  #Simulate the keyboard events "m", "y", "_" etc etc...
  app.gui.simulate_keyboard_events    "my_new_family_name_", []

  #Simulate a left click on the CPButton[cucappIdentifier='button-send']
  app.gui.simulate_left_click         "//CPButton[cucappIdentifier='button-send']", []
end
```

### Demo

A fully demo of what Cucapp can do is available [here](https://github.com/Dogild/Cucapp-demo).

