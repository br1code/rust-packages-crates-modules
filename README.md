# Managing Growing Projects with Packages, Crates, and Modules

As you write large programs, organizing your code will be important because keeping track of
your entire program in your head will become impossible.
By grouping related functionality and separating code with distinct features,
you’ll clarify where to find code that implements a particular feature and where to go to change how a feature works.

The programs we’ve written so far have been in one module in one file.
As a project grows, you can organize code by splitting it into multiple modules and then multiple files.
A package can contain multiple binary crates and optionally one library crate.

As a package grows, you can extract parts into separate crates that become external dependencies.

Note: For very large projects of a set of interrelated packages that evolve together, Cargo provides workspaces.

In addition to grouping functionality, encapsulating implementation details lets you reuse code at a higher level:
once you’ve implemented an operation, other code can call that code via the code’s public interface without knowing how the implementation works.
The way you write code defines which parts are public for other code to use and which parts are private implementation details that you reserve the right to change.

A related concept is *scope*: the nested context in which code is written has a set of names that are defined as “in scope.”
When reading, writing, and compiling code, programmers and compilers need to know whether a particular name at a
particular spot refers to a variable, function, struct, enum, module, constant, or other item and what that item means.
You can create scopes and change which names are in or out of scope.

You can’t have two items with the same name in the same scope; tools are available to resolve name conflicts.

Rust has a number of features that allow you to manage your code’s organization, including which details are exposed,
which details are private, and what names are in each scope in your programs.

These features, sometimes collectively referred to as the module system, include:
- Packages: A Cargo feature that lets you build, test, and share crates
- Crates: A tree of modules that produces a library or executable
- Modules and use: Let you control the organization, scope, and privacy of paths
- Paths: A way of naming an item, such as a struct, function, or module


# Packages And Crates

A crate is a binary or library.
The crate root is a source file that the Rust compiler starts from and makes up the root module of your crate.

A package is one or more crates that provide a set of functionality.
A package contains a Cargo.toml file that describes how to build those crates.

Several rules determine what a package can contain.
A package must contain zero or one library crates, and no more.
It can contain as many binary crates as you’d like, but it must contain at least one crate (either library or binary).

Let’s walk through what happens when we create a package. First, we enter the command ``cargo new``:

````commandline
$ cargo new my-project
Created binary (application) `my-project` package
$ ls my-project
Cargo.toml
src
$ ls my-project/src
main.rs
````

When we entered the command, Cargo created a ``Cargo.toml`` file, giving us a package.
Looking at the contents of ``Cargo.toml``, there’s no mention of ``src/main.rs`` because Cargo follows a convention that ``src/main.rs`` is the
crate root of a binary crate with the same name as the package.
Likewise, Cargo knows that if the package directory contains ``src/lib.rs``, the package contains a library crate with the same name as the package,
and ``src/lib.rs`` is its crate root. Cargo passes the crate root files to ``rustc`` to build the library or binary.

Here, we have a package that only contains ``src/main.rs``, meaning it only contains a binary crate named ``my-project``.
If a package contains ``src/main.rs`` and ``src/lib.rs``, it has two crates: a library and a binary, both with the same name as the package.
A package can have multiple binary crates by placing files in the ``src/bin`` directory: each file will be a separate binary crate.

A crate will group related functionality together in a scope so the functionality is easy to share between multiple projects.

For example, the ``rand`` crate provides functionality that generates random numbers.
We can use that functionality in our own projects by bringing the ``rand`` crate into our project’s scope.
All the functionality provided by the ``rand`` crate is accessible through the crate’s name, ``rand``.

Keeping a crate’s functionality in its own scope clarifies whether particular functionality is defined
in our crate or the ``rand`` crate and prevents potential conflicts.

For example, the ``rand`` crate provides a trait named ``Rng``.
We can also define a ``struct`` named ``Rng`` in our own crate.
Because a crate’s functionality is namespaced in its own scope, when we add ``rand`` as a dependency,
the compiler isn’t confused about what the name ``Rng`` refers to.
In our crate, it refers to the ``struct`` ``Rng`` that we defined.
We would access the ``Rng`` trait from the ``rand`` crate as ``rand::Rng``.

# Defining Modules to Control Scope and Privacy

Modules let us organize code within a crate into groups for readability and easy reuse.

Modules also control the privacy of items, which is whether an item can be used by outside code (public) or is an internal implementation detail and not available for outside use (private).

As an example, let’s write a library crate that provides the functionality of a restaurant. We’ll define the signatures of functions but leave their bodies empty to concentrate on the organization of the code, rather than actually implement a restaurant in code.

In the restaurant industry, some parts of a restaurant are referred to as front of house and others as back of house.
Front of house is where customers are; this is where hosts seat customers, servers take orders and payment, and bartenders make drinks. 
Back of house is where the chefs and cooks work in the kitchen, dishwashers clean up, and managers do administrative work.

To structure our crate in the same way that a real restaurant works, we can organize the functions into nested modules.

Create a new library named restaurant by running ``cargo new --lib restaurant``; then put the following code into ``restaurant/src/lib.rs`` to define some modules and functions signatures.
````rust
mod front_of_house {
    mod hosting {
        fn add_to_waitlist() {}

        fn seat_at_table() {}
    }

    mod serving {
        fn take_order() {}

        fn serve_order() {}

        fn take_payment() {}
    }
}
````

We define a module by starting with the ``mod`` keyword and then specify the name of the module (in this case, ``front_of_house``) and place curly brackets around the body of the module.

Inside modules, we can have other modules, as in this case with the modules ``hosting`` and ``serving``. 

Modules can also hold definitions for other items, such as structs, enums, constants, traits, or functions.

By using modules, we can group related definitions together and name why they’re related.
Programmers adding new functionality to this code would know where to place the code to keep the program organized.

Earlier, we mentioned that ``src/main.rs`` and ``src/lib.rs`` are called ``crate`` roots. 
The reason for their name is that the contents of either of these two files form a module named crate at the root of the crate’s module structure, known as the module tree.

Module tree for the structure of restaurant:
````
crate
 └── front_of_house
     ├── hosting
     │   ├── add_to_waitlist
     │   └── seat_at_table
     └── serving
         ├── take_order
         ├── serve_order
         └── take_payment
````

This tree shows how some of the modules nest inside one another (for example, ``hosting`` nests inside ``front_of_house``).

The tree also shows that some modules are siblings to each other, meaning they’re defined in the same module (``hosting`` and ``serving`` are defined within ``front_of_house``).

To continue the family metaphor, if module ``A`` is contained inside module ``B``, we say that module ``A`` is the child of module ``B`` and that module ``B`` is the parent of module ``A``.

> Notice that the entire module tree is rooted under the implicit module named ``crate``.

Just like directories in a filesystem, you use modules to organize your code. And just like files in a directory, we need a way to find our modules.

# Paths for Referring to an Item in the Module Tree

To show Rust where to find an item in a module tree, we use a path in the same way we use a path when navigating a filesystem. If we want to call a function, we need to know its path.

A path can take two forms:
- An absolute path starts from a crate root by using a crate name or a literal crate.
- A relative path starts from the current module and uses ``self``, ``super``, or an identifier in the current module.

Both absolute and relative paths are followed by one or more identifiers separated by double colons (``::``).

How do we call the ``add_to_waitlist`` function? This is the same as asking, what’s the path of the ``add_to_waitlist`` function?

> we simplified our code a bit by removing some of the modules and functions.

We’ll show two ways to call the ``add_to_waitlist`` function from a new function ``eat_at_restaurant`` defined in the crate root.
The ``eat_at_restaurant`` function is part of our library crate’s public API, so we mark it with the ``pub`` keyword.

> Note that this example won’t compile just yet; we’ll explain why in a bit.

````rust
mod front_of_house {
    mod hosting {
        fn add_to_waitlist() {}
    }
}

pub fn eat_at_restaurant() {
    // Absolute path
    crate::front_of_house::hosting::add_to_waitlist();

    // Relative path
    front_of_house::hosting::add_to_waitlist();
}
````

The first time we call the ``add_to_waitlist`` function in ``eat_at_restaurant``, we use an absolute path. 
The ``add_to_waitlist`` function is defined in the same crate as ``eat_at_restaurant``, which means we can use the ``crate`` keyword to start an absolute path.
After ``crate``, we include each of the successive modules until we make our way to ``add_to_waitlist``.

The second time we call ``add_to_waitlist`` in ``eat_at_restaurant``, we use a relative path.
The path starts with ``front_of_house``, the name of the module defined at the same level of the module tree as ``eat_at_restaurant``.

Choosing whether to use a relative or absolute path is a decision you’ll make based on your project. 
The decision should depend on whether you’re more likely to move item definition code separately from or together with the code that uses the item.

For example, if we move the ``front_of_house`` module and the ``eat_at_restaurant`` function into a module named ``customer_experience``, we’d need to update the absolute path to ``add_to_waitlist``, but the relative path would still be valid.
However, if we moved the ``eat_at_restaurant`` function separately into a module named ``dining``, the absolute path to the ``add_to_waitlist`` call would stay the same, but the relative path would need to be updated.

Our preference is to specify absolute paths because it’s more likely to move code definitions and item calls independently of each other.

If we try to compile the previous code, we'll get an error:

````commandline
$ cargo build
   Compiling restaurant v0.1.0 (file:///projects/restaurant)
error[E0603]: module `hosting` is private
 --> src/lib.rs:9:28
  |
9 |     crate::front_of_house::hosting::add_to_waitlist();
  |                            ^^^^^^^

error[E0603]: module `hosting` is private
  --> src/lib.rs:12:21
   |
12 |     front_of_house::hosting::add_to_waitlist();
   |                     ^^^^^^^

error: aborting due to 2 previous errors

For more information about this error, try `rustc --explain E0603`.
error: could not compile `restaurant`.

To learn more, run the command again with --verbose.
````

The error messages say that module hosting is private. In other words, we have the correct paths for the hosting module and the ``add_to_waitlist`` function, but Rust won’t let us use them because it doesn’t have access to the private sections.

Modules aren’t useful only for organizing your code. They also define Rust’s privacy boundary: the line that encapsulates the implementation details external code isn’t allowed to know about, call, or rely on. So, if you want to make an item like a function or struct private, you put it in a module.

The way privacy works in Rust is that all items (functions, methods, structs, enums, modules, and constants) are **private by default**.

Rust chose to have the module system function this way so that hiding inner implementation details is the default. That way, you know which parts of the inner code you can change without breaking outer code. But you can expose inner parts of child modules' code to outer ancestor modules by using the ``pub`` keyword to make an item public.

# Exposing Paths with the pub Keyword

We want the ``eat_at_restaurant`` function in the parent module to have access to the ``add_to_waitlist`` function in the child module, so we mark the hosting module with the ``pub`` keyword.

````rust
    pub mod hosting {
        fn add_to_waitlist() {}
    }
}

pub fn eat_at_restaurant() {
    // Absolute path
    crate::front_of_house::hosting::add_to_waitlist();

    // Relative path
    front_of_house::hosting::add_to_waitlist();
}
````

Unfortunately, this code still results in an error:

````commandline
$ cargo build
   Compiling restaurant v0.1.0 (file:///projects/restaurant)
error[E0603]: function `add_to_waitlist` is private
 --> src/lib.rs:9:37
  |
9 |     crate::front_of_house::hosting::add_to_waitlist();
  |                                     ^^^^^^^^^^^^^^^

error[E0603]: function `add_to_waitlist` is private
  --> src/lib.rs:12:30
   |
12 |     front_of_house::hosting::add_to_waitlist();
   |                              ^^^^^^^^^^^^^^^

error: aborting due to 2 previous errors

For more information about this error, try `rustc --explain E0603`.
error: could not compile `restaurant`.

To learn more, run the command again with --verbose.
````

What happened? Adding the ``pub`` keyword in front of mod hosting makes the module public. With this change, if we can access ``front_of_house``, we can access ``hosting``.

But the contents of ``hosting`` are still private; making the module public doesn’t make its contents public. The ``pub`` keyword on a module only lets code in its ancestor modules refer to it.

The previous error say that the ``add_to_waitlist`` function is private. The privacy rules apply to structs, enums, functions, and methods as well as modules.

Let’s also make the ``add_to_waitlist`` function public by adding the ``pub`` keyword before its definition.

````rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

pub fn eat_at_restaurant() {
    // Absolute path
    crate::front_of_house::hosting::add_to_waitlist();

    // Relative path
    front_of_house::hosting::add_to_waitlist();
}
````
In the absolute path, we start with crate, the root of our crate’s module tree. Then the ``front_of_house`` module is defined in the crate root. 
The ``front_of_house`` module isn’t public, but because the ``eat_at_restaurant`` function is defined in the same module as ``front_of_house`` 
(that is, ``eat_at_restaurant`` and ``front_of_house`` are siblings), we can refer to ``front_of_house`` from ``eat_at_restaurant``. 
Next is the ``hosting`` module marked with ``pub``. We can access the parent module of ``hosting``, so we can access ``hosting``. 
Finally, the ``add_to_waitlist`` function is marked with ``pub`` and we can access its parent module, so this function call works!

In the relative path, the logic is the same as the absolute path except for the first step: rather than starting from the crate root, the path starts from ``front_of_house``. 
The ``front_of_house`` module is defined within the same module as ``eat_at_restaurant``, so the relative path starting from the module in which ``eat_at_restauran``t is defined works. 
Then, because ``hosting`` and ``add_to_waitlist`` are marked with ``pub``, the rest of the path works, and this function call is valid!

# Starting Relative Paths with super

We can also construct relative paths that begin in the parent module by using ``super`` at the start of the path. This is like starting a filesystem path with the ``..`` syntax.

Consider the following code that models the situation in which a chef fixes an incorrect order and personally brings it out to the customer. 
The function ``fix_incorrect_order`` calls the function ``serve_order`` by specifying the path to ``serve_order`` starting with ``super``:

````rust
fn serve_order() {}

mod back_of_house {
    fn fix_incorrect_order() {
        cook_order();
        super::serve_order();
    }

    fn cook_order() {}
}
````

The ``fix_incorrect_order`` function is in the ``back_of_house`` module, so we can use ``super`` to go to the parent module of ``back_of_house``, which in this case is crate, the root.

# Making Structs and Enums Public

We can also use ``pub`` to designate structs and enums as public, but there are a few extra details. If we use pub before a ``struct`` definition, we make the ``struct`` public, but the struct’s fields will still be private. We can make each field public or not on a case-by-case basis.

Let's look at one example: we’ve defined a public ``back_of_house::Breakfast`` struct with a **public** ``toast`` field but a **private** ``seasonal_fruit field``.

This models the case in a restaurant where the customer can pick the type of bread that comes with a meal, but the chef decides which fruit accompanies the meal based on what’s in season and in stock. The available fruit changes quickly, so customers can’t choose the fruit or even see which fruit they’ll get.

````rust
mod back_of_house {
    pub struct Breakfast {
        pub toast: String,
        seasonal_fruit: String,
    }

    impl Breakfast {
        pub fn summer(toast: &str) -> Breakfast {
            Breakfast {
                toast: String::from(toast),
                seasonal_fruit: String::from("peaches"),
            }
        }
    }
}

pub fn eat_at_restaurant() {
    // Order a breakfast in the summer with Rye toast
    let mut meal = back_of_house::Breakfast::summer("Rye");
    // Change our mind about what bread we'd like
    meal.toast = String::from("Wheat");
    println!("I'd like {} toast please", meal.toast);

    // The next line won't compile if we uncomment it; we're not allowed
    // to see or modify the seasonal fruit that comes with the meal
    // meal.seasonal_fruit = String::from("blueberries");
}
````

Because the ``toast`` field in the ``back_of_house::Breakfast`` ``struct`` is public, in ``eat_at_restaurant`` we can write and read to the ``toast`` field using dot notation. Notice that we can’t use the ``seasonal_fruit`` field in ``eat_at_restaurant`` because ``seasonal_fruit`` is private.

Also, note that because ``back_of_house::Breakfast`` has a private field, the ``struct`` needs to provide a public associated function that constructs an instance of ``Breakfast`` (we’ve named it ``summer`` here).

If ``Breakfast`` didn’t have such a function, we couldn’t create an instance of ``Breakfast`` in ``eat_at_restaurant`` because we couldn’t set the value of the private ``seasonal_fruit`` field in ``eat_at_restaurant``.

In contrast, if we make an ``enum`` public, all of its variants are then public. We only need the ``pub`` before the ``enum`` keyword.

````rust
mod back_of_house {
    pub enum Appetizer {
        Soup,
        Salad,
    }
}

pub fn eat_at_restaurant() {
    let order1 = back_of_house::Appetizer::Soup;
    let order2 = back_of_house::Appetizer::Salad;
}
````

Because we made the ``Appetizer`` ``enum`` public, we can use the ``Soup`` and ``Salad`` variants ``in eat_at_restaurant``. Enums aren’t very useful unless their variants are public; it would be annoying to have to annotate all enum variants with ``pub`` in every case, so the default for enum variants is to be public. Structs are often useful without their fields being public, so struct fields follow the general rule of everything being private by default unless annotated with ``pub``.