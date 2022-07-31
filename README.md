# Find Merge 

Adds a `Pages::findMerge()` method that allows multiple PageFinder selectors to be merged into an efficient paginated set of results.

This can be useful when you need more sophisticated sorting of results than what would be possible using only the `sort` value in a single `$pages->find()`.

## Details

```php
$results = $pages->findMerge($selectors, $options);
```

`$selectors` is required and must be an array of selectors. Each selector can be in [string format](https://processwire.com/docs/selectors/) or [array format](https://processwire.com/blog/posts/processwire-3.0.13-selector-upgrades-and-new-form-builder-version/#selector-engine-array-support). The `findMerge()` method will loop over the selectors in the order supplied, adding matching pages to the final results.

`$options` is an optional associative array of options.
* `limit` (int) Limit for pagination.
* `start` (int) Manually override the start value rather than have it be automatically calculated from the current page number.
* `excludeExisting` (bool) Whether or not to exclude pages in each selector that have already been matched by a previous selector. Default is `true`.
* `keepFirst` (bool) When `excludeExisting` is false then a page might match more than one selector in the supplied array. But each page can only appear once in the results and if `keepFirst` is true then the page will appear in its earliest position in the results, whereas if `keepFirst` is false it will appear in its latest position in the results. Default is `true`.

As a shortcut you can supply an integer as the second argument and it will be treated as the limit for pagination.

## Basic usage

For most use cases only a limit will be needed for the `$options` argument.

```php
$selectors = [
    'title%=yellow', // title contains "yellow"
    'title^=z', // title starts with "z"
    'title=elephant', // title equals "elephant"
    'template=colour, sort=-title, limit=3', // 3 colours in reverse alphabetical order
    'template=country, sort=title, limit=40', // 40 countries in alphabetical order
];
$results = $pages->findMerge($selectors, 10);
if($results->count) {
    echo "<p>Showing results {$results->getPaginationString()}</p>";
    echo "<ul>";
    foreach($results as $result) {
        echo "<li><a href='$result->url'>$result->title</a></li>";
    }
    echo "</ul>";
    echo $results->renderPager();
}
```

![2022-07-31_160717](https://user-images.githubusercontent.com/1538852/182009602-4947bf33-8a0f-435d-8b55-dc4fc59fc9e2.png)

## Advanced usage

*The following notes are only relevant to rare cases and most users can safely skip this section.*

In the demo example the `colour` page Yellow will potentially match both the 1st selector and the 4th selector. Because of this the `excludeExisting` and `keepFirst` options will have an effect on the results.

### excludeExisting option false

Note that the 4th selector asks for 3 colour pages (limit=3).

By default `excludeExisting` is true, which means that when the 4th selector is processed it is interpreted as saying "find 3 colour pages in reverse alphabetical order *that have not already been matched in an earlier selector*". We can see that there are 3 pages in the results from that selector: Violet, Red, Orange.

But if `excludeExisting` is set to false then the results are different. The matches of the 1st selector (Yellow, Yellow Warbler) are not excluded from consideration by the 4th selector (the 4th selector matches will be Yellow, Violet, Red), and because each page can only appear once in the results this means that the 4th selector ends up only adding 2 more pages to the results.

```php
$selectors = [
    'title%=yellow', // title contains "yellow"
    'title^=z', // title starts with "z"
    'title=elephant', // title equals "elephant"
    'template=colour, sort=-title, limit=3', // 3 colours in reverse alphabetical order
    'template=country, sort=title, limit=40', // 40 countries in alphabetical order
];
$options = [
    'limit' => 10,
    'excludeExisting' => false,
];
$results = $pages->findMerge($selectors, $options);
```
![2022-07-31_162704](https://user-images.githubusercontent.com/1538852/182010119-c0c8b20b-a9c9-4b88-8904-7d56436d7218.png)

### keepFirst option false

As described above, the Yellow page potentially matches both the 1st and 4th selector. By default Yellow will appear in its earliest position within the results, i.e. the position resulting from it being matched by the 1st selector.

But if `keepFirst` is set to false (and `excludeExisting` is false) then it will appear in its latest position within the results, i.e. the position resulting from it being matched by the 4th selector.

```php
$selectors = [
    'title%=yellow', // title contains "yellow"
    'title^=z', // title starts with "z"
    'title=elephant', // title equals "elephant"
    'template=colour, sort=-title, limit=3', // 3 colours in reverse alphabetical order
    'template=country, sort=title, limit=40', // 40 countries in alphabetical order
];
$options = [
    'limit' => 10,
    'excludeExisting' => false,
    'keepFirst' => false,
];
$results = $pages->findMerge($selectors, $options);
```
![2022-07-31_164436](https://user-images.githubusercontent.com/1538852/182010560-a45ad25c-879d-4dae-bd07-519addea2887.png)

`keepFirst` has no effect when `excludeExisting` is true.