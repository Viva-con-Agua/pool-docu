# Store data in CiviCrm

## Concept

```
  "crm_data": {
    "campaign_id": 17,
    "drops_id": "540e695b-341e-4256-8a3b-9aa874332f60",
    "activity": ""
  }
```


## Run4Water workflow

![Workflow](Workflow_Run4Water.jpg)


### Activities
| Aktivity  | Data Model |
| ------------- | ------------- |
| CREATE_DROPS_USER    | INSERT USER  |
| EVENT_JOIN    | SIMPLE ACTIVITY  |
| PAYMENT_COMMIT   | PAYMENT DATA  |
| EVENT_FINISH    | SIMPLE ACTIVITY  |
| RUN_GOAL    | MOVE DATA  |
| RUN_FINISH    | MOVE DATA  |
| TEAM_CREATE    | GROUP DATA  |
| TEAM_JOIN    | GROUP DATA  |
| INVIDE_FRIEND    | SIMPLE MAIL  |


### SIMPLE ACTIVITY

```
  "crm_data": {
    "campaign_id": 17,
    "drops_id": "540e695b-341e-4256-8a3b-9aa874332f60",
    "activity": "EVENT_JOIN"
  }
```

### INSERT USER crm model
```
{
  "crm_data": {
    "campaign_id": 17,
    "drops_id": "540e695b-341e-4256-8a3b-9aa874332f60",
    "activity": "CREATE_DROPS_USER",
    "created": "01231231"
  },
  "crm_user": {
    "email": "dennis_kleber22@mailbox.org",
    "first_name": "dennis",
    "last_name": "kleber",
    "privacy_policy": true,
    "country": "DE"
  },
  "mail": {
    "link": "http://localhost:1323/auth/signup/confirm/MyPZF1HCqBKzCsCAfsoYiGigUbwRflzpf8Mt6NOmaTc="
  },
  "offset": {
    "known_from": "facebook",
    "newsletter": false
 }
}
```
### PAYMENT USER crm model

```
  "crm_data": {
    "campaign_id": 17,
    "drops_id": "540e695b-341e-4256-8a3b-9aa874332f60",
    "activity": "PAYMENT_COMMIT",
    "created": "01231231"
  },
  "payment":{
    "id":"",
    "provider":"paypal"
    "money":{
        "amount":12000,
        "currency":"EUR"
    }  
  }
```

### SEND MAIL

```
{
    "crm_data": {
        "campaign_id": 17,
        "drops_id": "540e695b-341e-4256-8a3b-9aa874332f60",
        "activity": "INVIDE_FRIEND",
        "created": "01231231"
    },
    "mail": {
        "email": "freund@gmx.org" //dürfen wir nicht als Kontakt anlegen. Sorry für die schlechte vorarbeit im letzten Model ^^
        "link": "link_mit_token"
    }
}
```

### MOVE DATA


```
  "crm_data": {
    "campaign_id": 17,
    "drops_id": "540e695b-341e-4256-8a3b-9aa874332f60",
    "activity": "RUN_INIT / RUN_FINISH",
    "created": "01231231"
  },
  "move": {
    "range": 100
    "meas": "km"
    "time": 0
  }
```

### GROUP DATA


```
  "crm_data": {
    "campaign_id": 17,
    "drops_id": "540e695b-341e-4256-8a3b-9aa874332f60",
    "activity": "GROUP_CREATE / GROUP_JOIN",
    "created": "01231231"
  },
  "group_name": "teamname"
