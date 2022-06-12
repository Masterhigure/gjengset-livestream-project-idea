# Elevator pitch

The [Bevy game engine](https://bevyengine.org/) uses an Entity-Component-System (ECS) engine to manage all the game objects and logic. It does some type system magic. I think type system magic is cool. I believe you have said that you think type system magic is cool. I would like to propose that you implement, live on a stream, some variation on the Bevy ECS.

# General ECS overview

A computer game has entities, and these entities have logic applied to them. Many entities have much of the same logic applied to them. The ECS model solves this the following way:

 - **Entity**: An entity is a game object. It has, a priori, no logic applied to it
 - **Component**: A component is a property put on an entity, and it is how entities hook into the game logic
 - **System**: A piece of game logic, a function that acts on entities through whatever components they may have

For instance, a game object can have a `position` component and a `velocity` component, each containing coordinates of relevant dimensions. A `move()` system would look through all game objects that have both `position` and `velocity`, and update the coordinates in their `position` accordingly. Objects that are placed at a point in the game world but should never move (at least not frame-by-frame, e.g. walls) won't have a `volocity` component. Some objects might even have a `velocity` component but no `position` (e.g. wind). The `move()` system doesn't touch those.

Even rendering is tied to components, where a component would store the sprite or model data needed to draw the object in question.

As a concrete example of a very simple ECS, the entities could be `usize` IDs, the components could be containers of some kind storing all the IDs that have the relevant component (along with whatever data such a component might carry), and systems would iterate over these containers, intersecting and zipping them over the ID where necessary, and applying whatever logic they are made to implement.

# Simple Bevy example

Here is a simple example of the Bevy ECS in action (taken from the Getting Started section of the [Bevy Book](https://bevyengine.org/learn/book/getting-started/ecs/)):

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
This piece of code creates an app, adds a plugin to get an event loop, registers five entities, with a `Name` component (by using `add_startup_system()`, which is a `system` that runs once at app startup). It then adds a system that in each time through the event loop looks for every entity with a `Name` component, through the `Query::iter()` call, and prints out a greeting from the name. The `commands: Commands` variable is a struct that wraps and exposes `app`'s game world to facilitate making changes to said game world in a nice and controlled manner.

# Slightly more advanced Bevy example

Here is a slightly more advanced example that showcases the full ECS functionality that I think is relevant to this project:

```rust
use bevy::app::ScheduleRunnerPlugin;
use bevy::prelude::{App, Commands, Component, Query, With};

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
        .insert(LastName("Proctor".to_string()));
    commands
        .spawn()
        .insert(Person)
        .insert(FirstName("Renzo".to_string()))
        .insert(LastName("Hume".to_string()));
    commands
        .spawn()
        .insert(Person)
        .insert(FirstName("Zayna".to_string()))
        .insert(LastName("Nieves".to_string()));
    commands
        .spawn()
        .insert(Person);
    commands
        .spawn()
        .insert(Person)
        .insert(FirstName("George".to_string()));
    commands
        .spawn()
        .insert(FirstName("Lake".to_string()))
        .insert(LastName("Ontario".to_string()));
}

fn greet_people(query: Query<(&FirstName, &LastName), With<Person>>) {
    for fname, lname in query.iter() {
        println!("Hello {} {}!", fname.0, lname.0);
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
