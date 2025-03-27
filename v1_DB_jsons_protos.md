proposition post DB v1 : 

# Overall ideas, questions, standars: 
- timestamps in DB should be in UTC.
- Timestamps format ?  :
	- "last_time_saw_at": 1743076209,		
	 or
	- "last_time_saw_at": "2025-03-27 11:50:09"	
- The endpoints should be thought in bulk and not single transfer to improve speed of data treatment.

---
## table name : participant_data
Table qui aura tout les participants donnée jsp si c'est neccesaire, age, etc etc


|  name 					|   datatype			|   attribut			|   comment			|
|---						|---            		|---					|---				|
|   **id**						|   int auto_increment	| PK  					||
|   **id_runner**				|   int					| 	 					||
|   **"prenom" ou "pseudo"**	|   tiny_string			|   -					||
|   **created_at**				|   timestamp			|  auto_created in POST	||


POST : json post prototype: 
```json
{
	"id_runner": 123,
	"pseudo": "nours"
}
```

---


## table_name : raw_data_income_ia
Table income data from IA.
|  name 					|   datatype			|   attribut			|   comment			|
|---						|---            		|---					|---				|
|   **id**						|   int auto_increment	| PK  					||
|   **runner_id_is_match**		|   tiny int (BOOL)		|   					| these is a check TRUE / FALSE in case we have a match or not from participant_data|
|   **income_id_runner**		|   int					| FK					| can be NULL. / in case we found the data in participant_table. |
| 	**image_id**				|   "int"	or "string" ?						|  FK ??				| id from the image where the IA was taking the information.|
| 	**last_time_saw_at**		|   timestamp			|  timestamp			| timestamp registered from the IA|
| 	**created_at**				|   timestamp			|  auto_created in POST	||
| 	**treated_at**				|   timestamp			|  auto_update in PUT	||
| 	**updated_at**				|   timestamp			|  auto_update in PUT	||
| 	**is_treated**				|   tiny int (BOOL)		|  DEFAULT= FALSE. after treating the data in back end we can update BOOL to TRUE like that we know wich datas need to be treated and wich not. ||





### POST POST prototype: 
##### Exemple in case we have a match od a runner - IA
```json
{
	"runner_id_is_match": true,
	"income_id_runner": 123,
	"image_id": 852489,				// "image_id": "ovfsofgnbdfknbl-852489", ?
	"last_time_saw_at": 1743076209,			// or "last_time_saw_at": 2025-03-27 11:50:09
}
```
##### Exemple in case we DON'T have a match od a runner - IA
```json
{
	"runner_id_is_match": false,
	"income_id_runner": 123,
	"image_id": 852489,				// "image_id": "ovfsofgnbdfknbl-852489", ?
	"last_time_saw_at": 1743076209,			// or "last_time_saw_at": 2025-03-27 11:50:09
}
```

### PUT prototype
In the moment we want to say that we treated these information.
**We neeed a way to treat these in bulk and not one by one.**
```json
{
	"id": 74156,
	"is_treated" : true,
	"income_id_runner": 46523 // Maybe they need to change the income_id_runner because it was false.
}
```
another version : 
in URL : https:/api/endpoint/:[array-of-IDs]
and the Query will set all of then to TRUE. and is SQL that will manage that.

In case we want to put a runner runner_id_is_match to true
```json
{
	"id": 74156,
	"runner_id_is_match": true,	// set is match to TRUE 
	"income_id_runner": 78456321 	// set updated id
}
```

### Maybe we would need a GET 
### json GET prototype: 
i think GET by is_treated=FALSE and runner_id_is_match=TRUE
In bulk will GET the data not treated.
```json
[
	{
		"id": 74156,
		"income_id_runner": 10,
		"last_time_saw_at": 1743076209,			// or "last_time_saw_at": 2025-03-27 11:50:09
	}, 
	{
		"id": 846,
		"income_id_runner": 60,
		"last_time_saw_at": 1743076209,			// or "last_time_saw_at": 2025-03-27 11:50:09
	}, 
	{
		"id": 197,
		"income_id_runner": 36,
		"last_time_saw_at": 1743076209,			// or "last_time_saw_at": 2025-03-27 11:50:09
	}, 
	{
		"id": 30289,
		"income_id_runner": 2,
		"last_time_saw_at": 1743076209,			// or "last_time_saw_at": 2025-03-27 11:50:09
	},
]
```


--- 
## Table_name : runner_status

- we can set to all runners a 0 lap with the timestamp of the begining of the race. 

table where the data is cleaned related to matched runners.
|  name 					|   datatype			|   attribut			|   comment			|
|---						|---            		|---					|---				|
|   **id**						|   int auto_increment	| PK  					|participant_data	|
|   **id_runner**				|   int					| FK					| conclution from:  raw_data_ia_runners  |
|   **id_raw_data**				|   int					| FK					|  					|
| 	**amount_of_laps_runned**	|   int					| 						| |
| 	**average_speed**			|   float				| 						| |
| 	**total_time_runned**		|   int					| 						| |
| 	**current_position**		|   int					| 						| |
| 	**time_spend_current_lap**	|   int					| 						| |
| 	**last_time_saw_at**		|   timestamp			|  timestamp			| timestamp registered from the IA|
| 	**created_at**				|   timestamp			|  auto_created in POST	||
| 	**updated_at**				|   timestamp			|  auto_created in PUT	||
| 	**has_shown**				|   tiny int (BOOL)		|  DEFAULT= FALSE. after treating the data in back end we can update BOOL to TRUE like that we know wich datas need to be treated and wich not. ||





### GET / PUT by has_shown=FALSE
after getting and receibing a check from the FRONT that i received the data we need to set all has_shown to FALSE.
In a query set all to false (we don't need to pass by an endpoint).
```json
[
	{
		"id": 78965432113,
		"id_runner": 123,
		"amount_of_laps_runned": 1,
		"average_speed": 6.5,
		"total_time_runned": 416523,
		"current_position": 15,
		"time_spend_current_lap": 15852852,
		"last_time_saw_at": 1743076209			// or "last_time_saw_at": 2025-03-27 11:50:09
	},
	{
		"id": 78965432113,
		"id_runner": 123,
		"id_raw_data": 852489,				// "image_id": "ovfsofgnbdfknbl-852489", ?
		"amount_of_laps_runned": 1,
		"average_speed": 6.5,
		"total_time_runned": 416523,
		"current_position": 15,
		"time_spend_current_lap": 15852852,
		"last_time_saw_at": 1743076209			// or "last_time_saw_at": 2025-03-27 11:50:09
	},
	{
		"id": 78965432113,
		"id_runner": 123,
		"id_raw_data": 852489,				// "image_id": "ovfsofgnbdfknbl-852489", ?
		"amount_of_laps_runned": 1,
		"average_speed": 6.5,
		"total_time_runned": 416523,
		"time_spend_current_lap": 15852852,
		"current_position": 15,
		"last_time_saw_at": 1743076209			// or "last_time_saw_at": 2025-03-27 11:50:09
	}
]
```






### POST prototype:

Before POSTING we will need to GET by last inputed data by id_runner to compare that and do the calculations updates. 
 - calculations of things : 
 	- "time_spend_current_lap" = "last_time_saw_at" - "current_time_saw" 
	- **"average_speed" = ~2.5km something with "time_spend_current_lap"**
	- **"current_position" = ??**
	- total_time_runned = start_race_timestamp (beginning 0 lap for everyone at the begining) - "last_time_saw_at"
	- "amount_of_laps_runned"  = "amount_of_laps_runned" + 1

```json
[
	{
		"id_runner": 123,
		"id_raw_data": 852489,				// "image_id": "ovfsofgnbdfknbl-852489", ?
		"amount_of_laps_runned": 1,
		"average_speed": 6.5,
		"total_time_runned": 416523,
		"time_spend_current_lap": 15852852,
		"current_position": 15,
		"last_time_saw_at": 1743076209			// or "last_time_saw_at": 2025-03-27 11:50:09
	},
	{
		"id_runner": 123,
		"id_raw_data": 852489,				// "image_id": "ovfsofgnbdfknbl-852489", ?
		"amount_of_laps_runned": 1,
		"average_speed": 6.5,
		"total_time_runned": 416523,
		"current_position": 15,
		"last_time_saw_at": 1743076209			// or "last_time_saw_at": 2025-03-27 11:50:09
	},
	{
		"id_runner": 123,
		"id_raw_data": 852489,				// "image_id": "ovfsofgnbdfknbl-852489", ?
		"amount_of_laps_runned": 1,
		"average_speed": 6.5,
		"total_time_runned": 416523,
		"current_position": 15,
		"last_time_saw_at": 1743076209			// or "last_time_saw_at": 2025-03-27 11:50:09
	}
]
```