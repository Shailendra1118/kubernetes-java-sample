# Token

Define a list of tokens that are to be used in the JSON. 
Tokens are basically used for binding the invoice data dynamically within the generated HTML. 

System defines a list of supported domain tokens which align with the taxonomy supported in the 'Reporting' domain model. 
Mustache (https://www.npmjs.com/package/mustache) library has been used to resolve the data. 
This library expects the tokens to be wrapped inside '{{' and '}}' to resolve their value against the given data.

All the tokens listed below can be used as data inside any component. 
Table component is a special case where it requires token that resolves to collection of data. Eg. Invoice lines, list of taxes, usage lines.
Table component will not accept any data that is not a collection.

