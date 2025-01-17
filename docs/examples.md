## Creating Test Fixtures

### Basic usage

Here are some examples of how to use the fixture factories.

One article with a random title, as defined in the factory [on the previous page](factories.md):
```php
$article = ArticleFactory::make()->getEntity();
```
Two articles with different random titles:
```php
$articles = ArticleFactory::make(2)->getEntities();
```
One article with title set to 'Foo':
```php
$article = ArticleFactory::make(['title' => 'Foo'])->getEntity();
```
Three articles with the title set to 'Foo':
```php
$articles = ArticleFactory::make(['title' => 'Foo'], 3)->getEntities();
```
or
```php
$articles = ArticleFactory::make(3)->patchData(['title' => 'Foo'])->getEntities();
```
or
```php
$articles = ArticleFactory::make(3)->setField('title', 'Foo')->getEntities();
```
or
```php
$articles = ArticleFactory::make()->setField('title', 'Foo')->setTimes(3)->getEntities();
```
or
```php
$articles = ArticleFactory::make([
 ['title' => 'Foo'],
 ['title' => 'Bar'],
 ['title' => 'Baz'],
])->getEntities();
```

In order to persist the data generated, use the method `persist` instead of `getEntity` resp. `getEntities`:
```php
$articles = ArticleFactory::make(3)->persist();
```

### Using `FactoryAwareTrait`
All examples above are using static getter to fetch a factory instance. As convenience and kinda syntactic sugar, you can use the `FactoryAwareTrait::getFactory` instead.

`getFactory` is more tolerant on provided name, as you can use plurals or lowercased names. All arguments passed after factory name will be cast to `BaseFactory::make`.

```php
use App\Test\Factory\ArticleFactory;
use CakephpFixtureFactories\Factory\FactoryAwareTrait;

class MyTest extends TestCase
{
    use FactoryAwareTrait;

    public function myTest(): void
    {
        // Static getter style
        $article = ArticleFactory::make()->getEntity();
        $article = ArticleFactory::make(['title' => 'Foo'])->getEntity();
        $articles = ArticleFactory::make(3)->getEntities();
        $articles = ArticleFactory::make(['title' => 'Foo'], 3)->getEntities();

        // Exactly the same in FactoryAwareTrait style
        $article = $this->getFactory('Article')->getEntity();
        $article = $this->getFactory('Article', ['title' => 'Foo'])->getEntity();
        $articles = $this->getFactory('Article', 3)->getEntities();
        $articles = $this->getFactory('Article', ['title' => 'Foo'], 3)->getEntities();
    }
}
```

### Chaining methods
The aim of the test fixture factories is to bring business coherence in your test fixtures.
This can be simply achieved using the chainable methods of your factories. As long as those return `$this`, you may chain as much methods as you require.
In the following example, we make use of a method in the Article factory in order to easily create articles with a job title.
It is a simple study case, but this could be any pattern of your business logic.
```php
$articleFactory = ArticleFactory::make(['title' => 'Foo']);
$articleFoo1 = $articleFactory->persist();
$articleFoo2 = $articleFactory->persist();
$articleJobOffer = $articleFactory->setJobTitle()->persist();
```

 The two first articles have a title set two 'Foo'. The third one has a job title, which is randomly generated by fake, as defined in the
 `ArticleFactory`.

### Populating associations: the _with_ method
If you have baked your factories with the option `-m` or `--methods`, you will have noticed that a method for each association
has been inserted in the factories. This will assist you creating fixtures for the associated models. For example, we can
create an article with 10 authors as follows:
```php
use App\Test\Factory\ArticleFactory;
use App\Test\Factory\AuthorFactory;
use Faker\Generator;
...
 $article = ArticleFactory::make()->with('Authors', AuthorFactory::make(10))->persist();
```
or using the method defined in our `ArticleFactory`:
```php
$article = ArticleFactory::make()->withAuthors(10)->persist();
```

If we wish to randomly populate the field `biography` of the 10 authors of our article, with 10 different biographies:
```php
$article = ArticleFactory::make()->withAuthors(function(AuthorFactory $factory, Generator $faker) {
    return [
        'biography' => $faker->realText()
    ];
}, 10)->persist();
```
It is also possible to use the _dot_ notation to create associated fixtures:
```php
$article = ArticleFactory::make()->with('Authors.Address.City.Country', ['name' => 'Kenya'])->persist();
```
will create an article, with an author having itself an address in Kenya.

The second parameter of the method with can be:
* an array of field and their values
* an integer: the number
* a factory

Ultimately, the square bracket notation provides a mean to specify the number of associated
data created:
```php
$article = ArticleFactory::make(5)->with('Authors[3].Address.City.Country', ['name' => 'Kenya'])->persist();
```
will create 5 articles, having themselves each 3 different associated authors, all located in Kenya.

It is also possible to specify the fields of a toMany associated model.
For example, if we wish to create a random country with two cities having known names:

```php
$country = CountryFactory::make()->with('Cities', [
    ['name' => 'Nairobi'],
    ['name' => 'Mombasa'],
])->persist();
```

This can be useful if your business logic uses hard coded values, or constants.

Note that when an association has the same name as a virtual field,
the virtual field will overwrite the data prepared by the associated factory.

### Factory injection

When building associations, you may simply provide a factory as parameter. Example:

```php
$country = CountryFactory::make()->with('Cities',
  CityFactory::make()->threeCitiesAndFiveVillages()
)->persist();
```
will provide a country associated with three cities and five villages.

### Entity injection

You may also inject an exiting entity. The previous example would be now:
```php
$threeCitiesAndFiveVillages = CityFactory::make()->threeCitiesAndFiveVillages()->getEntities();
$country = CountryFactory::make()->with('Cities', $threeCitiesAndFiveVillages)->persist();
```

You may also pass an array of factories:
```php
$threeCitiesAndFiveVillages = CityFactory::make()->threeCitiesAndFiveVillages()->getEntities();
$country = CountryFactory::make()->with('Cities', [
    CityFactory::make()->threeCitiesAndFiveVillages(),
    CityFactory::make()->capitalCity()
])->persist();
```

### With a callable

In case a given field has not been specified with `faker` in the `setDefaultTemplate` method,  all the generated fields of a given factory
will be identical. The following
generates three articles with different random titles:
```php
use App\Test\Factory\ArticleFactory;
use Faker\Generator;
...
$articles = ArticleFactory::make(function(ArticleFactory $factory, Generator $faker) {
   return [
       'title' => $faker->text,
   ];
}, 3)->persist();
```
