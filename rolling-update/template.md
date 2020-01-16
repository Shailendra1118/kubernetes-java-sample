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

