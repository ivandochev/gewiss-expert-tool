# GET (GEWISS Expert Tool)
The GEWISS Expert Tool (GET) is a simulation tool developed by [Thomas Preisler](mailto:thomas.preisler@haw-hamburg.de) and [Nils Weiss](mailto:nils.weiss@haw-hamburg.de) from the [HAW Hamburg](https://www.haw-hamburg.de) as part of the name giving [GEWISS](http://gewiss.haw-hamburg.de/) Project. The data model as well as the actual data is provided by  [Ivan Dochev](mailto:ivan.dochev@hcu-hamburg.de) from the [HCU Hamburg](https://www.hcu-hamburg.de/). Additional contributers are [Arjun Jamil](mailto:arjun.jamil@haw-hamburg.de) (HAW Hamburg, simulation logic and UI/UX design), Ev Köhler (HAW Hamburg, simulation logic) and [Hannes Seller](mailto:hannes.seller@hcu-hamburg.de) (HCU Hamburg, simulation logic). The project is lead by [Wolfgang Renz](mailto:wolfgang.renz@haw-hamburg.de) (HAW Hamburg) and [Irene Peters](mailto:irene.peters@hcu-hamburg.de) (HCU Hamburg).

The GET-Icon is made by [Freepik](www.freepik.com) from www.flaticon.com.

Among other things the project aims at simulating the development of the building stock in the city of Hamburg (Germany) until 2050. GET intends to let expert users, like city planners, simulation different development scenarios for a given building stock under certain conditions. These user can define certain conditions for the building stock development and then simulate how these conditions might influence the building stock in the future. Thereby, the users can get an idea about what measuresments are needed to reach certain goals regarding the reduction of heat demand and CO<sub>2</sub> emission.

The tool consists of two projects `get-simulator` contains the simulation logic, the data model and the required input data while `get-ui` contains everything related to the user interface. Both projects are realizes using Java 8 and the UI is build using the JavaFX Framework.

***Remark: This is not the repository used  for the actual development of the tool. Due to privacy reasons, we're not allowed to share all of our input data with the general public (see section Building Input Data). We use a private repository for the actual development and use this one to share the source code without the sensitive input data with the public. Therefore, only major updates are commited to this repository.***

The following describes briefly the simulation approach, the building input data and how to build and run the application on your computer.

## Simulation Approach
The simulation approach is mainly based on a ranking of all buildings according to their *likeliness* for (hull) renovation. Currently, all buildings are ranked according to the last year of renovation, so buildings that have been renovated a long time ago or old building that have not been renovated before a more likely to be renovated in the current year of the simulation. (The simulation is a discrete one, where each simulation step is one year.) After the ranking a user configurable amount of buildings with the highest ranking score are selected for (hull) renovation in the current year of simulation.

The user can affect the ranking of buildings with the definition of so called *Modifiers*. These modifiers specify a modifier factor for buildings that match certain criteria. E.g.: One-family houses build before 1953 located in a certain city quarter. If a building matches this criteria its ranking score is multiplied with the given modifier factor, thereby increasing/decreasing the scoring value and influencing the ranking.

A more detailed description of the simulation approach can be found in the paper "[Proposing Multiple-Criteria Ranking to Simulate Building Renovation in Cities](./Proposing_Multiple-Criteria_Ranking.pdf)" by Arjun Jamil, Nils Weiss, Thomas Preisler and Wolfgang Renz, that has been presented at the ASIM 2018 symposium in Hamburg.

## Git Checkout
You can checkout the GIT repository either via command line: 
```
git clone https://github.com/gewiss/gewiss-expert-tool.git
```
Or you can download it from your browser and then unzip it. 

## Building Input Data
The main objective of the GEWISS Expert Tool is to simulate the building stock development in the city of Hamburg, but the simulation approach is also suitable for other cities. Unfortunately, due to privacy reasons we are not allowed to publish the building data set of Hamburg as an example. Therefore, we created a small anonymized dataset for testing and publishing purposes.

To do that, we took a real constellation of buildings and moved them to a location in the North Sea, close to Neuwerk Island. The coordinates were affine transformed to make them unrecognizable, but still to resemble a real urban area. Attributes like street names, names of heating companies and so on were changed where needed. Also, the actual names of the quarters and districts were changed to real ones from other places in Hamburg. Finally, the building typology for non-residential buildings of our project partner [Ecofys](https://www.ecofys.com/) was changed to a generic one.

All the data required for the simulation is stored inside a SQLite database. So, in order to use your own data set you have to change the value of the `DB_FILE` constant in the `SQLiteConnection` Java file to the file name of your data set. The Java file can be found in the `get-simulator` project directory in the `de.hawhh.gewiss.get.simulator.db.dao` package folder. Your data file has to be copied to the `get-simulator/src/main/resources/` folder. Be aware that whenever you change the data set file you have to rebuild the application! How to do so is described in the next section.

### Table structure
The following briefly describes the SQL-tables required for the simulation tool.

#### Table gewiss_buildings_v_1 (Building table)
| Column Name   | Type      | Description |
|-------------  |---------- |-------------|
| alkis_id      | String    | Unique Building ID |
| geomwkt       | String    | 2D Geometrie in WKT format |
| wohnfl        | Double    | Living Area in square meter |
| nwg_ngf       | Double    | Square meter of usage space for non-residential buildings |
| bezirk        | String    | The city district |
| stadtteil     | String    | The city quarter |
| stat_gebiet   | String    | Statistical area (below quarter) |
| baublock      | String    | Building block (below statistical quarter) |
| bj_alk_dt     | Integer   | Year of construction |
| dt_san_year   | Integer   | Last year of renovation |
| dt_heiztyp    | String    | Heating system type |
| iwu_typ       | String    | Residential building type |
| nwg_typ       | String    | Non-residential building type |
| bak_fin       | String    | Construction age class |
| cluster       | String    | ID of building cluster |

#### Table baualtersklassen (Construction age class)
| Column Name   | Type      | Description |
|-------------  |---------- |-------------|
| klasse        | String    | Building class |
| von           | Integer   | From year |
| bis           | Integer   | To year|

#### Table waermebedarf (Heat demand)
| Column Name   | Type      | Description |
|-------------  |---------- |-------------|
| typ           | String    | Building type |
| warmwasser0   | Double    | Warm water heat demand (non renovated) |
| warmwasser1   | Double    | Warm water heat demand (EnEV 2014 standard) |
| warmwasser2   | Double    | Warm water heat demand (Passive house standard) |
| heizwaerme0   | Double    | Heat demand (non renovated) |
| heizwaerme1   | Double    | Heat demand (EnEV 2014 standard) |
| heizwaerme2   | Double    | Heat demand (Passive house standard) |

#### Table districts_and_quarters (City districts and quarters)
| Column Name   | Type      | Description |
|-------------  |---------- |-------------|
| id            | Integer   | Unique table id |
| district      | String    | Name of the district |
| quarter       | String    | Name of the quarter |

#### Table primary_energy_factor_co2_factor (Primary energy and CO<sub>2</sub> factors)
| Column Name               | Type      | Description |
|-------------------------- |---------- |-------------|
| id                        | Integer   | Unique table id |
| typ                       | String    | Type of the heating system |
| Primaerenergiefaktor      | Double    | Primary energy factor of the heating system type |
| CO2                       | Double    | CO<sub>2</sub> factor of the heating system type |

## Building and Running the Application
The application is based on  [Apache Maven](https://maven.apache.org/) so, once you have checked out the repository you can either import the Maven projects `get-simulator` (simulation logic) and `get-ui` (JavaFX UI) to the IDE of your choice (we used Netbeans, but Eclipse or IntelliJ work as well) to build and run the it or use Maven and the command line, as decribed in the following part.

### Building and Running with Maven
The following assumes that you have installed [Apache Maven](https://maven.apache.org/) and can access it via the command line. Use the following command to check if Maven is installed:
```
mvn -version
```
If Maven is installed correctly use the command line to open the `get-simulator` folder and build the simulator:
```
cd get-simulator/
mvn clean install
```
This may take some time and in the end you should see something like this on the command line:
```
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 48.676 s
[INFO] Finished at: 2018-11-27T15:47:18+01:00
[INFO] Final Memory: 23M/275M
[INFO] ------------------------------------------------------------------------
```
Now change to the `get-ui` folder and build the JavaFX UI:
```
cd get-ui/
mvn arcgis-java:arcgis
mvn clean install
```
Again it should display `BUILD SUCCESS`. The `mvn arcgis-java:arcgis` command is required to download the [ArcGIS Java SDK](https://developers.arcgis.com/java/)'s native libraries to the user's home directory. It is only required to be run once. We us the ArcGIS Java to show the buildings in the simulation on a map.

In order to start the application stay in the `get-ui` folder and type the following Maven command in the command line:
```
mvn jfx:run
```

#### Building a native executable
To build a native executable for the platform you are running on (Windows, Mac or Linux) first you have to copy the ArcGIS Java SDK's native libaries from your home directory to the `get-ui` folder. To do so, you have to create a new folder called `arcgis` inside the `get-ui` folder. Then you have to copy the `jniLibs` folder from e.g. `/home/thomas/.arcgis/100.2.1/jniLibs` to the newly created `arcgis` folder.

Also you have to get a free licence key for the ArcGIS SDK, you obtain one is described here: https://developers.arcgis.com/java/latest/guide/license-your-app.htm

Once you have obtained the key you have to uncomment line 47 in the `GETAPP.java` file found in `get-ui/src/main/java/de/hawhh/gewiss/get/fx` and enter your licence key here:
```
ArcGISRuntimeEnvironment.setLicense("YOUR-LICENCE-KEY");
```
Afterwards you can build the native executable with the following command:
```
mvn jfx:build-native
```
Keep in mind that this might require additional third party tools depending on your operating system. Maven will state on the command lines which tools are needed if not already installed and a web search should quickly lead to the required tools and instructions on how to install them.

Afterwards you can find the native executable as well as an installer for your platform in the folder `get-ui/target/jfx/native`.
