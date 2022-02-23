# json-logic-py

This parser accepts [JsonLogic](http://jsonlogic.com) rules and executes them in Python.

This is a fork of the [nadirizr](https://github.com/nadirizr/json-logic-py), adding the possibility for dealing with operations with list of objects, adding some custom operations (filter, any, all), and using numpy for the majority of the existent operations.

## Examples

```python
from json_logic import jsonLogic
import json
```

Firstly, we define a fake data object, that takes into account the following data model V2 IA (https://drive.google.com/file/d/1zuO1IXE5TS2EjP4k255FDWW_hxmP5R2b/view?usp=sharing)
```
# Generate a fake data corresponding to a single GaviusUser
data = {
    "GaviusUsers":{
      "AgeRange": ">65",
      "TotalIncomes": 30000,
      "DisabilityDegree": 2,
      "NPropCars":1,
      "NPropDwellings":1,
      "EmploymentStatus": "pension"
    },
    "DetailedAddress": {
      "ZIPCode":"25006",
      "CensusTract":"2511001001",
      "Municipality": {
        "NationalStatisticsId": "25110"
      }
    },
    "BuildingParts": [
      {
        "Id":"14155122",
        "HasKitchen":True,
        "Propietary":True,
        "MainResidence":True,
        "Area":102.32,
        "Buildings": {
          "Id": "471223",
          "TotalFloors": 12,
          "DetailedAddress": {
            "CensusTract": "2502301001",
            "ZIPCode": "25110"
          }
        }
      },
      {
        "Id":"14155122",
        "Area":99.11,
        "Buildings": {
          "Id": "472914",
          "TotalFloors": 2,
          "DetailedAddress": {
            "CensusTract": "2511001001",
            "PostalCode": "25006"
          }
        }
      }
    ],
    "Projects": [
      {
        "Type": "Photovoltaics",
        "Budget": 6900,
        "Status": "Finished",
        "Building": "471223"
      },
      {
        "Type": "Retroffiting",
        "Budget": 9120,
        "Status": "Projected", 
        "Building": "471223"
      }
    ],
    "CohabitationUnits": {
      "NumerousFamily": False,
      "NUnemployed": 0,
      "NAdults": 2,
      "Nminors": 2,
      "People":[
        {
          "AgeRange": "60-64",
          "TotalIncomes": 15000,
          "NPropCars":2
        },
        {
          "AgeRange": "12-17",
          "TotalIncomes": 0
        },
        {
          "AgeRange": "12-17",
          "TotalIncomes": 0
        }
      ]
    },
    "Expenses": [
      {
        "Cost": 24.5,
        "DateIssuance": "2020-01-07",
        "TransportType": "Bus nit"
      },
      {
        "Cost": 70.5,
        "DateIssuance": "2021-12-31",
        "SupplyType": "Electricity",
        "SocialBonus": False
      },
      {
        "Cost": 270.5,
        "DateIssuance": "2022-01-03",
        "SupplyType": "Gas",
        "SocialBonus": False
      }
    ]
  }
```  
In this first example, the following three conditions must be accomplished:
1. Less than 500 € of expenses during last year.
2. The person must have more than 65 years old, or more than 60 years old and somebody in their cohabitation unit with more than 65.
3. The person must not be propietary of an apartment with an area greater than 100 m2.
```{python}
rule_json = json.dumps({
    "and": [
      {
        "<": [
          {
            "+": {
              "filter": [{"var":"Expenses.*.Cost"},{">": [{"var":"Expenses.*.DateIssuance"},"2021-02-22"]}]
            }
          },
          500
        ]
      },
      {
        "or":[
          {"in": [{"var":"People.AgeRange"},">65"]},
          {
            "and":[
              {"in": [{"var":"People.AgeRange"},"60-64"]},
              {"any": [{"in": [{"var":"CohabitationUnits.People.*.AgeRange"},[">65"]]}]}
            ]
          }
        ]
      },
      {
        "!":{
          "any":[
            {
              "filter":[
                {
                  ">":[{"var":"BuildingParts.*.Area"}, 100]
                },
                {
                  "==":[{"var":"BuildingParts.*.Propietary"}, True]
                }
              ]
            }
          ]
        }
      }
    ]
  })
  
 
print(rule_json)
result = jsonLogic(json.loads(rule_json), data)
result
```

## Installation

The best way to install this library is via [PIP](https://pypi.python.org/pypi/):

```bash
pip install git+https://github.com/gmor/json-logic-py.git
```
