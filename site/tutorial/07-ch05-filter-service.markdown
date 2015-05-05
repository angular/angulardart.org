---
layout: tutorial
title: Introducing Formatters and Services
previous: 06-ch04-directive.html
next: 08-ch06-view.html
---

# {{page.title}}


<p>In this chapter, we will add two features to our app. First, we will
get the recipe data from a service instead of hardcoding mock data into
the app. Second, we will look at another handy tool in Angular’s toolbox:
formatters. When you’re finished, you will be able to use built-in
 formatters as well as create your own custom formatters. You will also be
 able to read data from a server-side service like a real app.</p>

<hr />

<h3 id="running-the-sample-app">Running the sample app</h3>
<p>The code for this chapter is in the
<em><a href="https://github.com/angular/angular.dart.tutorial/tree/master/Chapter_05">
    Chapter_05</a></em> directory of the
    <a href="https://github.com/angular/angular.dart.tutorial/archive/master.zip">
    angular.dart.tutorial download</a>.
View it in Dart Editor by using
<strong>File &gt; Open Existing Folder...</strong> to open the
<strong>Chapter_05</strong> directory.</p>

<p>Now run the app. In Dart Editor’s Files view, select
<strong>Chapter_05/web/index.html</strong>, right-click, and choose
<strong>Run in Dartium</strong>.</p>

<p>Notice the text box and checkboxes at the top of the app. You can use
these inputs to control which recipes are shown in the list. Start
typing into the text box and watch the list of visible recipes shrink
to match your search criteria. Next, limit your results further by
checking a few categories to limit to. Try using both filters together
by typing in a search criteria and also limiting based on category.
Lastly, clear the filters by pressing the “Clear Filters” button, and
watch all the recipes reappear.</p>

<hr class="spacer" />

<h3 id="introducing-formatters">Introducing formatters</h3>
<p>Formatters let you change how your model data is displayed in the view
without changing the model data itself. They do things like allow you to
show parts of the model data, or display data in a particular format.
You can also use Angular’s custom formatters feature to create your own
formatters to do anything you want.</p>

<hr class="spacer" />

<h4 id="built-in-formatters">Built-in formatters</h4>
<p>Angular has some built-in formatters that provide handy functionality.</p>

<p>These are pretty self explanatory. <a href="https://docs.angulardart.org/#angular/angular-formatter.Currency"><code>Currency</code></a>
formats numbers like money. <a href="https://docs.angulardart.org/#angular/angular-formatter.Date"><code>Date</code></a> formats dates.
<a href="https://docs.angulardart.org/#angular/angular-formatter.Uppercase"><code>Uppercase</code></a> and <a href="https://docs.angulardart.org/#angular/angular-formatter.Lowercase"><code>Lowercase</code></a> do
what you would expect. <a href="https://docs.angulardart.org/#angular/angular-formatter.LimitTo"><code>LimitTo</code></a> limits the number
of results for a list model object that will appear in the view.</p>

<p>In our app, we use the
<a href="https://docs.angulardart.org/#angular/angular-formatter.Filter">
<code>Filter</code></a> class. Like the <code>LimitTo</code> formatter,
<code>Filter</code> also limits the number of list model
objects that will appear in the view. It chooses which items to display
based on whether they satisfy the filter criteria.</p>

<p>Here is how we use the <code>Filter</code> object.</p>

<p>First, we create a text input box and bind it through
<code>ng-model</code> to a property called <code>nameFilterString</code>.
As the user types into the text input box, it updates the model object’s
<code>nameFilterString</code> property.</p>

<script type="template/code">
<input id="name-formatter" type="text" ng-model="nameFilterString">
</script>

<p>Next, we pipe the <code>ng-repeat</code> criteria through the formatter,
and tell the Filter formatter (registered as <code>filter</code>)
to use <code>nameFilterString</code> as the
property to filter against.</p>

<script type="template/code">
<ul>
    <li class="pointer"
        ng-repeat="recipe in recipes | filter:{name:nameFilterString}">
    ...
    </li>
</ul>
</script>

<p>Lastly, we create a property on our <code>RecipeBookComponent</code>
to store the <code>nameFilterString</code> property for the input.</p>

<script type="template/code">
String nameFilterString = "";
</script>

<p>That’s all you need to start filtering your results.</p>

<hr class="spacer" />

<h4 id="custom-formatter">Custom formatters</h4>
<p>Built-in formatters are nice, but what if you want to do something more
specific. In our app, we also want to be able to reduce the number of
recipes shown, to those in a particular category. For this, we write a
custom formatter called <code>CategoryFilter</code>.</p>

<p>A custom formatter in Angular is simply a Dart class that declares a
<code>call</code> method with at least one argument:</p>

<script type="template/code">
class MyFormatter {
  call(valueToFormat, optArg1, ..., optArgN) { ... }
}
</script>

<p>The first argument is the incoming model object to be formatted; in our
case, it will be the recipe list. The remaining (0 or more) optional
arguments (named <code>optArg</code> above)  serve as input from the
view, and they will be applied in some way to the
<code>valueToFormat</code> to perform the formatting. In our case, as
will be explained below, we need only one optional parameter to let us
weed out recipes from the incoming list that we don’t want to
display.</p>

<p>The <code>call</code> method should return the formatted value. In our
case, it is a new recipe list that is a subset of the incoming recipe
list.</p>

<p>Here is our call method from <code>CategoryFilter</code>:</p>

<script type="template/code">
  List call(recipeList, filterMap) {
    if (recipeList is Iterable && filterMap is Map) {
      // If there is nothing checked, treat it as "everything is checked"
      bool nothingChecked = filterMap.values.every((isChecked) => !isChecked);
      return nothingChecked
          ? recipeList.toList()
          : recipeList.where((i) => filterMap[i.category] == true).toList();
    }
    return const [];
  }
</script>

<p>The <code>filterMap</code> parameter deserves some further explanation.
It’s the data that comes in off of the checkbox inputs. We will talk a
little bit more about how checkboxes work in Angular in the next
section.</p>

<p>Next, annotate the class to publish it as a formatter:

<script type="template/code">
@Formatter(name: 'categoryfilter')
class CategoryFilter { ... }
</script>

<p>Add the new class to the bootstrapping code:</p>

<script type="template/code">
class MyAppModule extends Module {
  MyAppModule() {
    ...
    bind(CategoryFilter)
    ...
  }
}
</script>

<p>and then use it from the view:</p>

<script type="template/code">
<div>
    <label>Filter recipes by category
        <span ng-repeat="category in categories">
          <label>
            <input type="checkbox"
                   ng-model="categoryFilterMap[category]">{% raw %}{{category}}{% endraw %}
          </label>
        </span>
    </label>
</div>
</script>

<p>Our view creates a checkbox input and label for each category. Angular
stores each checkbox value as a boolean - true if checked, or false if
not checked.</p>

<p>To create the checkboxes, we added a new piece of data to our app - a
list of categories. We use the <code>ng-repeat</code> directive to
iterate through the list and create a checkbox and label for each
category. Inputs in Angular are bound to a model object with the
<code>ng-model</code> directive. Here, we bind to a map called
<code>categoryFilterMap</code>. The map’s keys are the category names,
and the values are true or false depending on whether or not they’re
checked.</p>

<p>Next, we plug the custom formatter in the same way we would plug in a built-in formatter:</p>

<script type="template/code">
<li class="pointer" ng-repeat="recipe in recipes | categoryfilter:categoryFilterMap">
</script>

<hr class="spacer" />
<h4 id="formatter-chaining">Formatter chaining</h4>
<p>Our app uses a feature called formatter chaining to apply more than one
formatter to the same view element. Below, we see the
<code>ng-repeat</code> directive has three formatters separated by pipes.
This is how Angular applies multiple formatters to a single
<code>ng-repeat</code>.</p>

<script type="template/code">
<li class="pointer"
    ng-repeat="recipe in recipes | orderBy:'name' | filter:{name:nameFilterString} | categoryfilter:categoryFilterMap">
</script>


<hr class="spacer" />
<h3 id="introducing-the-http-service">
Introducing the Http service</h3>

<p>Our last example had the data hard coded in the app. In reality, you’d
request data from a server. Angular provides a built-in
service called the
<code><a href="https://docs.angulardart.org/#angular/angular-core.Http">Http</a></code>
Service that handles making HTTP
requests to the server. First let’s look at the two new files we’ve
added to the project: <strong>recipes.json</strong> and
<strong>categories.json</strong>. These contain data that’s been
serialized as a JSON string. We will use the <code>Http</code> service
to make an HTTP request to the web server to fetch this data. Let’s look
at how to do this.</p>

<p>First, we declare a property on the <code>RecipeBookComponent</code>
class. Ours is called <code>_http</code>. The <code>Http</code> service
is part of the core Angular package, so you don’t need to import
anything new. Next, look at the <code>RecipeBookComponent</code>’s
constructor. We’ve added a parameter and assigned it to the
<code>_http</code> property. Angular instantiates the
<code>RecipeBookComponent</code> class using
<a href="http://en.wikipedia.org/wiki/Dependency_injection">
  Dependency Injection</a>. In the main method, you set up the injector
with your app’s module where you included the code to construct a
<code>RecipeBookComponent</code>. The call to
<code>addModule()</code> includes the <code>AngularModule</code>,
which contains injection rules for all of Angular’s core features,
including the <code>Http</code> service.</p>

<script type="template/code">
class RecipeBookComponent {
  final Http _http;
  ...
  RecipeBookComponent(this._http) {
    _loadData();
  }
  ...
}
</script>

<script type="template/code">
class MyAppModule extends Module {
  MyAppModule() {
    ...
    bind(RecipeBookComponent);
    ...
  }
}

void main() {
  applicationFactory()
      .addModule(new MyAppModule())
      .run();
}
</script>

<p>Next, let’s look at how to use the
Http service to fetch
data from the server. Look at the changes we made to the
<code>_loadData()</code> method. Here is the new code:</p>

<script type="template/code">
Future _loadData() async {
  recipesLoaded = false;
  ...
  try {
    var response = await _http.get('recipes.json');

    recipes = response.data.map((d) => new Recipe.fromJson(d)).toList();
    recipesLoaded = true;
  } on Error catch (e) {
    print(e);
    recipesLoaded = false;
    message = ERROR_MESSAGE;
  }
  ...
}
</script>

<p>Using HTTP to fetch data from a server introduces a problem that's
familiar in web-app development: if the HTTP connection is slow, the
app could hang until the data is returned.</p>

<p>To avoid this problem, the <code>_loadData()</code> method declares
itself to be asynchronous by using the <code>async</code> modifier
and immediately returning a 
<a href="http://api.dartlang.org/docs/releases/latest/dart_async/Future.html">
Dart Future</a>. A <code>Future</code> is the promise of a value sometime
in the future. It is at the core of
<a href="https://www.dartlang.org/docs/dart-up-and-running/contents/ch03.html#ch03-asynchronous-programming">
asynchronous programming in Dart</a>. In its simplest form, an asynchronous
method contains at least one <code>await</code> expression, which pauses execution
of all subsequent expressions until the <code>await</code> expression completes.</p>

<p>When the <code>_loadData()</code> method is called, it immediately returns
a <code>Future</code>. Then it sets <code>recipesLoaded</code> to false. The next
expression is an <code>await</code> expression, so the method pauses until the
HTTP response is returned and successfully processed. Once the HTTP response is
returned, the method loads the response into a <code>List&lt;Recipe&gt;</code>,
which is published in the model as <code>recipes</code>:</p>

<script type="template/code">
List<Recipe> recipes = [];
</script>

<p>Then, <code>loadData()</code> sets  <code>recipesLoaded</code> to true. Because
<code>recipesLoaded</code> is available in the model, the component's template can
conditionally display either a “Loading...” message or the
whole app view. While we could use <code>ng-if</code> to implement the
conditional display logic, let&rsquo;s use an <code>ng-switch</code>
directive instead.</p>

<script type="template/code">
  <div ng-switch="recipesLoaded && ctrl.categoriesLoaded">
    <div ng-switch-when="false">
      {% raw %}{{message}}{% endraw %}
    </div>
    <div ng-switch-when="true">
      <h3>Recipe List</h3>
      ...
    </div>
  </div>
</body>
</script>

<p>Note that we could replace one of the <code>ng-switch-when</code>
directives above by <code>ng-switch-default</code>. Also, loosely speaking,
expressions of most types are suitable as arguments to
an <code>ng-switch</code>, as long as switch case values can be assigned
to <code>ng-switch-when</code> attributes. For more
details, see the
<a href="https://docs.angulardart.org/#angular/angular-directive.NgSwitch">
<code>NgSwitchDirective</code></a> API documentation.</p>

<p>You can see the “Loading...” feature work by simulating a really slow
loading data source. Put a breakpoint at the await expression and load
the app. Notice that while you’re stopped at the breakpoint, your
app shows the  “Loading...” message. Now get out of the breakpoint and
notice that your app’s real view pops into place. </p>

<hr class="spacer" />

<h3 id="angular-features">Angular features</h3>
<h4><a href="https://docs.angulardart.org/#angular/angular-directive.NgCloak">
<code>ng-cloak</code></a></h4>
<p>You probably noticed that in previous examples, when you first loaded
your app, you briefly saw curly braces like <code>{% raw %}{{someVar}}{% endraw %}</code>
before your app “popped” into place, and the correct values appeared.
The <code>ng-cloak</code> directive combined with some CSS rules that
you add to your app’s main CSS file allow you to avoid this blink. The
blink happens between the time your HTML loads and Angular is
bootstrapped and has compiled the DOM and has substituted in the real
values for the uncompiled DOM values. While Angular is compiling the
DOM, you can hide the page (or sections of it) by using
<code>ng-cloak</code>:</p>

<script type="template/code">
<body class="ng-cloak">
</script>

<p>or</p>

<script type="template/code">
<body ng-cloak>
</script>

<p>The CSS rule causes the view to be hidden:</p>

<script type="template/code">
[ng-cloak], .ng-cloak {
   display: none !important;
}
</script>

<p>A set of basic CSS rules are provided as part of Angular.Dart in the
<a href="https://github.com/angular/angular.dart/blob/master/lib/css/angular.css">angular.dart/lib/css/angular.css</a>
file</p>.

<p>Once Angular is finished compiling the DOM, it secretly removes the
<code>ng-cloak</code> class or directive from the DOM and allows your
app to be visible.</p>

<p>The directive can be applied to the <code>&lt;body&gt;</code> element,
but the preferred usage is to apply multiple <code>ngCloak</code>
directives to small portions of the page to permit progressive rendering
of the browser view.</p>

<hr class="spacer" />

<h3 id="explore-more">Try it yourself</h3>
<p>Now it’s your turn. Here are a few ways in which you can extend the app
version described in this chapter.</p>

<ol>
<li>Write a simple <code>String</code> formatter to replace all the
  occurrences of “sugar” and “powdered sugar” by “maple syrup”, and
  apply it to both the ingredients and the recipe directions.
  <strong>Hint</strong>: Dart has a <code>String</code> function named
  <code>replaceAll()</code> that could be handy in this case.</li>
<li>Write a formatter to convert <em>degrees Fahrenheit</em> (F) into
  <em>degrees Celsius</em> (C) in the recipe directions. Round the
  resulting degrees Celsius to the nearest multiple of 5: e.g., 300
  degrees F would become  150 C. <strong>Hint</strong>: Dart
  <a href="https://api.dartlang.org/apidocs/channels/stable/#dart:core.RegExp">
    regular expressions</a> are likely to be useful.</li>
<li>Assume that you are cooking for a large party.
  Create a formatter that will multiply all the amounts in the recipes.
  <strong>Hint:</strong> You’ll have to change the recipe model to
  include a more full featured “Ingredient” field that contains an
  amount, and then possibly a unit, and finally, an item name. Next, add
  a “multiplier” input that will allow the user to double, triple, or
  quadruple the recipe. Lastly, write a custom formatter that multiplies
  each amount by the number specified in the multiplier input.
<li>Finally, if you have not done so already, adapt your solutions to
  the first two problems above so that application of each formatter is
  controlled by a checkbox.</li>
</ol>
