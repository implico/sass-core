# sass-core

A SASS-based framework with useful mixins. Originally a part of the [frontend-starter] framework.

## Features
* [media queries with Breakpoint library][sass-breakpoint], configurable breakpoints
* design breakpoints - set the desing (e.g PSD) pixel dimensions and use mixins to calculate percentage or vw units
* font sets - set the standard font sizes (for headings, paragraphs) and use a mixin to automatically generate media queries for rem/px/vw (or use converter functions if you want to add media queries manually)
* responsive sprites (requires [spritesmith] - available also in grunt/gulp version)
* utilities: clearfix, font-face, placeholder styling

## Installation
T.B.C. - via Bower

## Manual
By default, sass-core uses the [SASS Breakpoint][sass-breakpoint] and [mobile first approach](http://www.google.com/search?q=mobile+first). The predefined in `_config.scss` breakpoints are (you can change any options by setting variables before importing the framework):
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
    width: 50%;
  }

  /* desktop */
  @include respond-to(desktop) {
    width: 25%;
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
width: unit-vw(50px, mobile);
//or
width: unit-pc(50px, mobile);

font-size: unit-rem(10px, $font-size-mobile);
//or
font-size: unit-vw(16px, mobile);

@include respond-to(tablet) {
  width: unit-vw(100px, tablet);
  //or
  width: unit-pc(100px, tablet, 1/12);  //for e.g. Bootstrap's col-sm-12, specify the ratio

  font-size: unit-rem(12px, $font-size-tablet);
  //or
  font-size: unit-vw(14px, tablet);
}
```


### Fonts

#### Enabling font-face
Use the mixin `font-face-custom` (located in `mixins/fonts`).

#### Responsive size: px, rem, vw
Use the `font-px` mixin, specifying the font sizes for the breakpoints, i.e.:
```sass
@include font-vw($mobile: 15px, $tablet: 13px, $desktop: 17px);
```

Use the `font-rem` mixin, specifying the font sizes and base font sizes for the breakpoints, i.e.:
```sass
@include font-rem($sizesPxRem: (mobile: 10px, tablet: 15px), $mobile: 15px, $tablet: 13px);
```

To make the font size dependent on the viewport width, use the `font-vw` mixin, specifying the maximum font sizes for the breakpoints, i.e.:
```sass
@include font-vw($mobile: 15px, $tablet: 13px, $desktop: 17px);
```

Media queries will be automatically created. Produced CSS code will be similar to (example for vw unit; px size is just a fallback for browsers not supporting this unit):
```css
/* mobile */
font-size: 12px;
font-size: 1.69492vw;

/* tablet */
@media (min-width: 768px) {
  font-size: 14px;
  font-size: 1.51362vw;
}

/* desktop */
@media (min-width: 992px) {
  font-size: 17px;
}
```

If you don't want to create media queries, you can use the `unit-rem` and `unit-vw` directly.
```sass
font-size: unit-vw(15px, mobile);
```

Available breakpoints: `mobile`, `tablet`, `desktop`, `mobile-sm`, `tablet-sm`. You can override media query breakpoints with [design breakpoints](#styles-design-breakpoints).


#### Responsive size with font sets
It often happens that you have some number of standard font sizes on the website. To use the `font-*` mixins more conveniently, you can predefine these font sizes, before the framework SASS import:
```sass
$font-sets: (

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
@include font-vw($mobile: 15px, $tablet: 13px, $desktop: 17px);
```

When using `font-rem` mixin, you must aways specify the sizes for the desired breakpoints, like:
```sass
@include font-rem(sm, (mobile: 21px, tablet: 20px));
```



<a name="styles-design-breakpoints"></a>
### Design breakpoints
By default the base width used to calculate vw/percentage width is the width of a breakpoint passed as the second argument. However, usually layout widths on the design do not meet the "system" (media query) breakpoints. For example, you can get a PSD design for mobile at 600px width (rather than 767px), and would like to calculate units according to this size. To deal with it, add a design breakpoint:
```sass
$design-breakpoints: (mobile: 600px);
```

In this case, all units for mobile, including font-vw, will be calculated according to this size.


### Grids
Quickly create a grid with the following mixins:
```sass
.container {

  @include respond-to(tablet) {
    @include grid-cont(20px);
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



### Responsive sprites
Generate sprites using [spritesmith] and import the stylesheet(s). To create a responsive sprite icon, use the mixins:

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


## TODO
* register in [Bower][bower]
* and more



[frontend-starter]: https://github.com/implico/frontend-starter
[bower]: http://bower.io/
[compass]: http://compass-style.org/
[nodejs]: https://nodejs.org/
[sass]: http://sass-lang.com/
[sass-breakpoint]: http://breakpoint-sass.com/
[spritesmith]: https://github.com/Ensighten/spritesmith