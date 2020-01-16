# Template Schema 

# Background

Template Generator is used for creating different inv layouts that can be used for the Marketplace and for billing of core services, allowing it to be fully branded and supporting mapping of database fields relevant for presentation on an inv. The inv templates will be defined using JSON format with pre-defined properties. 

# Goal
Define the JSON structure of inv template to support different inv layouts and formats. The JSON structure should be extensible, in that it has the ability to support more complex layouts over time as we gather feedback from customers.

# Design
Rows and cells act as containers for the actual data. Rows are vertically stacked while cells are horizontally stacked. All the elements added to the cells are implicitly stacked vertically. Rows cannot directly contain data components. They are implicit to the cell and are mainly used to layout data into different columns. A Cell is always wrapped within a Row and cannot have another Cell as its direct child. All of this is represented using a JSON structure. This structure is built to contain different components of the inv.

# Components

Component can be added to define the layout or the content of the template. For example Text, Field (Combination of label and value), Page Number, etc.

Component will have properties to define -

- UI aspect (Styles)
- Layout (Full-width,2-Columns)
- Data binding (via Tokens)

Components can be broadly classified into following categories -

* Elementary Component / Data Component:
   -  Defines the elementary UI component like Text, Image etc
   - Is atomic and cannot contain any child component.
* Layout Component:
   - Defines the container with implicit layout like Columns .  
   - It can hold other UI Components(Elementary, Container)

```

"component": {
    "data" : "#data-binding-expression",
    "type" : "#component-type",
    "description" : "Component Metadata",
    "properties" : {
        "visible": [true, false],
        "src" : "http://foo"
    },
    "style": {
        "margin": "0 0 0 0",
        "border": "1px solid #000000",
        "border-radius": "5",
        "padding": "0 0 0 0",
        "background-image": "url('')",
        "background-color": "#RRGGBB",
        "height": "100px",
        "width": "200px",
        "font-family": "Helvetica",
        "font-size": "12px",
        "color": "#RGB",
        "position": "[relative]",
        "align":"[left, right, center]",
        "opacity": "0.0..1.0"
    }
}
```

Not all the properties are applicable for all the components. Eg. Row and Cell components do not have data property but they have 'components' where we can add other layout and data components. Similarly, data components do not 'components' property.

Below are the examples of text component and image component. Text component contains data but image does not. Instead it has a 'src' attribute defined in 'properties' which accepts URL of the image -
Text JSON
```
{
  "type": "text",
  "properties": {
     "visible" : "true,
   },
  "style": {
    "color": "#333"
    "font-size": "12px"
  }
  "data": "Hello {{customer-name}} invoice on ${invoice-due-date}",
}
```
Image JSON -
```
{
  "type": "image",
  "properties": {
    "visible" : "true,
    "src": "http://path-2-img"
  },
  "style": {
    "height": "100px",
    "width": "200px"
  }
}
```
