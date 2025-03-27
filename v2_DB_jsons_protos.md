proposition post DB v1 : 

# Overall ideas, questions, standars: 
- timestamps in DB should be in UTC.
- Timestamps format ?  :
	- "last_time_saw_at": 1743076209,		
	 or
	- "last_time_saw_at": "2025-03-27 11:50:09"	
- The endpoints should be thought in bulk and not single transfer to improve speed of data treatment.
- We will have a lot of data in a short period of time, avoid Joins



1.  **Service for:** table."raw_data_income_ia" : constantly checking the table and updating it with flaging:
	- copy cats
	- what is important and not important to take in count.
2.  **Service for:** table."runner_status" : checking new income data from table."raw_data_income_ia" calculating and adding updates to  table."runner_status"
---
## table name : checkpoints
maybe we will have 1 or 2 or 4 stations. 
Crucial information we need to know if its the begining point checkpoint.

|  name 						|   datatype			|   attribut			|   comment			|
|---							|---            		|---					|---				|
|   **id**						|   int auto_increment	| PK  					||
|   **position**				|   tiny_string			| 	 					||
|   **admin_station_name**		|   tiny_string			|   -					||
|   **type_of_camera**			|   tiny_string			| can be null 			||
| 	**is_starting_checkpoint**	|   tiny int (BOOL)		|						| we need to know wich one is the starting point |


---
## table name : participant_data
Table qui aura tout les participants donnée jsp si c'est neccesaire, age, etc etc


|  name 						|   datatype			|   attribut			|   comment			|
|---							|---            		|---					|---				|
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
REMINDER : Nours will check in another script if the runners_id exists.
REMINDER : we will have multiple amount of income data from the same checkpoint we should take count of the first one the others is rubish.

|  name 							|   datatype			|   attribut			|   comment			|
|---								|---            		|---					|---				|
|   **id**							|   int auto_increment	| PK  					||
|   **checkpoint_id**				|   int					| FK  					||
| 	**is_starting_checkpoint**		|   tiny int (BOOL)		|						|(Already present in checkpoints table) if yes then we will need to increase the amount of laps later |
|   **income_id_runner**			|   int					| FK					| can be NULL. / in case we found the data in participant_table. |
| 	**image_id**					|   string				| -						| id from the image where the IA was taking the information.|
| 	**last_time_saw_at**			|   timestamp			| timestamp				| timestamp registered from the IA|
| 	**created_at**					|   timestamp			| auto_created in POST	||
| 	**treated_at**					|   timestamp			| auto_update in PUT	||
| 	**updated_at**					|   timestamp			| auto_update in PUT	||
| 	**is_treated**					|   tiny int (BOOL)		| DEFAULT=FALSE 		| after treating the data in back end we can update BOOL to TRUE like that we know wich datas need to be treated and wich not.|
| 	**is_noise_data**				|   tiny int (BOOL)		| DEFAULT=FALSE 		| REMINDER: by checkpoint we will have several almost same data (maybe some seconds of differnce ), we need to take the first one the others almost the same we don't use them |





### POST POST prototype: 
```json
[
	{
		"income_id_runner": 123,
		"image_id": "ovfsofgnbdfknbl-852489",
		"last_time_saw_at": 1743076209,			// or "last_time_saw_at": 2025-03-27 11:50:09
		"checkpoint_id": 3,
		"is_starting_checkpoint": false
	},
	{
		"income_id_runner": 123,
		"image_id": "ovfsofgnbdfknbl-852489",
		"last_time_saw_at": 1743076209,			// or "last_time_saw_at": 2025-03-27 11:50:09
		"checkpoint_id": 3,
		"is_starting_checkpoint": false
	},
	{
		"income_id_runner": 123,
		"image_id": "ovfsofgnbdfknbl-852489",
		"last_time_saw_at": 1743076209,			// or "last_time_saw_at": 2025-03-27 11:50:09
		"checkpoint_id": 3,
		"is_starting_checkpoint": false
	}
]
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

---

### Maybe we would need a GET 
### json GET prototype: 
- GET by is_treated=FALSE && is_noise_data=FALSE

In bulk will GET the data not treated.

```json
[
	{
		"id": 74156,
		"income_id_runner": 10,
		"last_time_saw_at": 1743076209,			// or "last_time_saw_at": 2025-03-27 11:50:09
		"checkpoint_id": 3,
		"is_starting_checkpoint": false
	}, 
	{
		"id": 846,
		"income_id_runner": 60,
		"last_time_saw_at": 1743076209,			// or "last_time_saw_at": 2025-03-27 11:50:09
		"checkpoint_id": 3,
		"is_starting_checkpoint": false
	}, 
	{
		"id": 197,
		"income_id_runner": 36,
		"last_time_saw_at": 1743076209,			// or "last_time_saw_at": 2025-03-27 11:50:09
		"checkpoint_id": 3,
		"is_starting_checkpoint": false
	}, 
	{
		"id": 30289,
		"income_id_runner": 2,
		"last_time_saw_at": 1743076209,			// or "last_time_saw_at": 2025-03-27 11:50:09
		"checkpoint_id": 3,
		"is_starting_checkpoint": false
	},
]
```

















--- 
## Table_name : runner_status

- we can set to all runners a 0 lap with the timestamp of the begining of the race. 

table where the data is cleaned related to matched runners.

| Name                          | Datatype            | Attribute            | Comment                                      |
|-------------------------------|---------------------|----------------------|----------------------------------------------|
| **id**                        | int auto_increment  | PK                   | participant_data                             |
| **id_runner**                 | int                 | FK                   | Conclusion from: raw_data_ia_runners         |
| **id_raw_data**               | int                 | FK                   |                                              |
| **checkpoint_id**				| int			 	  | FK 					 |												|
| **is_starting_checkpoint**	|   tiny int (BOOL)	  |						 | (Already present in checkpoints table) if yes then we will need to increase the amount of laps later |
| **amount_of_laps_runned**     | int                 |                      |                                              |
| **average_speed**             | float               |                      |                                              |
| **total_time_runned**         | int                 |                      |                                              |
| **runner_current_position**   | int                 |                      |                                              |
| **time_spend_current_lap**    | int                 |                      |                                              |
| **last_time_saw_at**          | timestamp           | timestamp            | Timestamp registered from the IA             |
| **created_at**                | timestamp           | auto_created in POST |                                              |
| **updated_at**                | timestamp           | auto_created in PUT  |                                              |
| **has_shown_in_front**        | tiny int (BOOL)     | DEFAULT = FALSE      | After treating the data in the backend, we can update BOOL to TRUE to know which data needs to be treated and which does not. |




### GET / PUT by has_shown_in_front=FALSE
1. GET by has_shown_in_front=FALSE
2. PUT set all ids to has_shown_in_front=TRUE
after getting and receibing a check from the FRONT that i received the data we need to set all has_shown_in_front to FALSE.
In a query set all to false (we don't need to pass by an endpoint).
```json
[
	{
		"id": 78965432113,
		"id_runner": 123,
		"amount_of_laps_runned": 1,
		"average_speed": 6.5,
		"total_time_runned": 416523,
		"runner_current_position": 15,
		"time_spend_current_lap": 15852852,
		"checkpoint_id": 3,
		"is_starting_checkpoint": false,
		"last_time_saw_at": 1743076209			// or "last_time_saw_at": 2025-03-27 11:50:09
	},
	{
		"id": 78965432113,
		"id_runner": 123,
		"id_raw_data": 852489,				// "image_id": "ovfsofgnbdfknbl-852489", ?
		"amount_of_laps_runned": 1,
		"average_speed": 6.5,
		"total_time_runned": 416523,
		"runner_current_position": 15,
		"time_spend_current_lap": 15852852,
		"checkpoint_id": 3,
		"is_starting_checkpoint": false,
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
		"runner_current_position": 15,
		"checkpoint_id": 3,
		"is_starting_checkpoint": false,
		"last_time_saw_at": 1743076209			// or "last_time_saw_at": 2025-03-27 11:50:09
	}
]
```



### POST prototype:

Before POSTING we will need to GET by last inputed data by id_runner to compare that and do the calculations updates. 
 - calculations of things : 
 	- "time_spend_current_lap" = "last_time_saw_at" - "current_time_saw" 
	- **"average_speed" = ~2.5km something with "time_spend_current_lap"**
	- **"runner_current_position" = ??**
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
		"runner_current_position": 15,
		"checkpoint_id": 3,
		"is_starting_checkpoint": false,
		"last_time_saw_at": 1743076209			// or "last_time_saw_at": 2025-03-27 11:50:09
	},
	{
		"id_runner": 123,
		"id_raw_data": 852489,				// "image_id": "ovfsofgnbdfknbl-852489", ?
		"amount_of_laps_runned": 1,
		"average_speed": 6.5,
		"total_time_runned": 416523,
		"runner_current_position": 15,
		"checkpoint_id": 3,
		"is_starting_checkpoint": false,
		"last_time_saw_at": 1743076209			// or "last_time_saw_at": 2025-03-27 11:50:09
	},
	{
		"id_runner": 123,
		"id_raw_data": 852489,				// "image_id": "ovfsofgnbdfknbl-852489", ?
		"amount_of_laps_runned": 1,
		"average_speed": 6.5,
		"total_time_runned": 416523,
		"runner_current_position": 15,
		"checkpoint_id": 3,
		"is_starting_checkpoint": false,
		"last_time_saw_at": 1743076209			// or "last_time_saw_at": 2025-03-27 11:50:09
	}
]
```