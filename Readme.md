Microsoft Fabric offers a suite of handy tools to accelerate the implementation of "near real-time" projects.

In this article, we'll look at an example of how to implement these tools to set up a "near real-time" Power BI report from the data in this [website](https://bctransit.com/open-data). More precisely, we will use the data found [here](https://bct.tmix.se/gtfs-realtime/vehicleupdates.js?operatorIds=12).

Below is an overview of the solution:

![Overview](/Pictures/001.jpg)



# Prerequisites

- One [Fabric Capability](https://learn.microsoft.com/fr-fr/fabric/enterprise/buy-subscription#buy-an-azure-sku)
- One [Azure subscription](https://azure.microsoft.com/en-ca/free/)
- An [Azure Logic Apps deployed] (https://learn.microsoft.com/fr-fr/azure/logic-apps/logic-apps-overview#get-started)





# Microsoft Fabric
## KQL Database

In order to store data, we're going to retrieve and create "near real-time" reports, the KQL database is the ideal candidate here.

To get started, sign in to the [Microsoft Fabric](https://fabric.microsoft.com) portal.

[Create a new workspace](https://learn.microsoft.com/fr-fr/fabric/get-started/create-workspaces) in order to deploy the various services.

Once in your newly created workspace, choose the "Real-Time Analytics" persona.

![Persona](/Pictures/002.jpg)

Click on the "+ New" button and then on "KQL Database".

![KQL](/Pictures/003.png)

Give your KQL database a name and click on the "Create" button.

![Create](/Pictures/004.png)

Once you've created your KQL, you should get a window similar to the one below:

![Create](/Pictures/005.png)

## Eventstream

From your workspace, click on the "+ New" button and then on "Eventstream".

![Eventstream](/Pictures/006.png)

Give your "Eventstream" a name and click "Create".

![Eventstream](/Pictures/007.png)



### Creating the Event Entry

A window similar to the one below should open. Click on "New source" and then on "Custom App".

![Input](/Pictures/008.png)

On the right, a pane should open. Give the source a name. Click on the "Add" button.

![Eventstream](/Pictures/009.png)

Once your entry is created, click on it. In the lower middle part of the screen, click on "Keys".

Note the values for the following items that we'll need later:
- Event hub name
- Primary Key

![Eventstream](/Pictures/010.png)

Before we go any further with Microsoft Fabric Eventstream, we need to send data into the ingestion engine in order to maintain the data schema.

In our example, we're going to deploy an Azure Logic Apps to retrieve the data and send it to Microsoft Fabric Eventstream. (it is of course possible to use other solutions such as "Azure Functions").


# Azure
# Azure logic apps

From the [Azure portal](https://portal.azure.com), go to your Azure logic app service.
In the left panel, click on "Workflows" and then on the "Add" button, give your workflow a name and select "Statefeul..." then click "Create".

![LogicApps](/Pictures/015.png)

Once your workflow is created, click on it.

![LogicApps](/Pictures/016.png)

Click on "Designer" and then on "Add a trigger".

![LogicApps](/Pictures/017.png)

Since we want to recover the data every 30 seconds, we will choose a recurrence trigger. 
In the search box, enter "recurrence" and then select the "Schedule/Recurrence" trigger.

![LogicApps](/Pictures/019.png)

Set the parameters for your trigger. Here I'm asking for a trigger every 30 seconds.

![LogicApps](/Pictures/020.png)

Under the "Recurrence" trigger, click on the "+" sign, search with the keyword "https" and then choose the "HTTP" action.

![LogicApps](/Pictures/021.png)

In the "Parameters" tab, enter the following values for the fields:
- **URI**: https://bct.tmix.se/gtfs-realtime/vehicleupdates.js?operatorIds=12
- **Method**: GET

![LogicApps](/Pictures/022.png)

Below the "HTTP" action, click on the "+" sign, search for "Event hub" and select "Event Hubs / Sent **E**vent"

![LogicApps](/Pictures/023.png)

If no connection exists, you should have the following window:

![LogicApps](/Pictures/024.png)

The information about the "Connection string" can be found in the entry of your Microsoft Fabric Eventstream (the information was noted earlier in this article).
Click on the "Create New" button.

The following window will appear. Fill in the name of your Eventstream in the "Event Hub Name" field. From the "Advanced parameters" drop-down list, select "Content".

![LogicApps](/Pictures/025.png)

Click in the "Content" field to make the little blue bubble appear. Click on the lightning bolt. 

![LogicApps](/Pictures/026.png)

Then, under the "HTTP" action, select "Body".

![LogicApps](/Pictures/027.png)

You should have a window similar to the one below:

![LogicApps](/Pictures/028.png)

Click on the "Save" button.

![LogicApps](/Pictures/029.png)


In order to check that everything is going well, after saving your workflow, click on "Overview" and check that your "workflow" is triggered every 30 seconds and that no errors have been reported in the "Status" column.

![LogicApps](/Pictures/030.png)



# Microsoft Fabric (continued)
 
After you've created your Azure Logic Apps workflow, go back to Microsoft Fabric, and more specifically to your Eventstream. 
Click on your Evenstream to verify that the events are coming correctly.



![LogicApps](/Pictures/031.png)

### Creating the Event Destination

Click on "New destination" and then on "KQL Database".

![Output](/Pictures/011.png)

Choose "Direct ingestion" (In case you want to do transformations before sending to the destination, you have the option to choose the "Event processing before ingestion" option). 
Then fill in the information in your KQL database. Click on "Add and configure".

![Output](/Pictures/012.png)

Now, we're going to define which table to send the data to. Click on "New table".

![Picture](/Pictures/013.png)

Give your table a name and then click "Next".

![Picture](/Pictures/014.png)

In the "Inspect the data" step, the wizard should be able to find a sample of the data, click on "Finish".

![Picture](/Pictures/032.png)

Then during the "Summary" step, click on "Close".

![Picture](/Pictures/033.png)

Your ingestion process is now ready!

![Picture](/Pictures/034.png)



## KQL Database

If all goes well, your table must have been created in your KQL database. Below you can see my "FranmerBronze" table. Click on the 3 small dots to the right of the table to get the command "Show any 100 records".

![Picture](/Pictures/035.png)

The "Explore your data" window will open and give you an overview of the data that arrives in your table.

![Picture](/Pictures/036.png)

As it stands, the data is not easily usable for the creation of a "near real-time" report. This is where the Kusto language comes into play.


### Kusto Query (KQL Queryset)

Thanks to the Kusto language, it's relatively simple enough to create a script that makes the data more usable. 
We're going to use the "KQL Queryset" feature to write our query.

From your workspace, click on the "New" button and then on the "KQL Queryset" button.

![Picture](/Pictures/037.png)

Give your "Queryset" a name and click "Create".

![Picture](/Pictures/038.png)

Select your KQL database and click on the "Connect" button.

![Picture](/Pictures/039.png)

You should have a window similar to the one below:

![Picture](/Pictures/040.png)

Clear the sample scripts in the main window and copy the code below:


```java

.create-or-alter function with (docstring="A function to flatten the BCT JSON", folder="Transformation functions") FlattenBctJson(){
FranmerBronze
| extend MyIngestionTime = ingestion_time()
| extend gtfs_realtime_version    = toreal(header.gtfs_realtime_version)
        ,incrementality           = tostring(header.incrementality)
        ,incrementalitySpecified  = tobool(header.incrementalitySpecified)
        // Had to rename the next two fields due to duplicate names further down in the JSON
        // Note how we are converting the unix timestamp on the fly
        ,HeaderTimestamp          = unixtime_seconds_todatetime(tolong(header.timestamp)) 
        ,HeadertimestampSpecified = tobool(header.timestampSpecified) 
        // 
| mv-expand entity 
| extend id	                    = toint(entity.id)
        ,is_deletedSpecified	= tobool(entity.is_deletedSpecified)
        ,vehicle		        = entity.vehicle
        ,Alert                  = entity.alert
| project-away entity, header // cleaning up to free memory
| extend congestion_level               = tostring(vehicle.congestion_level)
        ,congestion_levelSpecified      = tobool(vehicle.congestion_levelSpecified)
        ,consist                        = vehicle.consist // This is an array but is empty now. We should ask if this need to be expanded eventually
        ,current_status                 = tostring(vehicle.current_status)
        ,current_statusSpecified        = tobool(vehicle.current_statusSpecified)
        ,current_stop_sequence          = toint(vehicle.current_stop_sequence)
        ,current_stop_sequenceSpecified = tobool(vehicle.current_stop_sequenceSpecified)
        ,occupancy_statusSpecified      = tobool(vehicle.occupancy_statusSpecified)
        ,position                       = vehicle.position
        ,stop_id                        = tolong(vehicle.stop_id)
        ,stop_idSpecified               = tobool(vehicle.stop_idSpecified)
        ,timestamp                      = unixtime_seconds_todatetime(tolong(vehicle.timestamp))
        ,timestampSpecified             = tobool(vehicle.timestampSpecified)
        ,trip                           = vehicle.trip
| project-away vehicle // cleaning up to free memory
// Unpacking the "position bag"
| extend Latitude           = toreal(position.latitude)
        ,Longitude          = toreal(position.longitude)
        ,Bearing            = toreal(position.bearing)
        ,BearingSpecified   = tobool(position.bearingSpecified)
        ,Odometer           = toreal(position.odometer)
        ,odometerSpecified  = tobool(position.odometerSpecified)
        ,speed              = toreal(position.speed)
        ,speedSpecified     = tobool(position.speedSpecified)
| project-away position // cleaning up to free memory
// Unpacking the Trip bag
| extend  direction_id                   = toint(trip.direction_id)
         ,direction_idSpecified          = tobool(trip.direction_idSpecified)
         ,route_id                       = tostring(trip.route_id)
         ,route_idSpecified              = tobool(trip.route_idSpecified)
         ,schedule_relationship          = tostring(trip.schedule_relationship)
         ,schedule_relationshipSpecified = tobool(trip.schedule_relationshipSpecified)
         ,start_date                     = tostring(trip.start_date)
         ,start_dateSpecified            = tobool(trip.start_dateSpecified)
         ,start_time                     = tostring(trip.start_time) // Not sure why they separate the two... will create new column to recombine
         ,start_timeSpecified            = tobool(trip.start_timeSpecified)
         ,trip_id                        = tostring(trip.trip_id)
         ,trip_idSpecified               = tobool(trip.trip_idSpecified)
| project-away trip // cleaning up to free memory        
// Now we reshuffle the ugly date and recombine the parts to build a proper ISO-8601 date
// Note how Kusto converts it to UTC as God intended!         
| extend TripStartDate = todatetime(
                               strcat(
                                        substring(start_date,0,4)
                                       ,'-'
                                       ,substring(start_date,4,2)
                                       ,'-'
                                       ,substring(start_date,6,2)
                                       , ' '
                                       ,start_time 
                                      )
                                   )
}

```

You should get something similar on the screen below:
![Picture](/Pictures/041.png)

This script will create a function with the name "FlattenBCTJson".
Click on the "Run" button.

![Picture](/Pictures/042.png)

If all goes well, you should get a result similar to the one below:

![Picture](/Pictures/043.png)



### Update policy

We will now use the "Update Policy" feature to execute the function when an event arrives in the "FranmerBronze" table.
Since the purpose of this function is to make the data easily usable for analysis or reporting, we'll keep the result of the function in a second table that I'll call "FranmerGold".

So, we're going to create a new table with the schema corresponding to the output of the previously created function. 
Click on the "+" button to create a new page.

![Picture](/Pictures/044.png)

Copy the following code and run it by clicking on the "Run" button (In case the Run button is grayed out, switch windows while staying within Microsoft Fabric by going to another artifact, then go back to your Queryset):



```java
//Create target table with schema only
.set-or-append FranmerGold <| FlattenBctJson | limit 0

```

![Picture](/Pictures/045.png)

You should get a second table as shown below:

![Picture](/Pictures/046.png)

Now we are ready to create our "Update Policy".
Copy the code below into a new window and run the code:



```java
//Create a table policy
.alter table FranmerGold policy update "[{\"IsEnabled\":true,\"Source\":\"FranmerBronze\",\"Query\":\"FlattenBctJson\",\"IsTransactional\":false,\"PropagateIngestionProperties\":true,\"ManagedIdentity\":null}]"

```

![Picture](/Pictures/047.png)

If all goes well, you should get a similar result:

![Picture](/Pictures/048.png)

Go back to your KQL database (You can use the "switches" on the left to quickly change artifacts). You should see your new table (click "Refresh" if not).
Query your new table to see the new data coming in.

![Picture](/Pictures/049.png)

Now that the data arrives in near real-time formatted to our liking, we're going to create a Power BI report directly from this table.
Click on the 3 small dots to the right of your table name and select "Build Power BI Report".

![Picture](/Pictures/050.png)

The Power BI report editor opens in another window. You can now create a report directly from the columns in the table. On the right, you will find all the useful columns to build your new report.


![Picture](/Pictures/051.png)

To ensure that the report responds in near real-time, **DON'T FORGET** to set the report to automatically update at the desired frequency.
Click a **blank** space in the report. Then click on the "Format" icon and then turn on "Page Refresh". Set the automatic update period. Here I choose 30 seconds.

![Picture](/Pictures/052.png)

All you have to do is build a nice almost real-time report!

# Kusto Tip

My colleague [Gilles L'herault](https://www.linkedin.com/in/gilleslherault/), suggested the following trick to generate a script to completely rebuild the KQL database in another environment. Just use the following command:

'''Java
.show database YourKqlDbName schema as csl script
'''

![Picture](/Pictures/053.png)