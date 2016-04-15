# SASS-core

A [SASS][sass]-based framework with useful mixins. Originally a part of the [Frontend-starter][frontend-starter] framework.

## Features
* [media queries with Breakpoint library][sass-breakpoint], configurable breakpoints
* design breakpoints - set the desing (e.g PSD) pixel dimensions and use mixins to calculate percentage or vw units
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
//or pass container width as a value
font-size: unit-pc(100px, 250px); //40%

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
* `$exts`: file extensions separated by space; defaults to `eot woff woff2 ttf`
* `$ie8fix`: adds the IE8 support


#### Responsive sizes

The following mixins produce appropriate media queries:

##### px unit
Use the `font-px` mixin, specifying font sizes for the breakpoints, i.e.:
```sass
@include font-px((mobile: 15px, tablet: 13px, desktop: 17px));
```

##### rem unit
Use the `font-rem` mixin, specifying target font sizes as a map, i.e.:
```sass
@include font-rem((mobile: 15px, tablet: 13px));
```

The default [font set](#styles-font-sets) is taken, you can also pass a different font set name or a breakpoint font size map as a second parameter.

##### vw unit
To make the font size dependent on the viewport width, use the `font-vw` mixin, specifying the maximum font sizes for the breakpoints, i.e.:
```sass
@include font-vw((mobile: 15px, mobile-sm: 14px, tablet: 13px, desktop: 17px));
```
For desktop breakpoints, px font size will be used - you can change the `$sc-font-conv-vw-exclude` config variable (list) or pass it as a `$vwExcludeBreakpoints` parameter.

##### The result
Media queries will be automatically created. Produced CSS code will be similar to (example for vw unit; px size is just a fallback for browsers not supporting this unit, multiplied by a 0.9 ratio):
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

##### Functions: rem, vw
If you don't want to create media queries, you can use the `unit-rem` and `unit-vw` directly.
```sass
font-size: unit-vw(15px, 500px);  //500px: base width to calculate vw
font-size: unit-vw(15px, mobile); //mobile: design breakpoint name

font-size: unit-rem(15px, 20px);  //20px: base font size to calculate rem
font-size: unit-rem(15px, md);    //md: font set name
font-size: unit-rem(15px);        //with no second argument, the default font set is assumed as a base
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

This will produce same code as:
```sass
@include font-vw((mobile: 15px, tablet: 13px, desktop: 17px));
```

You can change any font set size, by passing a map as a second parameter:
```sass
@include font-vw(sm, (mobile: 12px)); //mobile size changed, other sizes left unmodified according to the font set
```


###### Font sets and rems
When using `font-rem` mixin, you must aways specify the sizes for the desired breakpoints (because font set px sizes are taken as a base), like:
```sass
@include font-rem(md, (mobile: 21px, tablet: 20px));
```
Notice, that if you pass a map instead of a string as the first parameter and not pass the second, the default font set is assumed as a base and the parameter is considered as font sizes. Shortly, the above code is equal to:
```sass
@include font-rem((mobile: 21px, tablet: 20px));
```

Default font set is defined by `$sc-font-set-default` variable (defaults to `md`). Set your root font size like this:
```sass
html {
  @include font-px(md);
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
  @include sprite($file-name);
}
```

To create a **responsive sprite icon** you can wrap it in `sprite-wrap-rwd` mixin, so the `max-width` property will be set to the actual sprite dimensions:
```sass
.sprite-wrap {
  @include sprite-wrap-rwd($file-name);

  .sprite {
    @include sprite-rwd($file-name);
  }
}
```

Sprite position and dimensions will adapt to the `.sprite-wrap` container width.


### Utilities
As for now, the framework comes with 2 helper mixins: `clearfix` and `input-placeholder` (for placeholder styling).

```sass
@include clearfix;

@include input-placeholder {
  color: #000;
}
```



[frontend-starter]: https://github.com/implico/frontend-starter
[bower]: http://bower.io/
[compass]: http://compass-style.org/
[nodejs]: https://nodejs.org/
[sass]: http://sass-lang.com/
[sass-breakpoint]: http://breakpoint-sass.com/
[spritesmith]: https://github.com/Ensighten/spritesmith