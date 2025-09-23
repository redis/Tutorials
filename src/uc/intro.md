**Redis** is a powerful in-memory data structure store that can be used for various use cases due to its speed, simplicity, and versatility. In this tutorial, we'll explore several common use cases: matchmaking, job queue, leaderboard, and session store.

If you haven't done so already, you can upload the sample data related to this tutorial by clicking the button below. You will also need support for [JSON](https://redis.io/docs/latest/develop/data-types/json/?utm_source=redisinsight&utm_medium=app&utm_campaign=RAG_tutorial) and [Search & query](https://redis.io/docs/latest/develop/ai/search-and-query/?utm_source=redisinsight&utm_medium=app&utm_campaign=RAG_tutorial) in your database to take advantage of this tutorial fully.

```redis:[run_confirmation=true] Upload Sample Data

JSON.SET sample_bicycle:1001 $ '{"model":"Racer","brand":"Speedster","price":320,"type":"Road","specs":{"material":"carbon fiber","weight":8},"description":"The Racer is built for speed and performance on the road!","addons":["water bottle holder","bike lock"],"helmet_included":true, "condition":"new"}'
JSON.SET sample_bicycle:1002 $ '{"model":"Explorer","brand":"Trailblazer","price":450,"type":"Mountain","specs":{"material":"steel","weight":15},"description":"The Explorer is your ultimate companion for off-road adventures!","addons":["mudguards","bike lights"],"helmet_included":true, "condition":"new"}'
JSON.SET sample_bicycle:1003 $ '{"model":"Cruiser","brand":"Beachcomber","price":280,"type":"Leisure","specs":{"material":"aluminium","weight":12},"description":"The Cruiser offers comfort and style for relaxed rides along the beach!","addons":["basket","bell"],"helmet_included":false, "condition":"new"}'
JSON.SET sample_bicycle:1004 $ '{"model":"Sprinter","brand":"Swift","price":380,"type":"Electric","specs":{"material":"aluminium","weight":18},"description":"The Sprinter gives you an extra boost for effortless commuting!","addons":["phone holder","kickstand"],"helmet_included":true, "condition":"new"}'
JSON.SET sample_bicycle:1005 $ '{"model":"Tracker","brand":"Pathfinder","price":410,"type":"Hybrid","specs":{"material":"carbon fiber","weight":11},"description":"The Tracker combines versatility and performance for urban and off-road trails!","addons":["rear rack","fenders"],"helmet_included":true, "condition":"new"}'
JSON.SET sample_bicycle:1006 $ '{"model":"Adventurer","brand":"Explorer","price":500,"type":"Gravel","specs":{"material":"titanium","weight":14},"description":"The Adventurer is designed to conquer any terrain with ease!","addons":["bike pump","repair kit"],"helmet_included":true, "condition":"new"}'
JSON.SET sample_bicycle:1007 $ '{"model":"Commuter","brand":"Urbanite","price":350,"type":"City","specs":{"material":"aluminium","weight":13},"description":"The Commuter offers practicality and style for everyday city commuting!","addons":["front light","rear reflector"],"helmet_included":true, "condition":"new"}'
JSON.SET sample_bicycle:1008 $ '{"model":"Stunt","brand":"Xtreme","price":300,"type":"BMX","specs":{"material":"chromoly steel","weight":11},"description":"The Stunt is built to handle the toughest tricks and jumps at the skatepark!","addons":["peg set","gyro"],"helmet_included":true, "condition":"new"}'
JSON.SET sample_bicycle:1009 $ '{"model":"Trailblazer","brand":"Rugged","price":470,"type":"Fat","specs":{"material":"aluminium","weight":20},"description":"The Trailblazer conquers sand, snow, and rough terrain with ease!","addons":["bike pump","saddle bag"],"helmet_included":true, "condition":"new"}'
JSON.SET sample_bicycle:1010 $ '{"model":"Speedster","brand":"Velocity","price":330,"type":"Road","specs":{"material":"carbon fiber","weight":7},"description":"The Speedster is engineered for maximum velocity and agility on the road!","addons":["bottle cage","bike computer"],"helmet_included":true, "condition":"new"}'
JSON.SET sample_bicycle:1011 $ '{"model":"Urbanite","brand":"CityCruiser","price":320,"type":"City","specs":{"material":"aluminium","weight":12},"description":"The Urbanite is your perfect companion for navigating busy city streets!","addons":["phone holder","rear rack"],"helmet_included":false, "condition":"new"}'
JSON.SET sample_bicycle:1012 $ '{"model":"Thrasher","brand":"Extreme","price":290,"type":"BMX","specs":{"material":"chromoly steel","weight":10},"description":"The Thrasher is for riders who dare to push their limits at the skatepark!","addons":["grind pegs","handlebar pad"],"helmet_included":true, "condition":"new"}'
JSON.SET sample_bicycle:1013 $ '{"model":"Navigator","brand":"Trekker","price":460,"type":"Hybrid","specs":{"material":"aluminium","weight":14},"description":"The Navigator guides you through urban streets and off-road trails with ease!","addons":["front basket","bike lock"],"helmet_included":true, "condition":"new"}'
JSON.SET sample_bicycle:1014 $ '{"model":"Freedom","brand":"Liberty","price":400,"type":"Electric","specs":{"material":"aluminium","weight":16},"description":"The Freedom empowers you with effortless mobility and eco-friendly commuting!","addons":["phone mount","rear light"],"helmet_included":true, "condition":"new"}'
JSON.SET sample_bicycle:1015 $ '{"model":"Trekker","brand":"Wanderlust","price":480,"type":"Touring","specs":{"material":"steel","weight":17},"description":"The Trekker is your reliable companion for long-distance journeys and exploration!","addons":["panniers","bottle holder"],"helmet_included":true, "condition":"new"}'
JSON.SET sample_bicycle:1016 $ '{"model":"Seeker","brand":"Nomad","price":430,"type":"Gravel","specs":{"material":"titanium","weight":13},"description":"The Seeker is built to explore rugged terrain and endless adventures!","addons":["tool kit","kickstand"],"helmet_included":true, "condition":"new"}'
JSON.SET sample_bicycle:1017 $ '{"model":"Pioneer","brand":"Frontier","price":380,"type":"Mountain","specs":{"material":"aluminium","weight":16},"description":"The Pioneer leads the way through challenging trails and mountainous terrain!","addons":["handlebar grips","bike lights"],"helmet_included":true, "condition":"new"}'
JSON.SET sample_bicycle:1018 $ '{"model":"Aviator","brand":"SkyRider","price":340,"type":"Leisure","specs":{"material":"aluminium","weight":11},"description":"The Aviator takes you on a leisurely journey with comfort and style!","addons":["cup holder","rear reflector"],"helmet_included":false, "condition":"new"}'
JSON.SET sample_bicycle:1019 $ '{"model":"Outlaw","brand":"Rebel","price":310,"type":"BMX","specs":{"material":"chromoly steel","weight":9},"description":"The Outlaw is for riders who challenge conventions and ride with rebellion!","addons":["stunt pegs","grip tape"],"helmet_included":true, "condition":"new"}'
JSON.SET sample_bicycle:1020 $ '{"model":"Storm","brand":"Thunder","price":520,"type":"Fat","specs":{"material":"aluminium","weight":22},"description":"The Storm conquers any terrain with its robust build and unstoppable performance!","addons":["saddlebag","bottle holder"],"helmet_included":true, "condition":"new"}'
JSON.SET sample_bicycle:1021 $ '{"model":"Glider","brand":"Aero","price":360,"type":"Electric","specs":{"material":"carbon fiber","weight":19},"description":"The Glider offers smooth and efficient rides, powered by cutting-edge technology!","addons":["cargo rack","bike bell"],"helmet_included":true, "condition":"new"}'
JSON.SET sample_bicycle:1022 $ '{"model":"Voyager","brand":"Wanderer","price":470,"type":"Touring","specs":{"material":"steel","weight":18},"description":"The Voyager takes you on epic journeys across continents with comfort and reliability!","addons":["bike pump","pannier rack"],"helmet_included":true, "condition":"new"}'
JSON.SET sample_bicycle:1023 $ '{"model":"Nomad","brand":"Explorer","price":340,"type":"Hybrid","specs":{"material":"aluminium","weight":13},"description":"The Nomad is your versatile companion for urban commuting and weekend adventures!","addons":["fenders","kickstand"],"helmet_included":true, "condition":"new"}'
JSON.SET sample_bicycle:1024 $ '{"model":"Cruiser","brand":"Coastline","price":290,"type":"Leisure","specs":{"material":"steel","weight":14},"description":"The Cruiser offers laid-back rides along the coast with comfort and style!","addons":["cup holder","rear rack"],"helmet_included":false, "condition":"new"}'
JSON.SET sample_bicycle:1025 $ '{"model":"Thrill","brand":"Velocity","price":380,"type":"BMX","specs":{"material":"chromoly steel","weight":10},"description":"The Thrill is designed for adrenaline junkies who seek excitement on two wheels!","addons":["foot pegs","bar ends"],"helmet_included":true, "condition":"new"}'
JSON.SET sample_bicycle:1026 $ '{"model":"Aero","brand":"Flyer","price":450,"type":"Road","specs":{"material":"carbon fiber","weight":8},"description":"The Aero is engineered for aerodynamic performance, perfect for speed enthusiasts!","addons":["aero bars","speedometer"],"helmet_included":true, "condition":"new"}'
JSON.SET sample_bicycle:1027 $ '{"model":"Explorer","brand":"Trekker","price":480,"type":"Mountain","specs":{"material":"aluminium","weight":17},"description":"The Explorer conquers rugged trails and mountain peaks with confidence and agility!","addons":["bike pump","hydration pack"],"helmet_included":true, "condition":"new"}'
JSON.SET sample_bicycle:1028 $ '{"model":"Cruiser","brand":"RetroRide","price":320,"type":"Leisure","specs":{"material":"steel","weight":13},"description":"The Cruiser brings back the nostalgia of classic rides with modern comfort and reliability!","addons":["basket","bell"],"helmet_included":false, "condition":"new"}'
JSON.SET sample_bicycle:1029 $ '{"model":"Navigator","brand":"Pathfinder","price":460,"type":"Hybrid","specs":{"material":"aluminium","weight":15},"description":"The Navigator guides you through city streets and nature trails with ease and style!","addons":["front basket","bike lock"],"helmet_included":true, "condition":"new"}'
JSON.SET sample_bicycle:1030 $ '{"model":"Freedom","brand":"EcoRide","price":420,"type":"Electric","specs":{"material":"aluminium","weight":17},"description":"The Freedom offers eco-friendly mobility without compromising on style and performance!","addons":["phone mount","rear light"],"helmet_included":true, "condition":"new"}'
JSON.SET sample_bicycle:1031 $ '{"model":"Wanderer","brand":"Nomad","price":490,"type":"Touring","specs":{"material":"steel","weight":19},"description":"The Wanderer takes you on epic adventures across varied landscapes with comfort and endurance!","addons":["pannier rack","bottle holder"],"helmet_included":true, "condition":"new"}'
JSON.SET sample_bicycle:1032 $ '{"model":"Seeker","brand":"Frontier","price":440,"type":"Gravel","specs":{"material":"titanium","weight":14},"description":"The Seeker is your ultimate companion for exploring gravel roads and rough terrains!","addons":["tool kit","kickstand"],"helmet_included":true, "condition":"new"}'
JSON.SET sample_bicycle:1033 $ '{"model":"Pioneer","brand":"Summit","price":390,"type":"Mountain","specs":{"material":"aluminium","weight":16},"description":"The Pioneer leads the way through challenging trails and rocky terrain with confidence!","addons":["handlebar grips","bike lights"],"helmet_included":true, "condition":"new"}'
JSON.SET sample_bicycle:1034 $ '{"model":"Aviator","brand":"SkyHigh","price":350,"type":"Leisure","specs":{"material":"aluminium","weight":12},"description":"The Aviator takes you on leisurely flights along bike paths with comfort and style!","addons":["cup holder","rear reflector"],"helmet_included":false, "condition":"new"}'
JSON.SET sample_bicycle:1035 $ '{"model":"Thrifty Rider","brand":"Pre-Owned Pedals","price":89,"type":"Kids","specs":{"material":"aluminum","weight":12.5},"description":"The Thrifty Rider offers affordable fun for young cyclists, perfect for their first biking experiences!","addons":["training wheels","bell"],"helmet_included":true, "condition":"used"}'
JSON.SET sample_bicycle:1036 $ '{"model":"Starter Steed","brand":"Secondhand Cycles","price":99,"type":"Kids","specs":{"material":"steel","weight":14.5},"description":"The Starter Steed is a reliable companion for young riders, providing stability and confidence on every journey!","addons":["basket","streamers"],"helmet_included":true, "condition":"used"}'
JSON.SET sample_bicycle:1037 $ '{"model":"Junior Jumper","brand":"Pre-Loved Wheels","price":109,"type":"Kids","specs":{"material":"aluminum","weight":11.5},"description":"The Junior Jumper inspires young adventurers to leap into outdoor exploration, offering reliable performance at a budget-friendly price!","addons":["water bottle","front basket"],"helmet_included":true, "condition":"used"}'
JSON.SET sample_bicycle:1038 $ '{"model":"Cozy Cruiser","brand":"Hand-Me-Down Bikes","price":79,"type":"Kids","specs":{"material":"steel","weight":13.5},"description":"The Cozy Cruiser provides comfort and joy for young riders, perfect for leisurely rides around the neighborhood!","addons":["handlebar pad","bike bell"],"helmet_included":true, "condition":"used"}'
JSON.SET sample_bicycle:1039 $ '{"model":"Junior Jetstream","brand":"Vintage Velo","price":129,"type":"Kids","specs":{"material":"aluminum","weight":10.5},"description":"The Junior Jetstream offers young riders a taste of nostalgia and adventure, providing a smooth and reliable ride!","addons":["rear reflector","bottle holder"],"helmet_included":true, "condition":"used"}'
JSON.SET sample_bicycle:1040 $ '{"model":"Tiny Trekker Plus","brand":"Recycled Rides","price":109,"type":"Kids","specs":{"material":"steel","weight":14.5},"description":"The Tiny Trekker Plus accompanies young adventurers on exciting journeys, offering durability and fun at an unbeatable price!","addons":["handlebar basket","training wheels"],"helmet_included":true, "condition":"used"}'
JSON.SET sample_bicycle:1041 $ '{"model":"Vintage Velo Junior","brand":"Retro Riders","price":99,"type":"Kids","specs":{"material":"aluminum","weight":11},"description":"The Vintage Velo Junior brings timeless style and joy to young riders, perfect for nostalgic adventures!","addons":["bike bell","streamers"],"helmet_included":true, "condition":"used"}'
JSON.SET sample_bicycle:1042 $ '{"model":"Retro Racer","brand":"Classic Cycles","price":119,"type":"Kids","specs":{"material":"steel","weight":13},"description":"The Retro Racer evokes memories of yesteryears, providing young cyclists with a taste of vintage fun and excitement!","addons":["basket","handlebar pad"],"helmet_included":true, "condition":"used"}'
JSON.SET sample_bicycle:1043 $ '{"model":"Hand-Me-Down Hero","brand":"Family Treasures","price":89,"type":"Kids","specs":{"material":"aluminum","weight":12},"description":"The Hand-Me-Down Hero carries on the tradition of family adventures, offering reliability and charm to young riders!","addons":["training wheels","bell"],"helmet_included":true, "condition":"used"}'
JSON.SET sample_bicycle:1044 $ '{"model":"Retro Rocket","brand":"Nostalgia Bikes","price":109,"type":"Kids","specs":{"material":"steel","weight":14},"description":"The Retro Rocket blasts young riders back in time with its vintage design and reliable performance, perfect for retro enthusiasts!","addons":["handlebar pad","bike bell"],"helmet_included":true, "condition":"used"}'
JSON.SET sample_bicycle:1045 $ '{"model":"Mini Rocket","brand":"Zoom Kids","price":149,"type":"Kids","specs":{"material":"aluminum","weight":12},"description":"The Mini Rocket delivers fun and excitement for young riders, perfect for their first biking adventures!","addons":["training wheels","bell"],"helmet_included":true, "condition":"new"}'
JSON.SET sample_bicycle:1046 $ '{"model":"Speedy Jr.","brand":"Velocity Kids","price":129,"type":"Kids","specs":{"material":"steel","weight":14},"description":"The Speedy Jr. is designed for budding cyclists, offering stability and confidence for learning riders!","addons":["basket","streamers"],"helmet_included":true, "condition":"new"}'
JSON.SET sample_bicycle:1047 $ '{"model":"Zoomster","brand":"Adventure Kids","price":169,"type":"Kids","specs":{"material":"aluminum","weight":11},"description":"The Zoomster inspires young adventurers with its vibrant design and easy-to-handle frame, perfect for outdoor exploration!","addons":["water bottle","front basket"],"helmet_included":true, "condition":"new"}'
JSON.SET sample_bicycle:1048 $ '{"model":"Tiny Turbo","brand":"Speedy Wheels","price":119,"type":"Kids","specs":{"material":"steel","weight":13},"description":"The Tiny Turbo sparks excitement with its sleek design and sturdy build, ideal for young cyclists on the move!","addons":["handlebar pad","bike bell"],"helmet_included":true, "condition":"new"}'
JSON.SET sample_bicycle:1049 $ '{"model":"Swift Rider","brand":"Little Racers","price":139,"type":"Kids","specs":{"material":"aluminum","weight":10},"description":"The Swift Rider offers young racers a taste of speed and agility, perfect for budding champions!","addons":["knee and elbow pads","bottle cage"],"helmet_included":true, "condition":"new"}'
JSON.SET sample_bicycle:1050 $ '{"model":"Mini Cruiser","brand":"Tiny Trekkers","price":159,"type":"Kids","specs":{"material":"steel","weight":15},"description":"The Mini Cruiser cruises through the neighborhood with style and ease, providing young riders with hours of fun!","addons":["rear cargo rack","bike horn"],"helmet_included":true, "condition":"new"}'
JSON.SET sample_bicycle:1051 $ '{"model":"Buddy Bike","brand":"Happy Pedals","price":129,"type":"Kids","specs":{"material":"aluminum","weight":11.5},"description":"The Buddy Bike is the perfect companion for young riders, offering stability and comfort for memorable adventures!","addons":["handlebar streamers","front light"],"helmet_included":true, "condition":"new"}'
JSON.SET sample_bicycle:1052 $ '{"model":"Junior Jet","brand":"Speedy Spokes","price":149,"type":"Kids","specs":{"material":"aluminum","weight":10.5},"description":"The Junior Jet zooms ahead with young riders, inspiring a love for cycling and outdoor exploration!","addons":["rear reflector","bottle holder"],"helmet_included":true, "condition":"new"}'
JSON.SET sample_bicycle:1053 $ '{"model":"Tiny Trekker","brand":"Adventure Wheels","price":159,"type":"Kids","specs":{"material":"steel","weight":14},"description":"The Tiny Trekker embarks on exciting journeys with young adventurers, offering reliability and fun on every ride!","addons":["handlebar basket","training wheels"],"helmet_included":true, "condition":"new"}'
JSON.SET sample_bicycle:1054 $ '{"model":"Mini Racer","brand":"Speedy Sprint","price":139,"type":"Kids","specs":{"material":"aluminum","weight":11},"description":"The Mini Racer ignites young imaginations with its sporty design and smooth ride, perfect for aspiring racers!","addons":["bike bell","streamers"],"helmet_included":true, "condition":"new"}'
JSON.SET sample_bicycle:1055 $ '{"model":"Tiny Thunder","brand":"Zoom Kids","price":119,"type":"Kids","specs":{"material":"steel","weight":13.5},"description":"The Tiny Thunder unleashes excitement for young riders, offering durability and thrill on every ride!","addons":["basket","handlebar pad"],"helmet_included":true, "condition":"new"}'
JSON.SET sample_bicycle:1056 $ '{"model":"Junior Cruiser","brand":"Happy Wheels","price":159,"type":"Kids","specs":{"material":"aluminum","weight":11},"description":"The Junior Cruiser cruises in style with young cyclists, providing comfort and joy on every adventure!","addons":["rear cargo rack","bike horn"],"helmet_included":true, "condition":"new"}'
JSON.SET sample_bicycle:1057 $ '{"model":"Sprinter","brand":"Velocity","price":299,"type":"Mountain","specs":{"material":"aluminum","weight":10},"description":"The Sprinter conquers rugged trails with ease, offering agility and durability for off-road adventures!","addons":["hydration pack","multi-tool"],"helmet_included":true, "condition":"new"}'
JSON.SET sample_bicycle:1058 $ '{"model":"Cruiser","brand":"Cyclopedic","price":249,"type":"City","specs":{"material":"steel","weight":12},"description":"The Cruiser navigates urban streets with style and comfort, perfect for leisurely rides around the city!","addons":["rear rack","kickstand"],"helmet_included":true, "condition":"new"}'
JSON.SET sample_bicycle:1059 $ '{"model":"Trailblazer","brand":"Adventure Seeker","price":369,"type":"Mountain","specs":{"material":"titanium","weight":9},"description":"The Trailblazer blazes new paths through rugged terrain, offering unmatched performance and durability for mountain enthusiasts!","addons":["bike pump","repair kit"],"helmet_included":true, "condition":"new"}'
JSON.SET sample_bicycle:1060 $ '{"model":"Cityscape","brand":"Urban Glide","price":279,"type":"City","specs":{"material":"aluminum","weight":11},"description":"The Cityscape glides through city streets with elegance and grace, offering style and convenience for urban commuters!","addons":["basket","bell"],"helmet_included":true, "condition":"new"}'
JSON.SET sample_bicycle:1061 $ '{"model":"Explorer","brand":"Terrain Tamer","price":399,"type":"Mountain","specs":{"material":"carbon fiber","weight":7},"description":"The Explorer conquers any terrain with precision and speed, making it the ultimate choice for adventurous mountain bikers!","addons":["GPS tracker","LED lights"],"helmet_included":true, "condition":"new"}'
JSON.SET sample_bicycle:1062 $ '{"model":"Tourer","brand":"Nomad Bikes","price":329,"type":"Touring","specs":{"material":"aluminum","weight":13},"description":"The Tourer is designed for long-distance adventures, offering comfort and reliability for cyclists exploring new horizons!","addons":["panniers","rear view mirror"],"helmet_included":true, "condition":"new"}'
JSON.SET sample_bicycle:1063 $ '{"model":"City Cruiser","brand":"Metro Bikes","price":289,"type":"City","specs":{"material":"steel","weight":12.5},"description":"The City Cruiser glides through urban landscapes with ease, offering comfort and style for city commuters!","addons":["front light","fenders"],"helmet_included":true, "condition":"new"}'
JSON.SET sample_bicycle:1064 $ '{"model":"Adventurer","brand":"Trailblazer Bikes","price":379,"type":"Mountain","specs":{"material":"titanium","weight":8},"description":"The Adventurer thrives on challenging trails, offering agility and control for riders seeking adrenaline-fueled adventures!","addons":["hydration pack","bike computer"],"helmet_included":true, "condition":"new"}'
JSON.SET sample_bicycle:1065 $ '{"model":"Commuter","brand":"City Commute","price":259,"type":"City","specs":{"material":"aluminum","weight":11},"description":"The Commuter is the perfect companion for daily city commutes, offering efficiency and comfort for urban cyclists!","addons":["rear rack","bell"],"helmet_included":true, "condition":"new"}'
JSON.SET sample_bicycle:1066 $ '{"model":"Explorer Pro","brand":"Offroad Masters","price":449,"type":"Mountain","specs":{"material":"carbon fiber","weight":6.5},"description":"The Explorer Pro is a beast on the trails, offering unmatched performance and agility for serious mountain bikers!","addons":["GPS tracker","LED lights"],"helmet_included":true, "condition":"new"}'
JSON.SET sample_bicycle:1067 $ '{"model":"Urban Glide","brand":"City Wheels","price":299,"type":"City","specs":{"material":"steel","weight":12},"description":"The Urban Glide effortlessly maneuvers through city streets, offering style and comfort for urban commuters!","addons":["basket","bell"],"helmet_included":true, "condition":"new"}'
JSON.SET sample_bicycle:1068 $ '{"model":"Trail Master","brand":"Trail Trek","price":379,"type":"Mountain","specs":{"material":"titanium","weight":8.5},"description":"The Trail Master dominates rugged terrain, offering superior control and durability for avid mountain bikers!","addons":["hydration pack","bike computer"],"helmet_included":true, "condition":"new"}'
JSON.SET sample_bicycle:1069 $ '{"model":"City Slicker","brand":"Urban Bikes","price":269,"type":"City","specs":{"material":"aluminum","weight":10.5},"description":"The City Slicker navigates city streets with finesse, offering efficiency and style for urban cyclists!","addons":["front light","fenders"],"helmet_included":true, "condition":"new"}'
JSON.SET sample_bicycle:1070 $ '{"model":"Explorer Elite","brand":"Offroad Experts","price":499,"type":"Mountain","specs":{"material":"carbon fiber","weight":6},"description":"The Explorer Elite sets new standards for mountain biking, offering unparalleled performance and agility for extreme riders!","addons":["GPS tracker","LED lights"],"helmet_included":true, "condition":"new"}'
JSON.SET sample_bicycle:1071 $ '{"model":"City Cruiser Pro","brand":"Metro Riders","price":329,"type":"City","specs":{"material":"steel","weight":11.5},"description":"The City Cruiser Pro glides through urban landscapes with precision, offering comfort and style for discerning city commuters!","addons":["front light","fenders"],"helmet_included":true, "condition":"new"}'
JSON.SET sample_bicycle:1072 $ '{"model":"Trailblazer Pro","brand":"Adventure Seeker","price":399,"type":"Mountain","specs":{"material":"titanium","weight":8},"description":"The Trailblazer Pro conquers challenging trails with ease, offering advanced features and durability for serious mountain bikers!","addons":["hydration pack","bike computer"],"helmet_included":true, "condition":"new"}'
JSON.SET sample_bicycle:1073 $ '{"model":"City Commuter","brand":"Urban Trek","price":279,"type":"City","specs":{"material":"aluminum","weight":11},"description":"The City Commuter is designed for seamless urban journeys, offering efficiency and comfort for daily commuters!","addons":["rear rack","bell"],"helmet_included":true, "condition":"new"}'
JSON.SET sample_bicycle:1074 $ '{"model":"Mountain Master","brand":"Offroad Warriors","price":449,"type":"Mountain","specs":{"material":"carbon fiber","weight":7},"description":"The Mountain Master dominates any trail with precision and speed, offering unmatched performance for hardcore mountain bikers!","addons":["GPS tracker","LED lights"],"helmet_included":true, "condition":"new"}'
JSON.SET sample_bicycle:1075 $ '{"model":"Urban Explorer","brand":"City Adventures","price":309,"type":"City","specs":{"material":"steel","weight":12},"description":"The Urban Explorer navigates bustling city streets with confidence, offering comfort and style for urban adventurers!","addons":["basket","bell"],"helmet_included":true, "condition":"new"}'
JSON.SET sample_bicycle:1076 $ '{"model":"Trail Conqueror","brand":"Mountain Legends","price":389,"type":"Mountain","specs":{"material":"titanium","weight":8.5},"description":"The Trail Conqueror dominates rough terrain, offering superior control and durability for adventurous mountain bikers!","addons":["hydration pack","bike computer"],"helmet_included":true, "condition":"new"}'
JSON.SET sample_bicycle:1077 $ '{"model":"City Trekker","brand":"Urban Roam","price":289,"type":"City","specs":{"material":"aluminum","weight":10.5},"description":"The City Trekker navigates city streets with ease, offering efficiency and style for urban explorers!","addons":["rear rack","bell"],"helmet_included":true, "condition":"new"}'
JSON.SET sample_bicycle:1078 $ '{"model":"Mountain Maverick","brand":"Trail Blazers","price":429,"type":"Mountain","specs":{"material":"carbon fiber","weight":7.5},"description":"The Mountain Maverick blazes new trails with agility and precision, offering top-tier performance for experienced mountain bikers!","addons":["GPS tracker","LED lights"],"helmet_included":true, "condition":"new"}'
JSON.SET sample_bicycle:1079 $ '{"model":"Urban Voyager","brand":"City Riders","price":319,"type":"City","specs":{"material":"steel","weight":11.5},"description":"The Urban Voyager explores city streets with confidence, offering comfort and style for urban adventurers!","addons":["basket","bell"],"helmet_included":true, "condition":"new"}'
JSON.SET sample_bicycle:1080 $ '{"model":"Trail Seeker","brand":"Adventure Wheels","price":369,"type":"Mountain","specs":{"material":"titanium","weight":8},"description":"The Trail Seeker conquers challenging trails with ease, offering durability and control for avid mountain bikers!","addons":["hydration pack","bike computer"],"helmet_included":true, "condition":"new"}'
JSON.SET sample_bicycle:1081 $ '{"model":"City Roamer","brand":"Urban Cruisers","price":299,"type":"City","specs":{"material":"aluminum","weight":11},"description":"The City Roamer glides through city streets with elegance and grace, offering comfort and style for urban adventurers!","addons":["basket","bell"],"helmet_included":true, "condition":"new"}'
JSON.SET sample_bicycle:1082 $ '{"model":"Mountain Pro","brand":"Extreme Trails","price":469,"type":"Mountain","specs":{"material":"carbon fiber","weight":7.5},"description":"The Mountain Pro dominates rugged terrain with precision and speed, offering top-tier performance for hardcore mountain bikers!","addons":["GPS tracker","LED lights"],"helmet_included":true, "condition":"new"}'
JSON.SET sample_bicycle:1083 $ '{"model":"City Navigator","brand":"Urban Wheels","price":309,"type":"City","specs":{"material":"steel","weight":11},"description":"The City Navigator maneuvers through bustling streets with ease, offering efficiency and comfort for urban commuters!","addons":["basket","bell"],"helmet_included":true, "condition":"new"}'
JSON.SET sample_bicycle:1084 $ '{"model":"Trail Challenger","brand":"Mountain Movers","price":399,"type":"Mountain","specs":{"material":"titanium","weight":8.5},"description":"The Trail Challenger conquers any trail with confidence, offering durability and agility for adventurous mountain bikers!","addons":["hydration pack","bike computer"],"helmet_included":true, "condition":"new"}'
JSON.SET sample_bicycle:1085 $ '{"model":"City Rover","brand":"Urban Wheels","price":329,"type":"City","specs":{"material":"aluminum","weight":10.5},"description":"The City Rover navigates city streets with efficiency and style, offering comfort and convenience for urban adventurers!","addons":["basket","bell"],"helmet_included":true, "condition":"new"}'
JSON.SET sample_bicycle:1086 $ '{"model":"Mountain Explorer","brand":"Peak Riders","price":449,"type":"Mountain","specs":{"material":"carbon fiber","weight":7.5},"description":"The Mountain Explorer conquers any terrain with precision and speed, offering top-tier performance for serious mountain bikers!","addons":["GPS tracker","LED lights"],"helmet_included":true, "condition":"new"}'
JSON.SET sample_bicycle:1087 $ '{"model":"City Stroller","brand":"Urban Cruisers","price":289,"type":"City","specs":{"material":"steel","weight":11.5},"description":"The City Stroller glides through city streets with elegance and comfort, offering style and efficiency for urban commuters!","addons":["basket","bell"],"helmet_included":true, "condition":"new"}'
JSON.SET sample_bicycle:1088 $ '{"model":"Mountain Conqueror","brand":"Trail Blasters","price":419,"type":"Mountain","specs":{"material":"titanium","weight":8.5},"description":"The Mountain Conqueror dominates rough terrain with ease, offering durability and control for adventurous mountain bikers!","addons":["hydration pack","bike computer"],"helmet_included":true, "condition":"new"}'
JSON.SET sample_bicycle:1089 $ '{"model":"City Navigator Pro","brand":"Urban Wheels","price":339,"type":"City","specs":{"material":"aluminum","weight":10.5},"description":"The City Navigator Pro maneuvers through bustling streets with ease, offering efficiency and comfort for urban commuters!","addons":["basket","bell"],"helmet_included":true, "condition":"new"}'
JSON.SET sample_bicycle:1090 $ '{"model":"Mountain Voyager","brand":"Summit Seekers","price":459,"type":"Mountain","specs":{"material":"carbon fiber","weight":7.5},"description":"The Mountain Voyager conquers any trail with confidence, offering top-tier performance for serious mountain bikers!","addons":["GPS tracker","LED lights"],"helmet_included":true, "condition":"new"}'
JSON.SET sample_bicycle:1091 $ '{"model":"City Cruiser Plus","brand":"Urban Adventures","price":309,"type":"City","specs":{"material":"steel","weight":11.5},"description":"The City Cruiser Plus navigates city streets with style and comfort, offering efficiency and convenience for urban adventurers!","addons":["basket","bell"],"helmet_included":true, "condition":"new"}'
JSON.SET sample_bicycle:1092 $ '{"model":"Mountain Dominator","brand":"Peak Crushers","price":479,"type":"Mountain","specs":{"material":"titanium","weight":8.5},"description":"The Mountain Dominator conquers rough terrain with ease, offering durability and control for hardcore mountain bikers!","addons":["hydration pack","bike computer"],"helmet_included":true, "condition":"new"}'
JSON.SET sample_bicycle:1093 $ '{"model":"City Trekker Plus","brand":"Urban Wheels","price":349,"type":"City","specs":{"material":"aluminum","weight":10.5},"description":"The City Trekker Plus navigates city streets with ease and efficiency, offering comfort and style for urban adventurers!","addons":["basket","bell"],"helmet_included":true, "condition":"new"}'
JSON.SET sample_bicycle:1094 $ '{"model":"Mountain Warrior","brand":"Summit Crushers","price":489,"type":"Mountain","specs":{"material":"carbon fiber","weight":7.5},"description":"The Mountain Warrior dominates any trail with precision and speed, offering top-tier performance for serious mountain bikers!","addons":["GPS tracker","LED lights"],"helmet_included":true, "condition":"new"}'
JSON.SET sample_bicycle:1095 $ '{"model":"City Roamer Pro","brand":"Urban Adventures","price":329,"type":"City","specs":{"material":"steel","weight":11.5},"description":"The City Roamer Pro glides through city streets with precision and style, offering comfort and convenience for urban commuters!","addons":["basket","bell"],"helmet_included":true, "condition":"new"}'
JSON.SET sample_bicycle:1096 $ '{"model":"Mountain Maverick Plus","brand":"Trail Blazers","price":499,"type":"Mountain","specs":{"material":"carbon fiber","weight":7.5},"description":"The Mountain Maverick Plus sets new standards for mountain biking, offering unmatched performance and agility for extreme riders!","addons":["GPS tracker","LED lights"],"helmet_included":true, "condition":"new"}'
JSON.SET sample_bicycle:1097 $ '{"model":"City Navigator Elite","brand":"Urban Wheels","price":359,"type":"City","specs":{"material":"aluminum","weight":10.5},"description":"The City Navigator Elite maneuvers through bustling streets with ease, offering efficiency and comfort for urban commuters!","addons":["basket","bell"],"helmet_included":true, "condition":"new"}'
JSON.SET sample_bicycle:1098 $ '{"model":"Mountain Raptor","brand":"Ridge Riders","price":499,"type":"Mountain","specs":{"material":"titanium","weight":8.5},"description":"The Mountain Raptor conquers rugged trails with precision and speed, offering top-tier performance for hardcore mountain bikers!","addons":["GPS tracker","LED lights"],"helmet_included":true, "condition":"new"}'
JSON.SET sample_bicycle:1099 $ '{"model":"City Cruiser Pro Plus","brand":"Urban Adventures","price":369,"type":"City","specs":{"material":"steel","weight":11.5},"description":"The City Cruiser Pro Plus glides through city streets with precision and style, offering comfort and convenience for urban adventurers!","addons":["basket","bell"],"helmet_included":true, "condition":"new"}'
JSON.SET sample_bicycle:1100 $ '{"model":"Mountain Trailblazer","brand":"Summit Seekers","price":509,"type":"Mountain","specs":{"material":"carbon fiber","weight":7.5},"description":"The Mountain Trailblazer conquers any terrain with confidence, offering top-tier performance for serious mountain bikers!","addons":["GPS tracker","LED lights"],"helmet_included":true, "condition":"new"}'
JSON.SET sample_bicycle:1101 $ '{"model":"City Roamer Elite","brand":"Urban Riders","price":339,"type":"City","specs":{"material":"aluminum","weight":10.5},"description":"The City Roamer Elite navigates city streets with precision and style, offering comfort and convenience for urban commuters!","addons":["basket","bell"],"helmet_included":true, "condition":"new"}'
JSON.SET sample_bicycle:1102 $ '{"model":"Mountain Legend","brand":"Peak Crushers","price":499,"type":"Mountain","specs":{"material":"titanium","weight":8.5},"description":"The Mountain Legend conquers rough terrain with ease, offering durability and control for hardcore mountain bikers!","addons":["hydration pack","bike computer"],"helmet_included":true, "condition":"new"}'
JSON.SET sample_bicycle:1103 $ '{"model":"City Trekker Elite","brand":"Urban Wheels","price":359,"type":"City","specs":{"material":"aluminum","weight":10.5},"description":"The City Trekker Elite navigates city streets with ease and efficiency, offering comfort and style for urban adventurers!","addons":["basket","bell"],"helmet_included":true, "condition":"new"}'
JSON.SET sample_bicycle:1104 $ '{"model":"Mountain Dominator Pro","brand":"Peak Crushers","price":519,"type":"Mountain","specs":{"material":"carbon fiber","weight":7.5},"description":"The Mountain Dominator Pro conquers any trail with precision and speed, offering top-tier performance for serious mountain bikers!","addons":["GPS tracker","LED lights"],"helmet_included":true, "condition":"new"}'
JSON.SET sample_bicycle:1105 $ '{"model":"City Roamer Pro Plus","brand":"Urban Riders","price":349,"type":"City","specs":{"material":"aluminum","weight":10.5},"description":"The City Roamer Pro Plus glides through city streets with precision and style, offering comfort and convenience for urban commuters!","addons":["basket","bell"],"helmet_included":true, "condition":"new"}'
JSON.SET sample_bicycle:1106 $ '{"model":"Mountain Voyager Plus","brand":"Summit Seekers","price":529,"type":"Mountain","specs":{"material":"carbon fiber","weight":7.5},"description":"The Mountain Voyager Plus conquers any trail with confidence, offering top-tier performance for serious mountain bikers!","addons":["GPS tracker","LED lights"],"helmet_included":true, "condition":"new"}'



LPUSH sample_jobQueue:waitingList "sample_jobQueue:ticket:101"
LPUSH sample_jobQueue:waitingList "sample_jobQueue:ticket:102"
LPUSH sample_jobQueue:waitingList "sample_jobQueue:ticket:103"
HSET sample_jobQueue:ticket:101 id 101 user_id 123 description "Unable to login" priority "High" created_at "2024-04-09T10:00:00Z"
HSET sample_jobQueue:ticket:102 id 102 user_id 456 description "Network connectivity issue" priority "Medium" created_at "2024-04-09T10:05:00Z"
HSET sample_jobQueue:ticket:103 id 103 user_id 789 description "Software application crashing" priority "Low" created_at "2024-04-09T10:10:00Z"




ZADD sample_leaderboard:tetris 105000 "user1"
ZADD sample_leaderboard:tetris 145000 "user2"
ZADD sample_leaderboard:tetris 280000 "user3"
ZADD sample_leaderboard:tetris 325000 "user4"
ZADD sample_leaderboard:tetris 480000 "user5"
ZADD sample_leaderboard:tetris 510000 "user6"
ZADD sample_leaderboard:tetris 560000 "user7"
ZADD sample_leaderboard:tetris 640000 "user8"
ZADD sample_leaderboard:tetris 200000 "user9"
ZADD sample_leaderboard:tetris 180000 "user10"
ZADD sample_leaderboard:tetris 220000 "user11"
ZADD sample_leaderboard:tetris 420000 "user12"
ZADD sample_leaderboard:tetris 490000 "user13"
ZADD sample_leaderboard:tetris 570000 "user14"
ZADD sample_leaderboard:tetris 690000 "user15"
ZADD sample_leaderboard:tetris 125000 "user16"
ZADD sample_leaderboard:tetris 150000 "user17"
ZADD sample_leaderboard:tetris 300000 "user18"
ZADD sample_leaderboard:tetris 360000 "user19"
ZADD sample_leaderboard:tetris 540000 "user20"
ZADD sample_leaderboard:tetris 570000 "user21"
ZADD sample_leaderboard:tetris 630000 "user22"
ZADD sample_leaderboard:tetris 660000 "user23"
ZADD sample_leaderboard:tetris 230000 "user24"
ZADD sample_leaderboard:tetris 275000 "user25"
ZADD sample_leaderboard:tetris 350000 "user26"
ZADD sample_leaderboard:tetris 420000 "user27"
ZADD sample_leaderboard:tetris 520000 "user28"
ZADD sample_leaderboard:tetris 620000 "user29"
ZADD sample_leaderboard:tetris 670000 "user30"
ZADD sample_leaderboard:tetris 145000 "user31"
ZADD sample_leaderboard:tetris 265000 "user32"
ZADD sample_leaderboard:tetris 315000 "user33"
ZADD sample_leaderboard:tetris 490000 "user34"
ZADD sample_leaderboard:tetris 540000 "user35"
ZADD sample_leaderboard:tetris 580000 "user36"
ZADD sample_leaderboard:tetris 680000 "user37"
ZADD sample_leaderboard:tetris 175000 "user38"
ZADD sample_leaderboard:tetris 225000 "user39"
ZADD sample_leaderboard:tetris 365000 "user40"

HSET sample_session:123456789 user_id 123 username john_doe email john@example.com last_activity "2023-01-01 08:30:00"
HSET sample_session:234567890 user_id 234 username jane_smith email jane@example.com last_activity "2023-01-02 09:45:00"
HSET sample_session:345678901 user_id 345 username alice_green email alice@example.com last_activity "2023-01-03 11:00:00"
HSET sample_session:456789012 user_id 456 username bob_jones email bob@example.com last_activity "2023-01-04 12:15:00"
HSET sample_session:567890123 user_id 567 username emily_brown email emily@example.com last_activity "2023-01-05 13:30:00"
HSET sample_session:678901234 user_id 678 username chris_black email chris@example.com last_activity "2023-01-06 14:45:00"
HSET sample_session:789012345 user_id 789 username sophia_taylor email sophia@example.com last_activity "2023-01-07 16:00:00"
HSET sample_session:890123456 user_id 890 username david_wilson email david@example.com last_activity "2023-01-08 17:15:00"
HSET sample_session:901234567 user_id 901 username olivia_lee email olivia@example.com last_activity "2023-01-09 18:30:00"
HSET sample_session:012345678 user_id 012 username noah_hall email noah@example.com last_activity "2023-01-10 19:45:00"
HSET sample_session:112233445 user_id 112 username mia_evans email mia@example.com last_activity "2023-01-11 21:00:00"
HSET sample_session:334455667 user_id 334 username ethan_white email ethan@example.com last_activity "2023-01-12 22:15:00"
HSET sample_session:556677889 user_id 556 username ava_martin email ava@example.com last_activity "2023-01-13 23:30:00"
HSET sample_session:778899001 user_id 778 username logan_anderson email logan@example.com last_activity "2023-01-14 00:45:00"
HSET sample_session:990011223 user_id 990 username mia_thompson email mia@example.com last_activity "2023-01-15 02:00:00"
EXPIRE sample_session:123456789 1296000
EXPIRE sample_session:234567890 1728000
EXPIRE sample_session:345678901 2016000
EXPIRE sample_session:456789012 2592000
EXPIRE sample_session:567890123 3024000
EXPIRE sample_session:678901234 3456000
EXPIRE sample_session:789012345 3888000
EXPIRE sample_session:890123456 4320000
EXPIRE sample_session:901234567 4752000
EXPIRE sample_session:012345678 5184000
EXPIRE sample_session:112233445 5616000
EXPIRE sample_session:334455667 6048000
EXPIRE sample_session:556677889 6480000
EXPIRE sample_session:778899001 6912000
EXPIRE sample_session:990011223 7344000

FT.CREATE idx:smpl_bicycle ON JSON PREFIX 1 sample_bicycle: SCHEMA $.brand AS brand TEXT $.model AS model TEXT $.description AS description TEXT $.price AS price NUMERIC $.condition AS condition TAG SEPARATOR , $.type AS type TAG $.helmet_included AS helmet_included TAG $.specs.material AS material TAG $.specs.weight AS weight NUMERIC
```

### Matchmaking

Redis serves as a powerful document store that is well-suited for matchmaking use cases. For instance, consider a bike shop where you need to pair shoppers with bikes based on their preferences and budget.

You can store your bike inventory using the JSON data structure, where bikes have attributes such as type (e.g., mountain, road, electric), condition (new or used), price, etc. 

```redis:[run_confirmation=true] Add a bike as JSON
// Add a bike as JSON
JSON.SET sample_bicycle:2048 $ '{
  "model": "Ranger",
  "brand": "TrailBlazer",
  "price": 450,
  "type": "Mountain",
  "specs": {
    "material": "carbon",
    "weight": 12
  },
  "description": "The Ranger is designed for rugged trails and adventurous rides!",
  "addons": [
    "water bottle holder",
    "rear rack"
  ],
  "helmet_included": false
  }'
```

In Redis, you can easily index these attributes and perform complex queries efficiently.

```redis:[run_confirmation=true] Create a bike index
// Create a secondary index of your bike data
FT.CREATE idx:smpl_bicycle
    ON JSON 
      PREFIX 1 sample_bicycle: 
    SCHEMA 
      $.brand AS brand TEXT
      $.model AS model TEXT  
      $.description AS description TEXT
      $.price AS price NUMERIC 
      $.condition AS condition TAG SEPARATOR , 
      $.type AS type TAG 
      $.helmet_included AS helmet_included TAG 
      $.specs.material AS material TAG 
      $.specs.weight AS weight NUMERIC
```

You can then easily match a user to their preferred type of bike within a requested price bracket.

```redis:[run_confirmation=true] Search for a match
// Match leisure bikes within a price of 200 and 300
FT.SEARCH idx:smpl_bicycle "@type:{mountain} @price:[400 450]" RETURN 4 brand model type price
```

### Session store

Redis shines as a session store, offering speed and scalability for web applications. Using commands like HSET to store session data as hashes and HGET to retrieve it, Redis ensures rapid access to user sessions.

```redis:[run_confirmation=true] Create a session hash with expiration
// Store new session
HSET sample_session:074529275 user_id 7254 username gabe_jones email gabe@example.com last_activity "2024-06-01 10:24:00"

// Set an expiration time for the new session entry
EXPIRE sample_session:074529275 7344000
```

```redis:[run_confirmation=true] Retrieve a session
// Retrieve an existing session
HGETALL sample_session:074529275
```

### Job queue

In many applications, tasks need to be processed asynchronously, such as maintaining a waiting list for a helpdesk where incoming support tickets are queued for processing. A job queue helps manage these tasks efficiently. In Redis, a [job queue](https://redis.io/glossary/redis-queue/?utm_source=redisinsight&utm_medium=app&utm_campaign=RAG_tutorial) can be implemented using the List data structure. When a new task needs to be queued, it is added to the left end of the list (using the LPUSH command). When a worker is ready to process a task, it dequeues it from the right end of the list (using the RPOP command). 


```redis:[run_confirmation=true] Create a job ticket
// Create a new job as a Hash
HSET sample_jobQueue:ticket:199 id 199 user_id 723 description "Unable to access console" priority "High" created_at "2024-04-20T12:00:00Z"
```
```redis:[run_confirmation=true] Enqueue job
// Enqueue the new job to the waiting list
LPUSH sample_jobQueue:waitingList jobQueue:ticket:199
```
```redis:[run_confirmation=true] Show all jobs
// Show the full list of jobs
LRANGE sample_jobQueue:waitingList 0 -1
```
```redis:[run_confirmation=true] Dequeue a job
// Dequeue a job from the list
RPOP sample_jobQueue:waitingList
```

### Leaderboard

[Leaderboards](https://redis.io/solutions/leaderboards/?utm_source=redisinsight&utm_medium=app&utm_campaign=RAG_tutorial) are crucial for gaming applications to display rankings based on scores. Sorted Sets in Redis are perfect for this purpose due to their ability to store elements with associated scores, which can be easily queried and sorted.

```redis:[run_confirmation=true] Create a leaderboard score
// Add a new score to leaderboard
ZADD sample_leaderboard:tetris 670000 "user100"
```

```redis:[run_confirmation=true] Get users with scores
// Get the top 5 users on the leaderboard, with scores
ZRANGE sample_leaderboard:tetris 0 4 REV WITHSCORES
```

### Resources

Check out the [Redis for AI learning](https://university.redis.io/learningpath/hbykf3qrnhwccy?utm_source=redisinsight&utm_medium=main&utm_campaign=tutorials) path on Redis University to go deeper on this topic and get hands on experience.
