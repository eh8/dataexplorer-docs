---
title:  geo_distance_point_to_polygon()
description: Learn how to use the geo_distance_point_to_polygon() function to calculate the shortest distance between a coordinate and a polygon or a multipolygon on Earth.
ms.reviewer: mbrichko
ms.topic: reference
ms.date: 08/11/2024
---
# geo_distance_point_to_polygon()

> [!INCLUDE [applies](../includes/applies-to-version/applies.md)] [!INCLUDE [fabric](../includes/applies-to-version/fabric.md)] [!INCLUDE [azure-data-explorer](../includes/applies-to-version/azure-data-explorer.md)] [!INCLUDE [monitor](../includes/applies-to-version/monitor.md)] [!INCLUDE [sentinel](../includes/applies-to-version/sentinel.md)]

Calculates the shortest distance between a coordinate and a polygon or a multipolygon on Earth.

## Syntax

`geo_distance_point_to_polygon(`*longitude*`,`*latitude*`,`*polygon*`)`

[!INCLUDE [syntax-conventions-note](../includes/syntax-conventions-note.md)]

## Parameters

|Name|Type|Required|Description|
|--|--|--|--|
| *longitude* | `real` |  :heavy_check_mark: | Geospatial coordinate, longitude value in degrees. Valid value is a real number and in the range [-180, +180].|
| *latitude* | `real` |  :heavy_check_mark: | Geospatial coordinate, latitude value in degrees. Valid value is a real number and in the range [-90, +90].|
| *polygon* | `dynamic` |  :heavy_check_mark: | Polygon or multipolygon in the [GeoJSON format](https://tools.ietf.org/html/rfc7946).|

## Returns

The shortest distance, in meters, between a coordinate and a polygon or a multipolygon on Earth. If polygon contains point, the distance will be 0. If the coordinates or polygons are invalid, the query will produce a null result.

> [!NOTE]
>
> * The geospatial coordinates are interpreted as represented by the [WGS-84](https://earth-info.nga.mil/index.php?dir=wgs84&action=wgs84) coordinate reference system.
> * The [geodetic datum](https://en.wikipedia.org/wiki/Geodetic_datum) used for measurements on Earth is a sphere. Polygon edges are [geodesics](https://en.wikipedia.org/wiki/Geodesic) on the sphere.
> * If input polygon edges are straight cartesian lines, consider using [geo_polygon_densify()](geo-polygon-densify-function.md) to convert planar edges to geodesics.

**Polygon definition and constraints**

dynamic({"type": "Polygon","coordinates": [LinearRingShell, LinearRingHole_1, ..., LinearRingHole_N]})

dynamic({"type": "MultiPolygon","coordinates": [[LinearRingShell, LinearRingHole_1,..., LinearRingHole_N],..., [LinearRingShell, LinearRingHole_1,..., LinearRingHole_M]]})

* LinearRingShell is required and defined as a `counterclockwise` ordered array of coordinates [[lng_1,lat_1],...,[lng_i,lat_i],...,[lng_j,lat_j],...,[lng_1,lat_1]]. There can be only one shell.
* LinearRingHole is optional and defined as a `clockwise` ordered array of coordinates [[lng_1,lat_1],...,[lng_i,lat_i],...,[lng_j,lat_j],...,[lng_1,lat_1]]. There can be any number of interior rings and holes.
* LinearRing vertices must be distinct with at least three coordinates. The first coordinate must be equal to the last. At least four entries are required.
* Coordinates [longitude, latitude] must be valid. Longitude must be a real number in the range [-180, +180] and latitude must be a real number in the range [-90, +90].
* LinearRingShell encloses at most half of the sphere. LinearRing divides the sphere into two regions. The smaller of the two regions will be chosen.
* LinearRing edge length must be less than 180 degrees. The shortest edge between the two vertices will be chosen.
* LinearRings must not cross and must not share edges. LinearRings may share vertices.
* Polygon doesn't necessarily contain its vertices.

> [!TIP]
>
> * Using literal polygons may result in better performance.
> * If you want to know if polygon contains point, see [geo_point_in_polygon()](geo-point-in-polygon-function.md)

## Examples

The following example calculates shortest distance in meters from some location in NYC to Central Park.

:::moniker range="azure-data-explorer"
> [!div class="nextstepaction"]
> <a href="https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAAA1WQwWrDMAyG73sK41MLWbEtS7Jb9g67hxBMYkpYZofUl1D27ksaAttJ4tenn18aYxFdTGUOYzuF+Ut8iH5J4XvoTk9ZlinKq/zM43LPSVayy3nuhxRKfMhrXdfvDBdvPVZWXdiTb6pdQqfYEBlAQLMNnVLkSAGS9kYfmDNKe2usBWfdy4OcQWIDCpn5wBg0O0erFYLfMYuogVBt5cD+5mian/PtbZqHVMQ95rYfHiWkLrZTXqW25LV5HXXag6y+YttlhdW/f5x/Afn4cVQhAQAA" target="_blank">Run the query</a>
::: moniker-end

```kusto
let central_park = dynamic({"type":"Polygon","coordinates":[[[-73.9495,40.7969],[-73.95807266235352,40.80068603561921],[-73.98201942443848,40.76825672305777],[-73.97317886352539,40.76455136505513],[-73.9495,40.7969]]]});
print geo_distance_point_to_polygon(-73.9839, 40.7705, central_park)
```

**Output**

|print_0|
|---|
|259.940756070596|

The following example enriches the data with distance.

:::moniker range="azure-data-explorer"
> [!div class="nextstepaction"]
> <a href="https://dataexplorer.azure.com/clusters/help/databases/Samples?query=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" target="_blank">Run the query</a>
::: moniker-end

```kusto
let multipolygon = dynamic({"type":"MultiPolygon","coordinates":[[[[-73.991460000000131,40.731738000000206],[-73.992854491775518,40.730082566051351],[-73.996772,40.725432000000154],[-73.997634685522883,40.725786309886963],[-74.002855946639244,40.728346630056791],[-74.001413,40.731065000000207],[-73.996796995070824,40.73736378205173],[-73.991724524037934,40.735245208931886],[-73.990703782359589,40.734781896080477],[-73.991460000000131,40.731738000000206]]],[[[-73.958357552055688,40.800369095633819],[-73.98143901556422,40.768762584141953],[-73.981548752788598,40.7685590292784],[-73.981565335901905,40.768307084720796],[-73.981754418060945,40.768399727738668],[-73.982038573548124,40.768387823012056],[-73.982268248204349,40.768298621883247],[-73.982384797518051,40.768097213086911],[-73.982320919746599,40.767894461792181],[-73.982155532845766,40.767756204474757],[-73.98238873834039,40.767411004834273],[-73.993650353659021,40.772145571634361],[-73.99415893763998,40.772493009137818],[-73.993831082030937,40.772931787850908],[-73.993891252437052,40.772955194876722],[-73.993962585514595,40.772944653908901],[-73.99401262480508,40.772882846631894],[-73.994122058082397,40.77292405902601],[-73.994136652588594,40.772901870174394],[-73.994301342391154,40.772970028663913],[-73.994281535134448,40.77299380206933],[-73.994376552751078,40.77303955110149],[-73.994294029824005,40.773156243992048],[-73.995023275860802,40.773481196576356],[-73.99508939189289,40.773388475039134],[-73.995013963716758,40.773358035426909],[-73.995050284699261,40.773297153189958],[-73.996240651898916,40.773789791397689],[-73.996195837470992,40.773852356184044],[-73.996098807369748,40.773951805299085],[-73.996179459973888,40.773986954351571],[-73.996095245226442,40.774086186437756],[-73.995572265161172,40.773870731394297],[-73.994017424135961,40.77321375261053],[-73.993935876811335,40.773179512586211],[-73.993861942928888,40.773269531698837],[-73.993822393527211,40.773381758622882],[-73.993767019318497,40.773483981224835],[-73.993698463744295,40.773562141052594],[-73.993358326468751,40.773926888327956],[-73.992622663865575,40.774974056037109],[-73.992577842766124,40.774956016359418],[-73.992527743951555,40.775002110439829],[-73.992469745815342,40.775024159551755],[-73.992403837191887,40.775018140390664],[-73.99226708903538,40.775116033858794],[-73.99217809026365,40.775279293897171],[-73.992059084937338,40.775497598192516],[-73.992125372394938,40.775509075053385],[-73.992226867797001,40.775482211026116],[-73.992329346608813,40.775468900958522],[-73.992361756801131,40.775501899766638],[-73.992386042960277,40.775557180424634],[-73.992087684712729,40.775983970821372],[-73.990927174149746,40.777566878763238],[-73.99039616003671,40.777585065679204],[-73.989461267506471,40.778875124584417],[-73.989175778438053,40.779287524015778],[-73.988868617400072,40.779692922911607],[-73.988871874499793,40.779713738253008],[-73.989219022880576,40.779697895209402],[-73.98927785904425,40.779723439271038],[-73.989409054180143,40.779737706471963],[-73.989498614927044,40.779725044389757],[-73.989596493388234,40.779698146683387],[-73.989679812902509,40.779677568658038],[-73.989752702937935,40.779671244211556],[-73.989842247806507,40.779680752670664],[-73.990040102120489,40.779707677698219],[-73.990137977524839,40.779699769704784],[-73.99033584033225,40.779661794394983],[-73.990430598697046,40.779664973055503],[-73.990622199396725,40.779676064914298],[-73.990745069505479,40.779671328184051],[-73.990872114282197,40.779646007643876],[-73.990961672224358,40.779639683751753],[-73.991057472829539,40.779652352625774],[-73.991157429497036,40.779669775606465],[-73.991242817404469,40.779671367084504],[-73.991255318289745,40.779650782516491],[-73.991294887120119,40.779630209208889],[-73.991321967649895,40.779631796041372],[-73.991359455569423,40.779585883337383],[-73.991551059227476,40.779574821437407],[-73.99141982585985,40.779755280287233],[-73.988886144117032,40.779878898532999],[-73.988939656706265,40.779956178440393],[-73.988926103530844,40.780059292013632],[-73.988911680264692,40.780096037146606],[-73.988919261468567,40.780226094343945],[-73.988381050202634,40.780981074045783],[-73.988232413846987,40.781233144215555],[-73.988210420831663,40.781225482542055],[-73.988140000000143,40.781409000000224],[-73.988041288067166,40.781585961353777],[-73.98810029382463,40.781602878305286],[-73.988076449145055,40.781650935001608],[-73.988018059972219,40.781634188810422],[-73.987960792842145,40.781770987031535],[-73.985465811970457,40.785360700575431],[-73.986172704965611,40.786068452258647],[-73.986455862401996,40.785919219081421],[-73.987072345615601,40.785189638820121],[-73.98711901394276,40.785210319004058],[-73.986497781023601,40.785951202887254],[-73.986164628806279,40.786121882448327],[-73.986128422486075,40.786239001331111],[-73.986071135219746,40.786240706026611],[-73.986027274789123,40.786228964236727],[-73.986097637849426,40.78605822569795],[-73.985429321269592,40.785413942184597],[-73.985081137732209,40.785921935110366],[-73.985198833254501,40.785966552197777],[-73.985170502389906,40.78601333415817],[-73.985216218673656,40.786030501816427],[-73.98525509797993,40.785976205511588],[-73.98524273937646,40.785972572653328],[-73.98524962933017,40.785963139855845],[-73.985281779186749,40.785978620950075],[-73.985240032884533,40.786035858136792],[-73.985683885242182,40.786222123919686],[-73.985717529004575,40.786175994668795],[-73.985765660297687,40.786196274858618],[-73.985682871922691,40.786309786213067],[-73.985636270930442,40.786290150649279],[-73.985670722564691,40.786242911993817],[-73.98520511880038,40.786047669212785],[-73.985211035607492,40.786039554883686],[-73.985162639946992,40.786020999769754],[-73.985131636312062,40.786060297019972],[-73.985016964065125,40.78601423719563],[-73.984655078830457,40.786534741807841],[-73.985743787901043,40.786570082854738],[-73.98589227228328,40.786426529019593],[-73.985942854994988,40.786452847880334],[-73.985949561556794,40.78648711396653],[-73.985812373526713,40.786616865357047],[-73.985135209703174,40.78658761889551],[-73.984619428584324,40.786586016349787],[-73.981952458164173,40.790393724337193],[-73.972823037363767,40.803428052816756],[-73.971036786332192,40.805918478839672],[-73.966701,40.804169000000186],[-73.959647,40.801156000000113],[-73.958508540159471,40.800682279767472],[-73.95853274080838,40.800491362464697],[-73.958357552055688,40.800369095633819]]],[[[-73.943592454622546,40.782747908206574],[-73.943648235390199,40.782656161333449],[-73.943870759887162,40.781273026571704],[-73.94345932494096,40.780048275653243],[-73.943213862652243,40.779317588660199],[-73.943004239504688,40.779639495474292],[-73.942716005450905,40.779544169476175],[-73.942712374762181,40.779214856940001],[-73.942535563208608,40.779090956062532],[-73.942893408188027,40.778614093246276],[-73.942438481745029,40.777315235766039],[-73.942244919522594,40.777104088947254],[-73.942074188038887,40.776917846977142],[-73.942002667222781,40.776185317382648],[-73.942620205199006,40.775180871576474],[-73.94285645694552,40.774796600349191],[-73.94293043781397,40.774676268036011],[-73.945870899588215,40.771692257932997],[-73.946618690150586,40.77093339256956],[-73.948664164778933,40.768857624399587],[-73.950069793030679,40.767025088383498],[-73.954418260786071,40.762184104951245],[-73.95650786241211,40.760285256574043],[-73.958787773424007,40.758213471309809],[-73.973015157270069,40.764278692864671],[-73.955760332998182,40.787906554459667],[-73.944023,40.782960000000301],[-73.943592454622546,40.782747908206574]]]]});
let coordinates = 
    datatable(longitude:real, latitude:real, description:string)
    [
        real(-73.9741), 40.7914, 'Upper West Side',
        real(-73.9950), 40.7340, 'Greenwich Village',
        real(-73.8743), 40.7773, 'LaGuardia Airport',
    ];
coordinates
| extend distance = geo_distance_point_to_polygon(longitude, latitude, multipolygon)
```

**Output**

|longitude|latitude|description|distance|
|---|---|---|---|
|-73.9741|40.7914|Upper West Side|0|
|-73.995|40.734|Greenwich Village|0|
|-73.8743|40.7773|LaGuardia Airport|5702.15731467514|

The following example finds all states that are within 200-km distance, excluding state that contains the point.

:::moniker range="azure-data-explorer"
> [!div class="nextstepaction"]
> <a href="https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAAA12NwQrCMBAF737FHluodaNVEFTw4FEvxXMI7ZpG2iSkK1Lw442gVrw9hsfMuZQlK6Z+8gAf3JUqBqs6gi1cSPEtUJ9H7imwifO0Px4y8K4dtLO/H02uIw7DnyaD2vSsbPUSVmRaY3USv/KDpXfGsmQn39JkKoTI17jMoMB8tSi+uRRmIBAxjY17Q4FG9wbmiKBsPaId4BNva0k13AAAAA==" target="_blank">Run the query</a>
::: moniker-end

```kusto
US_States
| project name = features.properties.NAME, polygon = features.geometry
| project name, distance = ceiling(geo_distance_point_to_polygon(-111.905, 40.634, polygon) / 1000)
| where distance < 200 and distance > 0
```

**Output**

|name|distance|
|---|---|
|Idaho|152|
|Nevada|181|
|Wyoming|83|

The following example will return a null result because of the invalid coordinate input.

:::moniker range="azure-data-explorer"
> [!div class="nextstepaction"]
> <a href="https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAAAysoyswrUUjJLC5JzEtOVbBVSE/Nj4dx4wvygbLxJflARk5len6ehqmBgY6hTkplXmJuZrJGtVJJZUGqkpWCUgBEXklHKTk/vyglMy+xJLUYKBEdHW2gYxCrE20I1AejgRRIMDa2VlMTAGFm5geAAAAA" target="_blank">Run the query</a>
::: moniker-end

```kusto
print distance = geo_distance_point_to_polygon(500,1,dynamic({"type": "Polygon","coordinates": [[[0,0],[10,10],[10,1],[0,0]]]}))
```

**Output**

| distance |
|------------|
|            |

The following example will return a null result because of the invalid polygon input.

:::moniker range="azure-data-explorer"
> [!div class="nextstepaction"]
> <a href="https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAAA03JywqFIBAG4FeRWRXMQrdB79BeREQlhJqRmo1E756HONDqv3z1KCQqlVMCxaxmtWb2/+krd/XCvWxtZRoMGkyNwl7icIG0mmFSsLwKCJH5SIWC5LODtVajdmiNRvPN3+vcPY4PvW97rH8AAAA=" target="_blank">Run the query</a>
::: moniker-end

```kusto
print distance = geo_distance_point_to_polygon(1,1,dynamic({"type": "Polygon","coordinates": [[[0,0],[10,10],[10,10],[0,0]]]}))
```

**Output**

| distance |
|------------|
|            |