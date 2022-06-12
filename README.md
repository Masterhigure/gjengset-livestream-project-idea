# Elevator pitch

The [Bevy game engine](https://bevyengine.org/) uses an Entity-Component-System (ECS) library to manage all the game objects and logic. It does some kind of type system magic. I would like to propose that you implement, on a livestream, some (simplified) variation on the Bevy ECS.

# General ECS overview

A computer game has entities, and these entities have logic applied to them. Many entities have much of the same logic applied to them. The ECS model solves this the following way:

 - **Entity**: An entity is a game object. It has, a priori, no logic applied to it
 - **Component**: A component is a property put on an entity, and it is how entities hook into the game logic
 - **System**: A piece of game logic, a function that acts on entities through whatever components they may have

For instance, a game object can have a `position` component and a `velocity` component, each containing coordinates of relevant dimensions. A `move()` system would look through all game objects that have both `position` and `velocity`, and update the coordinates in their `position` accordingly. Objects that are placed at a point in the game world but should never move (at least not frame-by-frame, e.g. walls) won't have a `volocity` component. Some objects might even have a `velocity` component but no `position` (e.g. wind). The `move()` system doesn't touch those.

Even rendering is tied to components, where a component would store the sprite or model data needed to draw the object in question.

As a concrete example of a very simple ECS, the entities could be `usize` IDs, the components could be containers of some kind storing all the IDs that have the relevant component (along with whatever data such a component might carry), and systems would iterate over these containers, intersecting and zipping them over the ID where necessary, and applying whatever logic they are made to implement.

# Simple Bevy example

Here is a simple example of the Bevy ECS in action (taken from the Getting Started section of the [Bevy Book](https://bevyengine.org/learn/book/getting-started/ecs/), and using Bevy v0.7.0):

```rust
use bevy::app::ScheduleRunnerPlugin;
use bevy::prelude::{App, Commands, Component, Query};

#[derive(Component)]
struct Name(String);

fn add_people(mut commands: Commands) {
    commands
        .spawn()
        .insert(Name("Elaina Proctor".to_string()));
    commands
        .spawn()
        .insert(Name("Renzo Hume".to_string()));
    commands
        .spawn()
        .insert(Name("Zayna Nieves".to_string()));
}

fn greet_people(query: Query<&Name>) {
    for name in query.iter() {
        println!("Hello {}!", name.0);
    }
}

fn main() {
    App::new()
        .add_plugin(ScheduleRunnerPlugin::default())
        .add_startup_system(add_people)
        .add_system(greet_people)
        .run();
}
```
In the `main()` function we create an app, add a plugin to get an event loop, and register three entities with a `Name` component (by using `add_startup_system()`, which registers a system to run only once at app startup). We then add a system that in each time through the event loop looks for every entity with a `Name` component, through the `Query::iter()` call, and then it prints out a greeting using the `String` inside that `Name` component.

The `commands: Commands` variable is a struct that wraps and exposes the `App`'s game world to facilitate making changes to said game world in a nice and controlled manner.

The output is an infinite loop, printing
```
Hello Elaina Proctor!
Hello Renzo Hume!
Hello Zayna Nieves!
```
over and over to the terminal.

# Slightly more advanced Bevy example

Here is a second example that showcasesa bit more of what the Bevy ECS, and in particular the `Query` struct, is capable of:

```rust
use bevy::app::ScheduleRunnerPlugin;
use bevy::prelude::{App, Commands, Component, Query, With};

#[derive(Component)]
struct AccessCounter(u64);

#[derive(Component)]
struct Person;

#[derive(Component)]
struct FirstName(String);

#[derive(Component)]
struct LastName(String);

fn add_people(mut commands: Commands) {
    commands
        .spawn()
        .insert(Person)
        .insert(FirstName("Elaina".to_string()))
        .insert(LastName("Proctor".to_string()))
        .insert(AccessCounter(0));
    commands
        .spawn()
        .insert(Person)
        .insert(FirstName("Renzo".to_string()))
        .insert(LastName("Hume".to_string()))
        .insert(AccessCounter(0));
    commands
        .spawn()
        .insert(Person)
        .insert(FirstName("Zayna".to_string()))
        .insert(LastName("Nieves".to_string()));
    commands
        .spawn()
        .insert(Person)
        .insert(AccessCounter(0));
    commands
        .spawn()
        .insert(Person)
        .insert(FirstName("George".to_string()))
        .insert(AccessCounter(0));
    commands
        .spawn()
        .insert(FirstName("Lake".to_string()))
        .insert(LastName("Ontario".to_string()))
        .insert(AccessCounter(0));
}

fn greet_people(mut query: Query<(&FirstName, &LastName, &mut AccessCounter), With<Person>>) {
    for (fname, lname, mut counter) in query.iter_mut() {
        counter.0 += 1;
        println!("Hello {} {}, for the {}th time!", fname.0, lname.0, counter.0);
    }
}

fn main() {
    App::new()
        .add_plugin(ScheduleRunnerPlugin::default())
        .add_startup_system(add_people)
        .add_system(greet_people)
        .run();
}
```
This time the output is
```
Hello Elaina Proctor, for the 1th time!
Hello Renzo Hume, for the 1th time!
Hello Elaina Proctor, for the 2th time!
Hello Renzo Hume, for the 2th time!
Hello Elaina Proctor, for the 3th time!
Hello Renzo Hume, for the 3th time!
Hello Elaina Proctor, for the 4th time!
Hello Renzo Hume, for the 4th time!
```
(and so on...) What's going on here is a bit more advanced. We now have four components; a marker component `Person`, and splitting `Name` into `FirstName` and `LastName`. In addition we have a counter to keep track of how many times we've greeted each person.

The `Query` now fetches the `FirstName`, `LastName` and `AccessCounter` components on all the entities that have all three of them, and which also have the `Person` component. Which is to say, only the first two people ever get greeted. Also, the `AccessCoutner` component is accessed mutably so that we can actually increment the counter. `Query`'s first 

# Project proposal

There are a few core parts to this that I think are crucial.

 - An `App` struct where you can register entities, their components, and the systems that act on them
 - Systems can have any number of `Query` arguments
 - An event loop in `App::run()` which calls all the registered systems
 - The `Query` struct, which filters and iterates over entities, and zips their components (this struct seems like magic to me, and it's maybe the main reason for my interest in this project)

Some details from these two examples are _not_, in my opinion, crucial to this project. But they could make for nice extras:

 - The `#[derive(Component)]` macro
 - The fact that entities and components are registered inside a system, and similar implementation details
 - Or even the ability to register (or deregister) entities and components inside a system at all, and thus the need for a `Commands` argument
 - The ability to add or remove components from an entity after registration
 - The entire `Plugin` concept
 - `Resource`s, which are singleton structs carrying global game information (such as e.g. game time, game settings, sprite sheets, and renderers), available as optional arguments to systems the way `Query`s are
 - Performance

# Why it's important to teach

I think type system magic is cool, and I don't think I'm alone in that. How in the blazes they manage to do things like the system registration and the contruction of the `Query` objects for each system call is beyond me, and I don't think I'm alone in that. Also, Bevy is a cool project, and I think it would be cool to shed some light on what kind of things are going on (or at least might be going on) under the hood of their engine.

# Why it's possible to teach in the format that you do

This seems to me like a good fit alongside other livestreams you've done: Take some specification (either from some article or from an earlier implementation), and try to implement it or a variation of it yourself, while youtube and twitch watch.

# Why it's more important to teach than other things

I can't see many recent live coding videos from you where you dive deep into the actual type system in Rust and the powers it has. Looking back over the previous year, and judging from the titles and what little I can remember, it has been pretty focused on concurrency. Which, if you want to keep doing that, then I have no right or power to stop you. But if you want to change gears, then I think this is a good oppurtunity to look into a different aspect of Rust.

# Why you should be the one to teach it

I do seem to recall you saying you thought the type system of Rust is cool, so I hope this sounds intriguing to you. And I may not have scoured the internet looking for Rust videos over the years, but a few creators have popped up in my feed, and among those you are the only one who does longer coding content of the form that I suspect this requires.

# Disclaimer

I may have completely misjudged what my proposed project requires. It may be too long for your format, or too full of uninteresting boilerplate, or any number of other different reasons that it's not a good fit. I may have completely misjudged what kind of project this is (maybe there is no type magic going on, and everything just falls into place from using a few auto traits at clever and strategic locations). 
