# Elevator pitch

The Bevy game engine uses an Entity-Component-System (ECS) engine to manage all the game objects and logic. It does some type system magic. I think type system magic is cool. I believe you have said that you think type system magic is cool. Does this fall under your idea of cool type system magic? That's up to you. I would like to propose that you implement, live on a stream, a simplified version of the Bevy ECS.

# General ECS overview

A computer game has entities, and these entities have logic applied to them. Many entities have much of the same logic applied to them. The ECS model solves this the following way:

 - **Entity**: An entity is a game object. It has, a priori, no logic applied to it
 - **Component**: A component is a property put on an entity, and it is how entities hook into the game logic
 - **System**: A piece of game logic, a function that acts on entities through whatever components they may have

For instance, a game object can have a `position` component and a `velocity` component, each containing coordinates of relevant dimensions. A `move()` system would look through all game objects that have both `position` and `velocity`, and update the coordinates in their `position` accordingly. Objects that are placed at a point in the game world but should never move (at least not frame-by-frame, e.g. walls) won't have a `volocity` component. Some objects might even have a `velocity` component but no `position` (e.g. wind). The `move()` system doesn't touch those.

Even rendering is tied to components, where a component would store the sprite or model data needed to draw the object in question.

As a concrete example of a very simple ECS, the entities could be `usize` IDs, the components could be containers of some kind storing all the IDs that have the relevant component (along with whatever data such a component might carry), and systems would iterate over these containers, intersecting and zipping them over the ID where necessary, and applying whatever logic they are made to implement.

# The Bevy ECS

Here is a simple example of the Bevy ECS in action (taken from the getting started section of the [Bevy Book](https://bevyengine.org/learn/book/getting-started/ecs/)):

```rust
#[derive(Component)]
struct Person;

#[derive(Component)]
struct Name(String);

fn hello_world() {
    println!("hello world!");
}

fn add_people(mut commands: Commands) {
    commands.spawn().insert(Person).insert(Name("Elaina Proctor".to_string()));
    commands.spawn().insert(Person).insert(Name("Renzo Hume".to_string()));
    commands.spawn().insert(Person).insert(Name("Zayna Nieves".to_string()));
}

fn greet_people(query: Query<&Name, With<Person>>) {
    for name in query.iter() {
        println!("hello {}!", name.0);
    }
}

fn main() {
    App::new()
        .add_startup_system(add_people)
        .add_system(hello_world)
        .add_system(greet_people)
        .run();
}
```
