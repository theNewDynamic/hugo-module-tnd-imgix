# Imigx Hugo Module

## Spec

The component should be used on any project using IMGIX as an image CND and transforming API.

### GetImageSRC

For now I can think of only one public returning partial: `GetImageSRC`.
This would take the following argument:

1. A file path | String (.)
If the filepath does not include the IMGIX domain, we add it.
2. A map | Map (.)
  src: A file path | String
  ...any other key will match a transformation from the imgix API and its value the property to be passed.

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

### Defaults

User can add some sensible default transformation to the module. It will use them whenever a transformation query is passed on `GetImageSRC` function. If given key is passed, it will be overwritten. 

### IMGIX API mapping.

To abstract the Imgix terminology, most commonly used transformation keys should be interpreted by the module so user can use `width` and expect the function to find the proper Imgix key.

User can also add transformation key absent from the data file, the function will treat it as is.

With the following mapping:
```
width: 'w'
height: 'h'
quality: 'q'
```

The following arguments:
```
{{ $src := "/uploads/an-image.jpg" }}
{{ $args := dict "src" $src "width" 1024 "height" 100 "ch" "Width,DPR" }}
```

Will produce: `https://imgix.project.net/uploads/an-image.jpg?w=1024&h=100&ch=Width,DPR`

The API mapping will be stored as a data file. This has the benefit of being extended by an homonymous data file stored on the project level while a site
