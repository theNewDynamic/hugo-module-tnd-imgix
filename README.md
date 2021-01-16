# Imigx Hugo Module

The component should be used on any project using [imgix](https://www.imgix.com/) as an image transforming API.

## Requirements

Requirements:
- Go 1.14
- Hugo 0.61.0


## Installation

If not already, [init](https://gohugo.io/hugo-modules/use-modules/#initialize-a-new-module) your project as Hugo Module:

```
$: hugo mod init {repo_url}
```

Configure your project's module to import this module:

```yaml
# config.yaml
module:
  imports:
  - path: github.com/theNewDynamic/hugo-module-tnd-imgix
```

## Usage

### GetSRC

The only available public returning partial is `GetSRC`.

It returns, as a string, the Imigx URL of the passed image SRC with the optional passed transformation on top of the defaults transformations if any.

This takes two types of arguments.
- A file path | String (.)
  With the above, the function returns the file path prefixed with the imgix domain, with global defaults transformation if any.
- A map | Map (.)
  src: A file path | String
  ...any other key will match a transformation from the imgix API and its value the property to be passed.
  With the above, the function returns the file's imgix URI including the transformation query.

#### Examples
```
{{ $src := "/uploads/an-image.jpg" }}
{{ with partial "tnd-imgix/GetSRC" $src }}
  {{ $src = . }}
{{ end }}
```

Will produce: `https://imgix.project.net/uploads/an-image.jpg`

Note: [Default set transformations](#defaults) will be appended.

```
{{ $args := dict "src" "/uploads/an-image.jpg" "width" 1024 "height" 100 }}
{{ with partial "tnd-imgix/GetSRC" $args }}
  {{ $src = . }}
{{ end }}
```
Will produce: `https://imgix.project.net/uploads/an-image.jpg?w=1024&h=100`

### Settings

Settings are added to the project's parameter under the `tnd_imgix` map as show below.
Keys are detailed below

```yaml
# config.yaml
params:
  tnd_imgix:
    domain: imgix.project.net
    mapping:
      pixel: 'dpr'
    defaults:
      auto: format
      ch: Width,DPR
      q: 95
    allowed_extensions:
    - jpg
    - png
```

#### Domain (required)

This is the imgix domain of your project.

#### IMGIX API mapping.

To abstract the imgix terminology, the module interprets commonly used transformation keys. 
User can therefor use `width` and expect the function to find the proper Imgix key (`w`).

Key absent from mapping settings will be passed as is.

Default mapping is:
```yaml
  width: 'w'
  height: 'h'
  quality: 'q'
  text: 'txt'
```

User can overwrite the above mapping with adding their own mapping keys to the imgix parameters.

```yaml
params:
  tnd_imgix:
    mapping:
      pixel: 'dpr'
```


Given the example above, passing the following arguments to `tnd-imgix/GetSRC`
```
{{ $src := "/uploads/an-image.jpg" }}
{{ $args := dict "src" $src "width" 1024 "pixel" 2 "ch" "Width,DPR" }}
```

Will produce: `https://imgix.domain.com/image.jpg?w=1024&dpr=2&ch=Width,DPR`

#### Defaults

User can add some sensible default transformations to the module. It will use them whenever the `GetSRC` function is called. 
If one of the default keys is passed, its value will overwrite the default's.

```yaml
params:
  tnd_imgix:
    defaults:
      auto: format
      ch: Width,DPR
      q: 95
```

Defaults can be passed using the mapped keys discussed above (`width`, `quality` etc...)

#### Allowed Extensions

The `allowed_extensions` key lets you define a list of extensions for which the module will apply transformations.
This is to make sure no transformation is applied to a PDF for example.

Defaults are as followed:
```yaml
params:
  tnd_imgix:
    allowed_extensions:
    - jpg
    - jp2
    - jpeg
    - png
    - gif
    - webp
    - tif
```
The settings does not extends aforementioned defaults with user's own. In order to complement the defaults, you should copy/paste the above to your settings and append with new extensions.

## Markup

With a tiny bit of work you can have imgix applied to images rendered through Markdown.

```markdown
![Picture of a guitar](/uploads/guitar.jpg "I love my guitar!")
```

This will make the above rendered as:

```html
<img src="https://my-domain.imgix.net/uploads/guitar.jpg?auto=format&ch=Width,DPR&q=95&w=700" alt="Picture of a guitar" title="I love my guitar!">
```

You'll need to:

1. Make sure your project is indeed using [Goldmark](https://gohugo.io/getting-started/configuration-markup#goldmark) as Markdown renderer (by default it should).
2. Create at the root of your project a `layouts/_default/render-image.html`
3. Populate given file with your own HTML and a touch of GetSRC.  
The module sports its own unused [template file](https://github.com/theNewDynamic/hugo-module-tnd-imgix/blob/master/markup/render-image.html) which you can use for reference.

## theNewDynamic

This project is maintained and love by [thenewDynamic](https://www.thenewdynamic.com).
