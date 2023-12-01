# Introduction

This Javascript library provides support for a chemical composition input type with the following features:
- Periodic table based element picker
- Support for formula, atomic percentage, or weight percentage display
- Automatic coversion between display formats
- Support for user defined pallette of "favourite" compounds
- Formulas stored as simple human readable strings suitable for storage in standard database string field

# Formula Storage

The code uses the following syntax to load and save chemical formulae
```
<symbol><ratio - can be floating point>,<symbol><ratio>,... [display format]
```
__N.B. The amount of each chemical are always specified as floating point atomic ratios even if the specied display format is weight percentage__

- The display format can be either "formula" "atomic" "weight" these also be written as "formula" "atomic%" "weight%" or "formula" "at%" "wt%". If this is ommitted "formula" is assumed.
- If the display format is set to "weight" (or "wt%") the ratios will be treated as weight ratios. If format is ommitted, or set to formula or atomic, then the ratios will be treated as atomic.
- Commas separating each element-number pair are optional.

## Examples
```
H2O
```
```
H2,S1,O4 formula
```
```
Ti88.75,V4.5,Al6.75 wt%
```
```
Ti83.38V11Al3.4Fe2.22 wt%
```
In the example above the optional commas have been omitted. 

# API

```
// All options are optional
let chemicalInput = new chemicalInput({
    'favouritesSaveHandler' : function(favouriteName, formulaString) {
        // returns boolean true on success
        // returns error message string on failure
    },
    'favouritesLoadHandler' : function(){
        // returns an object consisting formula strings keyed on favourite name
    },
    'inputs' : [ <DOM element>, <DOM element> ... ]
}]
// If save or load handler is missing then favourites interface will be hidden

// To add another Dom element(s) call the add method
chemicalInputs->add(<DOM element>);
// or
chemicalInputs->add([<DOM element>, <DOM element> ... ]);
```

An array of picker elements can be passed when the periodicTablePicker object is created (see 'inputs' option above), or the elements can be passed in later by calling the add method.

The code will also look for all elements of type="chemical" or with a data-type="chemical" attribute. These will be instantiated as chemical inputs on page load and the original inputs converted to hidden fields.

You can either define your own load and save handler or you can use one of the supplied ready made handlers.
There are 2 ready made handlers:

## inputChemicalFavourites_local.js
This defines two functions: inputChemicalFavourites_local_loadHandler and inputChemicalFavourites_saveHandler. These can be passed as options when instantiating the chemicalInput object (see example below).

These read from and write to browser local storage. On the plus side this requires no server-side support, but the down side is that if the user switches to a different browser their favourites will not be available.

### Example Usage
```
<script src="src/inputChemical.js"></script>
<script src="src/inputChemicalFavourites_local.js"></script>

<script>
// This will automatically find any inputs with type="chemical" or data-type="chemical" attributes and convert them into hidden inputs.
let picker = new periodicTablePicker({
    'favouritesSaveHandler' : inputChemicalFavourites_local_saveHandler,
    'favouritesLoadHandler' : inputChemicalFavourites_local_loadHandler
});
</script>

<input type="chemical" name="myChemical" />
```

## inputChemicalFavourites_formField.js

This defines two functions: inputChemicalFavourites_formField_loadHandler and inputChemicalFavourites_formField_saveHandler. These can be passed as options when instantiating the chemicalInput object (see example below).

These functions will read the favourites from and write them to a form field value. The functions are not actually the handlers, but rather functions that take the form field as a parameter and return a function that is the handler configured to write to the specified form field. The form field can be specified either using a CSS selector, or a DOM object. Obviously the same form field should be used for the load and save handlers.

This provides a simple way (without using AJAX), of storing the data on the server side assuming of course that there is a form on the page that will be submitted back to the server. 

The form field used to store favourites can be hidden, text input, or textarea. If a textarea of input textbox is used then this will be hidden when the periodic table picker fires up.  Whenever favourites change, the updated favourites data will be written to this form field.

The favourites will be encoded one-per-line thus (see section about about formula storage syntax):
```
<favouriteName>: <formula> <display format>
<favouriteName>: <formula> <display format>
...
```
e.g.

```
Water: H2,O formula
Sulphuric acid:
Ti-6Al-4V: Ti88.75,V4.5,Al6.75 wt%
Ti-10V-2Fe-3Al: Ti83.38,V11,Al3.4,Fe2.22 wt%
...
```

### Example Usage
```
<script src="src/inputChemical.js"></script>
<script src="src/inputChemicalFavourites_formField.js"></script>

<script>
// This will automatically find any inputs with type="chemical" or data-type="chemical" attributes and convert them into hidden inputs.
let picker = new periodicTablePicker({
    'favouritesSaveHandler' : inputChemicalFavourites_formField_saveHandler('#favouriteStore'),
    'favouritesLoadHandler' : inputChemicalFavourites_formField_loadHandler('#favouriteStore')
});

<form method="post" action="processForm.php">
    ...
    <input type="chemical" name="myChemical" />
    <input id="favouriteStore" type="hidden" name="myFavouriteChemicals" />
    ...
</form>
</script>
```
