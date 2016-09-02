# SASS-core

A [SASS][sass]-based framework with useful mixins. Originally a part of the [Frontend-starter][frontend-starter] framework.

## Features
* [media queries with Breakpoint library][sass-breakpoint], configurable breakpoints
* design breakpoints - set the desing (e.g PSD) pixel dimensions and use mixins to automatically calculate percentage or vw units
* font sets - set the standard font sizes (for headings, paragraphs) and use a mixin to automatically generate media queries for rem/px/vw (or use converter functions if you want to add media queries manually)
* grid building mixins
* responsive sprites (requires [spritesmith] - available also in grunt/gulp version)
* utilities: clearfix, font-face, placeholder styling

## Installation

### Bower install
Add `sass-core` to your `bower.json` dependencies.

### Manual install
Unpack the contents of `dist` directory to the desired dir.

### Importing
**First** import [SASS Breakpoint][sass-breakpoint] file, then SASS-core file (`_sass-core.scss`).

<br><br>
## Manual
By default, SASS-core uses the [SASS Breakpoint][sass-breakpoint] and [mobile first approach](http://www.google.com/search?q=mobile+first). The predefined in `_config.scss` breakpoints are (you can change any options by setting variables before importing the framework):
* main, defined as min-width: `mobile`, `tablet`, `desktop` (when you want to target viewport with at least specified width)
* main, defined as exact ranges: `mobile-ex`, `tablet-ex`, `desktop-ex` (when you want to target only the specified range)
* auxiliary (small and large variations), defined as exact ranges: `mobile-sm`, `mobile-lg`, `tablet-sm`, `tablet-lg`, `desktop-sm`, `desktop-lg`

Build your stylesheet like this:

```sass
.col {

  /* common */
  background: #fff;
  
  /* tablet and desktop */
  @include respond-to(tablet) {
    float: left;
    width: 50%; //2 columns
  }

  /* desktop */
  @include respond-to(desktop) {
    width: 25%; //4 columns
  }

  //non-standard breakpoints
  @include breakpoint(400px 600px) {
    ...
  }

  @include breakpoint(max-width 500px) {
    ...
  }

}
```

### Units
Convert px units easily to vw or percentage. Just get the px dimensions of an element from the design (e.g. PSD), and use:
```sass
//pass the width and container width as a breakpoint/design breakpoint name
width: unit-vw(50px, mobile); //~6,519vw if mobile breakpoint is 767px
//you can omit breakpoint name for mobile, as it is default, so the previous rule is same as:
width: unit-vw(50px);
//...or pass container width as a value
width: unit-pc(100px, 250px); //40%

@include respond-to(tablet) {
  width: unit-vw(100px, tablet);
  //or
  width: unit-pc(100px, 500px);
}
```


### Fonts

#### Embedding font-face
Use the mixin `sc-font-face`:
```sass
@mixin sc-font-face($name, $filenameBase: null, $weight: normal, $style: null, $dir: $sc-font-dir, $exts: $sc-font-formats, $ie8fix: true);

//example
@include sc-font-face(OpenSans, OpenSans-Bold, bold);
```

Notes:
* `$name`: font-family name you will refer to in the stylesheets
* `$filenameBase`: if not set, the `$name` is assumed for the filename (without extension)
* `$dir`: font dir relative to the css output dir; defaults to the `$sc-font-dir` value, i.e. `fonts` (`css/fonts`)
* `$exts`: file extensions separated by space; defaults to the `$sc-font-formats` value, i.e. `woff woff2` (for full support, including old IE, set to `eot woff woff2 ttf svg`)
* `$ie8fix`: adds the IE8 support (if svg/eot formats are enabled)


#### Responsive sizes

The following mixins produce appropriate media queries:

##### px unit
Use the `font-px` mixin, specifying font sizes for the breakpoints, e.g.:
```sass
@include font-px((mobile: 15px, tablet: 13px, desktop: 17px));
```

##### rem unit
Use the `font-rem` mixin, specifying target font sizes as a map, e.g.:
```sass
@include font-rem((mobile: 15px, tablet: 13px));
```

The default [font set](#styles-font-sets) is taken, you can also pass a different font set name or a breakpoint font size map as a second parameter.

##### vw unit
To make the font size dependent on the viewport width, use the `font-vw` mixin, specifying the maximum font sizes for the breakpoints, e.g.:
```sass
@include font-vw((mobile: 15px, mobile-sm: 14px, tablet: 13px, desktop: 17px));
```
For desktop breakpoints (desktop, desktop-lg), the `px` font size unit will always be used - you can change the `$sc-font-conv-vw-exclude` config variable (list) or pass it as a `$vwExcludeBreakpoints` parameter.

##### The result
Media queries will be automatically created. Produced CSS code will be similar to (example for vw unit; px size is just a fallback for browsers not supporting this unit):
```css
/* mobile */
font-size: 14px;
font-size: 1.95567vw;

/* mobile-sm */
@media (max-width: 479px) {
  font-size: 13px;
  font-size: 2.92276vw;
}

/* tablet */
@media (min-width: 768px) {
  font-size: 12px;
  font-size: 1.31181vw;
}

/* desktop */
@media (min-width: 992px) {
  font-size: 17px;
}
```

The pixel fallback for vw units is a value multiplied by ratio defined in `$sc-font-conv-vw-fallback-ratio` (defaults to `0.8`). Set it to `0` to drop the fallback.


##### Functions: rem, vw
If you don't want to create media queries, you can use the `unit-rem` and `unit-vw` directly.
```sass
font-size: unit-vw(15px);           //by default, mobile breakpoint is assumed
font-size: unit-vw(15px, 500px);    //500px: base width to calculate vw
font-size: unit-vw(15px, desktop);  //desktop: design breakpoint name

font-size: unit-rem(15px);          //by default, the $sc-font-rem-base config value is used (defaults to 16px)
font-size: unit-rem(15px, 20px);    //20px: custom base font size to calculate rem
```


<a name="styles-font-sets"></a>
##### Responsive size with font sets
You will probably need a more handy way to define font sizes. It often happens that you have some number of standard font sizes on the website. To use the `font-*` mixins more conveniently, you can predefine these font sizes, before the framework SASS import:
```sass
$sc-font-sets: (

  /* small font */
  sm: (mobile: 15px, tablet: 13px, desktop: 17px),

  /* normal font */
  md: (mobile: 16px, tablet: 18px, desktop: 14px),

  /* header font */
  lg: (mobile: 18px, tablet: 22px, desktop: 22px)

  /*...other sets...*/
);
```

Then, use `font-*` mixin, specifying only the font set name, like:
```sass
@include font-vw(sm);
```

This will produce same code as (according to the `md` font set definition):
```sass
@include font-vw((mobile: 15px, tablet: 13px, desktop: 17px));
```

You can change any font set size, by passing a map as a second parameter:
```sass
@include font-px(sm, (mobile: 12px)); //mobile size changed, other sizes left unmodified according to the font set
```

Default font set is defined by `$sc-font-set-default` variable (defaults to `md`).

You can set your default font size like this:
```sass
body {
  @include font-px(md);
}
```

If you use `rem` units and change the default base font size, by assigning it to the `$sc-font-rem-base` variable, remember to set it also in your stylesheet:
```sass
html {
  font-size: $sc-font-rem-base;
}
```


<a name="styles-design-breakpoints"></a>
### Design breakpoints
By default the base width used to calculate vw/percentage width is the width of a breakpoint passed as the second argument. However, usually layout widths on the design do not meet the "system" (media query) breakpoints. For example, you can get a PSD design for mobile at 600px width (rather than 767px), and would like to calculate units according to this size. To deal with it, add a design breakpoint:
```sass
$sc-design-breakpoints: (mobile: 600px);
```

In this case, all units for `mobile`, including font-vw, will be calculated according to this size. If you don't set a particular design breakpoint (e.g `desktop`), the mixins fallback to the default breakpoints.


### Grids
Quickly create a grid with the following mixins:
```sass
.container {

  @include respond-to(tablet) {
    @include grid-cont(20px);   //20px gutter
  }

  .row {
    @include respond-to(tablet) {
      @include grid-row();
    }
    .col {
      @include respond-to(tablet) {
        @include grid-col(50%);
      }
      @include respond-to(desktop) {
        width: 25%; //no need to include grid-col if it was done in previous breakpoint
      }
    }
  }
}
```

This creates a 2-col grid for tablet and 4-col grid for desktop with a 20px gutter. Remember, that if you nest the grids, specify the gutter also for `grid-row` and `grid-col`, because these mixins take the last used width from a global variable set in `grid-cont`.

You can define custom grid classes at the top-level, to make them reusable.

This allows you to create fully customized column sizes (non-Bootsrap 12 cols scheme, rather like 13%/47%/40%) and gutters.



### Sprites
Generate sprites using [spritesmith] and import the stylesheet(s) (see the commented out line in `style.scss`). To use a sprite, simply use the mixin:
```sass
.sprite-icon {
  @include sprite($[prefix]file-name);

  //e.g. sprite of my-image.png (no prefix)
  @include sprite($my-image)
}
```

To create a **responsive sprite icon** you can wrap it in `sprite-wrap-rwd` mixin, so the `max-width` property will be set to the actual sprite dimensions:
```sass
.sprite-wrap {
  @include sprite-wrap-rwd($[prefix]file-name);

  .sprite {
    @include sprite-rwd($[prefix]file-name);
  }
}
```

Sprite position and dimensions will adapt to the `.sprite-wrap` container width. You should set the [spritesmith] `algorithm` option to `diagonal`. Be aware, that using this method (the mixin calculates background position and size in percentages) for large images will impact the performance in Chrome browser.


### Utilities
As for now, the framework comes with 2 helper mixins: `clearfix` and `input-placeholder` (for placeholder styling).

```sass
@include clearfix;

@include input-placeholder {
  color: #000;
}
```


### Helper variables
As for now there are only font-weight helpers. Use the following variables instead of literals like `300`, `800`:

```sass
$font-w-thin: 100 !default;
$font-w-xlight: 200 !default;
$font-w-light: 300 !default;
$font-w-normal: 400 !default;
$font-w-medium: 500 !default;
$font-w-semibold: 600 !default;
$font-w-bold: 700 !default;
$font-w-xbold: 800 !default;
$font-w-black: 900 !default;
```



[frontend-starter]: https://github.com/implico/frontend-starter
[bower]: http://bower.io/
[compass]: http://compass-style.org/
[nodejs]: https://nodejs.org/
[sass]: http://sass-lang.com/
[sass-breakpoint]: http://breakpoint-sass.com/
[spritesmith]: https://github.com/Ensighten/spritesmith