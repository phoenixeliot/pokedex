# Pokedex: Templates, Views and Routing!

## Phase 4: Templates

We've mixed our HTML code very heavily into our JavaScript; our
`Pokedex.RootView` methods use jQuery to build all the HTML of the
page.

This is not very nice; it's really hard for us to read the code and
know that it is generating the right HTML. It also makes it hard to
see what the `Pokedex.RootView` is doing, because we have to look past
a bunch of spaghetti logic to build HTML.

The traditional solution is to use *client-side templates*. What Rails
calls a *view* (`index.html.erb`, `index.json.jbuilder`) is more
properly called a *template*. We will start out this project building
templates. The big difference is that these templates **will not be
evaluated by the server**, but instead be used by the client to
generate HTML.

Let's start by writing the `pokemon-detail-template` in
`app/views/shared/_templates"`. In this file are a series of script
tags; your template code will go here.

Write an EJS template to display a single Pokemon, mimicing the parts
of `renderPokemonDetail` function that display the Pokemon (but not
the list of toys yet!). Render the partial in the `root.html.erb` so
that these script tags are included in the HTML served by the
server. Remember to use `<%%= ... %>` in your template, so that Rails
doesn't try to interpret it.

When you have written this, check that the following code works:

```javascript
var templateCode = $("#pokemon-detail-template").html();
var templateFn = _.template(templateCode);

var pokemon = new Pokedex.Models.Pokemon({ id: 1 });
pokemon.fetch({
  success: function () {
    console.log(templateFn({ pokemon: pokemon }));
  }
});
```

When this seems to be producing valid HTML, edit `pokedex-4.js` to
Load the template and compile it, saving the template function as
`JST["pokemonDetail"]`.

Last, modify your `renderPokemonDetail` to use
`JST["pokemonDetail"]`. You'll still have jQuery code for building the
list of toys, but this should reduce the amount of jQuery code. Verify
that this works correctly as before.

## Phase 4B: More Templates!

Just as you wrote `#pokemon-detail-template`, write the following
templates, along the way changing the relevant method to use the
template:

* `pokemon-list-item-template`, `addPokemonToList`
* Extend your `pokemon-detail-template` to hold an empty
  `ul.toys`. Then write `toy-list-item-teplate` and update
  `addToyToList`.
* `toy-detail-template`, `renderToyDetail`
    * Write the `select` tag! You'll want to pass `this.pokes` into
      the template so it can iterate through the pokemons and make an
      `option` for each.

## Phase 5: Views

Our `Pokedex.RootView` contains basically all the logic of our
application. We want to break it up into three main view classes:

* `Pokedex.View.PokemonIndex`
* `Pokedex.View.PokemonDetail`
* `Pokedex.View.ToyDetail`

We're going to slowly eliminate `RootView`. Let's start with the
`PokemonIndex`. Comment out the initialization code:

```javascript
// pokedex-0.js
/*
$(function() {
    var $rootEl = $('#pokedex');
    window.Pokedex.rootView = new Pokedex.RootView($rootEl);
    window.Pokedex.rootView.refreshPokemon();
});
*/
```

In the `PokemonIndex#initialize`, build an empty `Pokemon` collection
and save it to `this.collection`. Write a
`PokemonIndex#refreshPokemon` method that fetches the collection. Give
a success callback that calls the `PokemonIndex#render` method.

In the `PokemonIndex#render` method, first empty out the
`this.$el`. Then iterate through the collection, calling
`#addPokemonToList`. Your `#addPokemonToList` method should render the
`pokemonListItem` template, appending it to `this.$el`.

Last, uncomment the initialization logic at the bottom of
`pokedex-5.js`.

**You should be able to see the index of pokemon now. Call your TA
over to look over your code.**

**PokemonList Events**

We want to restore the functionality of being able to click on a
pokemon to display it. To do this, edit `events` so that it listens
for a `click` on a `li` in the view. Tell it to call the
`selectPokemonFromList` event handler.

To start out, let's write `selectPokemonFromList` to just find the
pokemon that was clicked on and print out its name. Check that this
works.

**PokemonDetail**

To actually display the details of the selected pokemon, let's
instantiate a `PokemonDetail` view in the `#selectPokemonFromList`
method. Pass in the selected pokemon at the selected pokemon as the
`model` parameter. Insert the `pokemonDetail.$el` into `$("#pokedex
.pokemon-detail")`. Last, call `render` on the `pokemonDetail` view.

We have to write the render method. Use your `pokemonDetail` template
to display just the pokemon details for now, not any toys. **Check
that this works**.

Next, we have to render the toys. Let's change our
`PokemonIndex#selectPokemonFromList` code so that instead of calling
`PokemonDetail#render` directly, it instead calls `#refreshPokemon`.

In the `#refreshPokemon`, fetch the pokemon to get its nested data,
calling `#render` in the success callback. Also, extend your `#render`
method to display the toys, by iterating through `model.toys()` and
using your `toysListItem` template.

You should now have be able to see a list of toys, but not click on
them yet.

**PokemonDetail Events**

Add a click handler for a click on `.toys li`. Use
`PokemonDetail#selectToyFromList` as a click handler. Instantiate a
`ToyDetail` view, inserting it into `$("#pokedex .toy-detail")` and
calling `ToyDetail#render`.

You'll have to write the `ToyDetail#render` method. When rendering the
`toyDetail` template, just pass an empty array for the `pokes`
parameter for now. We'll fix the dropdown to reassign the pokemon
later.

## Phase 6: Routing

**PokemonIndex, PokemonDetail**

In the previous section, when you click a pokemon, our `PokemonIndex`
view constructs a `PokemonDetail` view and inserts it into the
DOM. This is not great style; instead, the view should navigate to a
new URL, and a `Router` should construct the new view and insert it.

To begin, write a route in `pokedex-6.js` that maps the root URL to
the `pokemonIndex` method. Into this method, copy the initialization
code from `pokemon-5.js`. Comment out the old initialization code in
`pokemon-5.js`; we'll rely on the router now.

Next, we'll change `PokemonIndex#selectPokemonFromList`. Instead of
creating a view, use `Backbone.history.navigate` to move to a
`/pokemon/:id` URL. Next, write a second route in the router to show
construct/insert the `PokemonDetail` view. Your route function should
accept an `id` parameter. To start, just use `console.log` to verify
the route is invoked, and print out the `id` to check that the id is
set correctly. Remember to pass `{ trigger: true }` as an option to
`Backbone.history.navigate`.

Okay! Now we need to construct the `PokemonDetail` view. This mostly
involves moving the old code from `PokemonIndex#selectPokemonFromList`
to `Pokedex.Router#pokemonDetail`. The first step is finding the
proper `Pokemon` instance in `#pokemonDetail`. To do this, make sure
your `Router#pokemonIndex` method saves the `PokemonIndex` view in
`this._pokemonIndex`. Then you can access
`this.pokemonIndex.collection` in `#pokemonDetail`.

**Fixing Direct Navigation to PokemonDetail**

You'll notice that if you go directly to
`http://localhost:3000/#pokemon/1`, nothing will work. That's because
this will invoke `Router#pokemonDetail` when there is no
`this.pokemonIndex`. To solve this, in `#pokemonDetail`, check if there
is a `this._pokemonIndex` saved. If not, call `#pokemonIndex`.

This should get your Pokemon index rendering again, but you won't be
able to see your `PokemonDetail` still. That's because we have to wait
until after the `this._pokemonIndex` is fully fetched. To ensure this,
add a callback argument to `Router#pokemonIndex`, use this as a
success callback to `PokemonIndex#refreshPokemon`. You'll want to
modify your `PokemonIndex#refreshPokemon` to take a success callback
(it should render **and** call the callback).

Test this callback works properly by passing a simple success callback
to `Router#pokemonIndex`. Just have it print an alert after the index
is rendered.

When this is working, change your `Router#pokemonDetail` so that the
callback to the call to `Router#pokemonIndex` to **rerun** the
`pokemonDetail` method.

**ToyDetail**

Finally, let's write a `pokemon/:pokemonId/toys/:toyId` route. As
before, change `PokemonDetail#selectToyFromList` to use
`Backbone.history.navigate` (don't forget `{ trigger: true }`!).

Your `Router#toyDetail(pokemonId, toyId)` method should first get the
Toy from `this._pokemonDetail.model.toys()`. Therefore, have your
`Router#pokemonDetail` save a `_pokemonDetail` view.

Again, you'll have to deal with direct navigation to
`http://localhost:3000#pokemon/123/toy/456`. Use the same callback
trick with `Router#pokemonDetail` that you used on
`Router#pokemonIndex`.