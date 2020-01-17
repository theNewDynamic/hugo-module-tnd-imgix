# Imigx Hugo Module

The component should be used on any project using IMGIX as an image CND and transforming API.

## Requirements

Requirements:
- Go 1.12
- Hugo 0.61.0


## Installation

1. [Init](https://gohugo.io/hugo-modules/use-modules/#initialize-a-new-module) your project as Hugo Module:

```
$: hugo mod init
```

Simply add the following map to your project's `config.yaml`:

```
module:
  imports:
  - path: github.com/theNewDynamic/hugo-module-imgix
```

## Usage

### GetImageSRC

The only available public returning partial is `GetImageSRC`.
This take for argument either:

- A file path | String (.)
  With the above, the function returns the file path prefixed with the imgix domain.
- A map | Map (.)
  src: A file path | String
  ...any other key will match a transformation from the imgix API and its value the property to be passed.
  With the above, the function returns the file's imgix URI including the transformation query.

#### Examples

{{ $src := "/uploads/an-image.jpg" }}
{{ with partial "func/GetImageSRC" $src }}
  {{ $src = . }}
{{ end }}

Will produce: `https://imgix.project.net/uploads/an-image.jpg`

```
{{ $src := "/uploads/an-image.jpg" }}
{{ $args := dict "src" $src "width" 1024 "height" 100 }}
{{ with partial "func/GetImageSRC" $args }}
  {{ $src = . }}
{{ end }}
```
Will produce: `https://imgix.project.net/uploads/an-image.jpg?w=1024&h=100`

### Settings

Settings are added to the project's parameter under the `imgix` map:

```yaml
# config.yaml
params:
  imgix:
    domain: imgix.project.net
```

#### Domain

This is the imgix domain of your project. This will be used to transform file paths to imgix URIs.

### IMGIX API mapping.

To abstract the Imgix terminology, the module interprets commonly used transformation keys. User can therefor use `width` and expect the function to find the proper Imgix key (`w`).

Key absent from mapping setting will be passed as is.

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
  imgix:
    domain: imgix.project.net
    mapping:
      pixel: 'dpr'
```


Given the example above, the following arguments...
```
{{ $src := "/uploads/an-image.jpg" }}
{{ $args := dict "src" $src "width" 1024 "pixel" 2 "ch" "Width,DPR" }}
```

Will produce: `https://imgix.[...]image.jpg?w=1024&dpr=2&ch=Width,DPR`

#### Defaults

User can add some sensible default transformations to the module. It will use them whenever a transformation query is passed on `GetImageSRC` function. 
If one of the default keys is passed, its value will overwrite default's.

```
params:
  imgix:
    defaults:
      auto: format
      ch: Width,DPR
      q: 95
```

Defaults can be passed using the mapped arguments (`width`, `quality` etc...)

### Safe integration

To easily transition in or away from the module, and simplify image src references in your project you should create the following partials:

```
{{/*
  GetSRC
  Fetches Image's SRC using imgix module if available

  @author @regisphilibert

  @use
    - imgix/GetImageSRC

  @context Map
      - String (.src)
        ...String/Int (of any name)

  @access public

  @return String

  @example - Go Template
  {{ $transformArgs := dict "src" "/uploads/some-image.jpg" "width" 1024 "max-h" 15 }}
  {{ with partialCached "GetSRC" $transformArgs $transformArgs }}
    {{ $src = . }}
  {{ end }}

*/}}

{{ $return := $ }}
{{ if templates.Exists "partials/imgix/GetImageSRC.html" }}
  {{ $return = partialCached "imgix/GetImageSRC" $ $ }}
{{ end }}

{{ return $return }}
```