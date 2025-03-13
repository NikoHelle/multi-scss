# Multi-sass

This library was developed to publish CSS rules dynamically for multiple purposes. The outputted CSS needs to be partially exportable with or without selectors or with renamed selectors.

By default, the `multi-sass` output is the whole CSS because default arguments enable output for all selectors. This library splits (S)CSS into partials whose output is controlled with arguments. This way each partial (called entities) can be allowed or disallowed from the output.

The library is made for BEM methodology, and its arguments are named accordingly. It can be used for other purposes by just applying different configurations. Some special cases, like themes, variants, and media queries, require more fine-tuning, so there are also arguments unrelated to BEM.

The BEM methodology has three entities: `blocks`, `modifiers` and `elements` which are used in `multi-sass`.

[Read more about the BEM methodology](https://en.bem.info/methodology/).

## Dependencies

The [https://github.com/sass/dart-sass](sass) is required.

## Testing

All mixins, functions, and outputs are tested with Jest and [sass-true](https://www.npmjs.com/package/sass-true).

## Roadmap

The next release (1.0.0) will change how the configuration works. Using functions instead of multiple settings will make the library easier to use. Customising the selectors will be easier when all selectors are constructed with functions instead of string concatenation.

## Examples

### The source scss

```css
@use "@nikohelle/multi-sass/index.scss" as *;

@mixin my-button-component($args...) {
  @include multi-sass($block: "my-button", $args...) {
    @include block {
      // generic CSS for the block

      @include element("icon") {
        // CSS for the icon element
      }

      @include modifier("dark") {
        // CSS for the dark variant
        @include element("icon") {
          // CSS for the dark variant of the icon element
        }
      }
    }
  }
}
```

### Basic usage

Get all CSS rules and selectors:

```css
@include my-button-component();
```

The result:

```css
.my-button {
  // generic CSS for the block
}
.my-button__icon {
  // CSS for the icon element
}
.my-button--dark {
  // CSS for the dark variant
}
.my-button--dark__icon {
  // CSS for the dark variant of the icon element
}
```

### Advanced usage

Get only the core CSS rules without modifiers:

```css
@include my-button-component($modifiers: false);
```

The result:

```css
.my-button {
  // generic  CSS for the block
}
.my-button__icon {
  // CSS for the icon element
}
```

### Custom usage

Override the default config and discard the CSS of the icon of the dark variant.

```css
@include my-button-component(
    $modifiers: ("icon":false),
    $config:(
      "blockPrefix:"product-name-",
      "modifierDelimeter":".",
      "elementDelimeter":"-"
    )
);
```

The result

```css
.product-name-my-button {
  // generic  CSS for the block
}
.product-name-my-button-icon {
  // CSS for the icon element
}
.product-name-my-button.dark {
  // CSS for the dark variant
}
```

### Partial usage with renaming

It is possible to just pick the CSS content without selectors. the `$config.emitContentOnly` effect that the contents have when they are outputted.

Add a new selector and output the CSS contents only from the block and "dark" modifier to the same selector.

```css
.new-selector {
  @include my-button-component(
    $modifiers: (
      "dark": true
    ),
    $elements: false,
    $config: (
      emitContentOnly: "all"
    )
  );
}
```

The result:

```css
.new-selector {
  // generic  CSS for the block (cannot be discarded in this example)
  // CSS for the dark variant
}
```

The block's CSS can be discarded by wrapping it with [base content](#important-content-named-base).

See also selector renaming with [selector middleware](#selector-middleware).

## Arguments and configuration

Each mixin using the `multi-sass` should start and be wrapped with

```css
@include multi-sass($args...) {
  // other includes and CSS here
}
```

The `multi-sass `ensures components and mixins can be nested and different configurations can be used in each mixin. Even when nesting the same components within themselves. The `multi-sass` builds a hierarchy map of the nested entities so each entity knows its parents and can combine any preceding selectors with child selectors.

For example, the `block-element` finds the closest block entity and creates a selector by merging the block's selector with the given element.

```css
@include block-element("element-name") {
  //css
}

// results in <block selector><element delimeter><element selector>:

.block--element-name
  // CSS
);
```

### Arguments

Because the `multi-sass` was made for conditional CSS output, its arguments must control the output. Entities and their output are controlled with `rules`. Rules are basically `key:value` pairs with boolean or string values indicating whether an entity should be outputted or not.

In addition to rules, another argument is the `$config`. It does not control the output but defines selector prefixes and delimiters and where the CSS is outputted. If config is not passed, defaults are used. Detailed information in the [config section](#the-config-argument).

Note: `multi-sass` uses named arguments, so every argument has a `$` prefix. When named arguments are used, the order of arguments does not matter.

### The "$block" argument

Defines the selector for all blocks. The outputted selector is "." + the given block. No default. All blocks have the same selector. Usually there is only one block in one component when BEM is used.

Note: Blocks are always outputted if `@include block` used. Rules cannot control blocks. You can control the outputted CSS with [extras](#the-extras-argument) and [content](#content).

### The "$modifiers" argument

The `$modifiers` can be true/false, a string, or a map.

- Boolean value `true` is the default and rules that the CSS of all modifiers should be outputted.
- Boolean value `false` rules that none of the modifiers should be outputted.
- String value, like "dark," rules that only the modifier named "dark" should be outputted.
- Map of `string:boolean` pairs names each modifier, and rules should the named modifier be outputted or not.

Detailed information in the [rules section](#rules).

### The "$elements" argument

The `$elements` can be true/false, string, or a map. Behaves just like the `$modifiers` rule.

- Boolean value `true` is the default and rules that the CSS of all elements should be outputted.
- Boolean value `false` rules that none of the elements should be outputted.
- String value, like "dark," rules that only the element named "dark" should be outputted.
- Map of `string:boolean` pairs names each element, and rules should the named element be outputted or not.

Detailed information in the [rules section](#rules).

Note: The [content](#content) also uses element rules for output. There is reserved `base` name for content. Read more in the detailed section for [base](#important-content-named-base).

### The "$extras" argument

In addition to entities, `multi-sass` has an additional `extra` mixin. All entities, including CSS in it, are conditionally outputted. If an `extra` is allowed in rules, the contents are outputted. Otherwise, contents are ignored.

```css
@include extra("mobile") {
  // CSS for mobile
}

@include extra("desktop") {
  // CSS for desktop
}

@include extra("multi-select") {
  // CSS for select with multi select
}
```

The `$extras` can be true/false or string/boolean map.

- Boolean value `true` is the default and rules that the CSS of all extras should be outputted.
- Boolean value `false` rules that none of the extras should be outputted.
- Map of `string:boolean` pairs names each extra and rules should the named extra be outputted or not.

Detailed information in the [rules section](#rules).

### The "$config" argument

The `config` is a map and has the following properties and values.

- `blockPrefix` optional prefix for all blocks. Outputted selector is "." + blockPrefix + blockName. No default.
- `modifierDelimeter` delimeter between the parent selector and modifier selector. The outputted selector is parent selector + modifierDelimeter + modifier name. Default is `"--"`.
- `elementDelimeter` delimeter between the parent selector and element selector. The outputted selector is parent selector + elementDelimeter + element name. Default is `"__"`.
- `info` can be used for storing any data. Useful with [selector middleware](sSelector-middleware) and [renaming selectors with functions](#how-to-pass-information-to-the-middleware).
- `emitContentOnly`. Detailed information [below](#emitContentOnly).

#### $config.emitContentOnly

Sometimes the output only contains CSS without selectors. For example, if the output should be placed inside a custom selector. The `emitContentOnly` can have three values: null, `root` or `all`. Default is null.

For example, using the root value forwards the output of the root (usually a block entity) directly to the parent selector. The block selector is not outputted.

```css
.wrapper {
  @include myMultiSassComponent {
  }
}
```

could result in

```css
.wrapper .block {
  // block CSS
}
.wrapper .block--modifier {
  // modifier CSS
}
```

But with the 'root' value

```css
.wrapper {
  @include myMultiSassComponent(
    $config: (
      "emitContentOnly": "root"
    )
  );
}
```

results in

```css
.wrapper {
  // block CSS
}
.wrapper--modifier {
  // modifier CSS
}
```

And with the `all` value, all CSS is outputted to the nearest entity. This is useful when outputting a deeply nested CSS into a wrapper selector.

```css
.wrapper {
  @include myMultiSassComponent(
    $config: (
      "emitContentOnly": "all"
    )
  );
}
```

results in

```css
.wrapper {
  // block CSS
  // modifier CSS
}
```

The `emitContentOnly` enables possibility to wrap the CSS to a custom selector. When used with [rules](#rules) any part of the CSS could be picked in any selector.

## Mixins

The `_index.scss` exposes the following mixins.

### Block

The root entity in BEM. The outputted selector is defined in the [$block argument](#the-block-argument). The `block` mixin does not accept arguments.

Block is defined is the scss as

```css
@include block {
  // scss and/or CSS
}
```

### Modifier

Modifiers are outputted as `<parent selector>--<modifier name>`. The "--" prefix is automatically added. It can be overridden with `modifierDelimeter` in the [config](#the-config-argument).

A modifier is defined is the CSS as

```css
@include modifier(name-and-selector) {
  // scss and/or CSS
}
```

### Element

Elements are outputted as `<parent selector>__<element name>`. The "--" prefix is automatically added. It can be overridden with `elementDelimeter` in the [config](#the-config-argument).

A element is defined is the CSS as

```css
@include element(name-and-selector) {
  // scss and/or CSS
}
```

### Extra

Extra is used for outputting CSS conditionally and do not create selectors. The wrapped content is outputted if the named extra is allowed in rules.

```css
@include extra(name-matching-rules) {
  // content is outputted only if rules allow the given name.
}
```

Detailed information in the [$extra argument](#the-extras-argument).

### Custom

Custom is used for outputting any selector that is not any of the entities. Custom is outputted as `<parent selector><custom selector>`.

```css
@include custom(selector) {
  // scss and/or CSS
}
```

### Content

The "content" is a mixin used to mark CSS that is just CSS content, no selectors are created. Using the `@include content(<name>)` is useful when a CSS should not be outputted every time.

There is no `content` argument to keep arguments simpler. Content uses element names when checking is it allowed or not. The given `name` is compared to `$elements` rules.

Note that same logic can be achieved with `@include extra(<name>)`. `Content` is just a clearer way to indicate it is not an special case, it is just content.

#### Important content named "base"

Each selector has its so called "base" (aka. main/root/default) rules, the CSS right after the selector:

```css
// scss
@include block {
  // base CSS
}

// output
.block {
  // base CSS
}
```

There must a way to exclude that from the output. If the rules are just written there, they are always outputted. To control the output of base rules, `@include content("base")` can be used. There is no automatic way to pick base rules, so the CSS must be wrapped with an "@include".

`multi-sass` has a special and reserved element name for that: "base". If used manually, the scss would be:

```css
@include block {
  @include content("base") {
  }
}
```

There is a special shorthand `@include` for this.

```css
@include block {
  @include if-content-allowed {
  }
}
```

Note that the `if-content-allowed` can be used inside any entity, not just `block`.

**The base is affected by the same rules as all entities. It is not outputted if others are explicitly allowed.**

Detailed information in the [rules section](#rules).

Instead of writing base, a [constant can be used](#constants) or the [if-content-allowed](#if-content-allowed) mixin.

## Other exported mixins and includes

There are many [helper mixins](#helper-mixins-and-functions) for automatic selector linking, repetition and even creating selectors with custom functions.

## Other exported functions

### get-entity-name($entity)

Returns the name of the entity.

### is-block-entity($entity)

Returns true if given entity is a block.

### is-modifier-entity($entity)

Returns true if given entity is a modifier.

### is-element-entity($entity)

Returns true if given entity is an element.

### is-custom-entity($entity)

Returns true if given entity is a custom.

### get-entity-parent($entity)

Returns the parent of an entity or null if the entity is the root.

### find-closest-entity($startEntity, $type, $name:null)

Finds the closest entity in parent entities with given type and name. Name is optional. If name is not give, olnly types are compared.

### get-config-info-value($name)

Returns the named value from `config.info` or null if the `config.info[name]` does not exist.

### alias-middleware($entity, $config, $selectorList)

See [selector middleware](#selector-middleware).

### replace-selector-list($selectorList, $index, $new-item)

Replaces an item in the selector list passed to the `alias-middleware`.Returns new list. Note that in sass, indexes start from 1.

See [selector middleware](#selector-middleware).

### get-selector-by-index

Returns an item from the selector list passed to the `alias-middleware`. Note that in sass, indexes start from 1.

See [selector middleware](#selector-middleware).

## Selector middleware

The `$config.selectorMiddleware` is for customising selectors in the CSS output. The value is a function with the call signature: `func($entity, $config, $selectorList)` where:

- `$entity` is the currently processed entity the selector is created for. The entity has `type`, `name`, `parent`, `selector`, and other props for internal use.
- `$config` contains configuration props passed to the `multi-sass` function. It is a combination of passed and default props. Note that each nested `multi-sass` process has its own configuration.
- `$selectorList` list of strings that will form the selector. The list is usually (prefix or delimeter, entity name). For example, with modifiers, it is `"--", "modifier name"`. It is easier to parse and replace a list than a string. You can replace all delimeters by replacing the first item in the list. See [replace-selector-list](#replace-selector-list).

The middleware is called everytime an entity is created and a selector for it. If the entity is content only or is disallowed in the rules, the selector is not created.

The middleware function can return a list or string. A list is converted to a string with all falsy values discarded. The `$selectorList` should be returned if there is no need to modify the selector.

```css
@use "@nikohelle/multi-sass/index" as multisass;

@function my-rename-middleware($entity, $config, $selectorList) {
  $type: multisass.get-entity-type($level);
  $name: multisass.get-entity-name($level);
  @if multisass.is-block-entity($entity) {
    @return "#id.class";
  }
  @if multisass.is-modifier-type($type) {
    @return ".cool-#{$name}";
  }
  @if multisass.is-element-type($type) {
    @return " .element-#{$name}";
  }
  @return $selectorList;
}

@mixin my-button-component($args...) {
  @include multisass.multi-sass($block: "my-button", $args...) {
    @include modifier("mod1") {
      // CSS for the modifier
      @include element("elem1") {
        // CSS for the element
      }
    }
  }
}

// regular output

.my-button--mod1 {
  // CSS for the modifier
}
.my-button--mod1__elem1 {
  // CSS for the element
}

// add selector middleware

@include my-button-component(
  $config: (
    selectorMiddleware: meta.get-function("my-rename-middleware"),
  )
);

// output

.#id.class.cool-mod1 {
  // CSS for the modifier
}
.#id.class.cool-mod1 .element-elem1 {
  // CSS for the element
}
```

### How to pass information to the middleware

If a `multi-sass` component is used in multiple places and the middleware should make changes only in some instances, how to know? The [config.info](#config.info) can contain any data needed for middleware. The `info` can be fetched from `multi-sass` with [get-config-info](#get-config-info). It returns whatever was passed to `multi-sass` or `null`.

### Automatic renaming with config.info.alias and migration middleware

The previous version (0.9) of the `multi-sass` had automatic selector renaming with the `$alias` property. The property was a map of `entity-name:new-selector` and if an entity's name was found, the selector was used. The same process can be used with [config.info](#config.info) and selector middleware. The `alias` can be placed in the `config.info` and use the exported migration function:

```css
@use "@nikohelle/multi-sass/" as multisass;

$config: (
  info: (
    alias: (
      entityName: ".new-selector",
      entityX: "#id-block"
    )
  )
  selectorMiddleware: meta.get-function("alias-middleware", false, "multisass")
);
```

# Rules

The arguments affecting the output are called rules. In some cases you only want parts of the CSS, not everything. These rules control the outputted output. The output of a modifier, element or extra can be either allowed or disallowed. By default, all outputs is allowed.

`$modifiers` define the rules for the outputted CSS of all modifiers.
`$elements` define the rules for the outputted CSS of all elements.
`$extras` define the rules for other outputted CSS, like media queries and special cases.

Note that `block` entities cannot be controlled with rules. The outputted CSS can be controlled, if the CSS are wrapped with `content` or `extra` mixins.

## Rule logic

If the value is a boolean, "true" allows everything and "false" disallows.

If the value is a string, it names the allowed entity. All other ones are disallowed and not outputted.

If the value is a map, it has string/boolean pairs that names explicitly what modifiers/elements are allowed and outputted.

**Very important rule to remember: if one named entity is allowed, all other ones are disallowed!**

### The "$modifiers" rule

Defines which modifers are outputted. By default all modifiers are outputted.

The value can be a boolean, string or map. The map can be a nested map. Nested map is considered to contain rules for elements inside the modifier.

Boolean true/false allows/disallows all modifiers. Default is

```css
$modifiers: true;
```

If value is a string, only that modifier is outputted.

```css
$modifiers: "name of the allowed modifier";
```

If value is a map, modifiers are outputted according to the values.

```css
$modifiers: (
  "modifierA": false,
  "modifierB": false
);
```

All modifiers except "modifierA" and "modifierB" are outputted.

Note that the following settings are identical

```css
$modifiers: (
  "modifierA": true
);
$modifiers: "modifierA";
```

If an modifier is explicity allowed, all other modifiers are disallowed. Unless also allowed explicitly.

Disallowing 'modifierB' is not necessary here:

```css
$modifiers: (
  "modifierA": true,
  "modifierB": false
);
```

Multiple entities are allowed by naming them explicitly.

```css
$modifiers: (
  "modifierA": true,
  "modifierB": true
);
```

Only modifiers "modifierA" and "modifierB" are outputted.

**If a modifier is set, base elements are not allowed within that modifier!**

### Modifiers elements

Single element in a modifier can be allowed or disallowed

```css
$modifiers: (
  "modifierA": (
    "element": true
  ),
  "modifierB": "element"
);
```

The 'element' and only that element is outputted in the 'modifierA' and 'modifierB'.

**If `$modifiers` has elements defined, they override the rules in the `$elements`!**

### The "$elements" rule

Defines which elements are outputted. Rule logic is the same as with modifiers. Except the value cannot be a nested map, because there are nothing to target the nested map with.

#### The important "base" content

A reminder that "base" content is controlled with `$elements`! It should not be used as a element's name: `@include element("base")`.

**If an element is defined in the `$modifiers`, they override the rules in the `$elements`!**

### The "$extras" rule

The `$extras` should only contain "name:true/false" values. Usage is simple:

```css
@include extra("theme-black") {
}

@include my-button(
  $extra: (
    "theme-black": true
  )
);
```

## Helper mixins and functions

These mixings are helpers for creating complex selectors or joining selectors from parents. Selectors can also be created with passed custom functions.

### selector-list

Use `@include selector-list(<list>)` to output complex selectors with a one-liner. The only argument is a list of maps with entity type:value pairs.

```css
selector-list((('modifier': 'hello'), ('modifier': 'hello2'), ('element': 'elem'), ('custom': ' div'))) {
  // CSS
}
// outputs
<parent selector > --hello,
<parent selector > --hello2,
<parent selector > __elem,
<parent selector > div {
  // CSS
}
```

### linked-entities

Combines given selectors with selectors from closest entities of given type. This mixin has shorthands named `compound-entity`, `descendant-entity` and `compound-block-modifier`. They set the arguments automatically.

The mixin uses named arguments:

- `$typeAndName` defines what entity to look for. It is map with "type" and "name". Name is optional.
- `$separator` defines how selectors are separated. Value can be empty string ("") or a space (" ").
- third argument should be `$block`, `$modifier`, `$element` or `$custom` and the values should be name of the entity which is used in the selector.

```css
span.class {
  @include modifier("parent") {
    @include modifier("child")
      @include linked-entities(
        $typeAndName: (
          type: "modifier",
          name: "linked-to-parent",
        ),
        $separator: "",
        $modifier: "parent"
      ) {
        // CSS
      }
    }
  }
}
// outputs (excluded CSS from modifiers and elements)
// the "--parent--linked-to-parent" is from linked-entities
span.class--parent--child--parent--linked-to-parent {
  // CSS
}
```

### compound-entity

Shorthand to create a compound selector. Calls the `linked-entities` with `$separator: ""`.

### descendant-entity

Shorthand to create a descendant selector. Calls the `linked-entities` with `$separator: " "`.

### compound-block-modifier(modifier-to-add)

Shorthand to create a compound selector. Calls the `compound-entity` with given modifier in the `$typeAndName` and `$block:true`.

Finds the closest block, copies its selector and creates a new selector with a linked modifier. It merges the block's selector with the created selector. The selectors are separated with given separator "". Empty separator creates a compound selector of `.block.block--<name>`

```css
@include modifier("sibling-modifier") {
  // CSS of sibling-modifier
  @include compound-block-modifier("compound-modifier") {
    // CSS  of compound-modifier
    @include element("compound") {
      // CSS  of compound
    }
  }
}
// creates

.block--sibling-modifier {
  // CSS of sibling-modifier
}

.block--sibling-modifier.block--compound-modifier {
  // CSS  of compound-modifier
}

.block--sibling-modifier.block--compound-modifier__compound {
  // CSS  of compound
}
```

This decreases the amount of nested code.

### descendant-modifier-element(element-to-add,modifier-to-find)

Shorthand to create a descendant selector. Calls the `descendant-entity` with given element in the `$typeAndName` and `$modifier:modifier`. If `modifier` is not passed, modifier is not searched with a name, just type.

```css
@include modifier("mod1") {
  @include descendant-modifier-element("element1") {
  }
}
// results in
.block--mod1 .block--mod1__element1;
```

### block-element(element-to-add)

Shorthand to create a compound selector. Calls the `descendant-entity` with given element in the `$typeAndName` and `$block:true`.

```css
@include modifier('modifier') {
    @include block-element('block-element')
    }
}
// Results in
 .block--modifier .block__block-element
```

### descendant

Appends a selector to the parent as a descendant.

```css
 @include modifier('modifier') {
    @include @include descendant('p')
    }
 }
// Results in
 .block--modifier p {}
```

### class

Appends a class name to the parent selector .

```css
 @include modifier('modifier') {
    @include @include class('extra')
    }
 }
// Results in
 .block--modifier.extra {}
```

### extra-selector(rule, selector)

Wraps the `custom` mixin with `extra` to create a controlled `custom` selector.

```css
@include extra-selector("extra", "selector") {
  // CSS outputted with selector when 'extra' is allowed
}
```

### extra-descendant(rule, selector)

Same as `extra-selector`, but the selector has " " prefix so it is outputted as a descendant.

### if-content-allowed

The CSS inside this mixin is outputted only if the `base` rule is allowed in `$elements`. See also [content named base](#important-content-named-base)

### repeat($args...)

Finds the closest entity that matches the given arguments and repeats its selector again

```css
@include block() {
  @include repeat($block: true) {
    @include repeat($block: true) {
    }
  }
}
// Results in
.block .block .block {
}
```

### repeat-with-replace($string-to-replace, $replacement, $args...)

Like repeat, but finds and replaces strings in the selector.

```css
@include block() {
  @include repeat-with-replace($original: "-blockName", $replacement: "-replacement", $block: true) {
  }
}
// Results in
.block .replacement {
}
```

### create-custom-selector

Uses a passed function to create a selector. The passed function is called with rules, config and the current level. The function must return only a string.

```css
$passedFunction($ruleMap, $config, $currentLevel)
```

This mixin outputs nested CSS rules directly without creating a level.

### create-custom-level

Uses a passed function to create a new level as a child to current level. The passed function is called with rules, config and the current level. The function must return a valid level which must have a selector for output.

```css
$passedFunction($ruleMap, $config, $currentLevel)
```

## Constants

There are some constants used in `multi-sass` and those are also exported and can be used instead of strings.

The files can be imported with

```css
@use "@nikohelle/multi-sass/globals" as globals;
```

- for `emitContentOnly` use `globals.$EMIT_CONTENT_ONLY_IN_ROOT` or `globals.$EMIT_CONTENT_ONLY_ALL`
- for `base` use `globals.$BASE_CONTENT_NAME`

The `globals` also exports `$ELEMENT-DELIMETER` ("\_\_") and `$MODIFIER-DELIMETER` ("--").
