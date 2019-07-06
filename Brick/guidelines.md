How to Lay a BRICK
==================

A key concept is that few relations are needed to provide value, but for each added relation that value increases. In these guidelines we differentiate between a simple model and an extended model. The simple model represents a sensible minimum.

Relationships
-------------

Use hasPoint instead of isPointOf. Similar goes for the rest of the relations. This allows the preprocessor of [the application test script](/application_examples/RUN_APPS.py) to do less preprocessing and not be quite as slow.

Constructing a simple VAV
-------------------------

A VAV needs to be instantiated.

    VAV_1 a VAV

A VAV has a set of potential parts. Instances of such parts should be associated using the *hasPart* relationship.

    Supply_Damper_1 a Supply_Damper
    Return_Damper_1 a Return_Damper
    Cooling_Coil_1  a Cooling_Coil
    
    VAV_1 hasPart Supply_Damper_1
    VAV_1 hasPart Return_Damper_1
    VAV_1 hasPart Cooling_Coil_1

If one knows about sensors then they should be fully described through tags and *isPointOf* the VAV instance.

	Supply_Air_Temperature_Sensor_1 a Supply_Air_Temperature_Sensor
	Supply_Air_Temperature_Sensor_1 isPointOf VAV_1

Here, Supply_Air_Temperature_Sensor implies the tags Supply (the role), Air (the media), Temperature (the modality) and Sensor (the type). 

Extending the VAV Due to Increased Visibility
---------------------------------------------

If one has visibility into the internals of the VAV then the _feeds_ relationship may be used.

    Supply_Damper_1 feeds Cooling_Coil_1
    Cooling_Coil_1  feeds Supply_Air_Temperature_Sensor_1

The equipment class is relative to the output of its context. Due to its *partOf* relation to VAV_1 Supply_Air_Temperature_Sensor_1 thus measures the *temperature* of the *airflow* in the *supply* direction on the output side of VAV_1.

Extending the VAV Due to Increased Specificity
----------------------------------------------

If existing tagsets do not offer enough specificity then a subclass may be constructed. Imagine a temperature sensor -- within VAV_1 -- measuring the same the same the same (role,media,modality), but at the input side of the VAV. This sensor is on the output side of whatever is feeding VAV_1. For the sake of this example, lets call this an AHU.

    AHU_Supply_Air_Temperature_Sensor isSubClassOf Supply_Air_Temperature_Sensor
    AHU_Supply_Air_Temperature_Sensor_1 a AHU_Supply_Air_Temperature_Sensor
	AHU_Supply_Air_Temperature_Sensor_1 isPointOf VAV_1
	
    VAV_Supply_Air_Temperature_Sensor isSubClassOf Supply_Air_Temperature_Sensor
    VAV_Supply_Air_Temperature_Sensor_1 a VAV_Supply_Air_Temperature_Sensor
	VAV_Supply_Air_Temperature_Sensor_1 isPointOf VAV_1

For convenience, we can alias this to:

    Input_Supply_Air_Temperature_Sensor  hasSynonym AHU_Supply_Air_Temperature_Sensor
    Output_Supply_Air_Temperature_Sensor hasSynonym VAV_Supply_Air_Temperature_Sensor

Complex Entities
----------------

Certain equipments fit badly -- or not at all -- into this model. One example is the feeds relationships of a heat exchanger. A heat exchanger transfers heat between two flows without transferring matter between them. Essentially, it has 4 named ports. In reference to the primary and secondary flows, these are

1.    Primary Input
2.    Primary Output
3.    Secondary Input
4.    Secondary Output

It is impossible to distinguish between the primary and secondary flows if these are represented as a single entity. Instead, we define what is known as a functional block to group entities

    Function_Block                  isSubClassOf Thing
    Heat_Exchanger_Function_Block   isSubClassOf Function_Block
    Heat_Exchanger_Primary_Input    isSubClassOf Thing
    Heat_Exchanger_Primary_Output   isSubClassOf Thing
    Heat_Exchanger_Secondary_Input  isSubClassOf Thing
    Heat_Exchanger_Secondary_Output isSubClassOf Thing
    Heat_Exchanger_Primary_Input   feeds Heat_Exchanger_Primary_Output
    Heat_Exchanger_Primary_Input   feeds Heat_Exchanger_Secondary_Output
    Heat_Exchanger_Secondary_Input feeds Heat_Exchanger_Primary_Output
    Heat_Exchanger_Secondary_Input feeds Heat_Exchanger_Secondary_Output

and create an instance

    HX_1    a Heat_Exchanger
	HX_1_FB a Heat_Exchanger_Functional_Block
	HX_1_PI a Heat_Exchanger_Primary_Input
	HX_1_PO a Heat_Exchanger_Primary_Output
	HX_1_SI a Heat_Exchanger_Secondary_Input
	HX_1_SO a Heat_Exchanger_Secondary_Output

which we then tie together

    HX_1_FB isPartOf HX_1
    HX_1_PI isPartOf HX_1_FB
    HX_1_PO isPartOf HX_1_FB
    HX_1_SI isPartOf HX_1_FB
    HX_1_SO isPartOf HX_1_FB

This allows us to have feeds relationships on with HX_1_PI, HX_1_PO, HX_1_SI and HX_1_SO in the role of subject and object. 

Where Do We Go from Here?
-------------------------

Missing Tags, TagSets and Relations may be added [here](https://docs.google.com/spreadsheets/d/1QTSu0UxJ7UqRvgTW2P1Q4qudoBbvPqXpEhYiulyjcro/edit#gid=0). This spreadsheet is then exported to XL and processed by an IronPython notebook into the /BuildingSchema/Brick*.ttl files. 

The current schema layout is:

-   Tags are in [/BuildingSchema/BrickTag.ttl](/Brick/BrickTag.ttl)
-   TagSets are in [/BuildingSchema/Brick.ttl](/Brick/Brick.ttl)
-   Relations are in [/BuildingSchema/BrickFrame.ttl](/Brick/BrickFrame.ttl)
