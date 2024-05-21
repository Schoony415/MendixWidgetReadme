# Pull up a seat
#### Documented by Josh Schoonover the Intern
This will be the documentation with links on creating a widget in 2024 with all the difficultys of working with depercated libraries.

## NPM and Node.js

If you don't have admin, you will be required to use a portable version. I have had the best of luck with this:

Download Linke:
https://github.com/crazy-max/nodejs-portable/releases/download/2.10.0/nodejs-portable.exe

Just the Hub:
https://github.com/crazy-max/nodejs-portable

Follow the guide, or just create and empty folder in `Docs` and run it from inside that folder. It will create all the relevant items inside that folder.

I have found mixed success in editing the `.config` to address my file paths as such:
``` json
  "customPaths": [
    "C:/Users/*user*/Documents/PortableNode",
    "C:/Users/*user*/Documents/Widgets"
  ]
```

To get around npm wanting to use different locations for where to find node, you must always point it at the "local".  
This can be checked with `> npm config ls -l`. Check the variables that say they can be overwritten by `global`, and check the file path of `$PREFIX`, it should match your documents folder.  
To properly point your npm use on every command:
```shell
npm *commands* --globalconfig $PREFIX/etc/npmrc
```
Reference can be checked at the npm [page here](https://docs.npmjs.com/cli/v10/using-npm/config)

When creating the Map widget, because of the size of the ESRI modules I did have to get an admin to put node.js on my computer. When compiling the widget, the process would peak at 5gb of ram. 

## Getting setup with creating a widget
### Guides I started with

> #### Quick Note
> When you are using portable NPM you can't run `npx` because it isn't able to be overridden. You must use `npm exec` and tag in your global. The example will be 
> ```shell
> npm exec @mendix/generator-widget WidgetName --globalconfig $PREFIX/etc/npmrc
> ```
> reference [here](https://docs.npmjs.com/cli/v7/commands/npx)

#### Where to get the Mindex Template
There is a Widget Generator on npm [here](https://www.npmjs.com/package/@mendix/generator-widget) that will get you started. Use `npx` or `npm exec` as your situation allows.  
Give your app a good name, and Bippity Boppity Boop you are off to the races. `cd` into your new folder and get started.

> #### note
> I used typescript, functional, minimal boiler plate, tests, no end to end tests

## React Structure

React is just a way to arrange the javascript. It wants an `index` or similiar file to be the entry point for the program/script/widget. You should keep it simple at the index level. Then you will have your backbone component that index calls. It will be where you put in most of your top level variables and call apon the smaller componets that your program has. 

## Typescript

TS allows you to add typing to standard javascript. [Typescript website has a good writeup for coming from JS](https://www.typescriptlang.org/docs/handbook/typescript-in-5-minutes.html).

## Rollup

Rollup is what is the library that comes with Mendix widgets to orchistrate and compile the code into a widget format. It uses it's own plugins to get some of the jobs done. 

https://dev.to/proticm/how-to-setup-rollup-config-45mk  
https://rollupjs.org/configuration-options/  

## Learn about layers in arc

https://developers.arcgis.com/javascript/latest/api-reference/esri-layers-Layer.html

https://developers.arcgis.com/javascript/latest/tutorials/add-a-point-line-and-polygon/

https://developers.arcgis.com/javascript/latest/api-reference/esri-layers-FeatureLayer.html



## Hold this

`npm run build --globalconfig $PREFIX/etc/npmrc`

`--max_old_space_size=4096`
`set NODE_OPTIONS=--max_old_space_size=8192`
setx NODE_OPTIONS --max-old-space-size=8192

4096000000
4294967294
```shell
Error: Value for flag --max_old_space_size=5120000000 of type size_t is out of bounds [0-4294967295]
```


## Geocode links

reverse geocoding  
https://developers.arcgis.com/documentation/mapping-apis-and-services/geocoding/reverse-geocoding/  
tutorial  
https://developers.arcgis.com/javascript/latest/tutorials/reverse-geocode/  
react arc map  
https://developers.arcgis.com/javascript/latest/components-get-started-react/  
template?  
https://github.com/Esri/arcgis-maps-sdk-javascript-samples-beta/tree/main/packages/map-components/templates/react  

## Guides from a rather smart fellow:  

The original widget:  
https://medium.com/mendix/how-i-rewrote-the-mendix-arcgis-widget-in-react-typescript-4cf2158c2267  
https://github.com/ivosturm/ArcGIS  

https://medium.com/mendix/build-widgets-in-mendix-with-react-part-1-colour-counter-f1e400c3cdff
 
https://www.mendix.com/blog/build-widgets-in-mendix-with-react-part-2-timer/
 
https://www.mendix.com/blog/build-widgets-in-mendix-with-react-part-3-kanban/
 
https://www.mendix.com/blog/build-widgets-in-mendix-with-react-part-4-arcgis-maps/
 
https://www.mendix.com/blog/build-widgets-in-mendix-with-react-part-5-running-webassembly-in-mendix/

## To learn more about mapping

Lat Lon percision  
https://gis.stackexchange.com/questions/8650/measuring-accuracy-of-latitude-and-longitude  
https://www.explainxkcd.com/wiki/index.php/2170:_Coordinate_Precision  

## Mindex data types for javascript

https://docs.mendix.com/apidocs-mxsdk/apidocs/pluggable-widgets-property-types/#datasource

https://docs.mendix.com/apidocs-mxsdk/apidocs/pluggable-widgets-client-apis-list-values/#listvalue

https://docs.mendix.com/refguide/data-sources/#list-widgets

