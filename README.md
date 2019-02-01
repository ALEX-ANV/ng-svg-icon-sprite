# SVG icon sprite component for Angular 7

This library provides both a solution for generating SVG sprites and a [module](https://www.npmjs.com/package/ng-svg-icon-sprite) for including them.

## Demo

![Demo gif animation](svg-icon-sprite-anim.gif)

<a href="https://jannicz.github.io/ng-svg-icon-sprite/">
  Try out the ng-svg-icon-sprite demo
</a>

## Use Cases

- include single-color icons from a sprite
- fill and scale icons dynamically via CSS (i.e. hover, focus effects)
- meet accessibility requirements for inline SVGs

## Installation

After installing the package via `npm i ng-svg-icon-sprite -S` as dependency you can import it into
any application’s app.module.ts by simply including it in its `@NgModule` imports array:

```javascript
import { IconSpriteModule } from 'ng-svg-icon-sprite'; // <-- here

@NgModule({
  declarations: [
    AppComponent
  ],
  imports: [
    BrowserModule,
    IconSpriteModule // <-- here
  ],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

## Usage

To use your SVGs from a sprite you need to:

1. Convert your SVG icons into a sprite using a script
2. Include the `svg-icon-sprite` component with the sprite path and the icon name

### Step 1: Generate the sprite

First add the library [for sprite generation svg2sprite](https://github.com/mrmlnc/svg2sprite-cli) as a devDependency:
                                                                                      
```json
"devDependencies": {
  "svg2sprite-cli": "2.0.0"
}
```

Each time you add an icon, you need to run the script generating the sprite. You might want to add it to your npm scripts:

```json
"scripts": {
  "generate:svg-sprite": "svg2sprite ./src/assets/icons ./src/assets/sprites/sprite.svg --stripAttrs fill --stripAttrs stroke --stripAttrs id"
}
```

now execute the script:

```
npm run generate:svg-sprite
```

__Note: the fill and stroke properties are removed so the SVG can be filled via CSS. If don't need to apply color changes on your icons,
go for the multi-color pattern [described below](#user-content-dealing-with-multi-color-svgs-containing-inline-styles)__

Regardless of the method, the script will take all SVG icons from `src/app/assets/icons` and create a sprite SVG into
`src/app/assets/sprites` using the [svg symbols technique](https://css-tricks.com/svg-symbol-good-choice-icons/):

```
app
└── assets
    └── icons (icons source)
        └── icon-1.svg
        └── icon-2.svg
    └── sprites (sprite destination)
        └── sprite.svg
```

### Step 2: Use the component

Now you can include icons by using the `svg-icon-sprite` component directive:

```html
<!-- here including 'cart' SVG from the sprite -->

<svg-icon-sprite
  [src]="'assets/sprites/sprite.svg#cart'"
  [width]="'22px'"
  [classes]="'my-icon-class'"
></svg-icon-sprite>

<!-- or with a dynamic icon name -->

<svg-icon-sprite
  [src]="'assets/sprites/sprite.svg#' + iconName"
  [width]="'50%'"
></svg-icon-sprite>
```

## Options

- `src` - icon source name, the syntax is `path/file#icon` where `path` is relative to app folder, `file` is
the name of the sprite and `icon` is the filename of the svg icon.
- `width` *optional* - width of the svg in any length unit, i.e. `32px`, `50%`, `auto` etc., default is `100%`
- `height` *optional* - the height of the svg in any length unit, if undefined height will equal the width
- `classes` *optional* - class name(s) for this icon, default is `icon`
- `viewBox` *optional* - define lengths and coordinates in order to scale to fit the total space available (used for scaling up/down)
- `preserveAspectRatio` *optional* - manipulate the aspect ratio, only in combination with `viewBox`
- `title` - *optional* - text string that will be rendered into a title tag as the first child of the SVG node
- `attribute` - *optional* - tuple or array of tuples containing key/value pair that should be added as an attribute on the SVG node, i.e. `"['aria-hidden', 'true']"` becomes `<svg aria-hidden="true">`

## Styling

To change the sprite color add a CSS `color` property to the component invoking svg-icon-sprite. The SVG component uses
the `currentColor` value to [pass the ancestor's color](https://css-tricks.com/cascading-svg-fill-color) through to the SVG shapes:

```css
/* host component styles */
color: red;
```

## Advanced Configuration

### Assets folder

If you have another folder structure than above, you can pass both your input and output path using the npm script:

```
svg2sprite sourcefolder destination/filename.svg
```

### Custom Styling

To access inner SVG properties like `fill` or `stroke`, you need to use Angular's `::ng-deep` selector in
the host component and select the `use` tag inside the SVG:

```css
.host-component ::ng-deep svg.icon use {
  fill: orange;
}
```

__Note: make sure your CSS selector is strong enough here__

### Scaling and Sizing

If your SVG does not scale like expected (i.e. it is cropped or larger than desired) it might be lacking a `viewBox`.
You need to set the `viewBox` property manually to match the size of the exported shape. A combination of the correct
`viewBox` and width is required. Add the `viewBox` property and decrease/increase the last 2 values:

```html
<!-- i.e. lower '0 0 80 80' to '0 0 40 40' to scale up/down -->
<svg-icon-sprite
  [src]="'assets/sprites/sprite.svg#star'"
  [width]="'40px'"
  [viewBox]="'0 0 80 80'"
></svg-icon-sprite>
```

See the viewBox [example](https://jannicz.github.io/ng-svg-icon-sprite/#viewBox) for further details.

Still having trouble with scaling or sizing? Read [this article](https://css-tricks.com/scale-svg/) about SVG scaling.

### Dealing with multi color SVGs containing inline styles

If you wish to combine the single-color icon pattern with SVGs that contain inline styles (i.e. multi-color) that should not be overridden by CSS,
you will have to provide a separate sprite file that keeps the stroke and fill attributes:

```json
"scripts": {
  "generate:svg-multicolor-sprite": "svg2sprite ./src/assets/svg-images ./src/assets/sprites/image-sprite.svg"
}
```

The generated sprite will preserve it's original styles, but you won't be able to style it via CSS, [see demo](https://jannicz.github.io/ng-svg-icon-sprite/#multicolor).

### Setting a default sprite path for all icons

If your app uses one main sprite source, you can set it's path via the icon sprite service:

```javascript
// In your app.component import the service
import { IconSpriteService } from 'ng-svg-icon-sprite';

// Inject it
constructor(private iconSpriteService: IconSpriteService) {}

// And set the path
ngOnInit() {
  this.iconSpriteService.setPath('assets/sprites/sprite.svg');
}
```

[Like in the demo](https://jannicz.github.io/ng-svg-icon-sprite/#defaultpath), you can now leave out the path and just provide the icon name.

```html
<svg-icon-sprite [src]="'star'"></svg-icon-sprite>
```

Doing so you will still be able override the default path by using the full syntax for particular icons that should use a different sprite file.

## Browser Support
- Chrome (63)
- Firefox (57)
- Safari 11
- Edge
- IE 11 (with polyfill, see below)

### Polyfill for IE11 (and comparable)

Older browsers do not support referencing to (external) SVG symbols. To make it work for IE11 and lower you can add
[svg4everybody](https://github.com/jonathantneal/svg4everybody) to your `polyfills.ts` file:

```javascript
// Add the node module, import and execute it in polyfills.ts
import * as svg4everybody from 'svg4everybody/dist/svg4everybody.js';
svg4everybody();
```

## Accessibility

In order to support screen readers and make the icons meaningful, you can use following patters:
- add a `title` with descriptive text ([see demo](https://jannicz.github.io/ng-svg-icon-sprite/#a11y))
- optionally reference the title node using `aria-labelledby=”icon-title”`
- optionally set the node's `role` to image (`role=”img”`)

```html
<svg-icon-sprite
  [src]="'assets/sprites/sprite.svg#star'"
  [title]="'Some title text'"
  [attribute]="[['aria-labelledby', 'star-title'], ['role', 'img']]"
></svg-icon-sprite>
```

If you want to prevent the icon from being accessed by screen readers (i.e if you already have a descriptive text somewhere else),
set the `attribute` of `['aria-hidden', 'true']` instead.

Or use combinations of several methods to achieve better results, like described in this [article](https://css-tricks.com/accessible-svgs/).

## Compatibility

This library is optimized for Angular 7. For Angular 6 use [ng-svg-icon-sprite 1.2](https://www.npmjs.com/package/ng-svg-icon-sprite/v/1.2.1),
for Angular 4/5 use [ng-svg-icon-sprite 0.8](https://www.npmjs.com/package/ng-svg-icon-sprite/v/0.8.0).

## Author & License
- Jan Suwart | MIT License
