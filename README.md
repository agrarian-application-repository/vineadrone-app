## VINEA-DRONE - AGRARIAN Project

## 1.	Testbed
Testbed 2- Non-Geosynchronous Orbit (NGSO) Satellite Communications with EDGE capabilities;

## 2.	Node Type
Indicate the type of node hosting the application: edge, satellite, or cloud.
It's a mix of edge and satellite: Everything can run on the edge. Ideally, the POI identification and the Disease Detection would run on the satellite.

## 3.	Application Name: VINEA-DRONE

## 4.	Application Objective
VINEA-Drone provides a set of interconnected tools for semi-automated drone-based disease detection in vineyards. It follows a two-tier approach: the drone scans the whole vineyard, patch by patch, via high-altitude waypoints (e.g. 50-60 m). This can provide a fast scan, but insufficient resolution for disease detection. Hence, at this altitude, multispectral images are acquired, the NDVI of each patch is calculated, and points of interest (POIs) are identified. The drone then visits these POIs at very low altitude (e.g. 3-5m), getting high-resolution RGB images from which the presence and type of disease can be detected.
It consists of 3 main components:
a) POI identification receives multispectral images, computes the NDVI, detects anomalous points, and returns their coordinates.
b) Adaptive flight path integrates these coordinates into the drone's flight path in a way that minimises the overall distance travelled.
c) Disease detection analyses the RGB images taken at very low altitude and classifies them according to disease presence and severity/probability.

## 5.Use Case Description
Vinegrowers can use the solution to scan large vineyards routinely, reducing manual labour and redirecting it to more targeted inspections or verifications of particular patches with signs of disease. This can improve early detection and reactive mitigation measures, applying them in a more targeted, economical, and ecological way, rather than proactively and uniformly.

## 6.	Organization: KyberKore P.C.

## 7.	Use instructions
7a. Adaptive path simulator  
A. Required Inputs  
⦁	"pattern" may be serpentine or spiral  
⦁	"logic" may be shortest_detour or second_pass  
⦁	"scenario" must be a json following the example structure  
docker run --rm -v "[path to local folder containing the scenario json]:/data" vinea-drone-path-sim --scenario-config /data/[filename of the scenario] --pattern [pattern] --logic [logic]  
e.g.: docker run --rm -v "F:\VINEA-Drone:/data" vinea-drone-path-sim --scenario-config /data/scenario.json --pattern serpentine --logic second_pass  

B. Expected Outputs  
The total distance travelled, the time needed to complete the mission, and the number of low-altitude waypoints generated. If run locally (not in Docker), an interactive visualisation of the mission progress and the path adaptation will also appear.  

7b. POI identification  
A. Required Inputs  
⦁	a pair of multispectral image files (Red + NIR) in TIF format from a DJI drone (ideally Mavic 3M)  
docker run --rm -v "[local path where input images are stored]:/data"  vinea-drone-poi  --red /data/[filename if Red band image.TIF] --nir /data/[filename if NIR band image.TIF]  
e.g.: POI: docker run --rm -v "F:\VINEA-Drone:/data"  vinea-drone-poi  --red /data/DJI_20250717110426_0001_MS_R.TIF  --nir /data/DJI_20250717110426_0001_MS_NIR.TIF  

B. Expected Outputs  
⦁	a list of POIs as GPS coordinates. The list can contain 0-3 POIs, so an empty list is possible (and common, especially if the TIFs are from a healthy vineyard).  

7c. Disease detection  
A. Required Inputs  
⦁	a jpg image of grapevines  
docker run --rm -v "[local path where input images are stored]:/inputs" -v "${PWD}\outputs:/app/outputs" grapedisease:cpu --img_path "/inputs/[filename]"  
e.g.: docker run --rm -v "F:\VINEA-Drone:/inputs" -v "${PWD}\outputs:/app/outputs" grapedisease:cpu --img_path "/inputs/test1.jpg"   

B. Expected Outputs  
⦁	a list of detected unhealthy leaves, with the class of each and bounding box locations  

7d. Server and simulator (these work together, and simulate a mission on a vineyard. The server is responsible for POI identification, while the simulator handles path adaptation)  
A. Required Inputs  
⦁	a folder with Red+NIR pairs of multispectral DJI images in TIF format  
⦁	a scenarion json named: scenario_all_4x3.json (4 and 3 can be different numbers - they could denote the length and width)  
⦁	the IP and port of the server  
⦁	"pattern" may be serpentine or spiral  
⦁	"logic" may be shortest_detour or second_pass  
run the server with docker run --rm -p 5001:5001 vinea-drone-server  
run the simulator with docker run --rm -v "[folder of multispectral images and scenario]:/data" vinea-drone-sim-with-server --scenario-config /data/[scenario json] --image_folder /data --ip [IP of server] --port [port of server]  
e.g.: docker run --rm -v "F:\VINEA-Drone\MS_images:/data" vinea-drone-sim-with-server --scenario-config /data/scenario_all_6x9.json --image_folder /data --ip host.docker.internal --port 5001  

B. Expected Outputs  
⦁	log file with the times each multispectral pair was sent to the server, when receipt confirmation was received, and when the POI identification results of each pair were received.  
⦁	pkl (Python Pickle) files of the results and progress of each mission  
