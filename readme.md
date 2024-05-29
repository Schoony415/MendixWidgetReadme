# Pull up a seat

#### Documented by Josh Schoonover the Intern

This will be the documentation with links on creating a map widget in 2024 with all the difficulties of working with deprecated documentation.

## NPM and Node.js

For Node.js on Windows I have used the following installer:  
https://nodejs.org/en/download/prebuilt-installer/current

### Portable Node 
Skip this section if the above works for you.  
If you don't have admin, you will be required to use a portable version. I have had the best of luck with this:

Download Linke:
https://github.com/crazy-max/nodejs-portable/releases/download/2.10.0/nodejs-portable.exe

Just the Hub:
https://github.com/crazy-max/nodejs-portable

Follow the guide, or just create and empty folder in `Docs` and run it from inside that folder. It will create all the relevant items inside that folder.

I have found mixed success in editing the `.config` to address my file paths as such:
```json
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

If npm runs out of ram while trying to run you can use `--max_old_space_size=#` to increase it. I have not successfully found a place to put this command that will increase it past 1gb of ram. It seems to be a built in cap to the portable version. I have attached a error below showing the max bound that you may use. It is supposed to be in mb, however it was acting as though it was using bytes as the number.

```shell
Error: Value for flag --max_old_space_size=5120000000 of type size_t is out of bounds [0-4294967295]
```

When creating the Map widget, because of the size of the ESRI modules I did have to get an admin to put node.js on my computer. When compiling the widget, the process would peak at 5gb of ram. I suspect this is solely due to ESRI's package size.

## Git

If you are going to be using git to track changes, the desktop app works well for windows. You can find it here:  
https://desktop.github.com/

## Getting setup with creating a widget  
### Guides I started with  

> #### Quick Note
> When you are using portable NPM you can't run `npx` because it isn't able to be overridden. You must use `npm exec` and tag in your global. The example will be 
> ```shell
> npm exec @mendix/generator-widget WidgetName --globalconfig $PREFIX/etc/npmrc
> ```
> reference [here](https://docs.npmjs.com/cli/v7/commands/npx)

#### Where to get the Mendix Template
There is a Widget Generator on npm [here](https://www.npmjs.com/package/@mendix/generator-widget) that will get you started. Use `npx` or `npm exec` as your situation allows.  
Give your app a good name, and Bippity Boppity Boop you are off to the races. `cd` into your new folder and get started.

> #### note
> I used typescript, functional, minimal boiler plate, tests, no end to end tests

## React Structure

React is just a way to arrange the javascript. It wants an `index` or similar file to be the entry point for the program/script/widget. You should keep it simple at the index level. Then you will have your backbone component that index calls. It will be where you put in most of your top level variables and call upon the smaller components that your program has. 

## Typescript

TS allows you to add typing to standard javascript. [Typescript website has a good writeup for coming from JS](https://www.typescriptlang.org/docs/handbook/typescript-in-5-minutes.html).

## Rollup

Rollup is what is the library that comes with Mendix widgets to orchestrate and compile the code into a widget format. It uses its own plugins to get some of the jobs done. I suggest messing with it as little as possible until things don't compile because of rollup errors. Once you start overwriting it, you will have to put in a lot of work.

https://dev.to/proticm/how-to-setup-rollup-config-45mk  
https://rollupjs.org/configuration-options/  

Inside your `rollup.config.js` file:  
```javascript
export default args => {
    const result = args.configDefaultConfig;
    console.warn ('Custom roll up')
    
    return result.map((config) => {
      /* Your pre-plugin changes go here*/
      const plugins = config.plugins || []
      config.plugins = [
        ...plugins,
        /* your plugin changes go here */
      ]
      return config;
    });
};
```

## Adjusting the XML to talk with Mendix

You can create attributes and data inputs using the `./src/appName.xml`. It can be picky but you can nest `<propertyGroup>` to create some nice menus in Mendix. Roughly speaking you will only be given feedback on format of properties when you try to put your widget into mendix. 

To link a variable and then the attributes on that variable can be complex, the key is using the `dataSource=nameofdatasource` option on a property to link back to the `type=datasource`. I have found it useful to get ahead of errors by putting datasourced friends into one property group. 

```xml
<propertyGroup caption="Big Data">
  <property key="ListOfPoints" type="datasource" isList="true" required="false">
    <caption>Point Layer List</caption>
    <description/>
  </property>

  <property key="PointName" type="attribute" required="false" dataSource="ListOfPoints">
    <caption>Point Names</caption>
    <description>The name to tie to your points</description>
    <attributeTypes>
      <attributeType name="String"/>
    </attributeTypes>
  </property>
</propertyGroup>
```

### Prop Propagation

From the xml, a type file will be autogenerated upon compile. From there you can pull in your new variables. 

```typescript
export interface WidgetNameProps {
  ListOfPoints?: ListValue;
  PointName?: ListAttributeValue<string>;
}
```

```typescript
export function WidgetName({
ListOfPoints,
PointName
}:WidgetNameProps): ReactElement{
  /*code*/
}
```

## Pulling and Pushing Data to Mendix  
### Using getters to grab your data  

There is two ways to handle the Mendix data in ts, in both cases you should check the status of the data before you grab it. I found useEffect the best way to complete this.

If you were given a list from Mendix you will use the attribute to access a property of that list like so:
```typescript
const variableToUseTheData: {name:string}[] = []

useEffect(()=>{
  if(ListOfPoints.status !== "available"){return;}
  /*
    put in some logic to check your variableToUseTheData and the ListOfPoints
    The following bit assumes you just want it all.
  */
  ListOfPoints.items!.forEach((p)=>{
    variableToUseTheData.push({name: PointName!.get(p).value!})
  })
},[ListOfPoints, ListOfPoints.status])
```
In the above code I am telling typescript that I am guaranteeing  values in places that I am using `!`. You should put in a few if checks to make sure the user has infact filled in your fields on the Mendix side.

For getting from just a base type will look similar. With a base type you don't need to check availability.

```typescript
// pulling in prettyText from Mendix as string
const [textToStyle, setTextToStyle] = useState<string>();

useEffect(()=>{
  setTextToStyle(prettyText)
},[prettyText])
```

### Using Setters to give back to Mendix

To give your data back to Mendix, the best way is to use a state that and then wait for Mendix to be ready. We will once again check for status to be available. It will look a lot like the getter.

Below will be an example of multiple grouped attributes to a no persistent Mendix object, so not a list. We use `Partial<>` because we will be undefining the items as we set them.

```typescript
export function WidgetName({
  EventName // EditableValue<string>;
  EventTime // EditableValue<Big>;
}:WidgetNameProps): ReactElement{

  const [holdthis, setHoldthis] = useState<
    Partial<{name: string, time: number}> & { doAction: boolean }
  >();

  useEffect(()=>{
    if(!(holdthis && hodlthis.name && EventName.status === "available")){return;}

    EventName.setValue(holdthis.name)
    setHoldthis({...holdthis, name: undefined})
  
  },[ holdthis?.name && EventName] )

  /*
    Code that will setup holdthis when we want data to go back to Mendix
  */
}
```

I have not found a good way to set an item in a Mendix list yet.

### Having a callback function

If you want a widget to setoff a flow on Mendix, it's fairly simple. 

```xml
<property key="callBackAction" type="action" required="false">
  <caption>After Action</caption>
  <description>If a Mendix action should happen after a doing a thing</description>
</property>
```

On the Mendix side you can load several things into an action. I just used a Microflow

```typescript
callBackAction?: ActionValue;
```

Typescript will call this new friend an `ActionValue` which only has a few properties on it. Online says that the `isExecuting` can be unreliable because of the complexity of what might actually be happening on the Mendix side.  
We will treat the callback similar to our getters and setters. 

```typescript
const [doAction, setDoAction] = useState<boolean>(false);

useEffect(()=>{
  // first check if you want to do the thing and the thing is ready to do
  if(!(doAction && callBackAction && callBackAction.canExecute)){return;}
  // set your state back, and kick off to Mendix
  setDoAction(false)
  callBackAction.execute()
},[doAction, callBackAction?.canExecute])

// code that will want to queue up my action to mendix
setDoAction(true)
```



## Mindex data types for javascript

https://docs.mendix.com/apidocs-mxsdk/apidocs/pluggable-widgets-property-types/#datasource

https://docs.mendix.com/apidocs-mxsdk/apidocs/pluggable-widgets-client-apis-list-values/#listvalue

https://docs.mendix.com/refguide/data-sources/#list-widgets

## Guides from a rather smart fellow:  

The original widget:  
https://medium.com/mendix/how-i-rewrote-the-mendix-arcgis-widget-in-react-typescript-4cf2158c2267  
https://github.com/ivosturm/ArcGIS  

https://medium.com/mendix/build-widgets-in-mendix-with-react-part-1-colour-counter-f1e400c3cdff
 
https://www.mendix.com/blog/build-widgets-in-mendix-with-react-part-2-timer/
 
https://www.mendix.com/blog/build-widgets-in-mendix-with-react-part-3-kanban/
 
https://www.mendix.com/blog/build-widgets-in-mendix-with-react-part-4-arcgis-maps/
 
https://www.mendix.com/blog/build-widgets-in-mendix-with-react-part-5-running-webassembly-in-mendix/

## Learn about layers in arc

https://developers.arcgis.com/javascript/latest/api-reference/esri-layers-Layer.html

https://developers.arcgis.com/javascript/latest/tutorials/add-a-point-line-and-polygon/

https://developers.arcgis.com/javascript/latest/api-reference/esri-layers-FeatureLayer.html

## Geocode links

reverse geocoding  
https://developers.arcgis.com/documentation/mapping-apis-and-services/geocoding/reverse-geocoding/  
tutorial  
https://developers.arcgis.com/javascript/latest/tutorials/reverse-geocode/  
react arc map  
https://developers.arcgis.com/javascript/latest/components-get-started-react/  
template?  
https://github.com/Esri/arcgis-maps-sdk-javascript-samples-beta/tree/main/packages/map-components/templates/react  

## To learn more about mapping

Lat Lon precision  
https://gis.stackexchange.com/questions/8650/measuring-accuracy-of-latitude-and-longitude  
https://www.explainxkcd.com/wiki/index.php/2170:_Coordinate_Precision  

```typescript
```